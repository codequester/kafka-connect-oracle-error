# kafka-connect-oracle-error
The Docker-Compose file includes everything to rebuild the error we have with cp-kafka-connect and the jdbc-source connector with oracle.
For now when we try to add a jdbc-source-connector on an oracle database, we get the http error 500.

Tickets for that issue can be found here:
https://issues.apache.org/jira/browse/KAFKA-5938
https://github.com/confluentinc/cp-docker-images/issues/338

To reproduce it do the following:

1. Start Zookeeper by entering 
    ```
    docker-compose up -d zookeeper
    ```
2. Start everything else
    ```
    docker-compose up -d
    ```
3. Wait some time (60 sec.) to get all components up and running. Test it by entering the UI's and docker ps
    ```
    Kafka Topics UI: http://172.30.0.12:8000
    Kafka Schema Registry UI: http://http://172.30.0.8:8000
    Kafka Connect UI: http://172.30.0.11:8000
    ```
4. Try to add an oracle jdbc-source-connector:
    ```
    curl -X POST -H "Content-Type: application/json" --data '{ "name": "countries", "config": { "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector", "tasks.max": 1, "connection.url": "jdbc:oracle:thin:@172.30.0.15:1521:xe", "connection.user": "sample", "connection.password": "sample", "mode": "bulk", "query": "select from HR.COUNTRIES", "topic.prefix": "countries", "poll.interval.ms": 100000} }' http://172.30.0.10:8082/connectors
    ```
5. Look at the Logs:
    ```
    docker-compose logs kafkaconnect
    ```
    You will see following Stacktrace at the end of the logs:
```
kafkaconnect_1      | [2017-09-19 14:03:28,093] DEBUG Uncaught exception in REST call:  (org.apache.kafka.connect.runtime.rest.errors.ConnectExceptionMapper)
kafkaconnect_1      | org.apache.kafka.connect.runtime.rest.errors.ConnectRestException: Request timed out
kafkaconnect_1      | at org.apache.kafka.connect.runtime.rest.resources.ConnectorsResource.completeOrForwardRequest(ConnectorsResource.java:267)
kafkaconnect_1      | at org.apache.kafka.connect.runtime.rest.resources.ConnectorsResource.createConnector(ConnectorsResource.java:100)
kafkaconnect_1      | at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
kafkaconnect_1      | at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
kafkaconnect_1      | at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
kafkaconnect_1      | at java.lang.reflect.Method.invoke(Method.java:498)
kafkaconnect_1      | at org.glassfish.jersey.server.model.internal.ResourceMethodInvocationHandlerFactory$1.invoke(ResourceMethodInvocationHandlerFactory.java:81)
kafkaconnect_1      | at org.glassfish.jersey.server.model.internal.AbstractJavaResourceMethodDispatcher$1.run(AbstractJavaResourceMethodDispatcher.java:144)
kafkaconnect_1      | at org.glassfish.jersey.server.model.internal.AbstractJavaResourceMethodDispatcher.invoke(AbstractJavaResourceMethodDispatcher.java:161)
kafkaconnect_1      | at org.glassfish.jersey.server.model.internal.JavaResourceMethodDispatcherProvider$ResponseOutInvoker.doDispatch(JavaResourceMethodDispatcherProvider.java:160)
kafkaconnect_1      | at org.glassfish.jersey.server.model.internal.AbstractJavaResourceMethodDispatcher.dispatch(AbstractJavaResourceMethodDispatcher.java:99)
kafkaconnect_1      | at org.glassfish.jersey.server.model.ResourceMethodInvoker.invoke(ResourceMethodInvoker.java:389)
kafkaconnect_1      | at org.glassfish.jersey.server.model.ResourceMethodInvoker.apply(ResourceMethodInvoker.java:347)
```    
    
