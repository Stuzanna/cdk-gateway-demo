# This is a Data Generator Service

## Env Configuration
```yaml
  KAFKA_BOOTSTRAP_SERVERS: for kafka bootstrap servers - default = localhost:9092
  KAFKA_SCHEMA_REGISTRY_URL: for schema registry url - default = http://localhost:8081
  KAFKA_KSQL_QUERIES_FILE: for ksql queries file - default = ./config/ksql-queries.sql
  CONFIG_FILE_PATH: for data generator config file path - default = ./config/config.json
```

## How to run
```bash
  docker-compose up -d
```

## How to config ksql
```sql
CREATE STREAM
    customers
WITH (
    KAFKA_TOPIC = 'customers',
    VALUE_FORMAT = 'AVRO'
);

CREATE STREAM products (
    id varchar KEY,
    name string,
    price bigint
)
WITH (
    KAFKA_TOPIC = 'products',
    VALUE_FORMAT = 'AVRO'
);

CREATE STREAM
    purchases
WITH (
    KAFKA_TOPIC = 'purchases',
    VALUE_FORMAT = 'AVRO'
);

CREATE TABLE
    purchase_per_product
    AS SELECT
        products.id as product,
        LATEST_BY_OFFSET(products.name) as product_name,
        sum(purchases.quantity) as total_quantity,
        sum(products.price * purchases.quantity) as total_price
    FROM purchases
    JOIN products WITHIN 5 MINUTES
    ON purchases.productId = products.id
    GROUP BY products.id;
```

## How to config data model

### Example config file
```json
[
  {
    "target": "kafka",
    "topic": "customers",
    "key": {
      "_gen": "with",
      "with": "#{Internet.uuid}"
    },
    "value": {
      "profile": {
        "name": {
          "_gen": "with",
          "with": "#{Name.full_name}"
        },
        "job_title": {
          "_gen": "with",
          "with": "#{Job.title}"
        },
        "blood_group": {
          "_gen": "with",
          "with": "#{Name.blood_group}"
        },
        "creditCardNumber": {
          "_gen": "with",
          "with": "#{Finance.credit_card}"
        }
      },
      "address": {
        "country": {
          "_gen": "with",
          "with": "#{Address.country}"
        }
      },
      "contact": {
        "email": {
          "_gen": "with",
          "with": "#{Internet.email_address}"
        },
        "phone": {
          "_gen": "with",
          "with": "#{PhoneNumber.phone_number}"
        }
      }
    },
    "config": {
      "throttle": {
        "ms": 5000
      }
    }
  },
  {
    "target": "kafka",
    "topic": "products",
    "key": {
      "_gen": "with",
      "with": "#{Internet.uuid}"
    },
    "value": {
      "name": {
        "_gen": "with",
        "with": "#{Commerce.product_name}"
      },
      "color": {
        "_gen": "with",
        "with": "#{Commerce.color}"
      },
      "price": {
        "_gen": "with",
        "with": "#{Number.number_between(40, 1000)}"
      }
    },
    "config": {
      "throttle": {
        "ms": 2000
      }
    }
  },
  {
    "target": "kafka",
    "topic": "purchases",
    "key": {
      "_gen": "with",
      "with": "#{Internet.uuid}"
    },
    "value": {
      "customerId": {
        "_gen": "matching",
        "matching": "customers.key"
      },
      "productId": {
        "_gen": "matching",
        "matching": "products.key"
      },
      "timestamp": {
        "_gen": "with",
        "with": "#{current_timestamp}"
      },
      "quantity": {
        "_gen": "with",
        "with": "#{Number.number_between(1, 100)}"
      }
    },
    "config": {
      "throttle": {
        "ms": 12000
      }
    }
  }
]
```

#### Configuration key

| Key         | Description                                                                                             |
|-------------|---------------------------------------------------------------------------------------------------------|
| `topic`     | Specifies the Kafka topic name for data publishing.                                                     |
| `key`       | A unique identifier for each Kafka message, serving as the message key.                                 |
| `_gen`      | Defines the data generation strategy: `with` for direct values, `matching` for referencing other values. |
| `matching`  | Used with `_gen: "matching"` to refer to the value of another field.                                    |
| `with`      | Provides the value when `_gen` is set to `"with"`, usually a placeholder for dynamic data generation.   |
| `throttle`  | Configures the delay between consecutive messages to control the data flow rate. |

#### Java Faker Commands for Data Generation

## Java Faker Mappings

