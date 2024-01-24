在Spring Boot 1.5中，你可以使用Swagger来自动生成API文档。Swagger是一个强大的开源框架，它可以帮助开发者设计、构建、记录和使用RESTful Web服务。Swagger UI是一个集成的部分，它可以生成一个交互式的API文档，让用户可以在浏览器中直接测试API。

以下是如何在Spring Boot 1.5中集成Swagger的步骤：

1. 首先，你需要在你的`pom.xml`文件中添加Swagger的依赖：

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

2. 然后，你需要创建一个配置类来启用Swagger：

```java
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableSwagger2
public class SwaggerConfig {                                    
    @Bean
    public Docket api() { 
        return new Docket(DocumentationType.SWAGGER_2)  
          .select()                                  
          .apis(RequestHandlerSelectors.any())              
          .paths(PathSelectors.any())                          
          .build();                                           
    }
}
```

在这个配置类中，我们创建了一个`Docket` Bean，它是Swagger的主要接口，用于配置Swagger的各种设置。我们使用`@EnableSwagger2`注解来启用Swagger。

3. 最后，你可以通过访问`http://localhost:8080/swagger-ui.html`（假设你的应用运行在本地的8080端口）来查看你的API文档。

请注意，你可能需要根据你的实际需求来调整Swagger的配置。例如，你可能想要只为特定的API生成文档，或者添加更多的文档信息，如描述、联系信息等。你可以查看[Swagger的官方文档](https://swagger.io/docs/)来了解更多信息。
