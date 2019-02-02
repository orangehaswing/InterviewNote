# springmvc学习笔记-入门配置(JavaConfig)

# WebMvcConfig

@Configuration:标注在类上，相当于把该类作为spring的xml配置文件中的``，作用为：配置spring容器(应用上下文)。

相当于：

```
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
                         http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context-3.2.xsd
                        http://www.springframework.org/schema/mvc
                        http://www.springframework.org/schema/mvc/spring-mvc.xsd">
     // write code                   
                        
 </beans>                     
```

@ComponentScan(basePackages = "org.springframework.samples.mvc") 相当于：

```
    <context:component-scan base-package="org.springmvcframework">
    </context:component-scan>
```

@EnableWebMvc：注解用来开启Web MVC的配置支持，也就是写Spring MVC时的时候会用到。

@EnableScheduling：注解开启计划任务的支持。 一般都需要@Scheduled注解的配合。

实现WebMvcConfigurer接口

```
// DispatcherServlet context: defines Spring MVC infrastructure
// and web application components

@Configuration
@ComponentScan(basePackages = "org.springframework.samples.mvc")
@EnableWebMvc
@EnableScheduling
public class WebMvcConfig implements WebMvcConfigurer {

	@Override
	public void addFormatters(FormatterRegistry registry) {
		registry.addFormatterForFieldAnnotation(new MaskFormatAnnotationFormatterFactory());
	}

	@Override
	public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
		resolvers.add(new CustomArgumentResolver());
	}

	// Handle HTTP GET requests for /resources/** by efficiently serving
	// static resources under ${webappRoot}/resources/

	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");
	}

	@Override
	public void addViewControllers(ViewControllerRegistry registry) {
		registry.addViewController("/").setViewName("home");
	}

	@Override
	public void configureViewResolvers(ViewResolverRegistry registry) {
		registry.jsp("/WEB-INF/views/", ".jsp");
	}

	@Override
	public void configurePathMatch(PathMatchConfigurer configurer) {
		UrlPathHelper pathHelper = new UrlPathHelper();
		pathHelper.setRemoveSemicolonContent(false); // For @MatrixVariable's
		configurer.setUrlPathHelper(pathHelper);
	}

	@Override
	public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
		configurer.setDefaultTimeout(3000);
		configurer.registerCallableInterceptors(new TimeoutCallableProcessingInterceptor());
	}

	@Bean
	public MultipartResolver multipartResolver() {
		return new CommonsMultipartResolver();
	}

}
```

# MvcShowcaseApplnitializer

启动时初始化Servlet容器

```
public class MvcShowcaseAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer{

	@Override
	protected Class<?>[] getRootConfigClasses() {
		return new Class[] { RootConfig.class };
	}

	@Override
	protected Class<?>[] getServletConfigClasses() {
		return new Class[] { WebMvcConfig.class };
	}

	@Override
	protected String[] getServletMappings() {
		return new String[] { "/" };
	}

	@Override
	protected Filter[] getServletFilters() {
		return new Filter[] { new DelegatingFilterProxy("csrfFilter") };
	}

}
```

# RootConfig

配置CSRF token验证

```
@Configuration
public class RootConfig {

	// CSRF protection. Here we only include the CsrfFilter instead of all of Spring Security.
	// See http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#csrf
	// for more information on Spring Security's CSRF protection

	@Bean
	public CsrfFilter csrfFilter() {
		return new CsrfFilter(new HttpSessionCsrfTokenRepository());
	}

	// Provides automatic CSRF token inclusion when using Spring MVC Form tags or Thymeleaf.
	// See http://localhost:8080/#forms and form.jsp for examples

	@Bean
	public RequestDataValueProcessor requestDataValueProcessor() {
		return new CsrfRequestDataValueProcessor();
	}
}
```











































