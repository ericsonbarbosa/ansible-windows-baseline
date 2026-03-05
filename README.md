# 🚀 Automação de Baseline Windows com Ansible

Transformando horas de cliques em *"Next, Next, Finish"* em segundos de execução via código. 

Este projeto é focado em **Infraestrutura como Código (IaC)** para gerenciar, padronizar e configurar estações de trabalho Windows de forma 100% automatizada, silenciosa e idempotente.

---

## 🛠️ O Que Foi Feito?

Criamos uma arquitetura modularizada baseada em *Roles* do Ansible para implantar o "Kit Básico" corporativo e softwares de nicho, resolvendo diversos desafios de instaladores complexos:

* 🛡️ **Segurança (Bitdefender):** Instalação silenciosa com injeção de arquivo `.xml` (Tenant/GravityZone) utilizando iteração de loops, garantindo que o agente comunique com a nuvem correta.
* 📊 **Inventário (GLPI Agent):** Instalação via `.msi` com argumentos customizados, validação de serviço ativo e automação de chamadas web (`win_uri` e `win_wait_for`) para forçar o inventário imediato na porta local `62354`.
* 🏢 **ERP (TOTVS RM T-Cloud):** Resolução de problemas com instaladores customizados/teimosos através do método "Bypass" (transferência via `.zip`, extração direta com `community.windows.win_unzip` e criação de atalhos globais via *PowerShell*).
* 📦 **Essenciais:** Gerenciamento do ecossistema básico (Navegadores, Chocolatey, GCPW, etc).

---

## 💻 Stacks Envolvidas

* **Ansible / Ansible-Core:** Orquestração central e gerenciamento de estado.
* **YAML:** Modelagem dos playbooks e variáveis.
* **PowerShell / WinRM:** Comunicação remota e execução de scripts de baixo nível.
* **Módulos Windows do Ansible:** `ansible.windows` e `community.windows` (win_copy, win_package, win_stat, win_unzip, win_uri, win_wait_for).

---

## 🧠 Skills Utilizadas

* **Idempotência:** Garantia de que o código pode rodar 1.000 vezes na mesma máquina sem quebrar nada, aplicando mudanças apenas onde é estritamente necessário.
* **Troubleshooting Avançado:** Engenharia reversa de instaladores (descobrir parâmetros silenciosos, empacotadores NSIS/InstallShield e criar métodos alternativos como injeção direta de ZIP).
* **Manipulação de Serviços:** Automação de espera ativa de portas lógicas (Listening) e status de serviços do Windows.
* **Arquitetura de TI:** Organização do código em *Roles* separadas e uso inteligente de *Tags* para execuções parciais.

---

## 🪄 Preparando a Estação Windows (A Mágica)

Para que o servidor Linux (Ansible) consiga controlar a estação Windows remotamente, precisamos habilitar o protocolo **WinRM** no desktop. 

Abra o **PowerShell como Administrador** na máquina Windows alvo e rode estes dois comandos mágicos oficiais do Ansible:

```powershell
# 1. Baixa o script oficial de preparação do WinRM para o Ansible
Invoke-WebRequest -Uri [https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1](https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1) -OutFile ConfigureRemotingForAnsible.ps1

# 2. Executa o script ativando a comunicação remota
powershell -ExecutionPolicy RemoteSigned .\ConfigureRemotingForAnsible.ps1
```

---

## 📖 Guia Avançado de Execução (Cheat Sheet Ansible)

Aqui estão os comandos e parâmetros mais poderosos para você ter controle cirúrgico sobre o que, onde e como seus playbooks vão rodar no seu parque de máquinas:

### 1. Execução Padrão
Roda todas as tasks (sem filtros) nas máquinas configuradas no seu arquivo de inventário padrão:
```bash
ansible-playbook playbooks/baseline.yml
```

### 2. Mirando em Alvos Específicos (--limit)
Use o parâmetro --limit para apontar para um IP único ou um grupo específico:
```bash
# Rodar apenas em uma máquina específica
ansible-playbook playbooks/baseline.yml --limit 192.168.3.100
```
```bash
# Rodar apenas no grupo de computadores do laboratório (exemplo de grupo no inventário)
ansible-playbook playbooks/baseline.yml --limit "laboratorio_informatica"
```

### 3. O Poder das Tags (--tags e --skip-tags)
Filtre exatamente quais blocos de código você quer executar, economizando muito tempo:
```bash
# Rodar APENAS as roles de Segurança (Bitdefender) e Inventário (GLPI)
ansible-playbook playbooks/baseline.yml --tags "security,glpi"
```
```bash
# Rodar TUDO do baseline, EXCETO a instalação do navegador Chrome
ansible-playbook playbooks/baseline.yml --skip-tags "chrome"
```

### 4. Modo Simulação / Teste Seguro (--check)
Quer ver o que o Ansible faria na máquina sem alterar nada de verdade? O modo check (dry-run) é perfeito para homologar scripts novos:
```bash
ansible-playbook playbooks/totvs.yml --check
```