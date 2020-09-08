---
layout:     post
title:      "Restful API的构建"
subtitle:   "Restful，架构，Jersey，SpringMVC"
date:       2019-04-14
author:     "Paul"
header-img: "img/11.jpg"
tags:
    - Restful
    - Jersey
    - SpringMVC
---



> Restful API的构建

# Restful是什么

@(<Inbox>)[Restful, Spring MVC, Jersey]

**RESTFUL**是一种遵循REST架构的设计风格。要了解RESTFUL首先需要理解四个名词：REST，REST服务，JAX-RS标准，Jersey。

* **REST**
  一种跨平台，跨语言的架构风格。
* **REST服务**
  REST服务是REST在Web领域的实现。
* **JAX-RS**
  JAX-RS标准是Java领域对REST式的Web服务指定的实现标准。
* **Jersey**
  Jersey是JAX-RS的标准实现，也是RESTFul的标准实现。

**简单理解REST**
比如对于优惠券的各种业务，可能的请求路径如下：
```
http://classes.wolfcode.cn/coupon_setting.do
//后台能够设置优惠券类型
http://classes.wolfcode.cn/coupon_create.do
//后台根据优惠券类型创建优惠券
http://classes.wolfcode.cn/coupon_list.do
//前台用户查看优惠券类型列表
http://classes.wolfcode.cn/coupon_fetch.do
//前台用户领取优惠券
http://classes.wolfcode.cn/coupon_using.do
//前台用户使用优惠券
```
这时你会发现，针对每一个业务，你都需要一个独立的URL去完成，因为对于优惠券所作出的动作都要反应在URL中，而且还有可能需要根据不同的响应方式来使用不同的URL去描述。

在REST中将资源（优惠券）和动作（增删改查）抽象了出来。
```
http://classes.wolfcode.cn/couponSettings
//优惠券类型
http://classes.wolfcode.cn/coupons
//优惠券
//所有的请求都可以围绕这两个URL来做

POST http://classes.wolfcode.cn/couponSettings
PATCH http://classes.wolfcode.cn/couponSettings
//优惠券类型的创建和修改

GET http://classes.wolfcode.cn/couponSettings/1
//优惠券类型的具体明细，1代表要查询的优惠券的ID。

POST http://classes.wolfcode.cn/couponSettings/1/coupons
//根据优惠券的类型创建优惠券
```
总结来讲：客户端通过HTTP，对服务器端的资源进行操作，使用HTTP动词促使服务器端资源状态发生变化。

**知识点补充**
接口一般分为对内接口和对外接口，对内接口一般就是内部系统之间的服务相互调用，比如商城平台中订单系统和仓储系统之间需要多种接口来处理业务关系，可以使用RPC，RMI，WebService等等。对外的接口，比如应用提供给APP端的接口，一般更多使用普通HTTP请求或者RESTful架构。

**资源设计**
在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能由名词，而且名词往往与数据库表名相对应。
```
https://api.example.com/v1/zoos：动物园资源
https://api.example.com/v1/animals：动物资源
https://api.example.com/v1/employees：饲养员资源
```
**动作设计**
GET：从服务器获取资源                      ----SELECT
POST：在服务器新建资源                    ----CREATE
PUT：在服务器更新资源（完整）       ----UPDATE
PATCH：在服务器更新资源（改变）   ----UPDATE
DELETE：从服务器删除资源                ----DELETE

**Content Type**
一个API可以允许返回JSON，XML等格式，具体格式由头信息规定。一个是请求头中的Accept Type，另一个是响应头中的Content Type。

**返回状态码**
2xx操作成功
3xx的重定向类RedirectionException。
4xx的请求错误类ClientErrorException。
5xx的服务器错误类ServerErrorException。

**常用的RESTful开发框架**
* Jersey
  Jersey RESTful框架是开源的RESTful框架，实现了JAX-RS规范。是一个产品级的RESTful service和client框架。
  JAX-RS（使用RESTful风格来开发web service服务的规范）实现要好于MVC框架。
* SpringMVC
  SpringMVC对RESTful也有完善的支持。



# 使用SpringMVC开发REST应用
创建Maven项目，并且加入相关的Spring，SpringMVC和jackson相关以来，当然也可以使用阿里的fastjson等工具包。
创建POJO，Employee，代表员工：
```
@Setter
@Getter
public class Employee {
	private Long id;
	private String name;
}

```

创建SpringMVC中的Controller层，因为只是例子，就不写对应的service和dao层了（对于Spring MVC项目来说，controller负责url mapping，service负责处理逻辑，dao负责数据库查询，各层之间分工比较明确，代码比较清晰）。
```
@Controller
@RequestMapping("employees")
public class EmployeeRestController {
 //限定只能使用GET方法请求，这时如果使用非GET方法请求就会曝出405异常
@RequestMapping(method = RequestMethod.GET)
@ResponseBody
	public List<Employee> listEmps() {
	    List<Employee> emps = new ArrayList<>();
	    emps.add(new Employee(1L, "emp1"));
	    emps.add(new Employee(2L, "emp2"));
	    return emps;
	}
}
//针对GET方法，也可以直接使用
@GetMapping注解
//@RestController相当于@Controller和@ResponseBody的符合注解
```

