SpringMVC基于ServletAPI构建，来源于Spring项目模块spring-webmvc

和其它WEB框架相同，SpringMVC也围绕着一个中心Servlet-DispatcherServlet，提供request的分发处理服务，该Servlet需在web.xml中进行配置或者直接在代码中进行初始化配置

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletCxt) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        ac.register(AppConfig.class);
        ac.refresh();

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```

