# Contexto do Projeto — Design Tools Suite

## Quem sou eu
Antonio Araujo, Senior Product Designer com mais de 10 anos de experiência. Atualmente no time de experiência da C&A, frente de descoberta do cliente (site e app). Background em teatro e terceiro setor antes do design. Trabalho com produto, UX, Design System e referência técnica para o time.

---

## Arquivos do projeto

| Arquivo | Função |
|---|---|
| `design-system.css` | Tokens centralizados — base de todos os arquivos |
| `login.html` | Tela de login, recuperação e nova senha |
| `admin.html` | Painel de gerenciamento de usuários |
| `index.html` | A3 Digital (ferramenta principal) |
| `request.html` | Página pública de envio de demanda (sem login) |
| `briefing-agente.html` | Agente de Briefing |

---

## Design System (`design-system.css`)

Todos os arquivos importam `design-system.css` via `<link rel="stylesheet">`. Nenhum valor de cor, fonte ou espaçamento é hardcoded — tudo referencia tokens do DS.

### Tokens principais
- **Cores:** `--color-cea: #104170`, `--color-electric-blue: #4100F7`, `--color-soft-blue: #0bacd3`
- **Fontes:** `--font-primary: Roboto` (títulos), `--font-ui: Inter` (interface)
- **Radii:** `--radius-1: 4px`, `--radius-2: 8px`, `--radius-3: 12px`, `--radius-full`
- **Sombras:** `--shadow-1`, `--shadow-2`
- **Sidebar:** CEA Blue `#104170`
- **Botão primário:** Electric Blue `#4100F7`
- **Gradiente IA:** `linear-gradient(135deg, #6C63FF, #A855F7, #EC4899)` — intencional, diferencia ações de IA

---

## Sistema de autenticação

### Credenciais Supabase
- **Project URL:** `https://yipvayrvkuxpegpgerpr.supabase.co`
- **Anon key:** `sb_publishable_pnfgOXeCPc8CbWZb5PyR5g_R5YcDTT7`
- **Service role key:** usada apenas no `admin.html`
- **Região:** South America (São Paulo)

### Fluxo de redirecionamento
| Quem entra | Vai para |
|---|---|
| `tchearaujo@gmail.com` (admin) | `admin.html` |
| Qualquer outro usuário | `index.html` |
| Sem sessão | `login.html` |

---

## Tela de Login (`login.html`)

- Split screen: painel esquerdo CEA Blue + painel direito branco
- Esquerdo: **"A3 Digital"** grande em Roboto Bold (sem logo, sem ícone)
- Direito: "Bem-vindo de volta." em uma linha só
- Telas: Login / Recuperação de senha / Nova senha

---

## Painel Admin (`admin.html`)

- Sem logo na sidebar — nav "Gerenciar" direto no topo
- Tabela: **admin aparece primeiro** (espaço de 6px separa dos usuários comuns)
- Botões Resetar / Excluir alinhados à direita
- Modais com botão X + botões empilhados verticalmente
- Contagem de A3s por usuário via service role key

### Criar usuário sem e-mail (via SQL Editor)
```sql
UPDATE auth.users 
SET encrypted_password = crypt('senha_aqui', gen_salt('bf')),
    email_confirmed_at = now()
WHERE email = 'email_do_usuario@exemplo.com';
```

---

## A3 Digital (`index.html`)

### Arquitetura
- HTML único, sem backend, sem framework
- API Anthropic direto do browser (`anthropic-dangerous-direct-browser-access: true`)
- **API Key isolada por usuário:** `anthropic_api_key_<user_id>` no localStorage
  - O modal de setup só abre **após** o auth do Supabase — garante que `currentUserId` está disponível antes de qualquer leitura/escrita da chave
  - Campo tem `autocomplete="off"` para evitar autofill do browser
  - Na primeira sessão, migra a chave legacy `anthropic_api_key` para a chave isolada e **apaga a legacy** para não vazar para outros usuários
- **Histórico isolado por usuário:** `a3_history_<user_id>` no localStorage
- Supabase para auth + sincronização de contagem em `a3_documents`
- **Sem header**, **sem dark mode**

