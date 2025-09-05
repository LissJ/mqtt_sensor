# Projeto MQTT (Broker + Sensor + Subscriber)

## Serviços
- **mqtt-broker** (Eclipse Mosquitto 2.0.20), com persistência e logs.
- **temperature-sensor-1** (Python 3.12 + Paho MQTT), publica temperatura a cada 5s.
- **mqtt-subscriber** (mosquitto_sub), assina `sensor/#` e mostra mensagens no log.

## Uso
```bash
docker compose up -d --build
docker compose logs -f mqtt-subscriber
```

## Notas
- Porta 1883 exposta no host. Se não precisar, remova `ports` do broker.
- `allow_anonymous true` apenas para laboratório. Em produção, configure `password_file`, `acl_file` e TLS.
