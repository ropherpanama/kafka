# kafka
Learning Kafka

### Iniciar Zookeeper: 

```D:\development\confluent-kafka-installation\confluent-6.1.1\bin\windows>zookeeper-server-start.bat ../../etc/kafka/zookeeper.properties```

Puede que el script de Windows venga con defectos, esto ocurre al levantar Zookeeper. Para solventarlo se debe ubicar el script **kafka-run-class.bat** y ubicar (*Ctrl+F*) dentro del archivo el texto *"rem Classpath addition for core"*

Una vez ubicado, colocar justo encima de esa linea, el siguiente bloque de código

```
rem Classpath addition for LSB style path
if exist %BASE_DIR%\share\java\kafka\* (
call :concat %BASE_DIR%\share\java\kafka\*
)
```

### Iniciar Kafka Server: 

```D:\development\confluent-kafka-installation\confluent-6.1.1\bin\windows>kafka-server-start.bat ../../etc/kafka/server.properties```

Una vez iniciado con exito, debe existir un kafka server ID:

``` 
[2021-03-28 13:50:49,200] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
```

### Crear un topic

Ubicar el script *kafka-topics.bat* y ejecutar la siguiente sentencia:

```
../kafka-topics.bat --create --topic test --partitions 1 --replication-factor 1 --bootstrap-server localhost:9092
```

Donde:
- **test** es el nombre del topic
- **partitions** es la cantidad de particiones que tendra el topic. Esto debe atender a 2 circunstancias: Requerimiento de almacenamiento y Procesamiento Paralelo, por ejemplo: el 1 significa que solo un consumidor podra consumir concurrentemente el topic y la data es pequeña por lo que no se requieren multiples particiones.
- **replication-factor** es la cantidad de copias de cada particion (fault tolerance feature), si una copia se pierde los consumidores pueden seguir obteniendo la data de las replicas restantes (si hay mas de 1)
- **bootstrap-server** es el servidor y puerto donde esta ejecutandose el servidor de kafka (*kafka-config propiedad port, default es 9092*)

Salida del comando:

```
Created topic test.
```

### Usar un productor por consola

Se pueden crear productores y consumidores para un topico utilizando los comandos de consola de la instalación de Kafka, el siguiente productor leerá un archivo (*data/sample1.csv*) y lo enviará al topic *test*

```
kafka-console-producer.bat --topic test --broker-list localhost:9092  < data/sample1.csv
```

### Usar un consumidor por consola

Se crea un consumidor para el topic *test* (hacerlo en un terminal diferente al del productor)

```
kafka-console-consumer.bat --topic test --bootstrap-server localhost:9092 --from-beginning
```

Ahora, cada vez que el productor envie un mensaje al topic, inmediatamente sera transmitido por consola por el consumidor.

## Broker Multinodo

Para crear un grupo de brokers en una misma maquina es preciso modificar algunos valores en el archivo de configuracion de kafka (*server.properties*), en un ambiente distrubuido real no es requerido ya que los brokers estaran en distintos servidores. Sin embargo, el unico punto conflictivo puede ser el *brokerID*, es posible configurarlo manualmente para cada servidor aunque tambien se puede configurar para que se asigne automaticamente un brokerID distinto para cada servidor.

En un caso multinodo en un solo equipo se debe realizar lo siguiente:

- Crear **n** copias del archivo *server.properties* (donde n es la cantidad de brokers que se desean configurar)
- Para cada copia del archivo los valores o propiedades a cambiar para evitar conflictos son: broker.id, listeners (apuntar a un puerto diferente por broker) y log.dirs

Por ejemplo:

```
server-0.properties (broker.id=0, listeners=PLAINTEXT://:9092, log.dirs=/tmp/kafka-logs-0)
server-1.properties (broker.id=1, listeners=PLAINTEXT://:9093, log.dirs=/tmp/kafka-logs-1)
server-2.properties (broker.id=2, listeners=PLAINTEXT://:9094, log.dirs=/tmp/kafka-logs-2)
```

Hecho lo anterior, se debe levantar el servicio de Zookeeper y luego levantar cada broker de kafka por cada archivo de configuracion.

*Dato: si se borra todo lo que existe en la carpeta /tmp es como iniciar el broker desde cero, toda data previa se elimará.*



