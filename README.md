# 🚀 Projeto: Meu Laboratório SOC (Wazuh SIEM / XDR)

Este repositório documenta a construção do meu laboratório de Segurança Defensiva e Operações de Segurança (SOC). O objetivo foi criar um ambiente controlado para centralização de logs, análise de vulnerabilidades, monitoramento de integridade e simulação de ataques reais baseados na matriz MITRE ATT&CK.

---

## 🛠️ Arquitetura do Ambiente

O laboratório foi construído utilizando o **VirtualBox** para virtualização e o **Docker/Portainer** para orquestração simplificada dos serviços do servidor.

*   **Servidor SOC:** Ubuntu Server executando a stack do **Wazuh (Single-Node)** via Docker Compose, gerenciado pelo **Portainer**.
*   **Endpoint Monitorado:** Windows 10 com o agente nativo do Wazuh instalado e integrado ao **Sysmon** para enriquecimento de logs.
*   **Máquina de Ataque:** Kali Linux utilizada para executar varreduras e simulações de intrusão.

---

##  Passo a Passo da Implementação

### 1. Infraestrutura e Dockerização do Server
*   Configuração do Ubuntu Server com IP estático e regras restritas de Firewall (UFW) para comunicação segura com os agentes.
*   Instalação do Docker e Portainer para deploy e gerenciamento dos containers do Wazuh Manager, Indexer e Dashboard.
*   Ajustes de segurança na stack, incluindo a rotação e troca de credenciais padrão de admin do Kibana/Wazuh.

### 2. Hardening e Enriquecimento do Endpoint (Windows)
*   Deploy do agente do Wazuh no Windows 10 apontando para o IP do servidor Ubuntu.
*   Instalação e integração do **Microsoft Sysmon** no Windows, mapeando os eventos de sistema diretamente no arquivo `ossec.conf` do agente para capturar telemetria detalhada de processos e rede.
*   Ativação dos módulos nativos do Wazuh:
    *   **SCA (Security Configuration Assessment):** Para auditoria de conformidade e hardening do SO.
    *   **Vulnerability Detection:** Para inventariar CVEs ativas na máquina.
    *   **FIM (File Integrity Monitoring):** Para monitorar alterações em arquivos críticos do sistema.
    *   **AuditD (Linux):** Implementação de monitoramento de chamadas de sistema em ativos Linux via CDB-Lists.

### 3. Integração com Threat Intelligence
*   Configuração do módulo de integração do Wazuh com a API do **VirusTotal** para análise automatizada de hashes suspeitos detectados nos endpoints.

---

## 🛡️ Simulações de Ataque & Threat Hunting (MITRE ATT&CK)

Para validar a eficácia das regras e alertas do SIEM, utilizei o framework **Atomic Red Team** no Windows e ferramentas do Kali Linux para simular técnicas reais de adversários.

### Caso 1: Persistência via Tarefas Agendadas (T1053.005)
*   **Simulação:** Execução de scripts para criar tarefas agendadas maliciosas no Windows.
*   **Resultado:** O Sysmon capturou a criação do processo e o Wazuh disparou o alerta correspondente na console. Desenvolvi um dashboard personalizado para rastrear os binários que mais agendaram tarefas.

📌 *[Insira aqui o Print do Dashboard/Alerta da técnica T1053.005]*

### Caso 2: Bypass de Defesa via Regsvr32 (T1218.010)
*   **Simulação:** Uso do utilitário legítimo do Windows (`regsvr32.exe`) para descarregar e executar scripts maliciosos remotamente, evadindo controles de aplicação comuns.
*   **Resultado:** Threat hunting focado em comportamentos anômalos do processo pai/filho na console do Wazuh.

📌 *[Insira aqui o Print do Alerta do Regsvr32]*

### Caso 3: Ataque Web e Brute-Force com Resposta Ativa (Active-Response)
*   **Simulação 1:** Varredura agressiva de diretórios web usando a ferramenta **Nikto** contra um servidor protegido por Apache + ModSecurity.
*   **Simulação 2:** Ataque de força bruta via SSH vindo do Kali Linux contra o servidor Ubuntu.
*   **Defesa Automatizada:** Configuração do **Active-Response** no Wazuh. Assim que o limite de falhas de login/ataques foi atingido, o Wazuh acionou automaticamente scripts de contenção (`firewall-drop` e `host-deny`), bloqueando o IP do atacante (Kali Linux) em tempo real diretamente no UFW.

📌 *[Insira aqui o Print da regra de Active Response mitigando o ataque]*

---

## 📈 Conclusão e Aprendizados

Este laboratório me permitiu entender na prática o ciclo completo de engenharia de detecção: desde a coleta do log bruto no endpoint (Sysmon/AuditD), a normalização e correlação pelo Wazuh Manager, até a criação de respostas automatizadas para mitigar incidentes antes que eles se espalhem pela rede.
