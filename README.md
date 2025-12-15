# 🔍 Relatório de Auditoria de Segurança
Este repositório documenta o resultado final do Desafio de Projeto da DIO. Realizado em ambiente isolado (Kali Linux vs. Metasploitable 2/DVWA), demonstra a validação de credenciais via Força Bruta (Medusa) e entrega um Relatório Técnico com Plano de Mitigação priorizado para vulnerabilidades críticas em serviços expostos (FTP, SMB, Web).

<p align="center">
  <a href="https://web.dio.me/track/santander-ciberseguranca-2025" target="_blank">
  <img
    src="https://img.shields.io/static/v1?label=DIO&message=Education&color=E94D5F&labelColor=202024" alt="DIO Project" />
</p>
    
<br>

## 🧩 Metodologia Utilizada
A documentação segue o ciclo de vida completo da exploração, garantindo que cada falha identificada seja rastreável à sua solução:
1.  **Reconhecimento (Nmap/Ping):** Mapeamento inicial para identificar o alvo e os serviços expostos.
2.  **Enumeração (enum4linux):** Mapeamento de usuários e recursos (SMB) para planejamento de ataque.
3.  **Ataque de Força Bruta (Medusa):** Validação da quebra de senhas para obter acesso.
4.  **Exploração:** Confirmação do acesso utilizando credenciais descobertas (`ftp`, `smbclient`).
5.  **Relatório de Risco:** Documentação detalhada de **Evidências** e **Impacto**.
6.  **Mitigação (Hardening):** Recomendações Priorizadas para correção da causa raiz das vulnerabilidades.

> As ferramentas e técnicas demonstradas são utilizadas exclusivamente como meio de **validação e educação** para guiar estratégias robustas de defesa.

<br>

## 🚪Ambiente Utilizado:
* Virtualização: VirtualBox;
* Sistema Atacante (auditoria): Kali Linux;
* Sistema Alvo: Metasploitable 2 (incluindo DVWA);
* Modo de Rede: Host-Only
  * Garante isolamento total do ambiente, impedindo tráfego para a internet real;
- **Ferramentas Para a Exploração:**
  * Nmap (enumeração de serviços);
  * Medusa (validação de falhas de autenticação);
  * SMBClient / Enum4linux (enumeração SMB).

<br>

## 1.0. 📑 Verificações Iniciais: Conectividade e Alvo
Todo teste de penetração começa com a confirmação da acessibilidade do host alvo na rede.

### 1.1. Obtenção e Identificação do Endereço IP
Para identificar o endereço IP da máquina alvo (**Metasploitable 2**), o seguinte comando foi executado no console do alvo (após inserir o login: `msfadmin` e a senha: `msfadmin`):
```bash
ip a
  ```
**Ação Executada:** Foi **identificado** o endereço IPv4 (`inet`) na interface de rede que se comunica com o ambiente de testes (Kali Linux).

### 1.2. Teste de Conectividade (Ping):
Em seguida, a comunicação ICMP foi validada a partir do Kali Linux, confirmando que o alvo estava ativo:
  ``` bash
  ping -c 3 192.168.56.102
   ```
| Parâmetro | Descrição | Função no Reconhecimento |
| :---: | :--- | :--- |
| `ping` | Programa que envia pacotes **ICMP Echo Request** (Solicitação de Eco). | Utilizado para testar se o host alvo está ativo e respondendo na rede. |
| `-c 3` | **Count** (Contagem) | Limita o envio a 3 pacotes ICMP, garantindo um teste rápido. |
| `192.168.56.102` | Endereço IP do host de destino. | O alvo primário do teste de penetração (Metasploitable 2). |

Abaixo, a evidência da execução do teste de ping bem-sucedido:
<img width="1365" height="642" alt="image" src="https://github.com/user-attachments/assets/b9e96ede-3993-4784-bee2-6b838aa21def" />

<br>

## 2.0. 🔎 Reconhecimento de Serviços (Nmap)
Após confirmar a conectividade com o alvo, o mapeamento de portas foi realizado para identificar serviços ativos e suas respectivas versões.

### 2.1. Varredura de Portas Específicas e Versões
Foi executada uma varredura focalizada (`-sV`) nas portas de serviços mais comuns e potencialmente vulneráveis (FTP, SSH, HTTP e SMB) no alvo **192.168.56.102**:
```bash
nmap -sV -p 21,22,80,445,139 192.168.56.102
   ```
