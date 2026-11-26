# 🛡️ Laboratório de Auditoria e Teste de Força Bruta com Medusa e Kali Linux

> **Projeto prático desenvolvido para o Bootcamp Santander Cibersegurança da [DIO](https://dio.me).**

---

## 📌 Visão Geral do Projeto

Este projeto tem como objetivo demonstrar, documentar e analisar técnicas de auditoria de autenticação e ataques de força bruta/dicionário em um ambiente estritamente controlado e isolado, utilizando o **Kali Linux** como estação de testes e o **Metasploitable 2** como alvo vulnerável.

Foram simulados três cenários fundamentais de auditoria:
1. **Força Bruta no protocolo FTP (Porta 21)**
2. **Força Bruta em Formulário Web com inspeção HTTP (DVWA - Porta 80)**
3. **Enumeração e Password Spraying no protocolo SMB (Porta 445)** com validação de acesso

Além da execução prática dos testes, este repositório enfatiza a **visão defensiva**, detalhando medidas de mitigação e boas práticas recomendadas para proteger ambientes corporativos contra essas ameaças.

---

## 🏗️ Topologia e Configuração do Ambiente

O laboratório foi estruturado em rede isolada para assegurar que nenhum tráfego vazasse para a rede externa ou doméstica.

```text
+--------------------------------------------------------------+
|                     VirtualBox (Host-Only)                   |
|                                                              |
|   +-----------------------+      +-----------------------+   |
|   |      Kali Linux       |      |    Metasploitable 2   |   |
|   |  (Estação de Testes)  | ---> |   (Alvo Vulnerável)   |   |
|   |   IP: 192.168.56.X    |      |   IP: 192.168.56.X    |   |
|   +-----------------------+      +-----------------------+   |
+--------------------------------------------------------------+
```

* **Virtualizador:** Oracle VM VirtualBox
* **Modo de Rede:** Placa exclusiva de hospedeiro (*Host-Only Network*)
* **Máquina de Auditoria:** Kali Linux
* **Máquina Alvo:** Metasploitable 2 (hospedando serviços FTP, Samba e a aplicação web DVWA)

---

## 🧰 Ferramentas Utilizadas

| Ferramenta | Finalidade no Laboratório |
| :--- | :--- |
| **Nmap** | Varredura de portas e enumeração de serviços ativos no alvo |
| **Medusa** | Auditoria paralela e modular de credenciais (módulos FTP, HTTP e SMB) |
| **Enum4linux** | Enumeração de usuários, grupos e políticas de compartilhamento SMB |
| **Smbclient** | Cliente de validação de acesso aos compartilhamentos autenticados |
| **DevTools (Browser)** | Inspeção dos parâmetros e cabeçalhos HTTP do formulário web |

---

## 🧪 Execução dos Cenários e Evidências

### 1. Reconhecimento Inicial (Nmap)

Foi realizada uma varredura para identificar as portas e serviços em execução no Metasploitable 2:

```bash
nmap -sV <IP_DO_METASPLOITABLE>
```

![Reconhecimento com Nmap](images/01-nmap-scan.png)

* **Serviços identificados como foco do teste:**
  * `21/tcp`: vsftpd 2.3.4 (FTP)
  * `80/tcp`: Apache HTTPD (hospedando o DVWA)
  * `139/tcp` / `445/tcp`: Samba (SMB)

---

### 2. Cenário 1: Auditoria em Serviço FTP (Porta 21)

O protocolo FTP transmite credenciais em texto claro e, por padrão, não implementa bloqueios nativos por taxa de erro.

* **Parâmetros conceituais do Medusa utilizados:**
  * `-h`: Endereço IP do alvo
  * `-u / -U`: Usuário ou lista de usuários (`wordlists/users.txt`)
  * `-p / -P`: Senha ou dicionário (`wordlists/passwords.txt`)
  * `-M ftp`: Módulo para o protocolo FTP
  * `-f`: Encerra o processo ao encontrar a primeira credencial válida

![Ataque de Força Bruta no FTP](images/02-medusa-ftp.png)

* **Resultado:** Credencial válida identificada com sucesso após teste automatizado da lista.

---

### 3. Cenário 2: Força Bruta em Formulário Web (DVWA)

Neste cenário, avaliou-se um formulário de login web sem proteção contra requisições automatizadas (como CAPTCHA ou rate limiting).

#### Passo 2.1: Acesso e Mapeamento da Aplicação
Acesso à interface do Damn Vulnerable Web Application (DVWA):

![Interface do DVWA](images/03-dvwa-interface.png)

#### Passo 2.2: Inspeção dos Parâmetros HTTP
Utilizando as ferramentas de desenvolvedor do navegador (*DevTools - Network*), foram capturados os campos enviados na requisição POST/GET (`username`, `password`, `Login`), o caminho da URL e o cookie de sessão:

![Inspeção com Ferramentas do Desenvolvedor](images/04-dvwa-dev-tools.png)

#### Passo 2.3: Identificação da Assinatura de Falha
Foi verificado qual padrão de resposta o sistema retorna ao errar as credenciais (ex: *"Login failed"*), permitindo ao Medusa diferenciar erros de acertos:

![Assinatura de Falha no Login](images/05-dvwa-login-fail.png)

#### Passo 2.4: Execução com Módulo Web do Medusa
O Medusa foi configurado para enviar as requisições automatizadas com os parâmetros do formulário:

![Execução do Medusa contra o Formulário Web](images/06-medusa-web-attack.png)

#### Passo 2.5: Validação do Acesso
Com a credencial obtida, o login com sucesso foi confirmado no painel da aplicação:

![Login com Sucesso no DVWA](images/07-dvwa-login-success.png)

---

### 4. Cenário 3: Enumeração e Password Spraying em SMB (Porta 445)

Em redes corporativas, o protocolo SMB é amplamente utilizado para compartilhamento de arquivos. O teste consistiu em enumerar usuários válidos e aplicar testes de credenciais comuns (*Password Spraying*).

#### Passo 3.1: Enumeração com Enum4linux
Enumeração prévia de contas de usuário e configurações da máquina alvo:

![Enumeração com Enum4linux](images/08-enum4linux-smb.png)

#### Passo 3.2: Ataque via Módulo SMB do Medusa
Auditoria de senhas contra as contas descobertas utilizando o módulo `smbnt`:

![Ataque no SMB com Medusa](images/09-medusa-smb.png)

#### Passo 3.3: Validação de Acesso com Smbclient
Com a credencial validada, foi efetuada a conexão direta e a listagem dos compartilhamentos protegidos via `smbclient`:

![Validação de Acesso com Smbclient](images/10-smbclient-validation.png)

---

## 🛡️ Análise de Riscos e Recomendações de Mitigação (Defesa)

A facilidade com que ferramentas automatizadas realizam centenas de tentativas em poucos segundos demonstra a fragilidade de sistemas que dependem apenas de senhas estáticas sem camadas defensivas.

### 1. Mecanismos de Bloqueio e Rate Limiting
* **Fail2ban / IP Banning:** Monitoramento contínuo dos logs de autenticação (`/var/log/auth.log`, logs do vsftpd e do servidor web). Configuração de regras que bloqueiam temporariamente no firewall (`iptables` / `nftables`) o endereço IP que exceder um limite seguro de falhas consecutivas (ex: 5 falhas em 10 minutos).
* **Account Lockout Policy:** Configuração de bloqueio temporário da conta de usuário no serviço de diretório (Active Directory / Samba) após sucessivas falhas, mitigando tentativas distribuídas de força bruta.

### 2. Proteção para Aplicações Web
* **Mecanismos Anti-Automação:** Implementação de CAPTCHA inteligente (ex: Cloudflare Turnstile, reCAPTCHA v3) após tentativas incorretas.
* **WAF (Web Application Firewall):** Regras de rate limiting por IP e por sessão para restringir o volume de requisições enviadas ao endpoint de login.

### 3. Políticas de Senhas Robustas
* Exigência de comprimento mínimo de no mínimo 12 a 16 caracteres.
* Verificação periódica de senhas contra dicionários de credenciais vazadas (*compromised password lists*).

### 4. Autenticação Multifator (MFA / 2FA)
* Implementação obrigatória de MFA (via aplicativo autenticador ou token FIDO2/WebAuthn), garantindo que a descoberta da senha estática não seja suficiente para a invasão.

### 5. Desativação de Serviços Legados e Uso de Criptografia
* O FTP padrão não utiliza criptografia (credenciais viajam em texto claro). A recomendação padrão de mercado é substituí-lo por **SFTP (SSH File Transfer Protocol)** ou **FTPS (FTP sobre TLS)**.
* Desativação de autenticação por senha no SSH em favor de chaves criptográficas (`Ed25519` / `RSA-4096`).

---

## 📁 Estrutura do Repositório

```text
├── README.md
├── images/
│   ├── 01-nmap-scan.png
│   ├── 02-medusa-ftp.png
│   ├── 03-dvwa-interface.png
│   ├── 04-dvwa-dev-tools.png
│   ├── 05-dvwa-login-fail.png
│   ├── 06-medusa-web-attack.png
│   ├── 07-dvwa-login-success.png
│   ├── 08-enum4linux-smb.png
│   ├── 09-medusa-smb.png
│   └── 10-smbclient-validation.png
└── wordlists/
    ├── users.txt
    └── passwords.txt
```

---

## ⚖️ Aviso Legal (Ethical Disclaimer)

> **Aviso:** Todas as ações, comandos e testes documentados neste repositório foram conduzidos em um ambiente virtualizado, isolado e controlado, com propósitos estritamente educacionais e em conformidade com as diretrizes do Bootcamp da Digital Innovation One (DIO). Não utilize essas ferramentas ou metodologias contra sistemas ou redes sem autorização prévia por escrito.

---

## 👤 Autor

* **Nome:** [Samuel]
* **Bootcamp:** Santander Cibersegurança - Do Zero à Prática (DIO)
* **LinkedIn:** [Adicione o link do seu LinkedIn aqui]
* **GitHub:** [Adicione o link do seu perfil GitHub aqui]
