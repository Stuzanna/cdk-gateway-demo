# cdk-gateway-demo
Run the compose. Login to Console is in the yaml config, bob@conduktor.io admin

If need a fresh Console, kill Console, Postgres and delete volume (`event-demo_pg_data`) before restart.

Add your interceptors, e.g. the example postman collection. Add the block large topic, the encrypt and the decrypt.
The encrypt interceptor encrypts any value in the field `password`. 

In the UI create a topic, produce JSON message e.g.;

```json
{"password": "password"}
```
To view it encryped switch cluster to the backing cluster. 


# WIP
Below is WIP or optional.

If want a more thorough message you can write the below the 
### create topic
```bash
docker compose exec kafka-client \
  kafka-topics \
    --bootstrap-server conduktor-gateway:6969 \
    --command-config /clientConfig/gateway.properties \
    --create --if-not-exists \
    --topic encryptedTopic
```

```bash
docker compose exec kafka-client \
  kafka-topics \
    --bootstrap-server conduktor-gateway:6969 \
    --command-config /clientConfig/gateway.properties \
    --create --if-not-exists \
    --topic general-topic
```

### produce data to encrypted topic
```bash
echo '{
    "name": "conduktor",
    "username": "test@conduktor.io",
    "password": "password1",
    "visa": "visa123456",
    "address": "Conduktor Towers, London"
}' | jq -c | docker compose exec -T schema-registry \
    kafka-json-schema-console-producer \
        --bootstrap-server conduktor-gateway:6969 \
        --producer.config /clientConfig/gateway.properties \
        --topic encryptedTopic \
        --property value.schema='{
            "title": "User",
            "type": "object",
            "properties": {
                "name": { "type": "string" },
                "username": { "type": "string" },
                "password": { "type": "string" },
                "visa": { "type": "string" },
                "address": { "type": "string" }
            }
        }'
```

### produce to a different topic
```bash
echo '{
    "name": "conduktor",
    "username": "test@conduktor.io",
    "password": "password1",
    "visa": "visa123456",
    "address": "Conduktor Towers, London"
}' | jq -c | docker compose exec -T schema-registry \
    kafka-json-schema-console-producer \
        --bootstrap-server conduktor-gateway:6969 \
        --producer.config /clientConfig/gateway.properties \
        --topic general-topic \
        --property value.schema='{
            "title": "User",
            "type": "object",
            "properties": {
                "name": { "type": "string" },
                "username": { "type": "string" },
                "password": { "type": "string" },
                "visa": { "type": "string" },
                "address": { "type": "string" }
            }
        }'
```

# getting-started