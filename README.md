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

-   **Protocolo:** O protocolo foi alterado de `MQTTv5` para `MQTTv3.1.1` (`MQTTv311`) para garantir compatibilidade com o Mosquitto 2.0.20.
-   **Host e Porta:** O host foi definido como `mqtt-broker` e a porta como `1883`, permitindo a comunicação interna via TCP.
-   **TLS:** A tentativa de usar TLS na comunicação interna foi removida, mantendo a criptografia apenas para a porta externa (8883).

Como resultado, o sensor agora publica leituras de forma contínua e confiável na rede interna.

### 2. Broker MQTT

O broker foi configurado para gerenciar o tráfego de maneira segura:

-   **Porta 1883:** Dedicada à comunicação interna de sensores, sem a necessidade de TLS.
-   **Porta 8883:** Designada para conexões externas, com **TLS** e **autenticação obrigatória**.
-   **Segurança:** A configuração `allow_anonymous false` na porta externa, o `password_file` e as **ACLs (Access Control Lists)** foram implementadas para controlar rigorosamente o acesso.
-   **Criptografia:** Certificados TLS foram gerados para garantir a criptografia, integridade e autenticidade das mensagens.

**Exemplo de ACL:**

```text
user liss
topic readwrite sensor/#
````

-----

## Como Executar o Projeto

1.  **Gere a senha de autenticação:**
    ```bash
    mosquitto_passwd -b ./mosquitto/config/mosquitto.passwd liss liss123
    ```
2.  **Inicie os serviços:**
    ```bash
    docker-compose up -d
    ```
    *Para reconstruir o sensor após alterações:*
    ```bash
    docker-compose build temperature-sensor-1
    docker-compose up -d --no-deps --build temperature-sensor-1
    ```

-----

## Testes

### Publicar manualmente (sensor interno)

```bash
mosquitto_pub -h localhost -p 1883 -t 'sensor/temperature' -m '27.5' -d
```

### Subscriber externo via TLS

```bash
mosquitto_sub -h localhost -p 8883 --cafile ./mosquitto/certs/ca.crt -t 'sensor/#' -v --tls-version tlsv1.2 -u liss -P liss123 -d
```

-----

## Resumo Técnico

  - O **Sensor Python** foi corrigido e está funcionando via **TCP** na rede interna.
  - O protocolo foi alterado de **MQTTv5** para **MQTTv3.1.1**.
  - O broker foi configurado com **TLS** e **autenticação** para conexões externas.
  - A segregação de portas (1883 para internos e 8883 para externos) garante a segurança do tráfego.
  - **Certificados TLS** foram criados para proteger a comunicação externa.

Este projeto demonstra boas práticas de segurança MQTT, mantendo o ambiente funcional e garantindo a integridade dos dados transmitidos.
