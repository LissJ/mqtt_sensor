# Projeto MQTT - Sensor Python com TLS e Autenticação

Este projeto demonstra um ambiente MQTT seguro, com um **broker Mosquitto**, um **sensor de temperatura em Python** e um **subscriber TLS**. O objetivo é garantir **autenticação, segregação de portas e criptografia** nas conexões externas, enquanto o sensor opera de forma funcional na rede interna.

---

## Estrutura do Projeto

-   `docker-compose.yml`: Define os serviços do broker, do sensor e do subscriber.
-   `src/temperature-sensor-1.py`: Código Python do sensor de temperatura.
-   `mosquitto/config/mosquitto.conf`: Arquivo de configuração do broker.
-   `mosquitto/config/mosquitto.passwd`: Senhas de autenticação.
-   `mosquitto/config/mosquitto.acl`: Regras de controle de acesso por tópico.
-   `mosquitto/certs/`: Certificados TLS (`ca.crt`, `server.crt`, `server.key`).

---

## Alterações e Correções

### 1. Sensor Python

O sensor Python foi corrigido para funcionar de maneira confiável. Os seguintes problemas foram resolvidos:

-   **Protocolo:** O protocolo (presente no .py) foi alterado de `MQTTv5` para `MQTTv3.1.1` (`MQTTv311`) para garantir compatibilidade com o Mosquitto 2.0.20.
-   **Host e Porta:** O host foi definido como `mqtt-broker` e a porta como `1883`, permitindo a comunicação interna via TCP.

Como resultado, o sensor agora publica leituras de forma contínua e confiável.

### 2. Broker MQTT

O broker foi configurado para gerenciar o tráfego de maneira segura:

-   **Porta 1883:** Dedicada à comunicação interna de sensores, sem a necessidade de TLS.
-   **Porta 8883:** Designada para conexões externas, com **TLS** e **autenticação obrigatória**.
-   **Segurança:** A configuração `allow_anonymous false` na porta externa, o `password_file` e as **ACLs (Access Control Lists)** foram implementadas para controlar rigorosamente o acesso.
-   **Criptografia:** Certificados TLS foram gerados para garantir a criptografia, integridade e autenticidade das mensagens.

-----

-----

## Autenticação Essencial

Esta seção detalha os passos essenciais para configurar a autenticação e garantir que o broker Mosquitto não aceite conexões anônimas.

### 1\. Gerenciando Permissões

Antes de criar qualquer arquivo, é crucial garantir que as permissões do diretório `mosquitto` sejam do usuário que irá executar os comandos, evitando problemas de acesso.

```bash
sudo chown -R user:user /home/user/mqtt-project/mosquitto
```

Este comando altera o proprietário do diretório para o usuário `user`, concedendo as permissões necessárias para criar e manipular os arquivos de configuração.

### 2\. Criando a Estrutura de Diretórios

Crie a estrutura de diretórios que o broker irá utilizar para armazenar configurações, dados, logs e certificados.

```bash
mkdir -p /home/user/mqtt-project/mosquitto/{config,data,log,certs}
ls -l /home/user/mqtt-project/mosquitto
```

O comando `mkdir -p` cria os diretórios necessários. O `ls -l` é usado para confirmar que a estrutura de pastas foi criada com sucesso.

### 3\. Configurando o Broker

No arquivo `mosquitto.conf`, adicione as seguintes linhas para desativar o acesso anônimo e apontar para o arquivo de senhas.

```ini
allow_anonymous false
password_file /home/user/mqtt-project/mosquitto/config/mosquitto.passwd
```

  * `allow_anonymous false`: Garante que apenas clientes com credenciais válidas possam se conectar.
  * `password_file`: Define o caminho para o arquivo que conterá as senhas criptografadas.

### 4\. Criando Usuário e Senha

Instale as ferramentas do Mosquitto, se ainda não as tiver, e utilize o comando `mosquitto_passwd` para criar o arquivo de senhas e adicionar um novo usuário.

```bash
sudo apt update && sudo apt install mosquitto mosquitto-clients -y
mosquitto_passwd -c /home/user/mqtt-project/mosquitto/config/mosquitto.passwd <USUÁRIO_CRIADO>
```

