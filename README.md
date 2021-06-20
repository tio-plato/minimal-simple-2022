# A minimal simple for issue [spring-data-rest#2022](https://github.com/spring-projects/spring-data-rest/issues/2022)

User
```java
public class User 
{
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String name;

    @ManyToOne
    private Role role;
}
```
Role
```java
public class Role
{
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "role")
    private List<User> users;
}
```
## Problem

First create a role "admin"
```shell
curl -L -X POST 'localhost:8080/roles' \
-H 'Content-Type: application/json' \
--data-raw '{
    "name": "admin"
}'
```
Then create a user with the role "admin" 

**In spring 2.4.5, use ref link to establish the relationship between user and role**
```shell
curl -L -X POST 'localhost:8080/users' \
-H 'Content-Type: application/json' \
--data-raw '{
    "name": "admin_user",
    "role": "/1"
}'
```
But in spring 2.5.0 or 2.5.1, it will get an exception
```shell
2021-06-20 18:06:04.615 ERROR 14848 --- [nio-8080-exec-2] o.s.d.r.w.RepositoryRestExceptionHandler : JSON parse error: Cannot construct instance of `com.example.demo.entity.Role` (although at least one Creator exists): no String-argument constructor/factory method to deserialize from String value ('/1'); nested exception is com.fasterxml.jackson.databind.exc.MismatchedInputException: Cannot construct instance of `com.example.demo.entity.Role` (although at least one Creator exists): no String-argument constructor/factory method to deserialize from String value ('/1')
 at [Source: (org.apache.catalina.connector.CoyoteInputStream); line: 3, column: 13] (through reference chain: com.example.demo.entity.User["role"])

org.springframework.http.converter.HttpMessageNotReadableException: JSON parse error: Cannot construct instance of `com.example.demo.entity.Role` (although at least one Creator exists): no String-argument constructor/factory method to deserialize from String value ('/1'); nested exception is com.fasterxml.jackson.databind.exc.MismatchedInputException: Cannot construct instance of `com.example.demo.entity.Role` (although at least one Creator exists): no String-argument constructor/factory method to deserialize from String value ('/1')
 at [Source: (org.apache.catalina.connector.CoyoteInputStream); line: 3, column: 13] (through reference chain: com.example.demo.entity.User["role"])
	at org.springframework.http.converter.json.AbstractJackson2HttpMessageConverter.readJavaType(AbstractJackson2HttpMessageConverter.java:389) ~[spring-web-5.3.8.jar:5.3.8]
	at org.springframework.http.converter.json.AbstractJackson2HttpMessageConverter.readInternal(AbstractJackson2HttpMessageConverter.java:350) ~[spring-web-5.3.8.jar:5.3.8]
	at org.springframework.http.converter.AbstractHttpMessageConverter.read(AbstractHttpMessageConverter.java:199) ~[spring-web-5.3.8.jar:5.3.8]
	at org.springframework.data.rest.webmvc.config.PersistentEntityResourceHandlerMethodArgumentResolver.read(PersistentEntityResourceHandlerMethodArgumentResolver.java:246) ~[spring-data-rest-webmvc-3.5.1.jar:3.5.1]
	at org.springframework.data.rest.webmvc.config.PersistentEntityResourceHandlerMethodArgumentResolver.lambda$read$6(PersistentEntityResourceHandlerMethodArgumentResolver.java:202) ~[spring-data-rest-webmvc-3.5.1.jar:3.5.1]
	at java.util.Optional.orElseGet(Optional.java:267) ~[na:1.8.0_292]
	at org.springframework.data.rest.webmvc.config.PersistentEntityResourceHandlerMethodArgumentResolver.read(PersistentEntityResourceHandlerMethodArgumentResolver.java:202) ~[spring-data-rest-webmvc-3.5.1.jar:3.5.1]
	at org.springframework.data.rest.webmvc.config.PersistentEntityResourceHandlerMethodArgumentResolver.resolveArgument(PersistentEntityResourceHandlerMethodArgumentResolver.java:132) ~[spring-data-rest-webmvc-3.5.1.jar:3.5.1]
	at org.springframework.web.method.support.HandlerMethodArgumentResolverComposite.resolveArgument(HandlerMethodArgumentResolverComposite.java:121) ~[spring-web-5.3.8.jar:5.3.8]
	at org.springframework.web.method.support.InvocableHandlerMethod.getMethodArgumentValues(InvocableHandlerMethod.java:170) ~[spring-web-5.3.8.jar:5.3.8]
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:137) ~[spring-web-5.3.8.jar:5.3.8]
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:106) ~[spring-webmvc-5.3.8.jar:5.3.8]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:894) ~[spring-webmvc-5.3.8.jar:5.3.8]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808) ~[spring-webmvc-5.3.8.jar:5.3.8]
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87) ~[spring-webmvc-5.3.8.jar:5.3.8]
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1063) ~[spring-webmvc-5.3.8.jar:5.3.8]
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:963) ~[spring-webmvc-5.3.8.jar:5.3.8]
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006) [spring-webmvc-5.3.8.jar:5.3.8]
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:909) [spring-webmvc-5.3.8.jar:5.3.8]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:652) [tomcat-embed-core-9.0.46.jar:4.0.FR]
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883) [spring-webmvc-5.3.8.jar:5.3.8]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:733) [tomcat-embed-core-9.0.46.jar:4.0.FR]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:227) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53) [tomcat-embed-websocket-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100) [spring-web-5.3.8.jar:5.3.8]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119) [spring-web-5.3.8.jar:5.3.8]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93) [spring-web-5.3.8.jar:5.3.8]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119) [spring-web-5.3.8.jar:5.3.8]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201) [spring-web-5.3.8.jar:5.3.8]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119) [spring-web-5.3.8.jar:5.3.8]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:97) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:542) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:143) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:78) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:357) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:374) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:893) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1707) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_292]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_292]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.46.jar:9.0.46]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_292]
Caused by: com.fasterxml.jackson.databind.exc.MismatchedInputException: Cannot construct instance of `com.example.demo.entity.Role` (although at least one Creator exists): no String-argument constructor/factory method to deserialize from String value ('/1')
 at [Source: (org.apache.catalina.connector.CoyoteInputStream); line: 3, column: 13] (through reference chain: com.example.demo.entity.User["role"])
	at com.fasterxml.jackson.databind.exc.MismatchedInputException.from(MismatchedInputException.java:63) ~[jackson-databind-2.12.3.jar:2.12.3]
	at com.fasterxml.jackson.databind.DeserializationContext.reportInputMismatch(DeserializationContext.java:1588) ~[jackson-databind-2.12.3.jar:2.12.3]
	at com.fasterxml.jackson.databind.DeserializationContext.handleMissingInstantiator(DeserializationContext.java:1213) ~[jackson-databind-2.12.3.jar:2.12.3]
	at com.fasterxml.jackson.databind.deser.std.StdDeserializer._deserializeFromString(StdDeserializer.java:311) ~[jackson-databind-2.12.3.jar:2.12.3]
	at com.fasterxml.jackson.databind.deser.BeanDeserializerBase.deserializeFromString(BeanDeserializerBase.java:1495) ~[jackson-databind-2.12.3.jar:2.12.3]
	at com.fasterxml.jackson.databind.deser.BeanDeserializer._deserializeOther(BeanDeserializer.java:207) ~[jackson-databind-2.12.3.jar:2.12.3]
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:197) ~[jackson-databind-2.12.3.jar:2.12.3]
	at com.fasterxml.jackson.databind.deser.impl.MethodProperty.deserializeAndSet(MethodProperty.java:129) ~[jackson-databind-2.12.3.jar:2.12.3]
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserializeFromObject(BeanDeserializer.java:402) ~[jackson-databind-2.12.3.jar:2.12.3]
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:195) ~[jackson-databind-2.12.3.jar:2.12.3]
	at com.fasterxml.jackson.databind.deser.DefaultDeserializationContext.readRootValue(DefaultDeserializationContext.java:322) ~[jackson-databind-2.12.3.jar:2.12.3]
	at com.fasterxml.jackson.databind.ObjectMapper._readMapAndClose(ObjectMapper.java:4593) ~[jackson-databind-2.12.3.jar:2.12.3]
	at com.fasterxml.jackson.databind.ObjectMapper.readValue(ObjectMapper.java:3601) ~[jackson-databind-2.12.3.jar:2.12.3]
	at org.springframework.http.converter.json.AbstractJackson2HttpMessageConverter.readJavaType(AbstractJackson2HttpMessageConverter.java:378) ~[spring-web-5.3.8.jar:5.3.8]
	... 54 common frames omitted
```

## Solution

Replacing the ref link with an Object will solve this problem
```shell
curl -L -X POST 'localhost:8080/users' \
-H 'Content-Type: application/json' \
--data-raw '{
    "name": "admin_user",
    "role": {"id":1}
}'
```



