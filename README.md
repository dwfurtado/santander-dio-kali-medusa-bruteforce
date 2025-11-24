# 🛡️ Relatório de Pentest: Do Reconhecimento à Exploração (FTP, Web & SMB)

> **Desafio de Projeto DIO:** Implementação de auditoria de segurança prática utilizando Kali Linux, Medusa e ferramentas de enumeração.

## 📋 Resumo do Projeto
Este relatório documenta a simulação de testes de intrusão (Pentest) em um ambiente controlado. O objetivo foi identificar vulnerabilidades, realizar ataques de força bruta e explorar falhas de configuração em serviços de rede e aplicações web.

---

## ⚙️ 1. Preparação do Ambiente
O laboratório foi montado utilizando o VirtualBox com rede em modo **Host-Only** para isolamento.
* **Atacante:** Kali Linux
* **Alvo:** Metasploitable 2

![Ambiente](./01_ambiente.jpg)

Validamos a comunicação entre as máquinas antes de iniciar os testes.
![Ping e Conectividade](./02_conectividade.jpg)

---

## 🔍 2. Reconhecimento (Scanning)
Utilizamos o **Nmap** para mapear a superfície de ataque.
**Comando:** `nmap -sV -p 21,22,80,445,139 192.168.239.129`

![Nmap Scan](./03_nmap.jpg)
> **Descoberta:** Portas abertas para FTP (21), HTTP (80) e SMB (139/445).

---

## ⚔️ 3. Vetor de Ataque 1: FTP (Porta 21)

### Preparação
Criamos wordlists iniciais simples para testar credenciais padrão.
![Wordlists FTP](./04_users_pass_lists_1.jpg)

### Execução (Brute Force)
Utilizamos o **Medusa** para testar as senhas contra o serviço FTP.
**Comando:** `medusa -h 192.168.239.129 -U users.txt -P pass.txt -M ftp -t 6`

![Ataque Medusa FTP](./05_medusa_1.jpg)

### Validação
Confirmamos o acesso logando no servidor FTP com a credencial encontrada (`msfadmin`).
![Acesso FTP](./06_ftp.jpg)

---

## 🌐 4. Vetor de Ataque 2: Aplicação Web (DVWA)

### Análise
No navegador, analisamos a requisição de login (POST) para entender os campos necessários.
![Login DVWA](./09_formulario_2.jpg)
![Análise de Rede](./07_formulario_1.jpg)

### Execução (Automação)
Utilizamos o **Hydra** para testar combinações de senha no formulário HTML.
![Ataque Hydra](./08_hydra.jpg)

### Validação
Acesso administrativo concedido ao painel do DVWA.
![Sucesso Web](./10_formulario_3.jpg)

---

## 🏢 5. Vetor de Ataque 3: SMB/Samba (Porta 445)

### Enumeração (Enumeration)
Para este serviço, utilizamos o **Enum4Linux** para descobrir nomes de usuários válidos no sistema e informações do domínio.
**Comando:** `enum4linux -a 192.168.239.129`

![Enum4Linux Start](./11_enumeracao_1.jpg)
![Enum4Linux SIDs](./12_enumeracao_2.jpg)

O comando listou diversos usuários locais:
![Usuários Encontrados](./13_enumeracao_3.jpg)

### Password Spraying
Com os usuários enumerados, criamos uma nova lista específica e realizamos um ataque de *Password Spraying* (tentar poucas senhas em muitos usuários).
![Novas Wordlists](./14_users_pass_lists_2.jpg)

Utilizamos o módulo `smbnt` do Medusa:
**Comando:** `medusa -M smbnt -U smb_users.txt -P senhas_spray.txt ...`

![Ataque SMB Medusa](./15_medusa_2.jpg)

### Validação
Conectamos via **smbclient** para listar os compartilhamentos disponíveis.
![SMB Client](./16_smbclient.jpg)

---

## 🛡️ Medidas de Mitigação
1.  **Bloqueio de Enumeração:** Configurar o Samba para não permitir conexões anônimas ou listagem de usuários (Restrict Anonymous).
2.  **Senhas Fortes:** Política de senhas complexas impediria os ataques de dicionário realizados.
3.  **Monitoramento:** Alertas para múltiplas falhas de login (Brute Force) em curto período.

---
*Projeto realizado para fins educacionais.*
