# Projeto: Infraestrutura Web na AWS com Monitoramento Automatizado

## Objetivo do Projeto

O objetivo deste projeto é desenvolver habilidades em Linux, AWS e automação através da implantação de uma infraestrutura web básica, segura e funcional na AWS, garantindo a disponibilidade de um site 24 horas por dia com monitoramento e alertas automáticos.

---

## Guia de Implementação

### Etapa 1: Configuração do Ambiente AWS

Nesta etapa, criamos a base de rede e o servidor na nuvem da AWS. Os passos foram realizados através do Console da AWS.

**1.1. Criação da VPC (Virtual Private Cloud)**

A VPC é a rede privada e isolada para nossos recursos.

1.  Acesse o serviço **VPC** no Console da AWS. <img width="1917" height="910" alt="image" src="https://github.com/user-attachments/assets/98eb2354-3299-45bf-8978-7f20444d70aa" />

2.  Utilize o assistente **"VPC e mais"** para criar a estrutura de forma simplificada. <img width="868" height="863" alt="image" src="https://github.com/user-attachments/assets/e8b23e26-70a5-41bb-8fee-fd1fc7295167" />

3.  Configure o assistente para criar: <img width="1916" height="862" alt="image" src="https://github.com/user-attachments/assets/32e58411-1d97-49cd-8b77-7cbc3b2e287c" />

    * 1 VPC.
    * 2 Sub-redes Públicas
    * 2 Sub-redes Privadas
    * 1 Internet Gateway para dar acesso à internet para as sub-redes públicas.

**1.2. Criação da Instância EC2 (Servidor Virtual)** <img width="1886" height="853" alt="image" src="https://github.com/user-attachments/assets/5492041d-e02a-48d5-ad62-d5538bf43acf" />


1.  Acesse o serviço **EC2** no Console da AWS e clique em **"Executar instância"**.
2.  **AMI e Sistema Operacional:** Escolha uma imagem como **Ubuntu** ou **Amazon Linux 2023**. <img width="1914" height="872" alt="image" src="https://github.com/user-attachments/assets/442e042f-26bb-485a-8c7a-f3bcbd567ab5" />

3.  **Par de Chaves:** Crie um novo par de chaves (`.pem`) e salve o arquivo em um local seguro no seu computador. Ele é essencial para o acesso SSH. <img width="1916" height="868" alt="image" src="https://github.com/user-attachments/assets/2996e504-c91e-4c5b-a64d-a1bbeef1ccdf" />

4.  **Configurações de Rede:**
    * Selecione a **VPC** criada no passo anterior. <img width="1914" height="830" alt="image" src="https://github.com/user-attachments/assets/3fc4f044-0eb5-4965-845d-76cc3a17b2e0" />

    * Escolha uma das **sub-redes públicas** para a instância.
    * Habilite a opção **"Atribuir IP público automaticamente"**.

**1.3. Configuração do Security Group (Firewall)**

Durante a criação da instância, crie um novo Security Group com as seguintes regras de entrada (Inbound Rules): <img width="1910" height="912" alt="image" src="https://github.com/user-attachments/assets/292ff58c-399a-4565-820a-9d4328d02764" />

* **Tipo SSH:** Porta `22`. Na origem, selecione `Meu IP` para segurança.
* **Tipo HTTP:** Porta `80`. Na origem, selecione `Qualquer lugar (0.0.0.0/0)` para permitir o acesso público ao site[cite: 40].

**1.4. Acesso via SSH**

Após a instância ser criada, a conexão é feita via terminal. <img width="711" height="500" alt="image" src="https://github.com/user-attachments/assets/06f7d137-97c4-459b-aadf-ffd17bbbae15" />


1.  Ajuste as permissões do arquivo da chave (execute este comando no seu terminal local, na pasta onde salvou a chave, se você estiver usando WSL):
    ```bash 
    chmod 400 sua-chave.pem
    ```
2.  Conecte-se à instância usando o comando SSH, substituindo os valores necessários:
    ```bash
    ssh -i sua-chave.pem ubuntu@seu_ip_publico
    ```

### Etapa 2: Configuração do Servidor Web

Com o acesso à instância estabelecido, o próximo passo é instalar e configurar o servidor web.

**2.1. Instalação do Servidor Nginx**

1.  Atualize a lista de pacotes do sistema:
    ```bash
    sudo apt update
    ```
2.  Instale o Nginx:
    ```bash
    sudo apt install nginx -y
    ```
3.  Verifique se o Nginx está em execução:
    ```bash
    sudo systemctl status nginx
    ```
    O status deve aparecer como `active (running)`. <img width="1076" height="346" alt="image" src="https://github.com/user-attachments/assets/2a21264e-dcab-4849-a0a4-34de584f8143" />


**2.2. Criação da Página Web**

1.  Navegue até o diretório raiz do Nginx:
    ```bash
    cd /var/www/html
    ```
2.  Crie o arquivo `index.html` com o editor de texto `nano`[cite: 19, 45]:
    ```bash
    sudo nano index.html
    ```