@RequestMapping的path参数化
```
@GetMapping(value = "depts/{deptId}/emps/{empId}")
public Employee getEmpBelongDept(@PathVariable("deptId") Long deptId,@PathVariable("empId") Long empId){
 
}
```
headers
headers属性和params差不多，headers是限定要处理的请求头信息，只有匹配该请求头内容的请求才会被该方法处理。
```
@RequestMapping(value = "/something", headers = "content-type=text/*")
```

@consumes 是指定处理请求的提交内容类型（Content-Type）。
@produces 指定返回内容类型，仅当request请求中的（Accept）类型中包含指定类型才返回。


# Jersey
虽然SpringMVC在开发REST应用时比较方便和简单，但是它并不支持JSR311/JSR339标准。最常用的实现了这两个标准的框架就是Jersey了。

**Hello Jersey**
Jersey是一个REST框架，提供了REST服务相关的一切东西，可以和SpringMVC做一下对比。
Jersey可以运行在Servlet环境下也可以脱离该环境。

* **基于Servlet容器**
  1 创建一个Maven的web项目，在pom.xml中只需要引入一个Jersey的Servlet容器依赖。
```
<dependency>
   <groupId>org.glassfish.jersey.containers</groupId>
    <artifactId>jersey-container-servlet</artifactId>
    <version>2.25</version>
</dependency>
```
这时会引入一些核心库，container-servlet-core是基于Servlet容器的Jersey核心库，common是Jersey的核心基础包，server是核心服务包，jersey-client是jersey的客户端包（使用这个包能非常方便的消费REST服务和测试）。

2 修改webx.xml，添加一个jersey的核心servlet（可以理解为MVC框架的前端控制器）
```
<servlet>
<servlet-name>JerseyServletContainer</servlet-name>
<servlet-class>org.glassfish.jersey.servlet.ServletContainer</servlet-class>
 <init-param>
      <param-name>jersey.config.server.provider.packages</param-name>
      <param-value>cn.wolfcode.jersey</param-value>
 </init-param>
<load-on-startup>1</load-on-startup>
</servlet>
 
<servlet-mapping>
    <servlet-name>JerseyServletContainer</servlet-name>
    <url-pattern>/webapi/*</url-pattern>
</servlet-mapping>
```
ServletContainer即为核心控制器，而jersey.config.server.provider.packages参数一看就是扫描jersey中REST服务类所在的包（SpringMVC中的component-scan）。

3 完成服务类
```
package cn.wolfcode.jersey._01hello;
@Path("hello")
public class HelloService {
 
    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hi(){
        return "hello jersey";
    }
}
```
@Path("hello")，代表资源根路径为hello，类似于SpringMVC的@RequestMapping。


* **使用内置容器**
  Jersey提供了使用内置服务器来发布REST服务。提供了对Jetty，JDK HttpServer, Grizzly HttpServer和一个内置的Simple HttpServer来部署。下面以Jetty为例子。
```
<dependency>
    <groupId>org.glassfish.jersey.containers</groupId>
    <artifactId>jersey-container-jetty-http</artifactId>
    <version>2.21</version>
</dependency>
```

2 创建一个Application类，用于设置发布环境：
```
package cn.wolfcode.jersey;
public class RestApplication extends ResourceConfig {
    public RestApplication(){
        this.packages("cn.wolfcode.jersey");
    }
}
```
我们的RestApplication类继承了ResourceConfig类，ResourceConfig是Jersey中用于配置应用资源的类，在Jersey中，把所有提供REST服务的类，都成为资源类。

ResourceConfig类继承了Application类，这是Jersey中一个非常基础的类，用于定义一个JAX-RS应用的基础组建。
该类实现了ServerConfig接口，该接口用于注册应用中的资源组件，该类还实现了Configurable接口，该接口会像当前应用提供一个上下文信息。

3 发布应用
```
public class App {
 
public static void main(String[] args) {
    JettyHttpContainerFactory.createServer(URI.create("http://localhost:8082/"), new RestApplication());
}
}
```

**资源设置**
Jersey中使用@Path注解来设置资源，也支持资源的继承和路径参数。
```
@Path("path")
public class PathRest {
 
/**
 * 映射url中匹配的占位符
 * 
 * @param id
 * @return
 */
@GET
@Path("{id}")
public String pathParam(@PathParam("id") Long id) {
    System.out.println(this);
    System.out.println(id);
    return "success";
}
}
```