| Parâmetro | Descrição | Função no Reconhecimento |
| :---: | :--- | :--- |
| `nmap` | Ferramenta de Mapeamento de Rede. | Realiza a varredura de portas e a detecção de serviços. |
| `-sV` | Service Version Detection | Tenta determinar a versão exata do software rodando (crucial para buscar exploits específicos). |
| `-p` | Especificação de portas | Define que a varredura deve ser realizada apenas nas portas listadas. |
| `21,22,80,445,139` | Lista de Portas | Portas de serviços-alvo comuns (FTP, SSH, HTTP e SMB). |
| `192.168.56.102` | Endereço IP do host de destino. | O alvo específico da varredura (Metasploitable 2). |

### 2.2. O que foi Descoberto? (Resultados Chave do Nmap)
Os resultados da varredura (`nmap -sV`) identificaram os seguintes serviços críticos e suas versões:
* **Porta 21 (FTP):** Serviço `vsftpd 2.3.4` ativo. **(Vulnerabilidade Conhecida)**
* **Porta 22 (SSH):** Serviço `OpenSSH 4.7p1 Debian 8` ativo.
* **Porta 80 (HTTP):** Serviço `Apache httpd 2.2.8` ativo.
* **Porta 139/445 (SMB):** Serviço `Samba smbd 3.0.20-Debian` ativo.

> **Conclusão:** A versão do **vsftpd 2.3.4** é notória por uma falha de *backdoor* (CVE-2011-2523), além de ser suscetível a credenciais padrão/fracas.

<br>

## 3.0. 🔑 Quebra de Senha via Força Bruta (Medusa)
Após identificar o serviço FTP na Porta 21, um ataque de dicionário foi executado para obter credenciais válidas.

### 3.1. Preparação dos Dicionários
Os arquivos de lista de palavras foram preparados, contendo usuários (`users.txt`) e senhas (`pass.txt`) simples e comuns:
* **Criação de `users.txt`:**
    ```bash
    echo -e "user\nmsfadmin\nadmin\nroot" > users.txt
    ```
* **Criação de `pass.txt`:**
    ```bash
    echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
    ```
**Arquivos Gerados:**
<img width="903" height="257" alt="image" src="https://github.com/user-attachments/assets/6c1e3aec-2068-4f04-b5e2-29faf7925d19" />

### 3.2. Execução do Ataque de Força Bruta
A ferramenta Medusa foi utilizada para realizar um ataque de dicionário no serviço FTP:
```bash
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 6
```
| Parâmetro | Descrição | Função no Ataque |
| :---: | :---: | :--- |
| **`medusa`** | Chamada da ferramenta de teste de força bruta paralela. | Automatiza múltiplas tentativas de autenticação. |
| **`-h`** | Host Alvo. | Especifica o endereço IP de destino: **`192.168.56.102`**. |
| **`-U`** | Arquivo de Usuários. | Carrega o dicionário de usuários (users.txt) criado. |
| **`-P`** | Arquivo de Senhas. | Carrega o dicionário de senhas (pass.txt) criado. |
| **`-M`** | Módulo de Serviço. | Define qual protocolo será atacado: ftp. |
| **`-t 6`** | Threads. | Define o número de conexões paralelas (6) para acelerar o processo. |

### 3.3. O que foi Descoberto? (Evidência da Credencial)
* O serviço FTP permitiu a execução do ataque de força bruta sem limitação de tentativas.
* Foi identificada a credencial válida: **`msfadmin:msfadmin`**.
* **Conclusão:** O serviço não possuía mecanismos de bloqueio de tentativas de autenticação, expondo contas com senhas fracas.

**Evidência:** O Medusa confirmou a credencial encontrada:
<img width="1179" height="555" alt="image" src="https://github.com/user-attachments/assets/5a336aa4-5ce3-4112-b86f-c1d37de474d4" />

