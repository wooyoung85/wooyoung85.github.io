---
layout: post
title: "ASP.NET Core Web API Project 만들기"
description: "ASP.NET Web API with ASP.NET Core MVC "
date: 2017-06-09
comments: true
share: true

sitemap :
  changefreq : weekly
  priority : 1.0
---
마이크로 서비스 구축 실습을 목표로 이것 저것 하나씩 연습해 보는 중입니다^^


**마이크로 서비스 실습 에정 순서**
>1. ASP.NET Core Web API Project 만들기
>2. Docker 배포
>3. MongoDB와 Web API 연결
>4. 의존성 주입(DI)과 보안을 고려해서 Web API 다시 만들기
>5. UI만들기(Node JS나 Angular JS를 활용해서 만들 예정)

<br>

일단 이번 포스팅에서는 ASP.NET Web API 프로젝트를 만들어 보도록 하겠습니다.

## Project 생성
###### 저는 Visual Studio 2017 Community 버전을 사용하고 있습니다.
[마이크로소프트에서 제공하고 있는 ASP.NET Tutorial](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-web-api)을 바탕으로 Web API프로젝트를 구성하겠습니다.

![image_1](/images/post_2/1.png)
- 파일 > 새로만들기 > 프로젝트
- 템플릿 중 ASP.NET Core 웹 응용 프로그램 (.NET Core) 선택

![image_2](/images/post_2/2.png)
- ASP.NET Core 버전은 1.1 선택
- Web API 선택

![image_3](/images/post_2/3.png)
- 솔루션 탐색기에 MVC형태 비슷하게(Models 폴더 추가가 필요함) 프로젝트 구성이 되어 있는 것을 확인 후 `Ctrl + Shift + B` 솔루션 전체 빌드
- 솔루션 밑에 생성된 프로젝트를 시작 프로젝트로 설정 후 `Ctrl + F5`로 디버깅 없이 시작

![image_4](/images/post_2/4.png)
- IIS Express를 통해 Web API가 실행되고, 작업을 따로 하지 않았기 때문에 위와 같이 나오게 됨

## InMemory DB 사용하기
Web API 개발에 집중하기 위해서 DB와 연결하는 작업은 일단 Pass하고 EntityFramework에서 사용할 수 있는 InMemory DB를 사용하도록 하겠습니다.

![image_5](/images/post_2/5.png)
- 도구 > NuGet 패키지 관리자 > 패키지 관리자 콘솔 에서 아래 명령어로 Install<br>

<code>PM>  Install-Package Microsoft.EntityFrameworkCore.InMemory</code>

![image_6](/images/post_2/6.png)
- 설치된 DB는 .csproj(프로젝트 파일)이나 Nuget 에서 확인 가능

## Model 추가
![image_7](/images/post_2/7.png)
- 프로젝트에서 우클릭 후 추가 > 새 폴더 (폴더명 : *Models*)

![image_8](/images/post_2/8.png)
- Models 폴더에서 우클릭 후 추가 > 클래스 추가
- 아래 코드 입력

~~~cs
namespace TodoApi.Models
{
    public class TodoItem
    {
        public long Id { get; set; }
        public string Name { get; set; }
        public bool IsComplete { get; set; }
    }
}
~~~

![image_9](/images/post_2/9.png)
- Models 폴더에서 우클릭 후 추가 > 클래스 추가
- 아래 코드 입력

~~~cs
using Microsoft.EntityFrameworkCore;

namespace TodoApi.Models
{
    public class TodoContext : DbContext
    {
        public TodoContext(DbContextOptions<TodoContext> options)
            : base(options)
        {
        }

        public DbSet<TodoItem> TodoItems { get; set; }

    }
}
~~~
- InMemory DB 사용을 위해 Microsoft.EntityFrameworkCore.DbContext를 상속 받는 Database Context 클래스 생성

## Database Context 등록

~~~cs
using Microsoft.AspNetCore.Builder;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using TodoApi.Models;

namespace TodoApi
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddDbContext<TodoContext>(opt => opt.UseInMemoryDatabase());
            services.AddMvc();
        }

        public void Configure(IApplicationBuilder app)
        {
            app.UseMvc();
        }
    }
}
~~~
- Startup.cs에 있는 ConfigureServices Method에 Database Context(TodoContext) 등록하기<br>
  <sup>Controller에서 생성자 주입 방식을 사용하여 Dependency Injection을 구현하게 되는데 이를 위해 등록해 놓는 코드</sup>
- ASP.NET Core는 기본적으로 Dependency Injection을 지원하고 활용하도록 설계됨<br>
  <sup>(이 부분은 나중에 자세히 포스팅 하겠습니다^^)</sup>

## 컨트롤러 추가
![image_9](/images/post_2/9.png)
- Controllers 폴더에서 우클릭 후 추가 > 새 항목 > MVC 컨트롤러 클래스 추가
- 아래 코드 입력

~~~cs
using System.Collections.Generic;
using Microsoft.AspNetCore.Mvc;
using TodoApi.Models;
using System.Linq;

namespace TodoApi.Controllers
{
    [Route("api/[controller]")]
    public class TodoController : Controller
    {
        private readonly TodoContext _context;

