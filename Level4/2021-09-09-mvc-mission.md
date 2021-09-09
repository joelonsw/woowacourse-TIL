### 2021-09-09 mvc mission

## 미션 구조
1. JwpApplication main()을 누르면

2. 톰캣 객체 생성 후 8080 포트를 매핑

3. addWebapp(Tomcat)
    - *참고: https://www.mulesoft.com/tcat/tomcat-context*
    - "app/webapp" 하위의 파일들을 Tomcat의 webapp 디렉토리에 추가 후 Context 객체 생성
        - Context Container: single web application running within a given instance of Tomcat
    ```
    9월 09, 2021 10:59:11 오후 org.apache.catalina.core.StandardContext setPath
    경고: A context path must either be an empty string or start with a '/' and do not end with a '/'. The path [/] does not meet these criteria and has been changed to []
    22:59:11.118 [INFO ] [main] [com.techcourse.JwpApplication] - configuring app with basedir: C:\Users\joel6\Desktop\wooteco\level4\mission\jwp-dashboard-mvc\app\webapp
    ```

4. skipBindOnInit(Tomcat)
    - *참고: https://velog.io/@minosmlee/Tomcat-bindOnInit-Socket-listener-binding-%EC%8B%9C%EC%A0%90%EC%9D%84-%EA%B2%B0%EC%A0%95*
    - `bindOnInit=true`: Connector가 생성되는 시점에 Socket listener가 바인딩 됨
    - `bindOnInit=false`: Connector가 생성되고 기동이 완료될 때 Socket listener가 바인딩 됨 (현재 프로젝트)

5. tomcat.start()
    - 톰캣 (서블릿 컨테이너) 시작!
    - getServer()
        - server = new StandardServer(); 를 통해 새로운 서버 할당
        - initBaseDir()
        - ConfigFileLoader.setSource(new CatalinaBaseConfigurationSource(new File(basedir), null));
        - server.addService(new StandardService()); 를 통해 서버에 스탠다드 서비스 할당
    ```
    9월 09, 2021 10:59:27 오후 org.apache.coyote.AbstractProtocol init
    정보: Initializing ProtocolHandler ["http-nio-8080"]
    9월 09, 2021 10:59:27 오후 org.apache.catalina.core.StandardService startInternal
    정보: Starting service [Tomcat]
    9월 09, 2021 10:59:27 오후 org.apache.catalina.core.StandardEngine startInternal
    정보: Starting Servlet engine: [Apache Tomcat/10.0.10]
    9월 09, 2021 10:59:27 오후 org.apache.catalina.startup.ContextConfig getDefaultWebXmlFragment
    정보: No global web.xml found
    ```
    - 카탈리나가 자주 나오네
        - *참고: https://gogoonbuntu.tistory.com/63*
        - 톰캣의 코어 컴포넌트
        - 톰캣 서블릿의 실질적인 구동 제공
        - 해당 동작은 config 파일을 통해 구현/제어 가능
            - catalina.policy: 자바 클래스의 톰캣 보안 정책
            - catalina.properties: 카탈리나 클래스를 위한 표준 자바 프로퍼티
            - logging.properties: 임계값, 로그값의 위치와 같은 카탈리나의 로깅 기능을 구성하는 방법
            - context.xml: 톰캣에 구동되는 웹앱에 대해 로드될 정보
            - server.xml: 톰캣의 메인 config 파일
            - tomcat-users.xml: 톰캣 서버의 많은 유저 정보
        ``` 
        [위키피디아]
        Catalina is Tomcat's servlet container. 
        Catalina implements Sun Microsystems' specifications for servlet and JavaServer Pages (JSP). 
        In Tomcat, a Realm element represents a "database" of usernames, passwords, and roles (similar to Unix groups) assigned to those users. 
        Different implementations of Realm allow Catalina to be integrated into environments where such authentication information is already being created and maintained, 
        and then use that information to implement Container Managed Security as described in the Servlet Specification.
        ```    

6. ServletContainerInitializer를 상속 받은 NextstepServletContainerInitializer 실행
    - onStartup() 으로 받은 Set<Class<?>> webAppInitializerClasses
        - 여기에 AppWebApplicationInitializer가 포함되어 있음
            - 이걸 새로운 인스턴스 생성함
    - 이제 생성한 인스턴스들 `initializer.onStartup(servletContext);` 메서드 호출

7. AppWebApplicationInitializer
    - onStartUp(ServletContext servletContext) 메서드
        - new DispatcherServlet() 생성
        - dispatcherServlet.addHandlerMapping(new ManualHandlerMapping());
            - 여기에 매핑해준 핸들러들이 구식 핸들러들
        - dispatcherServlet을 이제 ServletContext에 추가
            - 해당 디스패쳐는 모든 경로에 대한 요청을 받도록 "/"에 매핑
    ```
    22:59:41.197 [INFO ] [main] [c.t.AppWebApplicationInitializer] - Start AppWebApplication Initializer
    ```

8. Filter 생성 및 등록 후 init 호출
    - `CharacterEncodingFilter`: UTF-8로 요청을 처리할 수 있도록 설정
    - `ResourceFilter`: 정적 파일들을 RequestDispatcher를 통해 요청 처리
        ```
        [RequestDispatcher 설명]
        * Defines an object that receives requests from the client and sends them to
        * any resource (such as a servlet, HTML file, or JSP file) on the server. The
        * servlet container creates the <code>RequestDispatcher</code> object, which is
        * used as a wrapper around a server resource located at a particular path or
        * given by a particular name.
        ```

9. Servlet의 init 호출
    - DispatcherServlet 하나 밖에 없음
    ```java
    @Override
    public void init() {
        handlerMappings.forEach(HandlerMapping::initialize);
    }
    ```
    - 핸들러 매핑 촤르르
        ```
        22:59:44.695 [INFO ] [main] [com.techcourse.ManualHandlerMapping] - Initialized Handler Mapping!
        22:59:44.695 [INFO ] [main] [com.techcourse.ManualHandlerMapping] - Path : /login, Controller : class com.techcourse.controller.LoginController
        22:59:44.695 [INFO ] [main] [com.techcourse.ManualHandlerMapping] - Path : /login/view, Controller : class com.techcourse.controller.LoginViewController
        22:59:44.695 [INFO ] [main] [com.techcourse.ManualHandlerMapping] - Path : /logout, Controller : class com.techcourse.controller.LogoutController
        22:59:44.695 [INFO ] [main] [com.techcourse.ManualHandlerMapping] - Path : /register/view, Controller : class com.techcourse.controller.RegisterViewController
        22:59:44.695 [INFO ] [main] [com.techcourse.ManualHandlerMapping] - Path : /register, Controller : class com.techcourse.controller.RegisterController
        22:59:44.695 [INFO ] [main] [com.techcourse.ManualHandlerMapping] - Path : /, Controller : class nextstep.mvc.controller.asis.ForwardController
        9월 09, 2021 10:59:44 오후 org.apache.coyote.AbstractProtocol start
        정보: Starting ProtocolHandler ["http-nio-8080"]
        ```