3.  Cole o conteúdo HTML da sua página no editor. Exemplo: <img width="1104" height="556" alt="image" src="https://github.com/user-attachments/assets/500a455e-a684-44f6-b73b-6658b0ffcc2d" />

    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>Projeto AWS</title>
    </head>
    <body>
        <h1>Meu Site na AWS!</h1>
        <p>Esta página está sendo servida por um Nginx em uma instância EC2.</p>
    </body>
    </html>
    ```
4.  Salve e saia do `nano` (pressione `Ctrl + X`, depois `Y` e `Enter`).

**2.3. Verificação**

Abra um navegador e acesse o endereço IP público da sua instância. Sua página personalizada deve ser exibida. <img width="1921" height="938" alt="image" src="https://github.com/user-attachments/assets/8d0cea0d-11d8-47e4-9dd1-9263994f1e99" />


### Etapa 3: Script de Monitoramento e Alertas

Nesta etapa, criamos um sistema de monitoramento automatizado. Siga os comandos abaixo no terminal da sua instância EC2.

**3.1. Criar o Arquivo de Log**

Primeiro, criamos o arquivo de log e damos a permissão correta para que o nosso usuário possa escrever nele.

```bash
# Crie o arquivo de log vazio
sudo touch /var/log/monitoramento.log

# Dê ao seu usuário a permissão para escrever no arquivo
# Substitua 'ubuntu' pelo seu nome de usuário, se for diferente
sudo chown ubuntu /var/log/monitoramento.log
```

**3.2. Criar o Script de Monitoramento**

Agora, crie o arquivo do script usando o editor `nano`.

```bash
nano monitor.sh
```

Copie e cole o código abaixo dentro do editor `nano`. Este script verifica o site, registra o status no log e envia um alerta para o Discord em caso de falha.

<img width="1098" height="550" alt="image" src="https://github.com/user-attachments/assets/ac1fa9f7-8fc5-4614-9c1b-56d32196026a" />



```bash
#!/bin/bash

# --- Configurações ---
URL="http://localhost"
LOG_FILE="/var/log/monitoramento.log"
WEBHOOK_URL="COLE SEU WEBHOOK AQUI>
# --- Início do Script ---
HTTP_STATUS=$(curl -o /dev/null -s -w "%{http_code}" $URL)
TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

if [ $HTTP_STATUS -eq 200 ]; then
    # Site OK: registrar e notificar
    MESSAGE="✅ SUCESSO: Site está no ar. Status: $HTTP_STATUS"
    echo "[$TIMESTAMP] $MESSAGE" >> $LOG_FILE

    # Envia notificação de SUCESSO para o Discord
    curl -H "Content-Type: application/json" -d "{\"content\": \"$MESSAGE\"}" "$WEBHOOK_URL"
else
    # Site com Falha: registrar e notificar
    MESSAGE="🚨 FALHA: Site está fora do ar ou com erro. Status: $HTTP_STATUS"
    echo "[$TIMESTAMP] $MESSAGE" >> $LOG_FILE

    # Envia notificação de FALHA para o Discord
    curl -H "Content-Type: application/json" -d "{\"content\": \"$MESSAGE\"}" "$WEBHOOK_URL"
```
Para salvar e sair do `nano`, pressione `Ctrl + X`, depois `Y` e `Enter`.

**3.3. Tornar o Script Executável**

Por padrão, o arquivo é criado sem permissão de execução. Use o comando `chmod` para torná-lo um programa que pode ser executado.

```bash
chmod +x monitor.sh
```

**3.4. Configurar a Automação com Cron**

Finalmente, vamos agendar a execução automática do script a cada minuto usando o `cron`.

```bash
# Abra o editor de agendamento do cron
crontab -e
```
Adicione a seguinte linha no final do arquivo. Ela instrui o sistema a executar seu script a cada minuto.

```
* * * * * /home/ubuntu/monitor.sh
```
<img width="1105" height="572" alt="image" src="https://github.com/user-attachments/assets/9ad10c84-23cd-4fd3-8b1f-ef5b21b34a2d" />


Salve e saia do editor. O monitoramento agora está ativo e automatizado.

### Etapa 4: Testes da Solução

Para validar a implementação, foram realizados os seguintes testes:
* Acessar o IP público da instância para verificar se o site está online.
* [Parar o serviço do Nginx (`sudo systemctl stop nginx`) para simular uma falha. <img width="741" height="222" alt="image" src="https://github.com/user-attachments/assets/cdc4697a-4753-4bc0-a2dd-7f7cc3e50471" />

* Confirmar o recebimento do alerta de falha no Discord e o registro do erro no arquivo de log. <img width="604" height="108" alt="image" src="https://github.com/user-attachments/assets/e6bbf983-97e5-48b6-97bb-015c55e2664a" />

* Reiniciar o serviço do Nginx (`sudo systemctl start nginx`) e verificar o log retornando ao status de sucesso.
 <img width="760" height="231" alt="image" src="https://github.com/user-attachments/assets/238ea966-5a43-4298-b1e8-1ac901a05e10" />


---

### **Aviso de Segurança**

**Cuidado com dados que podem comprometer a segurança**. A URL do Webhook do Discord é uma informação sensível e não deve ser exposta em repositórios públicos.
