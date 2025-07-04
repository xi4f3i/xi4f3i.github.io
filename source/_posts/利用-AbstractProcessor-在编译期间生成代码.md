---
title: 利用 AbstractProcessor 在编译期间生成代码
date: 2020-02-01 19:44:52
tags: [Java]
---

# 背景

内部 Web 应用框架要求应用对外暴露的所有接口都要有对应的 Interface 和 Implement 。

示例如下：

每新增一个 HTTP 接口都需要在 ExampleSOAService Interface 中添加方法定义，并在 ExampleSOAServiceImpl 中添加方法实现。

```java
public interface ExampleSOAService {
    // 需新增的代码
    HelloWorldResponseType helloWorld(HelloWorldRequestType request) throws Exception;
}

@Service
public class ExampleSOAServiceImpl implements ExampleSOAService {
    @Autowired
    private ServiceInvoker serviceInvoker;

    // 需新增的代码
    public HelloWorldResponseType helloWorld(HelloWorldRequestType request) throws Exception {
        return serviceInvoker.invoke(HelloWorldService.class, request);
    }
}

@Service
public class DefaultServiceInvokerImpl implements ServiceInvoker {
    private ApplicationContext context;

    @Autowired
    public DefaultServiceInvokerImpl(ApplicationContext context) { this.context = context; }

    @Override
    public <TReq, TRes, TServ> TRes invoke(Class<TServ> cls, TReq req) throws Exception {
        return context.getBean(cls).service(req);
    }
}
```

由于每个接口都统一调用自定义实现的 ServiceInvoker.invoke 方法，所以每个接口的定义和实现只有名称不同。

# 实现方法

通过 AbstractProcessor 在编译期间生成接口定义和实现的相关代码。

## 自定义 Annotation

自定义一个 Annotation ，添加到每个接口的 Service 类上。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Service
public @interface WebService {
    String name() default "ExampleSOAService";
}

@WebService
public class HelloWorldService {
    public HelloWorldResponseType service(HelloWorldRequestType request) throws Exception {
        HelloWorldResponseType response = new HelloWorldResponseType();
        return response;
    }
}
```

## AbstractProcessor

在编译期间自动找到所有添加了 WebService 注解的类，并根据 WebServiceGenerateProcessor.process 内的逻辑生成 ExampleSOAService 和 ExampleSOAServiceImpl 文件。

```java
public class WebServiceGenerateProcessor extends AbstractProcessor {
    private Map<String, Pair<TypeSpec.Builder, TypeSpec.Builder>> files = Maps.newLinkedHashMap();
    private JavacElements elementUtils;
    private JavacTypes typeUtils;
    private Messager messager;

    @Override
    public Set<String> getSupportedAnnotationTypes() { return Collections.singleton(WebService.class.getName()); }

    @Override
    public SourceVersion getSupportedSourceVersion() { return SourceVersion.RELEASE_8; }

    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        this.elementUtils = (JavacElements) processingEnv.getElementUtils();
        this.typeUtils = (JavacTypes) processingEnv.getTypeUtils();
        this.messager = processingEnv.getMessager();
        this.messager.printMessage(Diagnostic.Kind.NOTE, "Generate WebServices...");
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        if (roundEnv.processingOver()) {
            /**
             * 写文件
             * JavaFile.builder
             */
        } else {
            /**
             * 生成文件内容
             * 通过 roundEnv.getElementsAnnotatedWith 获取 service class
             * 通过 TypeSpec.interfaceBuilder TypeSpec.classBuilder
             * MethodSpec.methodBuilder CodeBlock.builder AnnotationSpec.builder
             * 等方法生成文件内容
             */
        }
        return false;
    }
}
```
