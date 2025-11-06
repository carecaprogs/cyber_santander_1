# Desafio DIO: Simulação de Ataque de Força Bruta com Kali e Medusa

Este repositório documenta a execução de um desafio prático da plataforma DIO, focado em testes de segurança (Pentest) utilizando Kali Linux e a ferramenta Medusa contra ambientes vulneráveis controlados (Metasploitable 2 / DVWA).

O objetivo é simular cenários de ataque de força bruta em um laboratório isolado para compreender as vulnerabilidades e, o mais importante, propor as medidas de mitigação adequadas.

## 1. Configuração do Ambiente

O laboratório foi configurado em uma rede isolada para garantir que os testes fossem realizados com segurança e sem impacto externo.

* **Máquina Atacante:** Kali Linux 
* **Máquina Vítima:** Metasploitable 2 (Hospedando os serviços vulneráveis, incluindo o DVWA)
* **Endereço IP da Vítima:** `192.168.88.249`

## 2. Execução dos Testes

### 2.1. Fase 1: Reconhecimento (Nmap)

O primeiro passo foi realizar um scan com o Nmap para identificar os serviços e portas abertas na máquina vítima.

**Comando:**
```bash
nmap -sV -p 21,22,80,445,139 192.168.88.249
```

**Resultados do Nmap:**
O scan revelou diversos serviços conhecidos por serem vulneráveis:
* `21/tcp`: **vsftpd 2.3.4** (FTP)
* `22/tcp`: **OpenSSH 4.7p1** (SSH)
* `80/tcp`: **Apache httpd 2.2.8** (HTTP)
* `139/tcp` & `445/tcp`: **Samba smbd 3.X** (SMB)


### 2.2. Cenário 1: Força Bruta em FTP (vsftpd)

**Objetivo:** Obter acesso não autorizado ao serviço FTP (porta 21), que frequentemente possui credenciais fracas.

**Ferramenta:** Medusa

**Execução:**
Foi utilizada uma *wordlist* simples de usuários e senhas (contendo `admin`, `msfadmin`, `root`, `password`, `123456`, etc.) para atacar o serviço FTP.

**Resultados:**
O Medusa obteve sucesso rapidamente, identificando a credencial `msfadmin` / `msfadmin`.

**Validação:**
O acesso foi validado conectando-se diretamente ao serviço FTP com as credenciais encontradas, confirmando o "Login successful".

[Resultado do Medusa e Validação FTP]

**Recomendações de Mitigação (FTP):**
1.  **Desativação:** Se o FTP não for essencial, desative o serviço.
2.  **Protocolos Seguros:** Substituir o FTP por protocolos criptografados como **SFTP** (que utiliza o SSH) ou **FTPS**.
3.  **Senhas Fortes:** Implementar uma política de senhas fortes e complexas.
4.  **Bloqueio de Tentativas:** Utilizar ferramentas como `fail2ban` para banir IPs que falharem a autenticação múltiplas vezes.

---

### 2.3. Cenário 2: Força Bruta em Formulário Web (DVWA)

**Objetivo:** Obter acesso à área restrita do "Damn Vulnerable Web Application" (DVWA) através da página de login (Brute Force).

**Ferramenta:** Medusa (Módulo `web-form`)

**Execução:**
O DVWA foi acessado pelo navegador no IP da vítima (`http://192.168.88.249/dvwa/`). O nível de segurança foi mantido em "low" para o teste.


O Medusa foi configurado para enviar múltiplas tentativas de login ao formulário web, testando a mesma *wordlist* de usuários e senhas.

**Resultados:**
O ataque foi extremamente eficaz, descobrindo múltiplas combinações válidas em segundos, incluindo `admin` / `123456`, `msfadmin` / `msfadmin`, `password` / `password`


**Recomendações de Mitigação (Web Forms):**
1.  **CAPTCHA:** Implementar um reCAPTCHA ou CAPTCHA similar para validar que o usuário é humano.
2.  **Bloqueio de Conta:** Bloquear a conta temporariamente após um número de tentativas falhas (ex: 5 tentativas).
3.  **Rate Limiting:** Limitar o número de tentativas de login que um mesmo endereço IP pode realizar em um curto período.
4.  **Autenticação de Múltiplos Fatores (MFA):** Implementar MFA como uma camada adicional de segurança.


### 2.4. Cenário 3: Password Spraying em SMB (Próximos Passos)

**Objetivo (Sugerido):** Com base no scan do Nmap, o serviço SMB (portas 139/445) é outro vetor de ataque comum.

**Execução (Sugerida):**
Diferente do *brute force* (muitas senhas, um usuário), o *password spraying* (uma senha, muitos usuários) é eficaz contra o SMB. O Medusa pode ser usado para testar uma senha comum (ex: `admin` ou `123456`) contra a lista de usuários.


# Exemplo de comando para Password Spraying
medusa -h 192.168.88.249 -U lista_de_usuarios.txt -p "123456" -M smbnt -t 2 -T 50


**Recomendações de Mitigação (SMB):**
1.  **Políticas de Senha:** Implementar políticas de senha fortes via GPO (no Windows) ou PAM (no Linux).
2.  **Monitoramento:** Monitorar os logs de eventos de segurança para detectar um grande número de falhas de login vindas de um mesmo IP.
3.  **Firewall:** Restringir o acesso às portas SMB apenas para IPs autorizados na rede.

## 3. Conclusões

Este desafio prático demonstrou como serviços comuns (FTP, HTTP, SMB) podem ser facilmente comprometidos se não forem configurados corretamente. Ferramentas automatizadas como o Medusa tornam ataques de força bruta triviais. A implementação de medidas de mitigação básicas, como senhas fortes, bloqueio de tentativas e o uso de CAPTCHA, é essencial para a defesa de qualquer sistema.
