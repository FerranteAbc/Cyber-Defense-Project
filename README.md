# 🛡️ FIAP Challenge 2026 — Relatório Técnico de Avaliação de Segurança Web (Mentor Web)

> Repositório com a documentação técnica consolidada das Fases 1 a 4 do **FIAP Challenge (1TDCOR-2026)** no ambiente de homologação da aplicação **Mentor Web** (`target-host.local`), cobrindo reconhecimento, varreduras ativas, exploração controlada, métricas CVSS v3.1 e plano de mitigação estruturado.

---

## 📌 1. Visão Geral do Alvo & Metodologia

* **Instituição:** Faculdade de Informática e Administração Paulista (FIAP).
* **Turma:** 1TDCOR — 2026.
* **Autores:** Bruno Henrique Veiga Sabino e Pedro Henrique Ferrante Prado.
* **Orientadores:** Prof. Silvio Cesar Roxo Giavaroto e Prof. Marcelo Gomes dos Santos.
* **Alvo de Avaliação:** `https://<target-host>` (Caminho base: `/devSecurityG5/`).
* **Resolução DNS / IP:** `host.dyndns.org` ➔ `<TARGET_IP>` (Ambiente dinâmico via DynDNS).
* **Ambiente Analisado:** Homologação / Testes (Sistema de Gestão ERP Mentor Web).
* **Abordagem Operacional:** *Ethical Hacking* não intrusivo com aplicação rigorosa de *rate limiting* e *throttling* para preservar a estabilidade dos serviços.

---

## 🏗️ 2. Mapeamento Tecnológico (Fingerprinting)

A infraestrutura foi mapeada através da correlação entre cabeçalhos de resposta HTTP, análise do comportamento de servlets Java e extração de telemetria:

```text
               [ Tráfego Externo / Auditor ]
                             │
                             ▼ (Portas 80 / 443)
              ┌─────────────────────────────┐
              │  HAProxy 2.0.0+ (Reverse)   │
              │  Balanceador / Borda        │
              └──────────────┬──────────────┘
                             │
                             ▼ (Roteamento Interno)
              ┌─────────────────────────────┐
              │    Apache Tomcat 9.0.93.0   │
              │  JVM: OpenJDK 21.0.1+12-29  │
              ├─────────────────────────────┤
              │ Backend: Java J2EE + JSF    │
              │ Frontend: PrimeFaces        │
              │ Tema UI: Sentinel           │
              └─────────────────────────────┘
```
### 🧩 Componentes Mapeados

* **Proxy Reverso & Borda:** `HAProxy 2.0.0` ou superior (evidenciado via headers e scripts do Nmap).
* **Servidor de Servlets Web:** `Apache Tomcat 9.0.93.0` (Build de *Aug 2 2024 21:24:59 UTC*).
* **Runtime da Aplicação:** `OpenJDK 21.0.1+12-29` (Oracle Corporation / JVM).
* **Camada de Backend:** Java (J2EE) implementado com o framework `JavaServer Faces (JSF)`, evidenciado pelas rotas com extensão `.jsf` e ciclo de vida de sessão.
* **Biblioteca de UI:** `PrimeFaces` com Tema *Sentinel*.
* **Contextos Internos Localizados:** `/devSecurityG5`, `/devMentorIntegrador` e `/devReport`.

## 🛠️ 3. Ficha Técnica de Ferramentas

| Ferramenta | Versão | Função Principal | Parâmetros de Throttling & Operação |
| :--- | :---: | :--- | :--- |
| **Nmap** | `7.95` | Reconhecimento de portas e identificação de serviços | `-sV -sC -Pn -T3 --max-rate 10 -oN scan_nmap.txt <target-host>` |
| **Dirsearch** | `v0.4.3` | Descoberta automatizada de diretórios e arquivos | `-u https://<target-host> -w <wordlist> -t 5 -e php,txt,html,conf` |
| **cURL** | `8.x.x` | Requisições HTTP e execução de PoC de exploração | `curl -i -sL https://<target-host>/metrics` |
| **Brave Browser** | `Atual` | Validação manual e inspeção de cabeçalhos/DOM | DevTools (`F12`) / Network / Storage |

---

## 🔍 4. Análise de Vulnerabilidades por Fase

### Fases 1 e 2: Reconhecimento e Vulnerabilidades Preliminares

* **Cookie de Sessão sem `HttpOnly` (Risco Médio):**  
  O cookie `ETSS` é enviado com a diretiva `Secure`, mas sem a flag `HttpOnly` (`Set-Cookie: ETSS=...; Path=/; Secure`). Permite que ataques de Cross-Site Scripting (XSS) capturem o token via JavaScript (`document.cookie`), viabilizando o sequestro de sessão (*Session Hijacking*).

* **Exposição de Token de Sessão na URL (Risco Médio/Alto):**  
  O sistema transmite parâmetros sensíveis via método `GET` na URL (`?pcaes=a205de9c60d3992e6296830743168a74`) contendo hash de sessão/transação. Isso compromete a confidencialidade ao gravar o identificador em logs de servidores intermediários, proxies e no histórico do navegador.

* **Ausência de Cabeçalhos de Proteção:**  
  * **Falta do `X-Frame-Options`:** Expõe as páginas ao encapsulamento em iframes, viabilizando ataques de *Clickjacking*.  
  * **Falta do `X-Content-Type-Options`:** Permite interpretação indevida de tipos MIME (*MIME Sniffing*).

* **Desafio Metodológico (Soft 404 Dinâmico):**  
  O servidor responde com código `HTTP 200 OK` (páginas de ~25.7 KB) para requisições de arquivos inexistentes, gerando falsos positivos em ferramentas como Gobuster e exigindo validação manual.

