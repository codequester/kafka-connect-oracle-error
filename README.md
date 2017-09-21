# kafka-connect-oracle-error
The Docker-Compose file includes everything to rebuild the error we have with cp-kafka-connect and the jdbc-source connector with oracle.
For now when we try to add a jdbc-source-connector on an oracle database, we get the http error 500.

Tickets for that issue can be found here:

https://issues.apache.org/jira/browse/KAFKA-5938

https://github.com/confluentinc/cp-docker-images/issues/338

To reproduce it do the following:
1. Download Oracle driver: http://www.oracle.com/technetwork/apps-tech/jdbc-112010-090769.html (ojdbc6.jar) and put it into 
    ```
    kafka-connect/jars
    ```
2. go into Project root and start Zookeeper by entering 
    ```
    docker-compose up -d zookeeper
    ```
3. Start everything else
    ```
    docker-compose up -d
    ```
4. Wait some time (60 sec.) to get all components up and running. Test it by entering the UI's and docker ps
    ```
    Kafka Topics UI: http://172.30.0.12:8000
    Kafka Schema Registry UI: http://http://172.30.0.8:8000
    Kafka Connect UI: http://172.30.0.11:8000
    ```
5. Try to add an oracle jdbc-source-connector:
    ```
    curl -X POST -H "Content-Type: application/json" --data '{ "name": "countries", "config": { "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector", "tasks.max": 1, "connection.url": "jdbc:oracle:thin:@172.30.0.15:1521:xe", "connection.user": "sample", "connection.password": "sample", "mode": "bulk", "query": "select * from HR.COUNTRIES", "topic.prefix": "countries", "poll.interval.ms": 100000} }' http://172.30.0.10:8082/connectors
    ```
6. Look at the Logs:
    ```
    docker-compose logs kafkaconnect
    ```
    You will see following Stacktrace at the end of the logs:
