# Guns文档

## Swagger2

@ConditionalOnProperty

- 作用：通过属性name和havingValue来控制某个configuration是否生效。

  其中name用来读取application.properties或者application.yml中的某个属性值。

1. 如果该值为空，则返回false，即configuration不生效；
2. 如果值不为空，则将该值与havingValue指定的值进行比较，相同则返回true，configuration生效；否则返回false，configuration不生效。

@Configuration: springboot的配置项
@EnableSwagger2

代码模板：

```
@Configuration
@EnableSwagger2
@ConditionalOnProperty(prefix = "guns", name = "swagger-open", havingValue = "true")
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))                         		//这里采用包含注解的方式来确定要显示的接口
      			//.apis(RequestHandlerSelectors.basePackage
      			("cn.stylefeng.guns.modular.system.controller"))   
      			//这里采用包扫描的方式来确定要显示的接口
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Doc")
                .description("Api文档")
                .termsOfServiceUrl("https://github.com/")
                .contact("orangehaswing")
                .version("1.0")
                .build();
    }
}
```

## EhCache

xml文件

```
<defaultCache
            maxElementsInMemory="50000"
            eternal="false"
            timeToIdleSeconds="3600"
            timeToLiveSeconds="3600"
            overflowToDisk="true"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
/>
```

配置说明见springboot-ehcache

设置Config

```
@Configuration
@EnableCaching
public class EhCacheConfig {

    // EhCache的配置
    @Bean
    public EhCacheCacheManager cacheManager(CacheManager cacheManager) {
        return new EhCacheCacheManager(cacheManager);
    }
    
     // EhCache的配置
    @Bean
    public EhCacheManagerFactoryBean ehcache() {
        EhCacheManagerFactoryBean ehCacheManagerFactoryBean = new EhCacheManagerFactoryBean();
        ehCacheManagerFactoryBean.setConfigLocation(new ClassPathResource("ehcache.xml"));
        return ehCacheManagerFactoryBean;
    }
}
```

## Shiro

## Beetl模板

## 单/多数据源







