# 💥 Relatório de Análise de Vulnerabilidades e Teste de Invasão

---

## 🧩 1. Resumo

Este relatório apresenta os resultados de um teste de invasão realizado na **máquina virtual Metasploitable2**, com o objetivo de identificar e explorar vulnerabilidades conhecidas.  
O processo envolveu **escaneamento de portas**, **enumeração de serviços** e **ataques de força bruta** contra **FTP**, **HTTP** e **SMB**.

---

## 🎯 2. Informações do Alvo

| Informação | Detalhe |
|-------------|----------|
| 🖥️ Sistema | Metasploitable2 |
| 💿 Plataforma | VMware |
| 🌐 Endereço IP | `192.168.80.129` |
| 🧭 Endereço MAC | `00:0C:29:50:53:AC` |

---

## 🔍 3. Fase de Reconhecimento e Enumeração

### ⚙️ 3.1. Escaneamento de Portas e Serviços

**Comando utilizado:**
```bash
nmap -sV -p 21,22,80,445,139 192.168.80.129
```

**Explicação do comando:**
- `-sV`: Detecta a versão do serviço rodando em cada porta.
- `-p 21,22,80,445,139`: Define as portas específicas a serem escaneadas (FTP, SSH, HTTP, SMB).
- `192.168.80.129`: Endereço IP do alvo.

**Resultados principais:**
- FTP (21/tcp): vsftpd 2.3.4
- SSH (22/tcp): OpenSSH 4.7p1 Debian
- HTTP (80/tcp): Apache 2.2.8
- SMB (139/445): Samba 3.X - 4.X

---

### 🧾 3.2. Enumeração de Serviços

**Comando utilizado:**
```bash
enum4linux -a 192.168.80.129 | tee enum4_output.txt
```

**Explicação do comando:**
- `-a`: Executa uma enumeração completa (usuários, grupos, compartilhamentos, etc.).
- `tee enum4_output.txt`: Salva a saída no arquivo `enum4_output.txt` enquanto exibe na tela.

---

## 💣 4. Fase de Exploração: Ataques de Força Bruta

**Listas de palavras utilizadas (wordlists):**
- `users.txt`: user, msfadmin, admin, root
- `pass.txt`: 123456, password, qwerty, msfadmin
- `smb_users.txt`: user, msfadmin, service
- `senhas_spray.txt`: password, 123456, Welcome123, msfadmin

### 🔐 4.1. Ataque ao Serviço FTP

**Comando utilizado:**
```bash
medusa -h 192.168.80.129 -U users.txt -P pass.txt -M ftp -t 6
```

**Explicação:**
- `-h`: Define o host alvo.
- `-U users.txt`: Lista de usuários a testar.
- `-P pass.txt`: Lista de senhas a testar.
- `-M ftp`: Define o módulo de ataque (FTP).
- `-t 6`: Define 6 threads paralelas (velocidade).

**Credencial descoberta:** msfadmin : msfadmin

---

### 🧱 4.2. Ataque ao Serviço SMB (Samba)

**Comando utilizado:**
```bash
medusa -h 192.168.80.129 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
```

**Explicação:**
- `-M smbnt`: Usa o módulo SMB (NTLM).
- `-T 50`: Define o timeout máximo (50s).
- `-t 2`: Usa duas threads simultâneas.

**Credencial descoberta:** msfadmin : msfadmin (acesso permitido ao recurso ADMIN$)

---

### 🌐 4.3. Ataque ao Serviço HTTP (DVWA Login Page)

**Comando utilizado:**
```bash
medusa -h 192.168.80.129 -U users.txt -P pass.txt -M http -m PAGE:'/dvwa/login.php' -m FORM:'username=^USER^&password=^PASS^&Login=Login' -m 'FAIL=Login failed' -t 6
```

**Explicação detalhada:**
- `-M http`: Define o módulo HTTP.
- `-m PAGE:'/dvwa/login.php'`: Especifica a página de login.
- `-m FORM:'username=^USER^&password=^PASS^&Login=Login'`: Define o formato dos campos de login.
- `-m 'FAIL=Login failed'`: Identifica a resposta de falha de autenticação.
- `-t 6`: Usa 6 threads simultâneas.

**Credenciais encontradas:**
- msfadmin: 123456, password
- admin: 123456, password
- user: msfadmin, qwerty, 123456, password
- root: 123456, password, qwerty

---

## 🧠 5. Conclusão

**Resultado:** O teste obteve sucesso em comprometer múltiplos serviços do alvo.  
**Problema:** Uso de credenciais fracas e padrões.  

**Recomendações:**
- Alterar imediatamente as senhas vulneráveis.
- Implementar política de senhas fortes.
- Limitar tentativas de login por IP.
- Atualizar e proteger serviços expostos (FTP, HTTP, SMB).

---