**JSON和XML**
在Jersey中返回JSON类型和XML类型都是非常简单的事情，而且功能比SpringMVC更强。
```
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-jackson</artifactId>
    <version>2.25</version>
</dependency>

```
添加了jersey-media-json-jackson这个依赖包，使用Jackson来完成json和xml的转化。

定义一个POJO，为了支持XML生成，添加@XmlRootElement标签。
```
@Setter
@Getter
@NoArgsConstructor
@AllArgsConstructor
@XmlRootElement
public class Department {
    private Long id;
    private String name;
}
```
通过这两个注解来指定返回JSON还是XML。
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)


**请求类型设置**
在SpringMVC中，通过@RequestMapping(method=XXX)来设置，而Jersey则更为简单和直观，直接使用@GET，@POST，@PUT，@DELETE等注解实现。

**参数绑定**
* 绑定路径参数
  我们已经知道路径参数在针对某个资源，或者有子资源的情况下使用，比如/depts/1/emps/，查询id为1的部分下的所有员工。在Jesey中，使用@PathParam完成路径参数绑定：
```
/**
 * 对多个路径参数进行绑定
 * @param id
 * @param month
 * @return
 */
@GET
@Path("{id}/summary/{month}")
public String pathParam2(@PathParam("id") Long id,@PathParam("month")int month) {
    System.out.println(id);
    System.out.println(month);
    return "success";
}
```
* 映射普通请求参数
  获取url后面的参数，可以通过@QueryParam注解完成参数绑定。比如下面这个请求，/params/query?name=myname
```
/**
 * 映射请求参数，需要是GET/POST请求
 * 
 * @param name
 * @return
 */
@GET
@Path("/query")
public String queryParam(@QueryParam("name") String name) {
    System.out.println(name);
    return "success";
}
```

* 映射表单提交参数
  使用表单提交是互联网应用最为常见的方式。Jersey中使用@FormParam注解把表单中的数据映射出来。
```
/**
 * 映射表单提交参数，要求请求是POST,PUT,并且编码格式必须是x-www-form-urlencoded
 * 
 * @param name
 * @param age
 * @return
 */
@PUT
@Path("/form")
@Produces(MediaType.APPLICATION_JSON)
public Employee formParam(@FormParam("name") String name, @FormParam("age") int age) {
    return new Employee(1L, name, age);
}
```
* 映射请求头参数
  从请求头中获取一些参数，比如请求头中自定义的token等信息，可以直接使用Jersey提供的@HeaderParam注解完成。
```
/**
 * 通过@HeaderParam获取请求头内容
 */
@GET
@Path("/head")
public String headParam(@HeaderParam("token") String token) {
    System.out.println(token);
    return "success";
}
```
![token](/imgblog/token.png)

* 绑定矩阵参数
  矩阵参数听着很奇怪，比如做分页的时候，可能会收到类似的请求：/resource;pageSize=10;currentPage=2  普通的请求方式应该是/resource?pageSize=10&currentPage=2
  矩阵参数的好处是把这些参数看做一个资源。Jersey中提供了@MatrixParam标签来完成矩阵参数的绑定
```
/**
 * 从请求路径中分离出key=value的值
 * 
 * @param currentPage
 * @param pageSize
 * @param keyword
 * @return
 */
@GET
@Path("/matrix")
public String matrix(@MatrixParam("currentPage") int currentPage, @MatrixParam("pageSize") int pageSize,
        @MatrixParam("keyword") String keyword) {
    System.out.println("currentPage:" + currentPage);
    System.out.println("pageSize:" + pageSize);
    System.out.println("keyword:" + keyword);
    return "success";
}
```

# Jersey中的注入

**@Context注入特殊资源**
在SpringMVC中，使用@Auotwire能够注入一些非常特殊的对象，比如ApplicationEventPublisher，在Web环境下能注入ServletContext等等。在SpringMVC中，还能在每一个Controller方法参数中注入HttpServletRequest，HttpSession等特殊对象。在Jersey中也可以实现，需要用到@Context注解。

* 获取UriInfo
```
/**
 * 使用@Context获取请求上下文内容
 * 
 * @param ui
 * @return
 */
@GET
@Path("/formui")
public String formPojoParam(@Context UriInfo ui) {
    MultivaluedMap<String, String> qps = ui.getQueryParameters();
    MultivaluedMap<String, String> pps = ui.getPathParameters();
    System.out.println(qps);
    System.out.println(pps);
    return "success";
}
```
在JAX-RS中，一个UriInfo对象封装了应用相关信息和本次请求的相关信息。
* 获取请求头相关信息
```
@Context HttpHeaders headers
```
在JAX-RS中，提供了HttpHeaders接口来描述一个请求头信息，在该接口也提供了很多请求头相关的有用的方法。
* 获取请求处理相关信息
  使用@Context获取一个Request对象
