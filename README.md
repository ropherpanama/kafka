# kafka
Learning Kafka

- Iniciar Zookeeper: 

```D:\development\confluent-kafka-installation\confluent-6.1.1\bin\windows>zookeeper-server-start.bat ../../etc/kafka/zookeeper.properties```
- Iniciar Kafka Server: 

```D:\development\confluent-kafka-installation\confluent-6.1.1\bin\windows>kafka-server-start.bat ../../etc/kafka/server.properties```


Al instalar Kafka puede que el script de Windows venga con defectos, esto ocurre al levantar Zookeeper. Para solventarlo se debe ubicar el script **kafka-run-class.bat** y ubicar dentro del archivo el texto *"rem Classpath addition for core"*

Una vez ubicado, colocar justo encima de esa linea, el siguiente bloque de código

```
rem Classpath addition for LSB style path
if exist %BASE_DIR%\share\java\kafka\* (
call :concat %BASE_DIR%\share\java\kafka\*
)
```