### 3.4. 🛠️ Recomendações de Mitigação
As seguintes ações são mandatórias para mitigar a vulnerabilidade de exposição a ataques de Força Bruta:
1.  **Implementar Limitação de Tentativas:** Configurar o serviço FTP (ou um firewall intermediário) com um mecanismo de bloqueio ou atraso após um número mínimo de tentativas falhas de login.
2.  **Eliminar Credenciais Padrão/Fracas:** Alterar ou remover imediatamente as credenciais padrão (`msfadmin:msfadmin`) e implementar uma política rigorosa de senhas fortes e complexas.
3.  **Desabilitar Contas Genéricas:** Desabilitar ou remover contas genéricas (`user`, `admin`) que foram expostas ao serviço FTP.
4.  **Monitoramento de Autenticação:** Habilitar logs detalhados e monitoramento em tempo real para detectar padrões de múltiplas tentativas de login (indicando ataques de dicionário).

<br>

## 4.0. 💥 Exploração do Serviço FTP
A exploração da falha de autenticação foi realizada utilizando o par de credenciais (`msfadmin:msfadmin`) obtido na Seção 3.0. (Quebra de Senha). A conexão foi estabelecida através do cliente FTP.

### 4.1. Tentativa de Conexão e Autenticação
O comando a seguir foi executado para iniciar a conexão com o alvo:
```bash
ftp 192.168.56.102
```
| Parâmetro | Descrição | Importância Crítica na Exploração |
| :---: | :--- | :--- |
| `ftp` | Inicia o cliente **File Transfer Protocol**. | Abre o canal de comunicação para a tentativa de autenticação com a credencial descoberta. |
| `192.168.56.102` | Endereço IP do host de destino. | Alvo de rede comprometido (Metasploitable 2). |

### 4.2. Evidência do Acesso e Vulnerabilidade
**O que foi descoberto:** O serviço FTP estava acessível e permitiu autenticação bem-sucedida com a credencial fraca/padrão.
* **Credencial Fraca:** Autenticação bem-sucedida utilizando `User: msfadmin` e `Password: msfadmin`.
* **Vulnerabilidade Crítica:** Foi confirmada a execução do serviço `vsftpd 2.3.4`, notório por falha de *backdoor* (CVE-2011-2523), expondo o alvo a exploração remota.
* **Confirmação:** O *output* confirmou o sucesso do login (`230 Login successful.`).

<img width="1029" height="553" alt="image" src="https://github.com/user-attachments/assets/a6544b03-ac68-4690-962d-9654814ada3a" />

> A autenticação subsequente (usuário `msfadmin` e senha `msfadmin`) resultou na mensagem de sucesso `230 Login successful.`

### 4.3. 🛠️ Recomendações de Mitigação
As seguintes ações são mandatórias para mitigar a vulnerabilidade identificada no serviço FTP:
1.  **Remover Credenciais Padrão/Fracas:** Alterar imediatamente as credenciais padrão. Exigir senhas fortes, complexas e exclusivas para quaisquer contas com acesso ao serviço.
2.  **Manter o Serviço Atualizado:** O serviço **vsftpd 2.3.4** deve ser atualizado para a versão mais recente e estável para eliminar a vulnerabilidade de *backdoor* (CVE-2011-2523) e outras falhas conhecidas.
3.  **Restringir o Serviço FTP:** Desabilitar o serviço caso não seja estritamente necessário, ou substituí-lo por protocolos seguros (SFTP/FTPS) que criptografam a comunicação e as credenciais.
4.  **Aplicar Controle de Acesso por Rede:** Implementar regras de firewall (`iptables` ou equivalente) para permitir conexões FTP apenas de endereços IP ou faixas de rede **explicitamente autorizadas** (Princípio do Mínimo Privilégio).
5.  **Monitoramento de Autenticação:** Configurar logs detalhados do serviço e monitoramento em tempo real para detectar e alertar sobre múltiplas tentativas de autenticação (força bruta) ou acessos indevidos.

<br>

## 5.0. 🌐 Exploração do Formulário Web (Ataque de Força Bruta)
O objetivo desta seção é testar a robustez do formulário de *login* da aplicação web (DVWA) contra tentativas automatizadas de adivinhação de credenciais (ataque de dicionário). A página alvo é acessível em `192.168.56.102/dvwa/login.php`.