---

### Fase 3: Varreduras Ativas e Identificação de Endpoints

* **Portas Abertas:** TCP `80` (HTTP) e `443` (HTTPS), ambas intermediadas pelo HAProxy.
* **Endpoints Mapeados:**
  * `/robots.txt`: Contendo a regra `Disallow: /` (segurança por obscuridade).
  * `/probe/`: Rota com mecanismo de autenticação ativo.
  * `/metrics/`: Endpoint de monitoramento exposto publicamente sem qualquer restrição de acesso.

### Fase 4: Prova de Conceito (PoC) — Endpoint `/metrics/`

Execução controlada via terminal demonstrando o vazamento de dados de infraestrutura:

```bash
curl -i -sL https://<target-host>/metrics
```
### 🔍 Evidência Técnica Capturada:
HTTP/1.1 302 Found
location: /metrics/
transfer-encoding: chunked
date: Wed, 20 May 2026 22:29:38 GMT
strict-transport-security: max-age=31536000 includeSubDomains; preload;

HTTP/1.1 200 OK
content-type: text/plain;version=0.0.4;charset=utf-8
transfer-encoding: chunked
date: Wed, 20 May 2026 22:29:38 GMT
strict-transport-security: max-age=31536000; includeSubDomains; preload;
```bash
# HELP jvm_threads_current Current thread count of a JVM
# TYPE jvm_threads_current gauge
jvm_threads_current 169.0

# HELP jvm_info JVM version info
jvm_info{version="21.0.1+12-29", vendor="Oracle Corporation", runtime="OpenJDK Runtime Environment",} 1.0

# HELP tomcat_session_active_total Number of active sessions
tomcat_session_active_total{host="localhost", context="/devSecurityG5",} 23.0
tomcat_session_active_total{host="localhost", context="/devMentorIntegrador",} 41.0

tomcat_info{version="9.0.93.0", build="Aug 2 2024 21:24:59 UTC",} 1.0
```
## 📊 5. Classificação de Riscos (CVSS v3.1)

| Vulnerabilidade | Vetor CVSS v3.1 | Score | Severidade | Prioridade |
| :--- | :--- | :---: | :---: | :---: |
| **Exposição de Dados Técnicos e Contextos (`/metrics/`)** | `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N`[cite: 1] | **5.3**[cite: 1] | Média[cite: 1] | **Alta** (Facilidade de exploração e mapeamento de superfície)[cite: 1] |
| **Exposição de Token de Sessão via URL (`?pcaes=`)** | `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:L/I:L/A:N` | **5.4** | Média | **Alta** (Vazamento de credenciais em logs) |
| **Cookie de Sessão sem flag `HttpOnly` (`ETSS`)** | `CVSS:3.1/AV:N/AC:H/PR:N/UI:R/S:U/C:H/I:N/A:N` | **5.3** | Média | **Média** (Dependência de vetor XSS) |
| **Ausência de Security Headers (`X-Frame-Options`)** | `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:L/A:N` | **4.3** | Baixa | **Média** (Exposição a Clickjacking) |

## 🛡️ 6. Plano de Ação Técnica (Remediação)

* **1. Correção Imediata (Hotfix — HAProxy):**  
  Implementar ACL no balanceador para rejeitar qualquer acesso externo a rotas iniciadas por `/metrics`, respondendo com código `403 Forbidden`:
  ```haproxy
  acl is_metrics path_beg -i /metrics
  http-request deny if is_metrics
  ```
* **2. Correção Temporária (Workaround — Autenticação):** 
Caso haja necessidade de acesso de coletores remotos antes do ajuste definitivo, condicionar o endpoint a Autenticação HTTP Básica (Basic Auth) sob TLS.  
* **3. Correção Definitiva (Permanent Fix — Apache Tomcat):** 
Reconfigurar a aplicação e o Tomcat para realizar o bind das métricas exclusivamente na interface de loopback (127.0.0.1), permitindo a coleta apenas por agentes locais. 
* **4. Correções Adicionais de Aplicação:** 
```bash
Inserir <http-only>true</http-only> no bloco <session-config> do web.xml.  
Migrar a transmissão do token pcaes de GET para requisições POST seguras ou SessionStorage.  
Injetar os cabeçalhos X-Frame-Options: DENY e X-Content-Type-Options: nosniff no HAProxy[cite: 1].
```
## 🧪 7. Plano de Reteste e Critérios de Aceitação

Rotina de validação pós-mitigação via terminal:

```bash
curl -o /dev/null -s -w "%{http_code}\n" https://<target-host>/metrics
```
* **Critérios de Aceitação (Sucesso):** O retorno deve ser exclusivamente 401 Unauthorized, 403 Forbidden ou 404 Not Found.  
* **Critério de Reprovação (Falha Crítica):** O retorno do código 200 OK configura persistência da vulnerabilidade e não conformidade crítica
## 📁 8. Estrutura Sugerida do Repositório

```text
.
├── docs/
│   ├── relatorio-tecnico-final.pdf
│   └── arquitetura-haproxy-tomcat.md
├── scans/
│   ├── scan_nmap.txt
│   └── report_dirsearch.txt
├── poc/
│   ├── curl_metrics_evidence.log
│   └── token_leak_sample.png
├── configs/
│   ├── haproxy_acl_fix.cfg
│   └── web_xml_session_hardening.xml
└── README.md
````
## 👤 9. Autores
Bruno Henrique Veiga Sabino

Pedro Henrique Ferrante Prado
