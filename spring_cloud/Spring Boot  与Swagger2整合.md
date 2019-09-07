### Spring Boot  与Swagger2整合

------

1. 引入jar

   ```xml
   <!--swagger2-->
   <dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger2</artifactId>
   <version>${swagger2.version}</version>
   </dependency>
   <dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger-ui</artifactId>
   <version>${swagger2.version}</version>
   </dependency>
   ```

2. 编写配置类

   ```java
   @Configuration
   @EnableSwagger2
   public class Swagger2Config {
   
       @Bean
       public Docket createRestApi() {
           return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo()).select()
                   .apis(RequestHandlerSelectors.basePackage("com.paladin")).build();
       }
   
       private ApiInfo apiInfo() {
           return new ApiInfoBuilder()
                   //页面标题
                   .title("Spring Boot 测试使用 Swagger2 构建RESTful API")
                   //创建人
                   .contact(new Contact("Paladin", "http://www.baidu.com", ""))
                   //版本号
                   .version("1.0")
                   //描述
                   .description("API 描述")
                   .build();
       }
   }
   ```

   

3. 测试

   ```java
   @RestController
   @RequestMapping("/hello")
   @Api("测试swagger")
   public class HelloController {
   
       @ApiOperation(value = "根据id查询学生信息", notes = "查询数据库中某个的学生信息")
       @ApiImplicitParam(name = "id", value = "学生ID", paramType = "path", required = true, dataType = "Integer")
       @RequestMapping(value = "/{id}", method = RequestMethod.GET)
       public String getStudent(@PathVariable int id) {
           return "测试成功";
       }
   
   }
   ```