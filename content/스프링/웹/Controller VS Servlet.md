# Servlet

- Java EE 에서 제공한다.
- `HttpServlet` 을 상속하여 만들 수 있다.
- 저수준이다.

```java
@WebServlet(urlPatterns = "/servlet")
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(final HttpServletRequest req, final HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/plain");
        resp.setCharacterEncoding("UTF-8");
        resp.getWriter().write("<h1>Hello Servlet</h1>");
    }
}
```

# Controller

- Spring Web MVC 에서 제공한다.
- 일반 클래스에 `@Controller` 를 붙여서 만든다.
- 추상화 되어있는 고수준 모듈이다.
- `DispatcherServlet` 이라는 `Servlet`을 하나 두고, 해당 서블릿에 여러 개의 `Controller`가 연결되는 형태이다.

```java
@Controller
public class MyController {

    @GetMapping("/hello")
    @ResponseBody
    public String hello() {
        return "Hello World!";
    }
}
```