A flag `-c` cria o arquivo de senhas (`mosquitto.passwd`) e adiciona o usuário `<USUÁRIO_CRIADO>`. Você será solicitado a digitar e confirmar a senha.

### 5\. Verificando a Criação do Arquivo

Após a execução, um arquivo de senhas criptografadas será gerado. Você pode verificar se ele foi criado corretamente com o seguinte comando:

```bash
ls -l /home/user/mqtt-project/mosquitto/config/mosquitto.passwd
```

-----

## Como Executar o Projeto

Para executar este projeto, é necessário seguir alguns passos de configuração para garantir que o ambiente de segurança esteja pronto antes de iniciar os serviços.

### 1\. Criando Certificados TLS

Esses comandos devem ser executados no seu terminal **no diretório raiz do projeto** (`mqtt_sensor`), onde o `docker-compose.yml` está localizado.

1.  **Criação da estrutura de diretórios:**
    Garanta que a estrutura de pastas para os certificados e a configuração exista.

    ```bash
    mkdir -p mosquitto/{config,certs}
    ```

2.  **Geração dos Certificados TLS:**
    Os certificados são essenciais para criptografar a comunicação externa na porta 8883.

    ```bash
    # Navegue para o diretório de certificados
    cd mosquitto/certs

    # Gerar a chave privada da Autoridade Certificadora (CA)
    openssl genrsa -out ca.key 4096

    # Gerar o certificado autoassinado da CA
    openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -subj "/CN=MyTestCA" -out ca.crt

    # Gerar a chave privada e a solicitação de assinatura (CSR) do servidor
    openssl genrsa -out server.key 4096
    openssl req -new -key server.key -subj "/CN=mqtt-broker" -out server.csr

    # Assinar o certificado do servidor com a CA
    openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365 -sha256
    ```

3.  **Criação do Arquivo de Senhas:**
    Use o comando `mosquitto_passwd` para criar um arquivo de senhas para o broker.

    ```bash
    # Gere o arquivo de senha (no host, use o mosquitto_passwd)
    mosquitto_passwd -b ./mosquitto.passwd <USUÁRIO_CRIADO> <SENHA>

    # Mova o arquivo de senha para o diretório de configuração do broker
    mv mosquitto.passwd ../config/mosquitto.passwd
    ```

4.  **Criação do Arquivo de ACL:**
    Crie o arquivo `mosquitto/config/mosquitto.acl` com o seguinte conteúdo para definir as permissões de acesso ao tópico `sensor/#`:

    ```text
    user <USUÁRIO_CRIADO>
    topic read sensor/#
    ```

### 2\. Executando os Serviços

Com os certificados e arquivos de autenticação criados, você pode iniciar o projeto.

1.  **Inicie os serviços com Docker Compose:**
    ```bash
    docker compose up -d --build
    ```
    Isso iniciará o broker, o sensor e o subscriber.

-----

## Testes

### Publicar manualmente (sensor interno)

```bash
mosquitto_pub -h localhost -p 1883 -t 'sensor/temperature' -m '27.5' -d
```

### Subscriber externo via TLS

```bash
mosquitto_sub -h localhost -p 8883 --cafile ./mosquitto/certs/ca.crt -t 'sensor/#' -v --tls-version tlsv1.2 -u <USUÁRIO_CRIADO> -P <SENHA> -d
```

-----

## Resumo Técnico

  - A **autenticação** com usuário e senha foi adicionada ao projeto.
  - O **Sensor Python** foi corrigido e está funcionando via **TCP** na rede interna.
  - O protocolo foi alterado de **MQTTv5** para **MQTTv3.1.1**.
  - O broker foi configurado com **TLS** e **autenticação** para conexões externas.
  - A segregação de portas (1883 para internos e 8883 para externos) garante a segurança do tráfego.
  - **Certificados TLS** foram criados para proteger a comunicação externa.

Este projeto demonstra boas práticas de segurança MQTT, mantendo o ambiente funcional e garantindo a integridade dos dados transmitidos.