| Faker Method  | Class       | Command                                    | Description                                                  |
|---------------|-------------|--------------------------------------------|--------------------------------------------------------------|
| `ancient`     | Ancient     | `#{Ancient.god}`                           | Generates a name of an ancient god.                           |
|               |             | `#{Ancient.primordial}`                    | Generates a name of an ancient primordial entity.            |
|               |             | `#{Ancient.titan}`                         | Generates a name of an ancient titan.                         |
|               |             | `#{Ancient.hero}`                          | Generates a name of an ancient hero.                          |
| `app`         | App         | `#{App.name}`                              | Generates an application name.                                |
|               |             | `#{App.version}`                           | Generates an application version.                             |
|               |             | `#{App.author}`                            | Generates an application author name.                         |
| `artist`      | Artist      | `#{Artist.name}`                           | Generates an artist's name.                                   |
| `avatar`      | Avatar      | `#{Avatar.image}`                          | Generates a URL to an avatar image.                           |
| `aviation`    | Aviation    | `#{Aviation.aircraft}`                     | Generates an aircraft name.                                   |
|               |             | `#{Aviation.airport}`                      | Generates an airport name.                                    |
|               |             | `#{Aviation.metar}`                        | Generates a METAR code for weather info at an airport.        |
| `lorem`       | Lorem       | `#{Lorem.words(int)}`                      | Generates a specified number of lorem ipsum words.            |
|               |             | `#{Lorem.characters(int)}`                 | Generates a string of lorem ipsum characters.                 |
|               |             | `#{Lorem.sentence(int, int)}`              | Generates a lorem ipsum sentence with a specific range of words. |
|               |             | ...                                        |                                                              |
| `music`       | Music       | `#{Music.key}`                             | Generates a music key.                                        |
|               |             | `#{Music.instrument}`                      | Generates a musical instrument name.                          |
|               |             | `#{Music.genre}`                           | Generates a music genre.                                      |
| `name`        | Name        | `#{Name.full_name}`                        | Generates a full name.                                        |
|               |             | `#{Name.blood_group}`                      | Generates a blood group type.                                 |
|               |             | `#{Name.username}`                         | Generates a username.                                         |
| `number`      | Number      | `#{Number.number_between(int, int)}`       | Generates a number between the specified range.               |
| `internet`    | Internet    | `#{Internet.email_address}`                | Generates an email address.                                   |
|               |             | `#{Internet.url}`                          | Generates a URL.                                              |
|               |             | `#{Internet.domain_name}`                  | Generates a domain name.                                      |
|               |             | `#{Internet.uuid}`                         | Generates a UUID.                                             |
| `phoneNumber` | PhoneNumber | `#{PhoneNumber.phone_number}`              | Generates a phone number.                                     |
|               |             | `#{PhoneNumber.cell_phone}`                | Generates a cell phone number.                                |
| `pokemon`     | Pokemon     | `#{Pokemon.name}`                          | Generates a Pokémon name.                                     |
|               |             | `#{Pokemon.location}`                      | Generates a Pokémon location.                                 |
| `address`     | Address     | `#{Address.country}`                       | Generates a country name.                                     |
|               |             | `#{Address.zip_code}`                      | Generates a zip code.                                         |
|               |             | `#{Address.city}`                          | Generates a city name.                                        |
|               |             | `#{Address.street_address}`                | Generates a street address.                                   |
|               |             | ...                                        |                                                              |
| `dune`         | Dune            | `#{Dune.quote}`                                 | Generates a quote from Dune.                                        |
|                |                 | `#{Dune.quote(Quote)}`                          | Generates a quote from a specific character in Dune.                |
|                |                 | `#{Dune.title}`                                 | Generates a title within the Dune series.                           |
|                |                 | `#{Dune.planet}`                                | Generates a planet name from the Dune series.                       |
|                |                 | `#{Dune.saying(Saying)}`                     | Generates a saying from a specific character in Dune.               |

**Example enum values for `Dune.quote(Quote)` and `Dune.saying(Saying)`:**
- `Quote`: Can be one of `GUILD_NAIVGATOR, EMPEROR, PAUL, THUFIR, JESSICA, IRULAN, MOHIAM, GURNEY, LETO, STILGAR, LIET_KYNES, PARDOT_KYNES, BARON_HARKONNEN, PITER, ALIA, MAPES, DUNCAN, YUEH`
- `Saying`: Can be one of `BENE_GESSERIT, FREMEN, MENTAT, MUADDIB, ORANGE_CATHOLIC_BIBLE`

**Note**: Current support is limited to `enum`, `int`, and `String` input parameters. Please ensure you use the correct type for each command as per your data generation requirements.
1. `enum` values are case sensitive
2. example config for `int` value: `#{Number.number_between(1, 100)}`
3. example config for `String` value: `#{Internet.safe_email_address('abc')}`

**Important Node on Function Support**: The current implementation does not support direct invocation of functions like `Faker.botify(String)`