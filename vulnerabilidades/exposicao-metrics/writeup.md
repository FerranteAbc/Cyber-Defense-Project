# 📊 Exposição de Dados Técnicos — Endpoint `/metrics/`

**Fase:** 3 e 4 (Varredura Ativa & PoC)
**CVSS v3.1:** `AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` — **Score 5.3** (Média)
**Prioridade:** Alta (facilidade de exploração e mapeamento de superfície)

## Descrição

O endpoint `/metrics/` da aplicação **Mentor Web** estava publicamente acessível, sem qualquer mecanismo de autenticação ou restrição de rede, expondo um painel de telemetria no formato Prometheus com dados internos de infraestrutura: versão exata da JVM, fornecedor, contagem de threads ativas e número de sessões ativas por contexto da aplicação.

## Evidência Técnica (PoC)

```bash
curl -i -sL https://<target-host>/metrics
```

**Resposta observada** (ver evidência completa em [`poc/curl_metrics_evidence.log`](../../poc/curl_metrics_evidence.log)):

```text
HTTP/1.1 200 OK
content-type: text/plain;version=0.0.4;charset=utf-8

jvm_threads_current 169.0
jvm_info{version="21.0.1+12-29", vendor="Oracle Corporation", runtime="OpenJDK Runtime Environment",} 1.0
tomcat_session_active_total{host="localhost", context="/devSecurityG5",} 23.0
tomcat_session_active_total{host="localhost", context="/devMentorIntegrador",} 41.0
tomcat_info{version="9.0.93.0", build="Aug 2 2024 21:24:59 UTC",} 1.0
```

## Impacto

Essas informações permitem que um atacante faça *fingerprinting* preciso da stack (JVM, Tomcat, contextos internos) e meça a carga de usuários ativos — dados valiosos para planejar exploração de CVEs conhecidas das versões identificadas e para dimensionar ataques de negação de serviço.

## Remediação

1. **Hotfix imediato (HAProxy):** bloquear acesso externo ao path `/metrics` — ver [`configs/haproxy_acl_fix.cfg`](../../configs/haproxy_acl_fix.cfg).
2. **Workaround temporário:** condicionar o endpoint a Basic Auth sob TLS, caso coletores remotos precisem de acesso antes do fix definitivo.
3. **Correção definitiva (Tomcat):** fazer o *bind* do endpoint de métricas exclusivamente à interface de loopback (`127.0.0.1`), restringindo a coleta a agentes locais.

## Critério de aceitação do reteste

```bash
curl -o /dev/null -s -w "%{http_code}\n" https://<target-host>/metrics
```

- ✅ **Aprovado:** `401`, `403` ou `404`.
- ❌ **Reprovado:** `200 OK` — vulnerabilidade persiste.
