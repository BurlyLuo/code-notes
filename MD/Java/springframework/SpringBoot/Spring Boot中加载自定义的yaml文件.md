# Spring Boot中加载自定义的yaml文件

在 Spring Boot 项目中`resource`目录下创建一个`simple.yml`文件

```yaml
my:
  enjoy:
    website:
      - github
      - google
    open_source: spring
  like:
    food: chicken
    pc: Thinkpad,MacBook Pro
```

**方式一**：在 Spring Boot 管理的 Bean 中使用 `YamlPropertiesFactoryBean` 把 yaml 文件注入到系统配置中，示例如下：

```java
import lombok.Data;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.core.io.ClassPathResource;
import org.springframework.stereotype.Component;

import java.util.List;

@Data
@Component
@ConfigurationProperties(prefix = "my.enjoy")
public class SimpleYaml implements InitializingBean {
    private List<String> website;
    private String openSource;


    /**
     * 使用YamlPropertiesFactoryBean加载yaml配置文件
     *
     * @return
     */
    @Bean
    public static PropertySourcesPlaceholderConfigurer properties() {
        PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer = new PropertySourcesPlaceholderConfigurer();
        YamlPropertiesFactoryBean yaml = new YamlPropertiesFactoryBean();
        yaml.setResources(new ClassPathResource("simple.yml"));
        propertySourcesPlaceholderConfigurer.setProperties(yaml.getObject());
        return propertySourcesPlaceholderConfigurer;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println(this.toString());
    }
}
```

**方式二**：使用 `YamlMapFactoryBean` 加载 yaml 文件为 Map，自行读取，示例如下：

```java
import lombok.Data;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.config.YamlMapFactoryBean;
import org.springframework.core.io.ClassPathResource;
import org.springframework.stereotype.Component;

import java.util.Map;

@Data
@Component
public class SimpleYaml2 implements InitializingBean {
    private Map<String, Object> object = loadYaml();

    private static Map<String, Object> loadYaml() {
        YamlMapFactoryBean yaml = new YamlMapFactoryBean();
        yaml.setResources(new ClassPathResource("simple.yml"));
        return yaml.getObject();
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println(this.toString());
    }
}
```

**方式三**：扩展 `PropertySourceFactory`，使它支持加载 yaml 文件

1.扩展 `PropertySourceFactory`，实现一个加载 yaml 文件的配置文件工厂类

```java
import org.springframework.boot.env.YamlPropertySourceLoader;
import org.springframework.core.env.PropertySource;
import org.springframework.core.io.support.EncodedResource;
import org.springframework.core.io.support.PropertySourceFactory;

import java.io.IOException;
import java.util.List;

public class YamlPropertySourceFactory implements PropertySourceFactory {

    @Override
    public PropertySource<?> createPropertySource(String name, EncodedResource resource) throws IOException {
        List<PropertySource<?>> sources = new YamlPropertySourceLoader().load(resource.getResource().getFilename(), resource.getResource());
        return sources.get(0);
    }
}
```

2.在 `@PropertySource(value="xxx",factory="xxx")` 中指定工厂为自定义加载 yaml 文件的配置文件工厂类

```java
import lombok.Data;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

import java.util.List;

@Data
@Component
@ConfigurationProperties(prefix = "my.like")
@PropertySource(value = "classpath:simple.yml", factory = YamlPropertySourceFactory.class)
public class SimpleYaml3 implements InitializingBean {
    private String food;
    private List<String> pc;

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println(this.toString());
    }
}
```
