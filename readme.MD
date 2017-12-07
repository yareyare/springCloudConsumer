基于spring boot创建 服务消费者

1.引入依赖包  eureka ，ribbon
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-ribbon</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

2.创建启动项目主类 SpringCloudConsumerApplication，通过@EnableDiscoveryClient注解让该应用注册为Eureka客户端应用，以获得服务发现的能力。
同时，在该类中创建RestTemplate的SpringBean实例，并通过@LoadBalanced注解开启客户端负载均衡。
@EnableDiscoveryClient
@SpringBootApplication
public class SpringCloudConsumerApplication {
	@Bean
	@LoadBalanced
	RestTemplate restTemplate(){
		return new RestTemplate();
	}
	public static void main(String[] args) {
		SpringApplication.run(SpringCloudConsumerApplication.class, args);
	}
}

3.创建ConsumerController
实现/ribbon-consumer接口。在该接口中通过RestTemplate来实现对HELLO-SERVICE服务提供的/hello接口进行调用。
这里访问的是服务名HELLO-SERVICE（服务治理的重要表现），不是具体地址。
@RestController
public class ConsumerController {

    @Autowired
    RestTemplate restTemplate;

    @RequestMapping(value = "ribbon-consumer",method = RequestMethod.GET)
    public String helloConsumer(){
        return restTemplate.getForEntity("http://HELLO-SERVICE/hello",String.class).getBody();
    }
}

4.配置文件
server.port=8200
spring.application.name=ribbon-consumer


eureka.client.service-url.defaultZone=http://localhost:8000/eureka/

5.服务启动
启动后可以在注册中心看到RIBBON-CONSUMER注册成功
浏览器访问接口ribbon-consumer
http://127.0.0.1:8200/ribbon-consumer

观察每个服务端的后台打印信息，会显示所有这个接口服务的主机
ribbon就是按照此信息进行轮询访问，以实现基于客户端的负载均衡。



DynamicServerListLoadBalancer for client HELLO-SERVICE initialized:
DynamicServerListLoadBalancer:{NFLoadBalancer:name=HELLO-SERVICE,current list of Servers=[192.168.100.173:8101, 192.168.100.173:8102, 192.168.100.173:8100],Load balancer stats=Zone stats: {defaultzone=[Zone:defaultzone;	Instance count:3;	Active connections count: 0;	Circuit breaker tripped count: 0;	Active connections per server: 0.0;]
},Server stats: [[Server:192.168.100.173:8100;	Zone:defaultZone;	Total Requests:0;	Successive connection failure:0;	Total blackout seconds:0;	Last connection made:Thu Jan 01 08:00:00 CST 1970;	First connection made: Thu Jan 01 08:00:00 CST 1970;	Active Connections:0;	total failure count in last (1000) msecs:0;	average resp time:0.0;	90 percentile resp time:0.0;	95 percentile resp time:0.0;	min resp time:0.0;	max resp time:0.0;	stddev resp time:0.0]
, [Server:192.168.100.173:8101;	Zone:defaultZone;	Total Requests:0;	Successive connection failure:0;	Total blackout seconds:0;	Last connection made:Thu Jan 01 08:00:00 CST 1970;	First connection made: Thu Jan 01 08:00:00 CST 1970;	Active Connections:0;	total failure count in last (1000) msecs:0;	average resp time:0.0;	90 percentile resp time:0.0;	95 percentile resp time:0.0;	min resp time:0.0;	max resp time:0.0;	stddev resp time:0.0]
, [Server:192.168.100.173:8102;	Zone:defaultZone;	Total Requests:0;	Successive connection failure:0;	Total blackout seconds:0;	Last connection made:Thu Jan 01 08:00:00 CST 1970;	First connection made: Thu Jan 01 08:00:00 CST 1970;	Active Connections:0;	total failure count in last (1000) msecs:0;	average resp time:0.0;	90 percentile resp time:0.0;	95 percentile resp time:0.0;	min resp time:0.0;	max resp time:0.0;	stddev resp time:0.0]
]}ServerList:org.springframework.cloud.netflix.ribbon.eureka.DomainExtractingServerList@53eb3b3f