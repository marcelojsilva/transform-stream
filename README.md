## Link References
https://kafka-tutorials.confluent.io/
https://docs.confluent.io/current/streams/developer-guide/dsl-api.html
https://kafka-tutorials.confluent.io/transform-a-stream-of-events/kstreams.html
https://www.ru-rocker.com/2020/08/11/how-to-join-one-to-many-and-many-to-one-relationship-with-kafka-streams/
https://zcox.wordpress.com/2017/01/16/updating-materialized-views-and-caches-using-kafka/
https://www.atomiccommits.io/joining-streams/
https://events19.linuxfoundation.org/wp-content/uploads/2017/12/Beyond-the-DSL%E2%80%94Unlocking-the-Power-of-Kafka-Streams-with-the-Processor-API-Antony-Stubbs-Confluent-Inc..pdf
http://timvanlaer.be/2017-06-22/combining-the-dsl-and-processor-api-of-kafka-streams/


## Steps to run project
Run docker-compose
```
docker-compose up -d
```

Create gradle wrapper
```
gradle wrapper
```

Build
```
./gradlew build
```

Create shadowJar
```
./gradlew shadowJar
```
Run jar
```
java -jar build/libs/kstreams-transform-standalone-0.0.1.jar configuration/dev.properties
```

Run console producer EC
```
docker exec -i schema-registry /usr/bin/kafka-avro-console-producer --topic EstabComercial --broker-list broker:9092 --property value.schema="$(< src/main/avro/EstabComercial.avsc)"
```

Insert avro on console
```
{"id": 1, "CodEc": 121, "tipoEC": "ARTESP", "liberadoOperacao": 1, "codGrupoEc": 0}
{"id": 2, "CodEc": 122, "tipoEC": "ARTESP", "liberadoOperacao": 1, "codGrupoEc": 0}
{"id": 1, "CodEc": 121, "tipoEC": "ARTESP", "liberadoOperacao": 0, "codGrupoEc": 0}
{"id": 3, "CodEc": 123, "tipoEC": "ARTESP", "liberadoOperacao": 1, "codGrupoEc": 0}
{"id": 4, "CodEc": 124, "tipoEC": "ARTESP", "liberadoOperacao": 1, "codGrupoEc": 0}
{"id": 5, "CodEc": 125, "tipoEC": "ARTESP", "liberadoOperacao": 1, "codGrupoEc": 0}

{"id": 5, "CodEc": 125, "tipoEC": "ARTESP", "liberadoOperacao": 1, "codGrupoEc": 0}
{"id": 6, "CodEc": 126, "tipoEC": "ARTESP", "liberadoOperacao": 1, "codGrupoEc": 0}

{"id": 7, "CodEc": 127, "tipoEC": "ARTESP", "liberadoOperacao": 1, "codGrupoEc": 0}
```

Run console producer Identificador
```
docker exec -i schema-registry /usr/bin/kafka-avro-console-producer --topic Identificador --broker-list broker:9092 --property value.schema="$(< src/main/avro/Identificador.avsc)"
```

Insert avro on console
```
{"id": 1, "tipoIdent": "TAG", "dataAlteracao": "2020-10-10 10:10:00", "placa": "AAA1234", "ativo": true, "bloqueioTemp": false, "bloqueioSaldo": false, "codGrupoEc": 0}
{"id": 2, "tipoIdent": "TAG", "dataAlteracao": "2020-10-10 10:10:01", "placa": "AAA1235", "ativo": true, "bloqueioTemp": false, "bloqueioSaldo": false, "codGrupoEc": 0}

Run console consumer
```
docker exec -it schema-registry /usr/bin/kafka-avro-console-consumer --topic SituacaoIdentificador --bootstrap-server broker:9092 --from-beginning
```

### Result
Output example of two transformed lines for each EC
```
{"id":1,"CodEc":121,"idIdent":0,"tipoIdent":"tag","dataAlteracao":"2020-10-10 10:10:00","placa":"AAA1234","ativo":1,"bloqueado":0}
{"id":1,"CodEc":121,"idIdent":1,"tipoIdent":"tag","dataAlteracao":"2020-10-10 10:10:01","placa":"AAA1234","ativo":1,"bloqueado":0}
```


------------------
Annotations

docker exec -it ksqldb-cli ksql http://ksqldb-server:8088
SET 'auto.offset.reset' = 'earliest';

create stream EstabComercialStream with(kafka_topic='EstabComercial', value_format='AVRO');

create table EstabComercial_tbl 
    with (
        value_format='AVRO'
    ) as
    select id, latest_by_offset(codEc) codEc, latest_by_offset(tipoEc) tipoEc, latest_by_offset(liberadoOperacao) liberadoOperacao, latest_by_offset(codGrupoEc) codGrupoEc
    from ec1 
    group by id;