```
/**
 * 通过@Context获取请求信息
 * 
 * @param headers
 * @return
 */
@GET
@Path("/request")
public String request(@Context Request request) {
    System.out.println(request);
    return "success";
}
```
在JAX-RS中，这个Request对象并不代表的是请求对象本身，他是一个有关请求处理的辅助类，用来做请求预处理相关的内容。
* 获取Servlet相关对象
  上面几个例子通过@Context获取的是请求相关的抽象，因为Jersey可以脱离Servlet环境运行。其实@Context也可以获取Servlet相关对象，比如ServletContext，ServletRequest，ServletResponse。
```
@Context ServletContext ctx,
                      @Context HttpServletRequest req,@Context HttpServletResponse resp
```
需要注意的是，要获取Servlet相关对象，必须运行在Servlet环境下。

**资源类的生命周期**
随便写一个测试方法，在方法中打印this，连续两次请求该方法。
```
cn.wolfcode.jersey._04parameters.ParameterRest@c065304
cn.wolfcode.jersey._04parameters.ParameterRest@e5a25a9
```
每次请求都会创建一个全新的资源类对象。这样在请求过多时会有性能问题，可以使用如下注解
```
@Path("param")
@Singleton
public class ParameterRest {
```
再次请求的结果就变为单例了
```
cn.wolfcode.jersey._04parameters.ParameterRest@2f831231
cn.wolfcode.jersey._04parameters.ParameterRest@2f831231
```
按照这种使用方式，就和SpringMVC的controller类似了。子资源也可以使用@Singleton进行注解。

**注入的位置**
前面介绍的所有参数绑定注解，包括@PathParam，@QueryParam等等都可以在主资源类或者子资源类属性，构造方法参数，资源方法参数，Setter，子资源方法参数中。
```
public class ChildResource {
@Context
private ServletContext ctx;
 
@GET
@Path("{version}")
public String child(@PathParam("version") @DefaultValue("2.0") String version) {
    return version;
}
}
```

# Jersey的自定义配置

**Application**
在JAX-RS中，提供了一个非常重要的对象：javax.ws.rs.core.Application。该类定义了一个JAX-RS应用的基本组件和相关的信息。一般我们可以使用Application或者通过继承Application类来完成自己特定的配置。但是这种方式比较难，所以更多的时候我们采取了另外的方式。

**ResourceConfig**
为了方便我们自定义应用，Jersey提供了org.glassfish.jersey.server.ResourceConfig类来简化我们的操作。ResourceConfig类是Jersey自己实现的Application，并且实现了Configurat接口。
ResourceConfig类提供了非常多的方法来注册JAX-RS组件，比如自动扫描。如果想要使用ResourceConfig类来注册我们自己的组件，只需要继承ResourceConfig，并且在构造方法中，注册自己的组件即可。
```
public class RestApplication extends ResourceConfig {
 
public RestApplication(){
    this.packages("cn.wolfcode.jersey");
    this.registerClasses(MyResourceInotherPackage.class);
    this.register(MultiPartFeature.class);
    this.register(FastjsonBodyReader.class);
    this.register(FastjsonBodyWriter.class);
    this.property(CommonProperties.FEATURE_AUTO_DISCOVERY_DISABLE_SERVER, true);
}
}
```
1 继承ResourceConfig类，并且在类的构造方法中，完成字节的组件注册和配置。
2 常用方法：
	packages：提供自动扫描组件。
	registerClasses：提供手动注册组件，包括资源类，Provider，Feature等。
	property：提供手动添加配置选项的方法。
修改Config后，提供了在Servlet环境和内置环境下两种配置方式：
修改web.xml，去掉之前的自动扫描配置，替换为Application的配置，将value改为自己的ResourceConfig类即可。
```

<servlet>
    <servlet-name>JerseyServletContainer</servlet-name>
    <servlet-class>org.glassfish.jersey.servlet.ServletContainer</servlet-class>
    <init-param>
        <param-name>javax.ws.rs.Application</param-name>
        <param-value>cn.wolfcode.jersey._02Application.RestApplication</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

内置环境下配置
```
public class App {
 
public static void main(String[] args) {
    JettyHttpContainerFactory.createServer(URI.create("http://localhost:8082/"), new RestApplication());
}
}
```

# Entity Provider
RESTful框架一定会涉及到输入输出转化的问题。在SpringMVC里，是由HttpMessageConveter类来完成这个转化的，在Jersey中，就是依赖MessageBodyWriter和MessageBodyReader两个类来完成的。

简单理解：MessageBodyWriter作用就是把资源方法按照某种方式输出给客户端；MessageBodyReader就是把请求资源的方法按照某种方式转化成资源方法的参数。比如：
```
@Path("/emps/")
@GET
@Produces(MediaType.APPLICATION_JSON)
public Employee get(@QueryParam("no") Long no){
    return new Employee(no);
}
```
当我们请求GET /epms?no=100的时候，就需要指定MessageBodyReader来把请求中的no=100设置到Long no参数中。同理，当方法返回一个Employee对象，就需要一个指定的MessageBodyWriter把Employee对象使用某个json框架序列化为JSON字符串返回客户端。

**自定义MessageBodyWriter/MessageBodyReader**
工具包可以使用Jackson，当然也可以使用阿里的fastjson。为了简化开发难度，创建一个自定义注解FastJson，来标注哪些类使用FastJson来完成序列化和反序列化。
```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FastJson {
 
}
//增加pom依赖
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.7</version>
</dependency>
```
**实现MessageBodyWriter**
```
@Provider//8
public class FastJsonBodyWriter implements MessageBodyWriter<Object> {//1
 