### 5.1. Preparação e Execução do Ataque (Medusa HTTP)
O ataque de força bruta foi realizado injetando as listas de palavras (usuários e senhas) em uma requisição HTTP capturada do formulário de *login*. Foi utilizado o mesmo `users.txt` e `pass.txt` da Seção 3.0.
O comando de ataque foi o seguinte:
```bash
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \
-m PAGE: '/dvwa/login.php' \
-m FORM: 'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6
```
| Parâmetro | Descrição | Função no Ataque |
| :---: | :---: | :--- |
| **`medusa`** | Chamada da ferramenta de teste de força bruta paralela. | Automatiza múltiplas tentativas de autenticação. |
| **`-h`** | Host Alvo. | Especifica o endereço IP de destino: **`192.168.56.102`**. |
| **`-U`** | Arquivo de Usuários. | Carrega o dicionário de usuários (users.txt) para injeção. |
| **`-P`** | Arquivo de Senhas. | Carrega o dicionário de senhas (pass.txt) para injeção. |
| **`-M http`** | Módulo de Serviço. | Define qual protocolo será atacado: HTTP. |
| **`-m PAGE: ...`** | URL do Formulário. | Define o caminho da página que processa o login. |
| **`-m FORM: ...`** | Estrutura da Requisição POST. | Simula o envio de dados do formulário com placeholders (^USER^, ^PASS^). |
| **`-m 'FAIL=...'`** | Condição de Falha (Chave). | Define a string (Login failed) que, se presente na resposta, indica uma tentativa fracassada. |
| **`-t 6`** | Threads. | Define o número de conexões paralelas (6) para acelerar o processo. |

### 5.2. Evidência do Acesso e Vulnerabilidade
* O formulário de *login* **não possui mecanismos de defesa** contra ataques de força bruta, como *rate limiting* ou bloqueio de IP após tentativas falhas.
* O ataque validou as credenciais via **resposta HTTP**, buscando a ausência da *string* de falha (`Login failed`).
* **Credenciais Válidas Encontradas:** Foram descobertos múltiplos pares de credenciais válidas contidas nas listas de palavras, demonstrando o uso de senhas fracas.

**Evidência:**
```
2025-11-20 10:51:16 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: password [SUCCESS]
2025-11-20 10:51:16 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: msfadmin Password: qwerty [SUCCESS]
```
<img width="1251" height="607" alt="image" src="https://github.com/user-attachments/assets/2068f786-ab68-4490-82f5-79932d1fc090" />

> A mensagem `[SUCCESS]` valida as credenciais encontradas, comprovando que a aplicação aceitou os pares `user:password` e `msfadmin:qwerty`.

### 5.3. 🛠️ Recomendações de Mitigação
As seguintes ações são mandatórias para mitigar a vulnerabilidade de exposição a ataques de Força Bruta em formulários Web:
1.  **Limitação de Taxa de Requisição (*Rate Limiting*):** Implementar mecanismos para limitar o número de tentativas de login por endereço IP ou conta em um curto período de tempo.
2.  **Bloqueio de IP:** Bloquear temporariamente ou permanentemente endereços IP que excedam o limite de tentativas.
3.  **Implementar CAPTCHA:** Exigir a resolução de um CAPTCHA após um número de falhas de login.
4.  **Autenticação Multifator (MFA):** Implementar MFA como camada de defesa adicional, garantindo que mesmo que a senha seja quebrada, o acesso não será concedido sem um segundo fator.
5.  **Política de Senhas:** Reforçar a política de senhas para evitar o uso de credenciais fracas ou de dicionário.

<br>

## 6.0. 🗄️ Exploração do Serviço SMB/Samba
A exploração do serviço SMB/Samba (Portas 139 e 445) foi dividida em três fases: Enumeração de Reconhecimento, Ataque de Força Bruta para obter credenciais e, por fim, a Exploração do Compartilhamento.

### 6.1. Enumeração e Mapeamento de Recursos (enum4linux)
O comando abaixo executa uma varredura completa, buscando informações sobre usuários, grupos e compartilhamentos disponíveis, expondo a baixa segurança do serviço:
```bash
enum4linux -a 192.168.56.102 | tee enum4_output.txt
```
| Parâmetro | Descrição | Função no Ataque |
| :---: | :---: | :--- |
| **`enum4linux`** | Ferramenta de enumeração para Samba/SMB. | Executa diversas técnicas de reconhecimento, buscando usuários, grupos e compartilhamentos. |
| **`-a`** | Opção All (Todos). | Executa todas as opções de enumeração disponíveis (usuários, grupos, compartilhamentos). |
| **`192.168.56.102`** | Host Alvo. | O endereço IP do servidor que hospeda o serviço SMB/Samba. |
| **`tee`** | Comando tee. | Permite a saída simultânea dos dados na tela e a escrita em um arquivo. |
| **`enum4_output.txt`** | Nome do Arquivo. | O arquivo de texto onde o output completo da enumeração é salvo para fins de registro. |