### Sidebar (CEA Blue)
- Logo "A3 Digital" em Roboto Medium
- Botão "Novo A3 com IA" (gradiente roxo→rosa)
- Botão "Novo A3 em branco"
- Botão "Gerar link de demanda" (azul soft — intake)
- Histórico com badge "NOVO" para demandas recebidas
- User badge (avatar Electric Blue + nome) acima dos botões de config
- Modal API Key com botão X

### Seções do A3
| # | Seção |
|---|---|
| 01 | Contexto e background |
| 02 | Descrição do problema |
| 03 | Situação atual e evidências (cards de métricas) |
| 04 | Análise de causa raiz (5 Porquês) |
| 05 | Estado alvo e hipótese (cards de métricas) |
| 06 | Benefícios e custos do projeto |
| 07 | Plano de ação (tabela: #, Ação, Responsável, Status) |
| 08 | Acompanhamento e validação |
| 09 | Equipe de execução |
| 10 | Stack de tecnologia (tabela: #, Tecnologia/API/Linguagem) |

### Decisões de design
- Título do box à esquerda, número à direita em `--color-cea-lighter`
- Cabeçalho de seção com fundo `--color-surface-soft`
- Status pills: Rascunho / Em revisão / Em andamento / Concluído
- Seções preenchidas pela IA: borda Electric Blue + badge "IA"
- Plano de ação e Stack de tecnologia: mesmo componente de tabela
- Footer: **Salvar A3** (Electric Blue) + **Imprimir / PDF** — "Limpar tudo" removido
- Focus nos inputs: `outline: none` global, sem estilo visual

---

## Funcionalidade de Intake — Link de Demanda

### Tabelas Supabase
```sql
create table if not exists public.intake_links (
  token       uuid primary key default gen_random_uuid(),
  user_id     uuid not null references auth.users(id) on delete cascade,
  owner_name  text,
  created_at  timestamptz default now()
);
alter table public.intake_links enable row level security;
create policy "Leitura pública" on public.intake_links for select using (true);
create policy "Usuário cria" on public.intake_links for insert with check (auth.uid() = user_id);

alter table public.a3_documents
  add column if not exists intake_token uuid,
  add column if not exists intake_data  text;
create policy "Inserção pública via intake" on public.a3_documents for insert
  with check (intake_token is not null);
```

### Fluxo
1. Usuário clica "Gerar link de demanda" → token criado em `intake_links` (reutilizável)
2. Link: `request.html?token=<uuid>`
3. Pessoa externa preenche sem login → salvo em `a3_documents` com `status='Demanda recebida'`
4. Dono vê demanda na sidebar com badge "NOVO"
5. Clica → triage pré-preenchido → "Gerar A3" → IA processa → salvo automaticamente → intake removido da sidebar
6. Status atualizado para `'Processado'` no Supabase

### Estados de status
- `'Demanda recebida'` → aparece na sidebar
- `'Processado'` → convertido em A3, não reaparece
- `'Excluído'` → deletado, não reaparece

### Limpar intakes antigos
```sql
UPDATE public.a3_documents SET status='Excluído'
WHERE status='Demanda recebida' AND intake_token IS NOT NULL;
```

---

## Página de Demanda (`request.html`)

- Header: "A3 Digital" (esquerda) + "Envio da demanda" (direita), fundo CEA Blue
- Título: "Conte o que você precisa." em uma linha
- Campos: título*, descrição*, nome (sem e-mail)
- Referências abertas por padrão (links + imagens)
- Telas: formulário → sucesso → link inválido
- Sem login necessário

---

## Agente de Briefing (`briefing-agente.html`)

- 3 níveis: Ajuste / Feature / Iniciativa
- 7 blocos de perguntas calibradas com tags de prioridade
- Resumo final + envio para análise no Claude
- **Status:** Funcional. Sem autenticação integrada ainda.

---

## Rodar localmente
```bash
cd ~/Downloads/A3digital
python3 -m http.server 8080
```
Acesse: `http://localhost:8080/login.html`

---

## Próximos passos em aberto
- **Deploy no Vercel via GitHub**
- **Configurar domínio próprio no Resend** para e-mails sem restrição
- Possível: conectar Briefing Agent com A3 em fluxo único
- Possível: integrar autenticação no Briefing Agent

---

## Como continuar
Cole esse arquivo no início de uma nova conversa com Claude e diga o que quer evoluir. Os arquivos HTML precisam ser compartilhados novamente se quiser editar o código.