    @Override
    @Produces(MediaType.APPLICATION_JSON)//3
    public boolean isWriteable(Class<?> type, 
                      Type genericType, 
                      Annotation[] annotations, MediaType mediaType) {//2
        return type.isAnnotationPresent(FastJson.class);//4
    }
 
    @Override
    public long getSize(Object t, Class<?> type, 
                      Type genericType, 
                      Annotation[] annotations, 
                      MediaType mediaType) {//5
        return -1;
    }
 
    @Override
    public void writeTo(Object t, Class<?> type, 
                      Type genericType, 
                      Annotation[] annotations, 
                      MediaType mediaType,
                      MultivaluedMap<String, Object> httpHeaders, 
                      OutputStream entityStream)//6
            throws IOException, WebApplicationException {
        entityStream.write(JSONObject.toJSONString(t).getBytes());//7
    }
 
}
```
1 这个类作用在添加了@FastJson表现的任何对象，所以MessageBodyWriter的接口泛型类型就直接写Object。
2 isWriteable用户判断是否能用这个处理器来处理。
3 因为处理的是相应结果到response的内容转化，所以为了方便开发，可以可以在isWriterable方法添加@Produces标签来辅助限定该方法的响应类型。比如加了MediaType.APPLICATION_JSON，那么就要求作用的对象的@Produces也必须包含。
4 getSize 本来适用于标记响应的Content-Length属性的，在JAX-RS2.0中已经废弃。
5 writeTo： 真正的实施写入操作的方法；使用Fastjson提供的JSONObject.toJSONString 方法把目标转为json字符串输入即可。
6 @Provider标签，标记这是一个Provider类（可以理解为一个提供特殊服务的类），标记为@Provider的类可以被Jersey自动注册并发现。

**实现MessageBodyReader**
同理需要实现MessageBodyReader接口。
```
@Provider//1
public class FastJsonBodyReader implements MessageBodyReader<Object> {//2
    @Override
    @Consumes(MediaType.APPLICATION_JSON)//4
    public boolean isReadable(Class<?> type, 
                      Type genericType, 
                      Annotation[] annotations, MediaType mediaType) {//3
        return type.isAnnotationPresent(FastJson.class);//5
    }
 
    @Override
    public Object readFrom(Class<Object> type, 
                      Type genericType, Annotation[] annotations, 
                      MediaType mediaType,
                      MultivaluedMap<String, String> httpHeaders, 
                      InputStream entityStream)
            throws IOException, WebApplicationException {//6
        BufferedInputStream bis = new BufferedInputStream(entityStream);
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
 
        byte[] buffer = new byte[1024];
        int bytesRead = -1;
        while ((bytesRead = bis.read(buffer)) != -1) {
            baos.write(buffer, 0, bytesRead);
        }
        baos.flush();//7
 
        Object o = JSON.parseObject(baos.toByteArray(), type);//8
        return o;//9
    }
}
```

# Jersey客户端
Jersey客户端API能够让我们非常方便的创建出REST的Web服务客户端。相比于原生的HTTPUrlConnection或者Apache HttpClient，都更加方便和强大。

依赖
在JAX-RS中，提供了一些列的标准的Client API，而Jersey为了更好的实现和扩展这套API，提供了一种扩展机制，即实现org.glassfish.jersey.client.api.Connector接口，可以提供不同具体实现的Client API实现。比如默认使用JDK的Http(s)URLConnection，也可以使用HttpClient实现，还可以使用jetty或者grizzly。

**Client API的使用方式**
* 创建Client
  一般开始一个客户端代码，都是从创建一个client对象开始
```
Client client = ClientBuilder.newClient();
public static Client newClient() {
    return newBuilder().build();
}
```
* 定位一个资源
  创建好Client对象之后，就可以使用target方法定位一个资源。
```
WebTarget webTarget = client.target("http://example.com/rest");
```
得到一个WebTarget对象。这个类代表一个资源URI：
```
public interface WebTarget extends Configurable<WebTarget> 
```
该类继承了Configurable接口，所以我们还可以再某一个单独的URI上面配置Filter等。这个对象还提供了一下几个常用方法：
```
//创建一个子资源URI
public WebTarget path(String path);
//为当前URI设置矩阵参数（参考参数绑定一文）
public WebTarget matrixParam(String name, Object... values);
//为当前URI设置查询参数（参考参数绑定一文）
public WebTarget queryParam(String name, Object... values);
//发起一个请求，获取请求执行器
public Invocation.Builder request();
```
target方法用来定位WEB API的跟资源，在通过path方法创建具体请求的子资源。
```
WebTarget webTarget = client.target("http://www.wolfcode.com/api");
webTarget.register(FilterForExampleCom.class);
//针对应用根路径创建一个WebTarget，设置了一个过滤器，这个过滤器会被由WebTarget创建出来的子资源使用。
WebTarget resourceWebTarget = webTarget.path("resource");
//创建出第一个子资源resourceWebTarget，实际URI为http://www.wolfcode.com/api/resource;
WebTarget helloworldWebTarget = resourceWebTarget.path("helloworld");
//使用resourceWebTarget创建子资源，实际URI为：
http://www.wolfcode.com/api/helloworld;
WebTarget helloworldWebTargetWithQueryParam =
        helloworldWebTarget.queryParam("greeting", "HiWorld!");
