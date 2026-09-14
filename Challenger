## 📌 Visão Geral do Projeto

O projeto consiste no desenvolvimento e implementação de estratégias de segurança cibernética ofensiva e defensiva aplicadas a problemas reais propostos pelo ecossistema acadêmico da FIAP. 

As entregas foram estruturadas em fases cumulativas, integrando:
* Infraestrutura de redes e isolamento de tráfego.
* Auditoria e mapeamento de superfícies de ataque.
* Execução de testes de intrusão controlados e documentação de conformidade.
* Recomendações práticas de mitigação e *hardening*.

---

## 🛠️ Tecnologias & Ferramentas

| Categoria | Ferramentas |
| :--- | :--- |
| **Virtualização & SO** | Kali Linux, Metasploitable 2, Microsoft Hyper-V |
| **Enumeração & Varredura** | Nmap, Gobuster, Nikto |
| **Exploração & Pentest** | Metasploit Framework, scripts customizados |
| **Análise & Forense** | Wireshark, tcpdump |
| **Documentação** | Markdown, Git, Relatórios PoC |

---

## 🏗️ Arquitetura do Ambiente de Testes (Lab)

Para validar superfícies de ataque e testar controles defensivos sem expor serviços externos, foi provisionado um ambiente virtualizado isolado:

```text
       [ Host Físico (Windows / Hyper-V) ]
                       │
       [ Switch Virtual (Internal / Host-Only) ]
           ┌───────────┴───────────┐
           ▼                       ▼
    [ Kali Linux ]         [ Metasploitable 2 ]
  (Auditoria / Atacante)    (Ambiente Alvo / PoC)
Segmentação de Rede: Interfaces virtuais dedicadas para manter o tráfego de exploração estritamente local.

Contenção: Bloqueio de roteamento externo das máquinas-alvo durante a validação dos exploits.

📋 Fases do Projeto
1. Planejamento & Mapeamento de Superfície
[x] Levantamento de escopo e definição de requisitos de segurança.

[x] Varredura passiva e ativa de hosts, portas abertas e serviços expostos.

[x] Mapeamento de vetores de entrada potenciais via enumeração detalhada.

2. Análise de Vulnerabilidades & PoC
[x] Testes de exploração controlados para confirmação de falhas conhecidas.

[x] Coleta de evidências e logs de execução.

[x] Elaboração de relatório técnico de Prova de Conceito (Proof of Concept).

3. Mitigação & Estratégias Defensivas
[x] Proposição de plano de hardening (desativação de serviços legados e atualização de patches).

[x] Regras de filtragem de tráfego e segmentação de sub-redes.

[x] Alinhamento com práticas de Secure DevOps e governança de vulnerabilidades.

4. Demonstração Técnica
[x] Produção de apresentações em vídeo demonstrando a execução prática do laboratório e a defesa da solução.

📁 Estrutura do Repositório
Plaintext
.
├── docs/
│   ├── relatorio-analise-vulnerabilidades.pdf
│   └── arquitetura-ambiente-lab.md
├── poc/
│   ├── evidencias/
│   └── scripts/
├── network-analysis/
│   └── pcaps/
└── README.md
👤 Autor
Pedro Henrique Ferrante Prado — Defesa Cibernética, FIAP
