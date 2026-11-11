# 🏗️ Arquitetura Mapeada — Mentor Web

Mapeamento tecnológico (*fingerprinting*) da infraestrutura-alvo, obtido por correlação entre cabeçalhos de resposta HTTP, comportamento de servlets Java e telemetria exposta.

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

## Componentes mapeados

| Componente | Detalhe |
| :--- | :--- |
| **Proxy reverso / borda** | HAProxy 2.0.0 ou superior (evidenciado via headers e scripts do Nmap) |
| **Servidor de servlets** | Apache Tomcat 9.0.93.0 (build *Aug 2 2024 21:24:59 UTC*) |
| **Runtime da aplicação** | OpenJDK 21.0.1+12-29 (Oracle Corporation) |
| **Camada de backend** | Java (J2EE) com JavaServer Faces (JSF) — evidenciado pelas rotas `.jsf` |
| **Biblioteca de UI** | PrimeFaces, tema *Sentinel* |
| **Contextos internos localizados** | `/devSecurityG5`, `/devMentorIntegrador`, `/devReport` |

Essa superfície foi reconstruída sem acesso direto à infraestrutura, apenas a partir de respostas HTTP observáveis externamente — o que por si só já demonstra o risco de exposição de metadados (versões de JVM/Tomcat, contextos internos) discutido em [`vulnerabilidades/exposicao-metrics/`](../vulnerabilidades/exposicao-metrics/writeup.md).
