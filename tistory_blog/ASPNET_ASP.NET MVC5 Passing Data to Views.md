ASP.NET MVC5에서 View로 데이터를 전달하는 방법에 대해서 정리해 보았다.  
- ViewData
- ViewBag
- ViewModel

# ViewData

- ViewData는 dictionary 타입인 ViewDataDictionary로부터 파생됨
- ViewData는 current request 동안에만 유효함. redirection 이 발생하면 초기화됨
- <span style="color:red">ViewData는 사용하기 전에 type cast가 필수</span>

    ##### StudentController.cs 
    ```csharp
    public ActionResult Index()
    {
        IList<Student> studentList = new List<Student>();
        studentList.Add(new Student(){ StudentName = "Jiwoo" });
        studentList.Add(new Student(){ StudentName = "Yoono" });

        ViewData["students"] = studentList;

        return View();
    }
    ```
    ##### index.cshtml
    ```csharp    
    <ul>
    @foreach (var std in ViewData["students"] as IList<Student>)
    {
        <li>
            @std.StudentName
        </li>
    }
    </ul>
    ```

# ViewBag

- ViewBag C# 4.0부터 지원하는 dynamic features를 활용하는 동적 속성
- ViewBag은 내부적으로 ViewData에 저장하기 때문에 ViewData의 key 값과 ViewBag의 속성 값이 같으면 안됨 (ViewBag 실제로 ViewData를 감싸는 wrapper)
    ```csharp
    // 이렇게 같이 쓰면 안됨
    ViewData["TotalStudents"] = 10;
    ViewBag.TotalStudents = 20;
    ```
- ViewData와 동일하게 current request 동안에만 유효하고 redirection이 발생하면 초기화 됨. 
- <span style="color:red">사용하기 전에 type cast 필수</span>
- ~~왜 만들었는지 모르겠다;;~~

    ##### ViewBag Snippet 
    ```csharp
    ViewBag.students = studentList;
    ```

# ViewModel
- ViewData와 ViewBag은 Compile시 안전하지 않기 때문에(TypeSafe 하지 않음) ViewModel을 사용하여 View로 Data를 전달하는 것이 바람직 함
- ViewModel은 특별히 View를 위해 만들어진 Model클래스
- **Convention** : ViewModels라는 폴더 밑에 `~ViewModel` 로 끝나도록 Naming 해야 함
- 아래와 같이 2개 이상의 클래스의 정보를 ViewModel에 담아서 View로 전달 할 수 있음

    ##### StudentInfoViewModel.cs
    ```csharp
    public class StudentInfoViewModel  
    {  
        public Student Student { get; set; }  
        public List<Subject> Subjects { get; set; }  
    }
    ```
    ##### StudentController.cshtml
    ```csharp
    public ActionResult Index()  
    {  
        var student = new Student  
        {  
            StudentName = "Wooyoung"  
        };  
    
        var subjects = new List<Subject>  
        {  
            new Subject { Id = 1, SubjectName = "Math" },  
            new Subject { Id = 5, SubjectName = "Science" }  
        };  
    
        var viewModel = new StudentInfoViewModel  
        {  
            Student = student,  
            Subjects = subjects  
        };  
    
        return View(viewModel);  
    } 
    ```
    ##### index.cshtml
    ```csharp
    @model SCMS.ViewModels.StudentInfoViewModel   
    
    <h2>@Model.Student.Name</h2>  
    <h3>Subjects</h3>  
    <ul>  
        @foreach (var subject in Model.Subjects)  
        {  
            <li>@subject.SubjectName</li>  
        }  
    </ul> 
    ```