//添加请求参数，实际URI为http://www.wolfcode.com/api/helloworld?greeting=hiworld
```
* 执行一个HTTP请求
  获取一个资源URI后，就可以通过WebTarget对象的request方法创建一个Invocation.Builder类，这个类代表一个HTTP请求执行器：
```
Invocation.Builder invocationBuilder =
        helloworldWebTargetWithQueryParam.request(MediaType.TEXT_PLAIN_TYPE);
invocationBuilder.header("some-header", "true");
```
Invocation的方法如下：
```
//执行请求；得到一个响应对象
public Response invoke();
//执行请求；将响应转成一个指定类型对象；
public <T> T invoke(Class<T> responseType);
//异步执行请求（关于异步请求，SSE，WebSocket会单开专题）；
public Future<Response> submit();
```


# Jersey过滤器
在REST应用中，也会出现针对一组亲求需要在请求之前或者之后统一处理的情况，比如登陆检查，版本校对，额外的版权信息等，通过过滤器能够统一处理。

**服务端过滤其（Server Filter）**
Servlet中的过滤器Filter是一种双向过滤器，一个过滤器可以过滤请求和响应。在JAX-RS中，过滤器是单向的，要针对请求过滤就选请求过滤器，要针对响应过滤就要选响应过滤器。
javax.ws.rs.container.ContainerRequestFilter：针对请求的过滤器；
javax.ws.rs.container.ContainerResponseFilter：针对响应的过滤器；

加入我们请求了一个不存在的资源地址，返回404。这时，requestFilter一定要资源请求到了之后，才会执行，而responseFiter是只要由响应返回即可执行。
Jersey提供了@PreMatching注解，只要在对应的requestFilter类上添加该标签，请求过滤器会在执行资源匹配之前运行。

**客户端过滤器**
和服务端过滤器类似，Jersey也提供了两种客户端过滤器：
javax.ws.rs.client.ClientRequestFilter：客户端请求过滤器；
javax.ws.rs.client.ClientResponseFilter：客户端响应过滤器；


# Jersey 拦截器
过滤器主要用来处理请求头，响应头，请求URI地址等等，但是如果涉及到想要修改请求实体内容或者响应实体内容相关的统一业务，就需要使用Jersey提供的拦截器。

拦截器在服务端和客户端是相同的。
javax.ws.rs.ext.WriterInterceptor：写拦截器，可以在其中对于响应实体进行拦截操作；
javax.ws.rs.ext.ReaderInterceptor：读拦截器，可以在其中对于请求实体进行拦截操作；

例子，将输出流进行GZIP压缩。
```
public class GZIPWriterInterceptor implements WriterInterceptor {
 
    @Override
    public void aroundWriteTo(WriterInterceptorContext context)
                    throws IOException, WebApplicationException {
        final OutputStream outputStream = context.getOutputStream();
        context.setOutputStream(new GZIPOutputStream(outputStream));
        context.proceed();
    }
```

**拦截器和过滤器的执行流程**
背景：在服务端和客户端都配置了GZIP拦截器，并且配置了过滤器用于修改请求头。

客户端发起POST->客户端ClientRequestFilters执行，完成请求头修改->客户端GZIP拦截器执行，得到请求实体并压缩->客户端MessageBodyWriter执行，把实体内容通过GZIP写入输出流，真正发送请求。

**名称绑定**
用于对某个类或者方法绑定特定的过滤器或拦截器。JAX-RS提供了NameBinding机制，就是通过指定的过滤器/拦截器通过资源方法的名称绑定在某些匹配的资源方法上。
把对应的注解加到一些拦截器或者过滤器上就可以了。
可以通过@Priority设置优先级拦截器或者过滤器的优先级。

# 统一异常处理
统一异常处理是所有web应用都需要考虑的，在Jersey中，也提供了很简单的统一异常处理方式。

**基本思路**
服务层隐藏底层的受检异常，服务层的异常统一包装为RuntimeException抛出到Web层，由Web层统一对服务进行异常处理。针对Json格式的请求，返回包含异常代码，异常效益或者异常数据对象，针对web页面的请求，统一返回异常结果展示页。

* WebApplicationException
  使用Jersey本身的异常对象，在资源方法中使用HTTP代码来给客户端返回指定错误，比如直接返回500错误等。
```
@Path("exception")
public class ExceptionResource {
 
