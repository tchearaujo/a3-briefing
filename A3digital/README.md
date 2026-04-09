# A3 Digital — Design Tools Suite

Conjunto de ferramentas digitais para processos de design de produto, desenvolvidas para apoiar workflows de escopo e resolução de problemas.

Acesse em: [tchearaujo.com/experimentos/a3digital](https://tchearaujo.com/experimentos/a3digital)

---

## Ferramentas

| Arquivo | Rota | Descrição |
|---|---|---|
| `login.html` | `/experimentos/a3digital/login.html` | Tela de login, recuperação e nova senha |
| `index.html` | `/experimentos/a3digital/index.html` | A3 Digital — ferramenta principal de resolução de problemas |
| `admin.html` | `/experimentos/a3digital/admin.html` | Painel de gerenciamento de usuários |
| `request.html` | `/experimentos/a3digital/request.html` | Página pública de envio de demanda (sem login) |
| `briefing-agente.html` | `/experimentos/a3digital/briefing-agente.html` | Agente de Briefing para escopo de projetos |
| `design-system.css` | — | Tokens centralizados de design — base de todos os arquivos |

---

## A3 Digital

Ferramenta inspirada na metodologia A3 da Toyota, adaptada para processos de design de produto. Permite estruturar problemas em 10 seções, com suporte a IA (Anthropic API), histórico local e intake de demandas externas.

**Seções do A3:**
1. Contexto e background
2. Descrição do problema
3. Situação atual e evidências
4. Análise de causa raiz (5 Porquês)
5. Estado alvo e hipótese
6. Benefícios e custos do projeto
7. Plano de ação
8. Acompanhamento e validação
9. Equipe de execução
10. Stack de tecnologia

---

## Agente de Briefing

Ferramenta para escopo de projetos de design com fluxo de perguntas calibradas por nível de complexidade: Ajuste, Feature ou Iniciativa.

---

## Stack

- HTML + CSS + JavaScript (vanilla, single-file por ferramenta)
- [Supabase](https://supabase.com) — autenticação e banco de dados
- [Anthropic API](https://anthropic.com) — inteligência artificial (chave por usuário)
- [Resend](https://resend.com) — e-mails transacionais
- [Vercel](https://vercel.com) — hospedagem e deploy contínuo

---

## Rodar localmente

```bash
cd caminho/para/a3-briefing
python3 -m http.server 8080
```

Acesse: [http://localhost:8080/experimentos/a3digital/login.html](http://localhost:8080/experimentos/a3digital/login.html)

---

## Autor

**Antonio Araujo** — Senior Product Designer  
[tchearaujo.com](https://tchearaujo.com)
