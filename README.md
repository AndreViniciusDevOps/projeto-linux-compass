# Projeto: Infraestrutura Web na AWS com Monitoramento Automatizado

## Objetivo do Projeto

O objetivo deste projeto é desenvolver habilidades em Linux, AWS e automação através da implantação de uma infraestrutura web básica, segura e funcional na AWS, garantindo a disponibilidade de um site 24 horas por dia com monitoramento e alertas automáticos.

---

## Guia de Implementação

### Etapa 1: Configuração do Ambiente AWS

Nesta etapa, criamos a base de rede e o servidor na nuvem da AWS. Os passos foram realizados através do Console da AWS.

**1.1. Criação da VPC (Virtual Private Cloud)**

A VPC é a rede privada e isolada para nossos recursos.

1.  Acesse o serviço **VPC** no Console da AWS.
2.  Utilize o assistente **"VPC e mais"** para criar a estrutura de forma simplificada.
3.  Configure o assistente para criar:
    * 1 VPC.
    * 2 Sub-redes Públicas
    * 2 Sub-redes Privadas
    * 1 Internet Gateway para dar acesso à internet para as sub-redes públicas.

**1.2. Criação da Instância EC2 (Servidor Virtual)**

1.  Acesse o serviço **EC2** no Console da AWS e clique em **"Executar instância"**.
2.  **AMI e Sistema Operacional:** Escolha uma imagem como **Ubuntu** ou **Amazon Linux 2023**.
3.  **Par de Chaves:** Crie um novo par de chaves (`.pem`) e salve o arquivo em um local seguro no seu computador. Ele é essencial para o acesso SSH.
4.  **Configurações de Rede:**
    * Selecione a **VPC** criada no passo anterior[cite: 16].
    * Escolha uma das **sub-redes públicas** para a instância.
    * Habilite a opção **"Atribuir IP público automaticamente"**.

**1.3. Configuração do Security Group (Firewall)**

Durante a criação da instância, crie um novo Security Group com as seguintes regras de entrada (Inbound Rules):
* **Tipo SSH:** Porta `22`. Na origem, selecione `Meu IP` para segurança.
* **Tipo HTTP:** Porta `80`. Na origem, selecione `Qualquer lugar (0.0.0.0/0)` para permitir o acesso público ao site[cite: 40].

**1.4. Acesso via SSH**

Após a instância ser criada, a conexão é feita via terminal.

1.  Ajuste as permissões do arquivo da chave (execute este comando no seu terminal local, na pasta onde salvou a chave):
    ```bash (Caso esteja utilizando WSL)
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
    O status deve aparecer como `active (running)`.

**2.2. Criação da Página Web**

1.  Navegue até o diretório raiz do Nginx:
    ```bash
    cd /var/www/html
    ```
2.  Crie o arquivo `index.html` com o editor de texto `nano`[cite: 19, 45]:
    ```bash
    sudo nano index.html
    ```
3.  Cole o conteúdo HTML da sua página no editor. Exemplo:
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

Abra um navegador e acesse o endereço IP público da sua instância. Sua página personalizada deve ser exibida.

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

[cite_start]Agora, crie o arquivo do script usando o editor `nano`.

```bash
nano monitor.sh
```

Copie e cole o código abaixo dentro do editor `nano`. Este script verifica o site, registra o status no log e envia um alerta para o Discord em caso de falha.

```bash
#!/bin/bash

# --- Configurações ---
URL="http://localhost"
LOG_FILE="/var/log/monitoramento.log"
WEBHOOK_URL="cole sua webhook aqui"

# --- Início do Script ---
HTTP_STATUS=$(curl -o /dev/null -s -w "%{http_code}" $URL)
TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

if [ $HTTP_STATUS -eq 200 ]; then
    # Site OK
    echo "[$TIMESTAMP] SUCESSO: Site está no ar. Status: $HTTP_STATUS" >> $LOG_FILE
else
    # Site com Falha: registrar e notificar
    MESSAGE="FALHA: Site pode estar fora do ar ou com erro. Status: $HTTP_STATUS"
    echo "[$TIMESTAMP] $MESSAGE" >> $LOG_FILE
    
    # Envia notificação para o Discord
    curl -H "Content-Type: application/json" -d "{\"content\": \"🚨 Alerta: $MESSAGE\"}" $WEBHOOK_URL
fi
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
Salve e saia do editor. O monitoramento agora está ativo e automatizado.

### Etapa 4: Testes da Solução

Para validar a implementação, foram realizados os seguintes testes[cite: 24]:
* Acessar o IP público da instância para verificar se o site está online[cite: 64].
* [Parar o serviço do Nginx (`sudo systemctl stop nginx`) para simular uma falha[cite: 65].
* Confirmar o recebimento do alerta de falha no Discord e o registro do erro no arquivo de log[cite: 65].
* Reiniciar o serviço do Nginx (`sudo systemctl start nginx`) e verificar o log retornando ao status de sucesso.

---

### **Aviso de Segurança**

**Cuidado com dados que podem comprometer a segurança**. A URL do Webhook do Discord é uma informação sensível e não deve ser exposta em repositórios públicos.