#### 6.1.1. O que foi Descoberto na Enumeração?
* **Vulnerabilidade Crítica:** Esta versão do Samba (`3.0.20`) é historicamente conhecida por ser altamente vulnerável a diversas explorações, incluindo o famoso exploit `Username map script` (CVE-2007-2447).
* **Enumeração Completa:** Foi obtido acesso a listas de usuários, grupos, e nomes de domínio.
* **Compartilhamentos Padrão:** Foram identificados compartilhamentos administrativos padrão (`IPC$`, `ADMIN$`, `print$`).

**Evidência da Enumeração:**
<img width="1124" height="533" alt="image" src="https://github.com/user-attachments/assets/bac1ce94-299c-47e9-b607-b02916433e0e" />

### 6.2. Quebra de Senha (Força Bruta contra SMB)
O ataque de dicionário foi executado para obter credenciais válidas.
Este comando realiza um ataque de força bruta contra o serviço SMB:
```bash
medusa -h 192.168.56.102 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50 
```
| Parâmetro | Descrição | Função no Ataque |
| :---: | :---: | :--- |
| **`-h 192.168.56.102`** | Especifica o Host Alvo (Target Host) pelo seu endereço IP (192.168.56.102). | Foco: Onde o ataque será direcionado. |
| **`-U users.txt`** | Define o caminho para o arquivo que contém a lista de Nomes de Usuários a serem testados. | Foco: Iterar sobre a dimensão de usuários |
| **`-P senhas_spray.txt`** | Define o caminho para o arquivo contendo a lista de Senhas a serem testadas. | Foco: Iterar sobre a dimensão de senhas. |
| **`-M smbnt`** | Especifica o Módulo de Ataque: smbnt para testar o serviço SMB. | Foco: Direcionar o ataque ao serviço correto. |
| **`-t 2`** | Define o número de Threads por Host a serem usadas. | Otimização: Equilibrar velocidade e estabilidade. |
| **`-T 50`** | Define o número de Hosts Paralelos a serem atacados. | Otimização: Controle de recursos e escopo. |

**Evidência:**
<img width="1126" height="527" alt="image" src="https://github.com/user-attachments/assets/5f71c8fb-552c-4790-bc2f-61be9e9aac2c" />

### 6.3. Exploração e Acesso a Compartilhamentos
Utilizando a credencial (`msfadmin:msfadmin`) descoberta na fase anterior, foi obtido acesso aos recursos do servidor SMB:
O comando `smbclient` foi usado para listar os recursos disponíveis:
```bash
smbclient -L //192.168.56.102 -U msfadmin
```
| Parâmetro | Descrição | Função no Ataque |
| :---: | :---: | :--- |
| **`smbclient`** | Cliente de linha de comando para interagir com o protocolo SMB/CIFS. | Ferramenta essencial para auditoria de compartilhamentos. |
| **`-L //192.168.56.102`** | Listar (-L) os serviços disponíveis (compartilhamentos, impressoras) no host alvo (192.168.56.102). | Foco: Descobrir todos os recursos expostos pelo servidor alvo. |
| **`-U msfadmin`** | Especifica o Nome de Usuário (msfadmin) para autenticação. | Foco: Utilizar credenciais válidas para obter acesso. |

**Evidência do Acesso:**
<img width="1126" height="520" alt="image" src="https://github.com/user-attachments/assets/4f438458-3961-4625-8d09-a1d770f1eb81" />
**Resultado Final:** Acesso bem `sucedido`aos compartilhamentos ✔

### 6.4. 🛠️ Recomendações de Mitigação
As seguintes ações são críticas e devem ser implementadas para mitigar a vulnerabilidade do Samba, que permitiu enumeração e acesso não autorizado:
1.  **Atualização de Software (Prioridade Crítica):**
    * Atualizar imediatamente o serviço **Samba 3.0.20** para uma versão estável e mais recente.
    * Esta versão é conhecida por ser vulnerável ao exploit `Username map script` (CVE-2007-2447), que permite a **Execução Remota de Comandos (RCE)**. A atualização elimina esta falha grave.

