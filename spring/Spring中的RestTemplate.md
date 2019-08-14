# Spring中的RestTemplate

有的时候服务需要使用HTTP协议去调用外部接口，在Java中我们通常会使用Apache的HttpClient工具包去实现它。作为一个包罗万象的框架，Spring也提供了一个针对RESTful服务封装的HTTP调用工具——RestTemplate。

### 如何使用

在Spring Boot项目中使用RestTemplate是非常方便的。首先我们需要将RestTemplate对象注入到Spring的bean容器中：

```java
@Bean
public RestTemplate restTemplate() {
  return new RestTemplate();
}
```

在需要使用的地方引入：

```java
@Autowired
private RestTemplate restTemplate;
```

然后通过调用RestTemplate对象的方法实现对目标HTTP接口的调用：

```java
public String sayHello() {
    Map<String, String> map = new HashMap<>();
    map.put("name", "李四");
    String responseString = restTemplate.getForObject("http://HELLO-SERVICE/sayhello?name={name}", String.class, map);
    return responseString;
}
```

当然，具体使用RestTemplate的时候我们还会遇到很多细节问题，下面将会详细介绍这些内容。

### RestTemplate的连接配置信息

在创建RestTemplate对象bean时，我们可以通过ClientHttpRequestFactory对象实现对HTTP连接信息的配置，具体详情见示例：

```java
@Configuration
public class ApiConfig {
    @Bean
    public RestTemplate restTemplate(ClientHttpRequestFactory factory) {
        return new RestTemplate(factory);
    }

    @Bean
    public ClientHttpRequestFactory simpleClientHttpRequestFactory() {
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setReadTimeout(5000);	//连接读取时限，单位为ms
        factory.setConnectTimeout(5000);	//连接超时时限，单位为ms
        //factory.setProxy(proxy);	//设置代理
        factory.setBufferRequestBody(false);	//设置请求体缓存，默认true
        return factory;
    }
}
```

### RestTemplate中的方法

虽然RestTemplate可以发起HTTP协议请求，但是它毕竟是针对RESTful服务做的封装，所以它的内部方法封装同样也是针对RESTful协议来实现的。这里我们将分类对RestTemplate中的方法进行介绍。

#### GET

RESTful中使用GET请求来获取资源，RestTemplate中与GET请求有关的方法如下所示：

```java
public <T> T getForObject(String url, Class<T> responseType, Object... uriVariables);
public <T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables);
public <T> T getForObject(URI url, Class<T> responseType);
public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Object... uriVariables);
public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Map<String, ?> uriVariables);
public <T> ResponseEntity<T> getForEntity(URI url, Class<T> responseType);
```

uriVariables参数可以用来替换url中的占位符，例如示例代码中：

```java
Map<String, String> map = new HashMap<>();
map.put("name", "李四");
String responseString = restTemplate.getForObject("http://HELLO-SERVICE/sayhello?name={name}", String.class, map);
```

Map对象里key值“name”对应的value值“李四”将会替换掉url字符串中的“{name}”。

注意：如果Map对象（或者是Object数组）中的key值，没有在url字符串的占位符中出现，那这些值将**不会**进行替换操作，也**不会**被添加到url参数串中。

#### POST

RESTful中使用POST请求来创建资源，RestTemplate中与POST请求有关的方法如下所示：

```java
public URI postForLocation(String url, @Nullable Object request, Object... uriVariables);
public URI postForLocation(String url, @Nullable Object request, Map<String, ?> uriVariables);
public URI postForLocation(URI url, @Nullable Object request);
public <T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables);
public <T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables);
public <T> T postForObject(URI url, @Nullable Object request, Class<T> responseType);
public <T> ResponseEntity<T> postForEntity(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables);
public <T> ResponseEntity<T> postForEntity(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables);
public <T> ResponseEntity<T> postForEntity(URI url, @Nullable Object request, Class<T> responseType);
```

postForLocation方法表示的是这个post请求服务端的返回中只有url地址，详情请查阅RESTful服务中的定义。

request参数可以是一个`org.springframework.http.HttpEntity`对象，也可以是一个简单的POJO对象，方法里会将不是HttpEntity类型的对象封装到一个HttpEntity的body中。

