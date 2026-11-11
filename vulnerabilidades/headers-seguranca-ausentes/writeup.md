# 🧱 Ausência de Cabeçalhos de Proteção HTTP

**Fase:** 1 e 2 (Reconhecimento e Vulnerabilidades Preliminares)
**CVSS v3.1:** `AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:L/A:N` — **Score 4.3** (Média)
**Prioridade:** Média (exposição a Clickjacking)

## Descrição

A aplicação não envia dois cabeçalhos de segurança HTTP considerados padrão de mercado (*security headers baseline*):

- **`X-Frame-Options`** ausente — as páginas podem ser encapsuladas em um `<iframe>` por um site de terceiros.
- **`X-Content-Type-Options`** ausente — o navegador pode realizar *MIME sniffing* e interpretar um recurso com um tipo de conteúdo diferente do declarado.

## Impacto

- **Sem `X-Frame-Options`:** viabiliza ataques de **Clickjacking**, onde um site malicioso sobrepõe uma camada invisível da aplicação legítima sobre elementos clicáveis falsos, induzindo o usuário a executar ações não intencionais (ex.: alterar configurações, confirmar transações) dentro de uma sessão autêntica.
- **Sem `X-Content-Type-Options`:** aumenta a superfície para ataques de **MIME sniffing**, onde um arquivo malicioso disfarçado de um tipo benigno (ex.: imagem) é interpretado e executado como script pelo navegador.

## Remediação

Injetar os cabeçalhos diretamente no HAProxy (camada de borda), aplicando a correção a toda a aplicação sem alterar código:

```haproxy
http-response set-header X-Frame-Options DENY
http-response set-header X-Content-Type-Options nosniff
```

## Critério de aceitação do reteste

```bash
curl -sI https://<target-host>/ | grep -i "x-frame-options\|x-content-type-options"
```

- ✅ **Aprovado:** ambos os cabeçalhos presentes na resposta.
- ❌ **Reprovado:** qualquer um dos dois ausente.