    @POST
    @Path("register")
    public Response register(@FormParam("name")String username) {
        if ("admin".equals(username)) {
            throw new WebApplicationException("用户名已经存在!",
                    Response.Status.CONFLICT);
        } else {
            return Response.ok("注册成功!", MediaType.TEXT_PLAIN).build();
        }
    }
}
```
可以在资源方法中，或者Provider（拦截器，过滤器，Entity Provider）中抛出。
构造方法比较多，可以传入不同参数构造Response。
对于统一异常处理差距还是很远，因为客户端需要针对不同的状态码进行处理。
* Exception Mapper
  Jersey提供了ExceptionMapper接口来根据异常类型来执行对应的异常处理方法。
```
public interface ExceptionMapper<E extends Throwable> {
    Response toResponse(E exception);
}
```
在该接口中，定义了Response toResponse(E exception)，根据匹配的Exception去生成一个对应的Response对象。
例子，在Jackson框架中，如果在JSON->实体对象的映射过程中，出现解析错误，Jackson会抛出一个JsonMappingException，Jackson自己专门有一个Mapper去处理这个异常。
```
public class JsonMappingExceptionMapper implements ExceptionMapper<JsonMappingException> {
    @Override
    public Response toResponse(JsonMappingException exception) {
        return Response.status(Response.Status.BAD_REQUEST)
                  .entity(exception.getMessage())
                  .type("text/plain").build();
    }
}
JsonMappingExcpetionMapper实现了ExceptionMapper接口，在toResponse方法中，使用Response构造了Status.BAD_REQUEST，并传入异常消息返回。
```
要使用ExceptionMappper，添加@Provider注解或者通过ResourceConfig.register方法注册即可。

统一异常处理的简单例子：
1 首先创建一个异常枚举类，用来标识不同的应用异常类型和对应的异常状态码：
```
@Getter
public enum ExceptionCode {
    DEFAULT_ERROR(0), 
    PERMISSION_EXCEPTION(1), 
    MONEY_CHECK_EXCEPTION(2), 
    ACCOUNT_STATUS_ERROR(3);
 
    private ExceptionCode(int code) {
        this.code = code;
    }
 
    private int code;
}
```

应用异常基础类
```
@Getter
public abstract class ApplicationException extends RuntimeException {
 
    private static final long serialVersionUID = 1L;
    private ExceptionCode code = ExceptionCode.DEFAULT_ERROR;
 
    public ApplicationException(String msg) {
        super(msg);
    }
 
    public ApplicationException(String msg, ExceptionCode code) {
        super(msg);
        this.code = code;
    }
 
    public ApplicationException(String msg, ExceptionCode code,
            Throwable cause) {
        super(msg, cause);
        this.code = code;
    }
}
```
针对不同的服务，提供不同的异常子类，比如权限访问的异常，代码规定了异常状态类型。
```
public class PermissionException extends ApplicationException {
 
    private static final long serialVersionUID = 1L;
 
    public PermissionException(String msg, Throwable ex) {
        super(msg, ExceptionCode.PERMISSION_EXCEPTION, ex);
    }
}
```

创建自己的ExceptionMapper：
```
@Provider
public class ApplicationExceptionMapper
        implements ExceptionMapper<ApplicationException> {
 
    @Override
    public Response toResponse(ApplicationException exception) {
        AjaxResult result = new AjaxResult(false, exception.getMessage(), null,
                exception.getCode().getCode());
        return Response.ok(result, MediaType.APPLICATION_JSON).build();
    }
 
}
```

抛出异常的逻辑
```
@GET
@Path("resource")
@Produces(MediaType.APPLICATION_JSON)
public AjaxResult doSomething(@HeaderParam("token") String token) {
    if ("token".equals(token)) {
        return new AjaxResult(true, "正常访问资源", "some logic value", 0);
    } else {
        throw new PermissionException("没有权限访问该资源", null);
    }
}
```

# Spring集成Jersey
在Jersey中，默认使用的是HK2这个DI/AOP框架来完成服务管理和注入的。所以我们前面看到的@Context，@Service等都是HK2框架提供的。但是我们用Spring比较多，所以需要把Jersey和Spring集成起来。

引入依赖：
```
<dependency>
    <groupId>org.glassfish.jersey.ext</groupId>
    <artifactId>jersey-spring4</artifactId>
    <version>2.26</version>
</dependency>
```

创建一个简单的服务接口实现：
```
public interface ISomeService {
    void doSomething(String msg);
}
```
创建对应的实现：
```
public class SomeServiceImpl implements ISomeService {
    @Override
    public void doSomething(String msg) {
        System.out.println("do some thing:" + msg);
    }
 
}
```

创建资源类Controller
```
@Path("spring")
public class SpringResource {
 
