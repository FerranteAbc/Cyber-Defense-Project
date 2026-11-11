# 🔗 Exposição de Token de Sessão via URL (`?pcaes=`)

**Fase:** 1 e 2 (Reconhecimento e Vulnerabilidades Preliminares)
**CVSS v3.1:** `AV:N/AC:L/PR:N/UI:R/S:U/C:L/I:L/A:N` — **Score 5.4** (Média)
**Prioridade:** Alta (vazamento de credenciais em logs)

## Descrição

A aplicação transmite um parâmetro sensível via método **GET** na própria URL (`?pcaes=<hash-omitido>`), contendo um hash associado à sessão/transação do usuário.

## Impacto

Parâmetros transmitidos via `GET` ficam registrados em múltiplos pontos fora do controle da aplicação:

- Logs de acesso do próprio servidor web e de proxies/balanceadores intermediários (HAProxy).
- Histórico de navegação do navegador do usuário.
- Cabeçalho `Referer` de requisições subsequentes a recursos de terceiros.

Qualquer um desses pontos de persistência pode ser acessado por um atacante com acesso aos logs ou ao dispositivo do usuário, permitindo reuso do identificador de sessão (*session riding*).

## Remediação

- Migrar a transmissão do token `pcaes` de requisições `GET` para `POST` sobre TLS, removendo o identificador da URL.
- Alternativamente, armazenar o token em `SessionStorage` no cliente, nunca na *query string*.

## Critério de aceitação do reteste

Validar manualmente (Network tab do navegador / `curl -v`) que nenhuma requisição subsequente à autenticação carrega identificadores de sessão na URL — apenas em cookies com `Secure`, `HttpOnly` e `SameSite` configurados, ou no corpo de requisições `POST`.