        public TodoController(TodoContext context)
        {
            _context = context;

            if (_context.TodoItems.Count() == 0)
            {
                _context.TodoItems.Add(new TodoItem { Name = "Item1" });
                _context.SaveChanges();
            }
        }
    }
}
~~~
- TodoController 클래스 생성자에 TodoContext를 주입하는 의존성 주입 기법 사용하였고,
DB에 데이터가 없을 때 Name이 "Item1"인 데이터를 생성하게 함
- 전역 변수로 생성한 Database Context(TodoContext)는 컨트롤러에 CRUD를 구현할 때 사용함


## 컨트롤러에 CRUD 구현

TodoController 클래스에 아래 Method들을 추가
#### Create
~~~cs
[HttpPost]
public IActionResult Create([FromBody] TodoItem item)
{
    if (item == null)
    {
        return BadRequest();
    }

    _context.TodoItems.Add(item);
    _context.SaveChanges();

    return CreatedAtRoute("GetTodo", new { id = item.Id }, item);
}
~~~
#### Read
~~~cs
[HttpGet]
public IEnumerable<TodoItem> GetAll()
{
    return _context.TodoItems.ToList();
}

[HttpGet("{id}", Name = "GetTodo")]
public IActionResult GetById(long id)
{
    var item = _context.TodoItems.FirstOrDefault(t => t.Id == id);
    if (item == null)
    {
        return NotFound();
    }
    return new ObjectResult(item);
}
~~~

#### Update
~~~cs
[HttpPut("{id}")]
public IActionResult Update(long id, [FromBody] TodoItem item)
{
    if (item == null || item.Id != id)
    {
        return BadRequest();
    }

    var todo = _context.TodoItems.FirstOrDefault(t => t.Id == id);
    if (todo == null)
    {
        return NotFound();
    }

    todo.IsComplete = item.IsComplete;
    todo.Name = item.Name;

    _context.TodoItems.Update(todo);
    _context.SaveChanges();
    return new NoContentResult();
}
~~~
#### Delete
```cs
[HttpDelete("{id}")]
public IActionResult Delete(long id)
{
    var todo = _context.TodoItems.First(t => t.Id == id);
    if (todo == null)
    {
        return NotFound();
    }

    _context.TodoItems.Remove(todo);
    _context.SaveChanges();
    return new NoContentResult();
}
```

## PostMan을 사용하여 테스트
> `Ctrl + F5`로 Web API 실행하면 웹 브라우저에서 <u>http://localhost:port/api/values</u>와 같은 주소로 이동합니다.<br>
<sup>참고 : Controller 생성 시 Routing 속성(`[Route("api/[controller]")]`)을 넣어줬기 때문에 http://localhost:port/api/values 와 같은 형태로 실행됨</sup>

### Create
![image_10](/images/post_2/10.png)
- <mark>POST</mark> 방식으로 설정
- 화면 상단에서 Body --> raw 라디오 버튼을 클릭 후 JSON(application/json)을 선택
- 아래 코드 입력
~~~ json
{
    "name":"item2",
    "isComplete":true
}
~~~
- 생성된 데이터를 반환

### Read
![image_11](/images/post_2/11.png)
- <mark>GET</mark> 방식으로 설정
- 아이디 값을 파라미터로 넘겨주는 방식은 http://localhost:port/api/todo/<b>key</b>

### Update
![image_12](/images/post_2/12.png)
- <mark>PUT</mark> 방식으로 설정
- URL은 http://localhost:port/api/todo/<b>key</b> 아이디 값을 파라미터로 넘겨주는 방식으로 입력
- 아래 코드 입력
~~~ json
{
    "id": 1,
    "name": "UpdateItem1",
    "isComplete": true
}
~~~

### Delete
![image_13](/images/post_2/13.png)
- <mark>DELETE</mark> 방식으로 설정
- URL은 http://localhost:port/api/todo/**key** 아이디 값을 파라미터로 넘겨주는 방식으로 입력<br>
<br>

---

## GitHub 저장소에 소스코드 저장하기
![image_14](/images/post_2/14.png)
- 도구 > 확장 및 업데이트 에서 Online 항목 중 GitHub를 검색해서 GitHub Extension for Visual Studio를 설치합니다.

![image_15](/images/post_2/15.png)
- 설치가 완료되면 팀 탐색기에 GitHub 메뉴박스가 보이게 됩니다
- GitHub 메뉴박스에서 Connet 버튼을 Click

![image_16](/images/post_2/16.png)
- GitHub 로그인

![image_17](/images/post_2/17.png)
- 로그인이 되면 Create 버튼 Click (저장소 만들기)

![image_18](/images/post_2/18.png)
- 저장소 이름, 설명, Local PC에 소스 저장한 저장한 경로, gitignore(GitHub를 통해 관리 안 해도 되는 파일 설정) , License 등을 입력 후 Create

---
**참고한 사이트** <br>
[Create a web API with ASP.NET Core MVC and Visual Studio for Windows](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-web-api)<br>
[Introduction to Dependency Injection in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)<br>
[Setting up and using GitHub in Visual Studio 2017](https://blogs.msdn.microsoft.com/benjaminperkins/2017/04/04/setting-up-and-using-github-in-visual-studio-2017/)


