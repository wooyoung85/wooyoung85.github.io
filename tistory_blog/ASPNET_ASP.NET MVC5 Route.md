## Route

> A route is a URL pattern that is mapped to a handler.  
The handler can be a physical file, such as an .aspx file in a Web Forms application.  
A handler can also be a class that processes the request, such as a controller in an MVC application.  
To define a route, you create an instance of the Route class by specifying the URL pattern, the handler, and optionally a name for the route.  
<br />
reference : [MSDN - ASP.NET Routing](https://msdn.microsoft.com/en-us/library/cc668201.aspx#routes)

- Route : Handler와 매핑되는 URL 패턴
- WebForm형식의 어플리케이션에서는 `.aspx` 와 같은 파일이 핸들러가 될 수 있고, MVC 형식의 어플리케이션에서는 요청을 처리하는 `Controller` 같은 클래스가 핸들러가 될 수 있다.
- Route를 정의하기 위해 URL 패턴, Handler, Route 이름을 특정하는 Route 클래스 객체를 만들 수 있다.

## ASP.NET MVC5 Route

### **Convention-based Routes**

- `App_Start > RouteConfig.cs` 파일에 라우팅 규칙 추가
    ```c#
    routes.MapRoute(
        “MoviesByReleaseDate”,              // 이름
        “movies/released/{year}/{month}”,   // URL 패턴
        new { controller = “Movies”, action = “MoviesReleaseByDate” },  // 기본 경로 값
        new { year = @“\d{4}, month = @“\d{2}” }   // URL 파라미터 데이터 형식 제한
    );
    ```
- `rotes.MapRoute()` 는 순서대로 규칙이 적용됨
- specific한 규칙을 먼저 추가하고 default가 되는 규칙을 나중에 추가하면 된다.
- **URL 파라미터의 데이터 형식을 제한할 경우 정해놓은 규칙대로 URL 요청이 오지 않을 경우 404(Not Found) 에러가 발생한다**

### **Attribute Routes**

- 모든 라우팅 규칙을 `App_Start > RouteConfig.cs` 파일에 추가하면 가독성이 매우 떨어질 수 있기 때문에 아래와 같이 ActionResult 메서드에 어노테이션을 붙여서 표현할 수 있다.
    ```c#
    [Route(“movies/released/{year}/{month}”)
    public ActionResult MoviesByReleaseDate(int year, int month)
    {
        ...
    }
    ```
- colon(:)을 사용하여 데이터 형식을 제한할 수 있다.  
    `month:regex(\\d{2}):range(1,12)`
- 반드시 `App_Start > RouteConfig.cs` 에 아래와 같이 추가해서 ActionResult 메서드에 선언한 걸 사용하겠다고 알려줘야 함
    ```c#
    rotes.MapMvcAttributeRoutes();
    ```