    @Autowired
    private ISomeService someService;
 
    @Autowired
    private ApplicationContext ctx;
 
    public void setSomeService(ISomeService someService) {
        this.someService = someService;
    }
 
    @Path("resource1")
    @GET
    public String resource1(@QueryParam("msg") String msg) {
        System.out.println(this);
        System.out.println(ctx.getBeansOfType(SpringResource.class));
        this.someService.doSomething(msg);
        return "success";
    }
}
//使用@Autowired标签尝试从spring容器中注入ISomeService；
//因为没有使用Controller，service等注解，需要把SomeServiceImpl和Resource在applicationContext中配置。
```
配置web.xml
```
<filter>
    <filter-name>JerseyServletContainer</filter-name>
    <filter-class>org.glassfish.jersey.servlet.ServletContainer</filter-class>
    <init-param>
        <param-name>jersey.config.server.provider.packages</param-name>
        <param-value>cn.wolfcode.jersey</param-value>
    </init-param>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </init-param>
    <init-param>
        <param-name>jersey.config.server.provider.classnames</param-name>
        <param-value>org.glassfish.jersey.media.multipart.MultiPartFeature</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>JerseyServletContainer</filter-name>
    <url-pattern>/webapi/*</url-pattern>
</filter-mapping>
```
需要注意两个参数：
1 jersey.config.server.provider.packages，使用该参数田间Provider和Feature；
2 contextConfigLocation，通过该参数设置Spring的配置文件地址。默认是classpath:applicationContext.xml
3 需要在资源类上加@Component标签，以便能被Spring管理。

**shi用全注解方式**
使用全注解的方式，首先应该修改applicationContext.xml，配置包扫描。
```
<context:component-scan base-package="cn.wolfcode.jersey._10spring" />
```
Service实现类添加@Service注解。
Resurce添加@Component注解。


**使用ResourceConfig代替web.xml方式配置**
资源类，服务类，Spring配置文件都不用修改。
```
@ApplicationPath("webapi")
public class RestApplication extends ResourceConfig {
    public RestApplication() {
        this.packages("cn.wolfcode.jersey");
        this.register(MultiPartFeature.class); this.property("contextConfigLocation","classpath:applicationContext.xml");      this.register(MyRequestTestFilter.class).register(MyResponseTestFilter.class);
    }
}
```

**Provider的注入**
当Jersey容器交给Spring来管理后，Jersey中的各种Provider也可以交给Spring容器管理。这样就不用再resourceconfig中配置filter了。
```
@Provider
@Component
public class MyResponseTestFilter implements ContainerResponseFilter {
```


# SpringBoot集成Jersey
SpringBoot支持Jersey1.x和Jersey2.x。
1 导入jersey的starter 包。
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jersey</artifactId>
</dependency>
```
2 注备好对应的资源类，服务类，代码和Spring集成一样。
```
public interface ISomeService {
 
    void sayHi(String msg);
}
 
@Service
public class SomeServiceImpl implements ISomeService {
    @Override
    public void sayHi(String msg) {
        System.out.println(msg);
    }
}
@Component
@Path("resource")
public class SpringbootResource {
 
    @Autowired
    private ISomeService someService;
 
    @Path("sayhi")
    @GET
    public String sayHi(@QueryParam("msg") String msg) {
        this.someService.sayHi(msg);
        return "success";
    }
}
```

3 配置Jersey对象由三种方式，第一种是自定义的ResourceConfig，第二种是返回一个ResourceConfig类型的@Bean，第三种方式，配置一组ResourceConfigCustomizer对象。

创建自定义ResourceConfig：
```
@Component
public class JerseyConfig extends ResourceConfig {
    public JerseyConfig() {
        register(SpringbootResource.class);
    }
}
```
保证JerseyConfig在Application类能够扫描的包下即可。
SpringBoot默认把Jersey映射在根路径上的。对于自定义的ResoureConfig来说，只需要在类上面添加一个@ApplicationPath注解即可。
```
@Component
@ApplicationPath("webapi")
public class JerseyConfig extends ResourceConfig {
```

ResourceConfig类型的@Bean
```
@Bean
public ResourceConfig resourceConfig() {
    ResourceConfig config = new ResourceConfig();
    config.register(SpringbootResource.class);
    return config;
}
```
在application.properties中配置
spring.jersey.application-path=webapi

使用ResourceConfigCustomizer
Springboot提供了一个ResourceConfigCustomizer接口，让我们更灵活的对ResourceConfig对象进行配置。