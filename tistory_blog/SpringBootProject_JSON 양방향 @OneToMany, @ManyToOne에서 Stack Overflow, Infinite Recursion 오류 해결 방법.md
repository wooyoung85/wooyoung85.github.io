## spring data jpa infinite recursion

#### Question과 Answer 클래스 간 1대 다 관계

~~~java
@Entity
@Data
public class Question {
...
@OneToMany(mappedBy="question", fetch = FetchType.LAZY)
@OrderBy("id ASC")
@JsonIgnore
private List<Answer> answers;
...
}
~~~
----
~~~java
@Entity
@Data
public class Answer {
...
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(foreignKey = @ForeignKey(name = "fk_answer_to_question"))
private Question question;
...
}
~~~

#### 게시판에 대한 RestController create 메소드 구현 시 무한으로 Question과 Answer 객체를 호출해서 Stack Overflow 발생

~~~java
@RestController
@RequestMapping("/api/question/{questionId}")
public class ApiAnswerController {
    @Autowired
    QuestionService questionService;
    @Autowired
    AnswerService answerService;

    @PostMapping("/answer")
    public Answer create(@PathVariable Long questionId, String contents, HttpSession session) {
        User loginUser = HttpSessionUtils.getUserFromSession(session);
        Question question = questionService.findOne(questionId);
        Answer answer = new Answer(loginUser, question, contents);
        return answerService.create(answer);
    }
}
~~~

#### JPA, JSON 양방향 @OneToMany, @ManyToOne에서 Stack Overflow, Infinite Recursion 오류 해결 방법
- Jackson 1.6+ 에서 @JsonManagedReference, @JsonBackReference 사용
- Jackson 1.9+ 에서 @JsonIgnore 사용
- Jackson 2.0+ 에서 @JsonIdentityInfo 사용