注意：若服务端同样也是使用Spring框架实现，则服务端正确接受到RestTemplate发送的POST请求参数，需要使用到注解`@RequestBody`。代码示例如下：

```java
@PostMapping("/post")
public Result post(@RequestBody TestPojo pojo) {
    return Result.returnDataResult(pojo);
}
```

#### PUT

RESTful中使用PUT请求来替换已有资源，RestTemplate中与PUT请求有关的方法如下所示：

```java
public void put(String url, @Nullable Object request, Object... uriVariables);
public void put(String url, @Nullable Object request, Map<String, ?> uriVariables);
public void put(URI url, @Nullable Object request);
```

#### PATCH

RESTful中使用PATCH请求来修改或新增资源，RestTemplate中与PATCH请求有关的方法如下所示：

```java
public <T> T patchForObject(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables);
public <T> T patchForObject(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables);
public <T> T patchForObject(URI url, @Nullable Object request, Class<T> responseType);
```

#### DELETE

RESTful中使用DELETE请求来删除资源，RestTemplate中与DELETE请求有关的方法如下所示：

```java
public void delete(String url, Object... uriVariables);
public void delete(String url, Map<String, ?> uriVariables);
public void delete(URI url);
```

#### HEAD

RestTemplate提供相关方法来发送HEAD请求，用来获取请求的Headers信息，RestTemplate中与HEAD请求有关的方法如下所示：

```java
public HttpHeaders headForHeaders(String url, Object... uriVariables);
public HttpHeaders headForHeaders(String url, Map<String, ?> uriVariables);
public HttpHeaders headForHeaders(URI url);
```

#### OPTIONS

RestTemplate提供相关方法来发送OPTIONS请求，用来询问服务器所支持的请求方法有哪些，RestTemplate中与OPTIONS请求有关的方法如下所示：

```java
public Set<HttpMethod> optionsForAllow(String url, Object... uriVariables);
public Set<HttpMethod> optionsForAllow(String url, Map<String, ?> uriVariables);
public Set<HttpMethod> optionsForAllow(URI url);
```

### RestTemplate相关类

在使用RestTemplate的时候，除了它本身之外，我们还可能会用到其他一些类，这里我们将简单介绍比较常用的两个相关类。

#### HttpEntity

在上节介绍POST方法的时候有提到，HttpEntity类主要是用来封装请求报文内容的，下面我们将列举一些这个类的常用方法：

```java
public HttpEntity(T body);	//构造方法，指定body内容
public HttpEntity(MultiValueMap<String, String> headers);	//构造方法，指定headers内容
public HttpEntity(T body, MultiValueMap<String, String> headers);	//构造方法，指定body与headers内容
public HttpHeaders getHeaders();	//获取封装了head信息的hearders对象
public T getBody();	//获取body内容
public boolean hasBody();	//body是否存在
```

当我们希望在请求头中添加一些额外信息的时候，我们只需把这个对象作为参数request传入到方法中即可。

#### ResponseEntity

当我们使用`getForEntity`方法的时候，返回结果便是一个ResponseEntity类的对象。ResponseEntity类继承了HttpEntity类，除了HttpEntity中的方法，这个类中常用的新增方法有：

```java
public HttpStatus getStatusCode();	//获取HTTP响应状态枚举对象
public int getStatusCodeValue();	//获取HTTP相应状态值
```



###### 参考内容链接

1. [RestTemplate 详解](https://zhuanlan.zhihu.com/p/31681913)
2. [SpringBoot系列 - 使用RestTemplate](https://www.xncoding.com/2017/07/06/spring/sb-restclient.html)
3. [详解 RestTemplate 操作](https://blog.csdn.net/itguangit/article/details/78825505)
4. [Spring-RestTemplate之urlencode参数解析异常全程分析](https://www.jianshu.com/p/0bdcc6836eb3)
5. [架构实战篇（六）：Spring Boot RestTemplate的使用](https://www.jianshu.com/p/c96049624891)
6. [RESTful 架构详解](https://www.runoob.com/w3cnote/restful-architecture.html)
7. 源码——`org.springframework.web.client.RestTemplate`, `org.springframework.http.HttpEntity`, `org.springframework.http.ResponseEntity`