```
kafkaconnect_1      | [2017-09-21 11:53:11,521] DEBUG SelectChannelEndPoint@47684bc7{/172.30.0.1:54238<->8082,Open,in,out,-,-,30001/30000,HttpConnection}{io=0,kio=0,kro=1} idle timeout check, elapsed: 30001 ms, remaining: -1 ms (org.eclipse.jetty.io.IdleTimeout)
kafkaconnect_1      | [2017-09-21 11:53:11,521] DEBUG SelectChannelEndPoint@47684bc7{/172.30.0.1:54238<->8082,Open,in,out,-,-,30001/30000,HttpConnection}{io=0,kio=0,kro=1} idle timeout expired (org.eclipse.jetty.io.IdleTimeout)
kafkaconnect_1      | [2017-09-21 11:53:11,521] DEBUG ignored: WriteFlusher@21d031c8{IDLE} {} (org.eclipse.jetty.io.WriteFlusher)
kafkaconnect_1      | [2017-09-21 11:53:11,521] DEBUG Ignored idle endpoint SelectChannelEndPoint@47684bc7{/172.30.0.1:54238<->8082,Open,in,out,-,-,30001/30000,HttpConnection}{io=0,kio=0,kro=1} (org.eclipse.jetty.io.AbstractEndPoint)
kafkaconnect_1      | [2017-09-21 11:53:11,526] DEBUG Uncaught exception in REST call:  (org.apache.kafka.connect.runtime.rest.errors.ConnectExceptionMapper)
kafkaconnect_1      | org.apache.kafka.connect.runtime.rest.errors.ConnectRestException: Request timed out
kafkaconnect_1      | 	at org.apache.kafka.connect.runtime.rest.resources.ConnectorsResource.completeOrForwardRequest(ConnectorsResource.java:267)
kafkaconnect_1      | 	at org.apache.kafka.connect.runtime.rest.resources.ConnectorsResource.createConnector(ConnectorsResource.java:100)
kafkaconnect_1      | 	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
kafkaconnect_1      | 	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
kafkaconnect_1      | 	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
kafkaconnect_1      | 	at java.lang.reflect.Method.invoke(Method.java:498)
kafkaconnect_1      | 	at org.glassfish.jersey.server.model.internal.ResourceMethodInvocationHandlerFactory$1.invoke(ResourceMethodInvocationHandlerFactory.java:81)
kafkaconnect_1      | 	at org.glassfish.jersey.server.model.internal.AbstractJavaResourceMethodDispatcher$1.run(AbstractJavaResourceMethodDispatcher.java:144)
kafkaconnect_1      | 	at org.glassfish.jersey.server.model.internal.AbstractJavaResourceMethodDispatcher.invoke(AbstractJavaResourceMethodDispatcher.java:161)
kafkaconnect_1      | 	at org.glassfish.jersey.server.model.internal.JavaResourceMethodDispatcherProvider$ResponseOutInvoker.doDispatch(JavaResourceMethodDispatcherProvider.java:160)
kafkaconnect_1      | 	at org.glassfish.jersey.server.model.internal.AbstractJavaResourceMethodDispatcher.dispatch(AbstractJavaResourceMethodDispatcher.java:99)
kafkaconnect_1      | 	at org.glassfish.jersey.server.model.ResourceMethodInvoker.invoke(ResourceMethodInvoker.java:389)
kafkaconnect_1      | 	at org.glassfish.jersey.server.model.ResourceMethodInvoker.apply(ResourceMethodInvoker.java:347)
kafkaconnect_1      | 	at org.glassfish.jersey.server.model.ResourceMethodInvoker.apply(ResourceMethodInvoker.java:102)
kafkaconnect_1      | 	at org.glassfish.jersey.server.ServerRuntime$2.run(ServerRuntime.java:326)
kafkaconnect_1      | 	at org.glassfish.jersey.internal.Errors$1.call(Errors.java:271)
kafkaconnect_1      | 	at org.glassfish.jersey.internal.Errors$1.call(Errors.java:267)
kafkaconnect_1      | 	at org.glassfish.jersey.internal.Errors.process(Errors.java:315)
kafkaconnect_1      | 	at org.glassfish.jersey.internal.Errors.process(Errors.java:297)
kafkaconnect_1      | 	at org.glassfish.jersey.internal.Errors.process(Errors.java:267)
kafkaconnect_1      | 	at org.glassfish.jersey.process.internal.RequestScope.runInScope(RequestScope.java:317)
kafkaconnect_1      | 	at org.glassfish.jersey.server.ServerRuntime.process(ServerRuntime.java:305)
kafkaconnect_1      | 	at org.glassfish.jersey.server.ApplicationHandler.handle(ApplicationHandler.java:1154)
kafkaconnect_1      | 	at org.glassfish.jersey.servlet.WebComponent.serviceImpl(WebComponent.java:473)
kafkaconnect_1      | 	at org.glassfish.jersey.servlet.WebComponent.service(WebComponent.java:427)
kafkaconnect_1      | 	at org.glassfish.jersey.servlet.ServletContainer.service(ServletContainer.java:388)
kafkaconnect_1      | 	at org.glassfish.jersey.servlet.ServletContainer.service(ServletContainer.java:341)
kafkaconnect_1      | 	at org.glassfish.jersey.servlet.ServletContainer.service(ServletContainer.java:228)
kafkaconnect_1      | 	at org.eclipse.jetty.servlet.ServletHolder.handle(ServletHolder.java:812)
kafkaconnect_1      | 	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1669)
kafkaconnect_1      | 	at org.eclipse.jetty.servlets.CrossOriginFilter.handle(CrossOriginFilter.java:257)
kafkaconnect_1      | 	at org.eclipse.jetty.servlets.CrossOriginFilter.doFilter(CrossOriginFilter.java:220)
kafkaconnect_1      | 	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1652)
kafkaconnect_1      | 	at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:585)
kafkaconnect_1      | 	at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:221)
kafkaconnect_1      | 	at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1127)
kafkaconnect_1      | 	at org.eclipse.jetty.servlet.ServletHandler.doScope(ServletHandler.java:515)
kafkaconnect_1      | 	at org.eclipse.jetty.server.session.SessionHandler.doScope(SessionHandler.java:185)
kafkaconnect_1      | 	at org.eclipse.jetty.server.handler.ContextHandler.doScope(ContextHandler.java:1061)
kafkaconnect_1      | 	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:141)
kafkaconnect_1      | 	at org.eclipse.jetty.server.handler.HandlerCollection.handle(HandlerCollection.java:110)
kafkaconnect_1      | 	at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:97)
kafkaconnect_1      | 	at org.eclipse.jetty.server.handler.StatisticsHandler.handle(StatisticsHandler.java:159)
kafkaconnect_1      | 	at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:97)
kafkaconnect_1      | 	at org.eclipse.jetty.server.Server.handle(Server.java:499)
kafkaconnect_1      | 	at org.eclipse.jetty.server.HttpChannel.handle(HttpChannel.java:311)
kafkaconnect_1      | 	at org.eclipse.jetty.server.HttpConnection.onFillable(HttpConnection.java:257)
kafkaconnect_1      | 	at org.eclipse.jetty.io.AbstractConnection$2.run(AbstractConnection.java:544)
kafkaconnect_1      | 	at org.eclipse.jetty.util.thread.QueuedThreadPool.runJob(QueuedThreadPool.java:635)
kafkaconnect_1      | 	at org.eclipse.jetty.util.thread.QueuedThreadPool$3.run(QueuedThreadPool.java:555)
kafkaconnect_1      | 	at java.lang.Thread.run(Thread.java:745)
kafkaconnect_1      | [2017-09-21 11:53:11,528] DEBUG org.eclipse.jetty.server.HttpConnection$SendCallback@79d9fd4a[PROCESSING][i=ResponseInfo{HTTP/1.1 500 Internal Server Error,48,false},cb=org.eclipse.jetty.server.HttpChannel$CommitCallback@c953f8a] generate: NEED_HEADER (null,[p=0,l=48,c=8192,r=48],true)@START (org.eclipse.jetty.server.HttpConnection)
kafkaconnect_1      | [2017-09-21 11:53:11,529] DEBUG org.eclipse.jetty.server.HttpConnection$SendCallback@79d9fd4a[PROCESSING][i=ResponseInfo{HTTP/1.1 500 Internal Server Error,48,false},cb=org.eclipse.jetty.server.HttpChannel$CommitCallback@c953f8a] generate: FLUSH ([p=0,l=160,c=8192,r=160],[p=0,l=48,c=8192,r=48],true)@COMPLETING (org.eclipse.jetty.server.HttpConnection)
kafkaconnect_1      | [2017-09-21 11:53:11,529] DEBUG write: WriteFlusher@21d031c8{IDLE} [HeapByteBuffer@35da10ad[p=0,l=160,c=8192,r=160]={<<<HTTP/1.1 500 Inte....v20160210)\r\n\r\n>>>\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00...\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00},HeapByteBuffer@6b33b021[p=0,l=48,c=8192,r=48]={<<<{"error_code":500...est timed out"}>>>\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00...\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00}] (org.eclipse.jetty.io.WriteFlusher)
kafkaconnect_1      | [2017-09-21 11:53:11,529] DEBUG update WriteFlusher@21d031c8{WRITING}:IDLE-->WRITING (org.eclipse.jetty.io.WriteFlusher)
kafkaconnect_1      | [2017-09-21 11:53:11,529] DEBUG flushed 208 SelectChannelEndPoint@47684bc7{/172.30.0.1:54238<->8082,Open,in,out,-,W,8/30000,HttpConnection}{io=0,kio=0,kro=1} (org.eclipse.jetty.io.ChannelEndPoint)
kafkaconnect_1      | [2017-09-21 11:53:11,529] DEBUG update WriteFlusher@21d031c8{IDLE}:WRITING-->IDLE (org.eclipse.jetty.io.WriteFlusher)
kafkaconnect_1      | [2017-09-21 11:53:11,529] DEBUG org.eclipse.jetty.server.HttpConnection$SendCallback@79d9fd4a[PROCESSING][i=ResponseInfo{HTTP/1.1 500 Internal Server Error,48,false},cb=org.eclipse.jetty.server.HttpChannel$CommitCallback@c953f8a] generate: DONE ([p=160,l=160,c=8192,r=0],[p=48,l=48,c=8192,r=0],true)@END (org.eclipse.jetty.server.HttpConnection)
kafkaconnect_1      | [2017-09-21 11:53:11,530] INFO 172.30.0.1 - - [21/Sep/2017:11:51:41 +0000] "POST /connectors HTTP/1.1" 500 48  90011 (org.apache.kafka.connect.runtime.rest.RestServer)
kafkaconnect_1      | [2017-09-21 11:53:11,530] DEBUG RESPONSE /connectors  500 handled=true (org.eclipse.jetty.server.Server)
kafkaconnect_1      | [2017-09-21 11:53:11,530] DEBUG HttpChannelState@190ff7b4{s=DISPATCHED i=true a=null} unhandle DISPATCHED (org.eclipse.jetty.server.HttpChannelState)
kafkaconnect_1      | [2017-09-21 11:53:11,530] DEBUG unconsumed input HttpConnection@19ed9358[FILLING,SelectChannelEndPoint@47684bc7{/172.30.0.1:54238<->8082,Open,in,out,-,-,1/30000,HttpConnection}{io=0,kio=0,kro=1}][p=HttpParser{s=CONTENT,353 of 353},g=HttpGenerator{s=END},c=HttpChannelOverHttp@40eb5309{r=1,c=true,a=COMPLETED,uri=/connectors}] (org.eclipse.jetty.server.HttpConnection)
kafkaconnect_1      | [2017-09-21 11:53:11,530] DEBUG parseNext s=CONTENT HeapByteBuffer@51d77d9b[p=497,l=497,c=16384,r=0]={POST /connectors ....ms": 100000} }<<<>>>\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00...\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00} (org.eclipse.jetty.http.HttpParser)
kafkaconnect_1      | [2017-09-21 11:53:11,531] DEBUG ishut SelectChannelEndPoint@47684bc7{/172.30.0.1:54238<->8082,Open,in,out,-,-,2/30000,HttpConnection}{io=0,kio=0,kro=1} (org.eclipse.jetty.io.ChannelEndPoint)
kafkaconnect_1      | [2017-09-21 11:53:11,531] DEBUG HttpConnection@19ed9358[FILLING,SelectChannelEndPoint@47684bc7{/172.30.0.1:54238<->8082,Open,ISHUT,out,-,-,2/30000,HttpConnection}{io=0,kio=0,kro=1}][p=HttpParser{s=END,353 of 353},g=HttpGenerator{s=END},c=HttpChannelOverHttp@40eb5309{r=1,c=true,a=COMPLETED,uri=/connectors}] filled -1 (org.eclipse.jetty.server.HttpConnection)
kafkaconnect_1      | [2017-09-21 11:53:11,531] DEBUG atEOF HttpParser{s=END,353 of 353} (org.eclipse.jetty.http.HttpParser)
kafkaconnect_1      | [2017-09-21 11:53:11,532] DEBUG HttpInputOverHTTP@241b412e eof EOF (org.eclipse.jetty.server.HttpInput)
kafkaconnect_1      | [2017-09-21 11:53:11,532] DEBUG reset HttpParser{s=END,353 of 353} (org.eclipse.jetty.http.HttpParser)
kafkaconnect_1      | [2017-09-21 11:53:11,532] DEBUG END --> START (org.eclipse.jetty.http.HttpParser)
kafkaconnect_1      | [2017-09-21 11:53:11,532] DEBUG HttpChannelOverHttp@40eb5309{r=1,c=false,a=IDLE,uri=} handle exit, result COMPLETE (org.eclipse.jetty.server.HttpChannel)
kafkaconnect_1      | [2017-09-21 11:53:11,532] DEBUG atEOF HttpParser{s=START,0 of -1} (org.eclipse.jetty.http.HttpParser)
kafkaconnect_1      | [2017-09-21 11:53:11,532] DEBUG parseNext s=START HeapByteBuffer@50f98cca[p=0,l=0,c=0,r=0]={<<<>>>} (org.eclipse.jetty.http.HttpParser)
kafkaconnect_1      | [2017-09-21 11:53:11,532] DEBUG START --> CLOSED (org.eclipse.jetty.http.HttpParser)
kafkaconnect_1      | [2017-09-21 11:53:11,532] DEBUG onClose SelectChannelEndPoint@47684bc7{/172.30.0.1:54238<->8082,CLOSED,ISHUT,out,-,-,3/30000,HttpConnection}{io=0,kio=0,kro=1} (org.eclipse.jetty.io.AbstractEndPoint)
kafkaconnect_1      | [2017-09-21 11:53:11,532] DEBUG close SelectChannelEndPoint@47684bc7{/172.30.0.1:54238<->8082,CLOSED,ISHUT,out,-,-,3/30000,HttpConnection}{io=0,kio=0,kro=1} (org.eclipse.jetty.io.ChannelEndPoint)
kafkaconnect_1      | [2017-09-21 11:53:11,532] DEBUG Destroyed SelectChannelEndPoint@47684bc7{/172.30.0.1:54238<->8082,CLOSED,ISHUT,OSHUT,-,-,3/30000,HttpConnection}{io=0,kio=-1,kro=-1} (org.eclipse.jetty.io.SelectorManager)
kafkaconnect_1      | [2017-09-21 11:53:11,532] DEBUG onClose HttpConnection@19ed9358[FILLING,SelectChannelEndPoint@47684bc7{/172.30.0.1:54238<->8082,CLOSED,ISHUT,OSHUT,-,-,3/30000,HttpConnection}{io=0,kio=-1,kro=-1}][p=HttpParser{s=CLOSED,0 of -1},g=HttpGenerator{s=START},c=HttpChannelOverHttp@40eb5309{r=1,c=false,a=IDLE,uri=}] (org.eclipse.jetty.io.AbstractConnection)
kafkaconnect_1      | [2017-09-21 11:53:11,532] DEBUG onClose SelectChannelEndPoint@47684bc7{/172.30.0.1:54238<->8082,CLOSED,ISHUT,OSHUT,-,-,3/30000,HttpConnection}{io=0,kio=-1,kro=-1} (org.eclipse.jetty.io.AbstractEndPoint)
kafkaconnect_1      | [2017-09-21 11:53:11,533] DEBUG FILLING-->IDLE HttpConnection@19ed9358[IDLE,SelectChannelEndPoint@47684bc7{/172.30.0.1:54238<->8082,CLOSED,ISHUT,OSHUT,-,-,4/30000,HttpConnection}{io=0,kio=-1,kro=-1}][p=HttpParser{s=CLOSED,0 of -1},g=HttpGenerator{s=START},c=HttpChannelOverHttp@40eb5309{r=1,c=false,a=IDLE,uri=}] (org.eclipse.jetty.io.AbstractConnection)
```    
    
