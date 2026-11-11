# 🍪 Cookie de Sessão sem flag `HttpOnly` (`ETSS`)

**Fase:** 1 e 2 (Reconhecimento e Vulnerabilidades Preliminares)
**CVSS v3.1:** `AV:N/AC:H/PR:N/UI:R/S:U/C:H/I:N/A:N` — **Score 5.3** (Média)
**Prioridade:** Média (dependência de vetor XSS)

## Descrição

O cookie de sessão `ETSS` é emitido com a diretiva `Secure` (transmitido apenas via HTTPS), mas **sem a flag `HttpOnly`**:

```http
Set-Cookie: ETSS=...; Path=/; Secure
```

## Impacto

A ausência de `HttpOnly` permite que o valor do cookie seja lido via JavaScript do lado do cliente (`document.cookie`). Combinado com qualquer vulnerabilidade de **Cross-Site Scripting (XSS)** presente na aplicação — mesmo que não diretamente nesta avaliação —, um atacante poderia exfiltrar o cookie e realizar sequestro de sessão (*session hijacking*), assumindo a identidade do usuário autenticado sem precisar de suas credenciais.

## Remediação

Inserir a diretiva `http-only` no bloco `<session-config>` do `web.xml` da aplicação Tomcat:

```xml
<session-config>
  <cookie-config>
    <http-only>true</http-only>
    <secure>true</secure>
  </cookie-config>
</session-config>
```

Ver arquivo de referência em [`configs/web_xml_session_hardening.xml`](../../configs/web_xml_session_hardening.xml).

## Critério de aceitação do reteste

Inspecionar o cabeçalho `Set-Cookie` da resposta de autenticação e confirmar a presença simultânea de `Secure` e `HttpOnly` no cookie `ETSS`.