2.  **Restrição de Acesso por Rede:**
    * Implementar regras de firewall (`iptables` ou equivalente) para restringir o acesso às portas **139 (NetBIOS/SMB)** e **445 (SMB Direto)**.
    * O acesso deve ser permitido **apenas** a endereços IP ou faixas de rede internas e explicitamente autorizadas.

3.  **Remover Credenciais Padrão/Fracas:**
    * Alterar ou remover imediatamente credenciais padrão como `msfadmin:msfadmin`, que foram expostas ao ataque de Força Bruta (Seção 6.2.).
    * Implementar uma política rigorosa de senhas complexas.

4.  **Desativar o Serviço SMBv1:**
    * Garantir que o protocolo SMBv1, mais antigo e menos seguro, esteja desativado. Utilizar apenas versões mais recentes, como SMBv2 ou SMBv3.

5.  **Monitoramento de Eventos:**
    * Configurar logs detalhados do serviço Samba e monitoramento de segurança para detectar e alertar sobre atividades suspeitas, como enumeração de recursos e múltiplas tentativas de autenticação falhas.

<br>

# 7.0. 📝 Conclusão e Próximos Passos
O presente relatório detalhou a **Auditoria de Penetração** realizada no ambiente controlado (Metasploitable 2), comprovando a existência de falhas críticas de segurança que, em um ambiente de produção, resultariam em **comprometimento completo do sistema**.

### Impacto
Os testes confirmaram que a ausência de controles básicos permitiu:
1.  **Acesso Imediato:** Credenciais padrão e senhas fracas foram quebradas em serviços como **FTP**, **Web (DVWA)** e **SMB/Samba**, garantindo acesso não autorizado.
2.  **Risco de Execução Remota de Comandos (RCE):** A versão desatualizada do Samba (`3.0.20`) expõe o alvo à falha crítica **CVE-2007-2447**, o que é um risco inaceitável.
3.  **Falta de Defesa:** Nenhum dos serviços de autenticação explorados possuía mecanismos de defesa contra ataques de Força Bruta (*rate limiting* ou bloqueio de tentativas).

### ➡️ Próximos Passos (Plano de Mitigação)
O foco agora deve ser a **implementação imediata e priorizada** das recomendações detalhadas em cada seção do relatório. Sugere-se o seguinte plano de ação para o *hardening* do ambiente:

| Prioridade | Ação Mandatória | Serviços Afetados |
| :---: | :--- | :--- |
| **Crítica** | **Atualização de Software:** Corrigir o Samba 3.0.20 e o vsftpd 2.3.4. | SMB, FTP |
| **Alta** | **Controles de Autenticação:** Implementar *Rate Limiting* e CAPTCHA. | FTP, Web, SMB |
| **Alta** | **Alteração de Credenciais:** Remover contas padrão (`msfadmin`) e forçar senhas complexas. | FTP, Web, SMB |
| **Média** | **Restrição de Rede:** Bloquear portas 139/445 e 21 no *firewall* para tráfego externo. | SMB, FTP |

A implementação consistente dessas correções não apenas elimina as vulnerabilidades exploradas, mas também estabelece uma base de segurança robusta para proteger o ambiente contra ataques futuros e elevar o nível de maturidade de segurança da organização.
  
![](https://i.imgur.com/WTLoFrq.png)

## Aviso de Uso Ético
Este projeto é exclusivamente educacional e foi desenvolvido para testes em **ambientes isolados**. A execução de ataques de força bruta em sistemas reais, sem permissão explícita, é ilegal.

Leia a política completa em: 👉 [**SECURITY.md**](./SECURITY.md)

<h2> 🔗 Compartilhe com a comunidade 🧡 </h2>

Por favor, se esse conteúdo te ajudou, não esqueça de compartilhar 😁

[![GitHub Repo stars](https://img.shields.io/badge/share%20on-twitter-03A9F4?logo=twitter)](https://twitter.com/share?url=https://github.com/Luhrodrigues45/Auditoria-de-forca-bruta) [![GitHub Repo stars](https://img.shields.io/badge/share%20on-facebook-1976D2?logo=facebook)](https://www.facebook.com/sharer/sharer.php?u=https://github.com/Luhrodrigues45/Auditoria-de-forca-bruta) [![GitHub Repo stars](https://img.shields.io/badge/share%20on-linkedin-3949AB?logo=linkedin)](https://www.linkedin.com/shareArticle?url=https://github.com/Luhrodrigues45/Auditoria-de-forca-bruta)
