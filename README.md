# cdk-gateway-demo
Run the compose. Login to Console is in the yaml config, bob@conduktor.io admin

If need a fresh Console, kill Console, Postgres and delete volume (`event-demo_pg_data`) before restart.

## Add interceptors
Import the postman collection (event-demo/postman-collection), and the environment to have the variables.

Add your interceptors, using the POST calls. Add the 3, block large topic, encrypt and decrypt.
The encrypt interceptor encrypts any value in the field `password`. 

## Produce a message
In the UI create a topic, produce JSON message e.g.;

```json
{"password": "password"}
```
## View the message
To view it encryped switch cluster to the backing cluster.
Then switch back to GW to see it looks normal if not already shown that, as you have the decrypt on it.

You can play around with deeleting and readding the interceptor if you wish , the delete call is left in there.


## More full example data
Below is optional.

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

# getting-started