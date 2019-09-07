# Eureka配置安全账号

###  1、添加security依赖

```
<!--安全认证-->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-security</artifactId>
</dependency>

```
###  2、配置安全策略
```
@EnableWebSecurity
@Configuration
public class PalaEurekaConfig extends WebSecurityConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.NEVER);
        http.csrf().disable();
        //注意：为了可以使用 http://${user}:${password}@${host}:${port}/eureka/ 这种方式登录,所以必须是httpBasic,如果是form方式,不能使用url格式登录
        http.authorizeRequests()
                .antMatchers("/actuator/**").permitAll()
                .anyRequest().authenticated()
                .and()
                .httpBasic();
    }
}


```

###  3、配置文件

```yaml
server:
  port: 8761
spring:
  application:
    name: pala-eureka-server
  security:
    user:
      name: admin
      password: admin
  cloud:
    inetutils:
      ignored-interfaces:
        - docker0
        - veth.*
        - VM.*
      preferred-networks:
        - 192.168

eureka:
  server:
    enable-self-preservation: false
  instance:
    hostname: localhost
  client:
    healthcheck:
      enabled: true
    registerWithEureka: false
    fetchRegistry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

###  4.启动项目，打开eureka界面

  可以看到需要输入对应的用户名密码才能登陆。

![1555254147980](C:\Users\PC4\AppData\Roaming\Typora\typora-user-images\1555254147980.png)



