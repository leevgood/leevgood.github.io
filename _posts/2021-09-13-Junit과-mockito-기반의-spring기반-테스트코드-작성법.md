# Junit과-mockito-기반의-spring기반-테스트코드-작성법

<br>

## Mockito 소개 및 사용법
---

<br>

### Mockito란?

<br>

Mockito는 개발자가 동작을 직접 제어 할 수 있는 가짜(Mock) 객체를 지원하는 테스트 프레임 워크이다. 일반적으로 Spring과 같은 웹 어플리케이션을 개발한다고 하면, 여러 객체들 간의 의존성이 존재한다. 이러한 의존성은 단위 테스트 작성에 어려움이 될 수 있는데, 이를 해결하기 위해 가짜 객체를 주입시켜주는 Mockito 라이브러리를 활용 가능하다. Mockito를 활용함으로써 가짜 객체에 원하는 결과를 stub하여 단위테스트 진행이 가능하다. <b>물론 Mock을 하지 않아도 된다면 하지 않는 것이 더욱 좋다. </b>

<br>

### Mockito 사용법

<br>

1. Mock 객체 의존성 주입
<br>
Mockito에서 Mock 객체의 의존성 주입을 위해서는 크게 3가지 어노테이션이 사용된다.

- @Mock : Mock 객체를 만들어 반환해주는 어노테이션
- @Spy : Stub하지 않은 메소드들은 원본 메소드 그대로 사용하는 어노테이션
- @InjectoMocks : @Mock 또는 @Spy로 생성된 가짜 객체를 자동으로 주입시켜주는 어노테이션

예를 들어 UserController에 대한 단위 테스트를 작성하고자 할 때, UserService를 사용하고 있다면 @Mock 어노테이션을 통해 가짜 UserService를 만들고 @injectMocks를 통해 UserController에 이를 주입시킬 수 있다.

<br>

2. Stub

<br>

앞서 설명했듯이, 의존성이 있는 객체는 가짜 객체(Mock Object)를 주입하여 어떤 결과를 반환하라고 정해진 답변을 준비시켜야 한다. Mockito에서는 다음과 같은 Stub을 제공한다.
<br>
  - doReturn() : Mock 객체가 특정한 값을 반환해야 하는 경우
  - doNothing() : Mock 객체가 아무것도 반환하지 않는 경우(void)
  - doThrow() : Mock 객체가 예외를 발생시키는 경우

<br>

예를 들어 UserService의 findAllUser() 호출 시에 빈 ArrayList를 반환해야 한다면 다음과 같이 doReturn()을 사용 할 수 있다.

<br>

```java
doReturn(new ArrayList()).when(userService).findAllUseR();
```
<br>

3. Mockito와 Junit의 결합
Mockito도 테스팅 프레임워크이기 때문에 JUnit과 결합되기 위해서는 별도의 작업이 필요하다. 기존의 JUnit4에서 Mockito를 활용하기 위해서는 클래스 어노테이션으로 @RunWith(MockitoJUnitRunner.class)를 붙여주어야 연동이 가능했다. 하지만 SpringBoot 2.2.0부터 공식적으로 JUnit5를 지원함에 따라, 이제부터는 @ExtendWith(MockitoExtension.class)를 사용해야 결합이 가능하다.

<br>

## 2. Spring API 단위 테스트(Unit Test) 작성 예시
---

<br>

### 사용자 회원가입/목록 조회 API Java코드
<br>
예를 들어 다음과 같은 회원가입 API와 목록조회 API가 있다고 하자.
<br>


```java
@RequiredArgsConstructor
    @RestController
    @RequestMapping(value = "/user")
    public class UserController {
        private final UserService userService;

        @PostMapping(value = "/signUp")
        public ResponseEntity<String> signUp(
            @RequestBody final SignUpDTO signUpDTO) {

            return userService.isEmailExists(
                signUpDTO.getEmail()) ?
                ResponseEntity.badRequest().build() :
                ResponseEntity.ok(TokenUtils
                .generateJwtToken(userService.signUp(signUpDTO)));
        }

        @GetMapping(value = "/list")
        public ResponseEntity<UserListResponseDTO> findAll() {
            final UserListResponseDTO userListResponseDTO = 
            UserListResponseDTO.builder()
            .userList(userService.findAll()).build();
            
            return ResponseEntity.ok(userListResponseDTO);
        }
    }
```
<br>

우리는 위와 같은 UserController에 대한 단위 테스트 코드를 작성해주어야 한다.

<br>

### 단위 테스트 작성 준비

<br>

앞서 설명했듯이 JUnit5와 Mockito를 연동하기 위해서는 @ExtendWith(MockitoExtension.class)를 사용해야 한다. 이를 클래스 어노테이션으로 붙여 클래스를 다음과 같이 작성할 수 있다.

<br>

```java
@ExtendWith(MockitoExtension.class) class UserControllerTest { }
```

<br>

그리고 이제 의존성 주입을 처리해주어야 한다. 우선 UserController에서 UserService를 사용하고 있으므로 @Mock 어노테이션을 통해 UserService에 가짜 Mock 객체를 주입해주어야 한다. 그리고 테스트하고자 하는 UserController에 UserService를 주입시켜야 하는데, 이를 위해 @InjectMock를 붙여주어야 한다.

<br>

```java
    @ExtendWith(MockitoExtension.class)
    class UserControllerTest {
        @InjectMocks
        private UserController userController;
        @Mock
        private UserService userService;
    }
```

<br>

그리고 API의 경우 함수 실행을 위해 메소드가 아닌 API가 호출되므로 우리의 API 요청을 받아 전달하기 위한 별도의 객체가 필요하다. Spring Test에서는 이를 위해 MockMVC를 지원하고 있는데, 다음과 같이 초기화를 하면 된다.

<br>

```java
    @ExtendWith(MockitoExtension.class)
    class UserControllerTest {
        @InjectMocks
        private UserController userController;
        @Mock
        private UserService userService;
        private MockMvc mockMvc;

        @BeforeEach
        public void init() {
            mockMvc = 
            MockMvcBuilders.standaloneSetup(userController).build();
        }
    }

```

<br>

그러면 이제 UserController에 대한 API를 받아 넘겨줄 수 있는 MockMvc까지 준비가 되었으므로, 다음의 케이스들에 대해 테스트 코드를 작성해주자.

<br>

1. 회원가입 성공
2. 이메일이 중복되어 회원가입 실패
3. 사용자 목록 조회
   
<br>

### 1. 회원가입 성공

<br>

우선 회원가입 요청을 보내기 위해서는 SighUpDTO 객체 1개와 userService의 isEmailDuplicated와 signUp에 대한 stub이 필요하다. 이러한 준비 작업을 해주면 given 단계에 다음과 같은 테스트 코드가 작성된다.

<br>

```java
@DisplayName("회원 가입 성공")
    @Test
    void signUpSuccess() throws Exception {
        //final
        final SignUpDTO signUpDTO = signUpDTO();
        doReturn(false).when(userService)
        .isEmailDuplicated(signUpDTO.getEmail());
        
        doReturn(new User("a", "b", UserRole.ROLE_USER))
        .when(userService)
        .signUp(any(SignUpDTO.class));

    }

    private SignUpDTO signUpDTO() {

        final SignUpDTO signUpDTO = new SignUpDTO();

        signUpDTO.setEmail("test@test.test");

        signUpDTO.setPw("test");

        return signUpDTO;
    }
```

<br>

여기서 UserService의 signUp 함수에 대한 매개변수로 우리가 만든 signUpDTO가 아닌 어떠한 변수도 처리함을 뜻하는 any()가 사용됨에 주의해야 한다.<br><br> Spring에서 HTTP Body로 전달된 데이터는 MessageConverter에 의해 새로운 객체로 변환된다. 그런데 이것은 요청이 오면 Spring에서 변환을 하는 것이므로, 우리가 API로 전달되는 파라미터 SignUpDTO를 조작할 수 없다. 그렇기 때문에 어떠한 객체도 처리할 수 있도록 any()가 사용되었다. 추가로 signUpDTO는 다른 코드에서도 사용되므로 private 함수로 공통화 시켰다.<br><br>
그 다음 when 단계를 작성 해주어야 한다. 이때 mockMVC에 데이터와 함께 POST요청을 보내야 한다. 보내는 데이터는 객체가 아닌 JSON이어야 하므로 별도의 변환이 필요한데 Gson을 활용하였다. 

<br>

```java
    @DisplayName("회원 가입 성공")
    @Test
    void signUpSuccess() throws Exception {
        
        //given
        final SignUpDTO signUpDTO = signUpDTO();

        doReturn(false)
        .when(userService)
        .isEmailDuplicated(signUpDTO.getEmail());

        doReturn(new User("a", "b", UserRole.ROLE_USER))
        .when(userService)
        .signUp(any());

        //when
        final ResultActions resultActions = mockMvc
        .perform(MockMvcRequestBuilders
        .post("/user/signUp")
        .contentType(MediaType.APPLICATION_JSON)
        .content(new Gson().toJson(signUpDTO)));
    }
```

<br>

mockMvc의 perform에 요청에 대한 정보를 작성하여 넘겨주어야 한다. 요청 정보를 작성하기 위해서는 MockMvcRequestBuilders를 사용해야 하며 요청 메소드 종류, 내용, 파라미터 등을 설정할 수 있다.

<br>

마지막으로 호출된 결과를 검증하는 then 단계를 작성해주어야 한다. 회원가입 API 호출 결과로 200 Response와 JWT 토큰을 발급받고 있는데, 다음과 같이 이를 검증할 수 있다.

<br>

```java
    @DisplayName("회원 가입 성공")
    @Test
    void signUpSuccess() throws Exception {
        
        //given
        final SignUpDTO signUpDTO = signUpDTO();
        doReturn(false)
        .when(userService)
        .isEmailDuplicated(signUpDTO.getEmail());
        
        doReturn(new User("a", "b", UserRole.ROLE_USER))
        .when(userService).signUp(any());

        //when
        final ResultActions resultActions =
        mockMvc.perform(MockMvcRequestBuilders.post("/user/signUp")
        .contentType(MediaType.APPLICATION_JSON)
        .content(new Gson().toJson(signUpDTO)));

        //then
        final MvcResult mvcResult = 
        resultActions
        .andExpect(status().isOk()).andReturn();
        
        final String token = mvcResult.getResponse().getContentAsString();
        assertThat(token).isNotNull();

    }
```

<br>

Spring에서 HTTP Body로 전달된 데이터는 MessageConverter에 의해 새로운 객체로 변환된다. 그런데 이것은 요청이 오면 Spring에서 변환을 하는 것이므로, 우리가 API로 전달되는 파라미터 SignUpDTO를 조작할 수 없다. 그렇기 때문에 SignUpDTO 클래스의 어떠한 객체도 처리할 수 있도록 any()가 사용되었다. signUpDTO는 다른 코드에서도 사용되므로 private 함수로 공통화시켰다.

<br>

### 2. 이메일 중복으로 회원가입 실패

<br>

이메일이 중복되어 회원가입에 실패한 경우에는 우선 요청을 위한 SignUpDTO가 필요하며, isEmailDuplicated의 결과로 true가 반환되도록 given 단계를 변경해주어야 한다. 또한 then 단계에서는 Response Status가 BadRequest인지 확인해도록 변경해야 한다.

<br>

```java
@DisplayName("이메일이 중복되어 회원 가입 실패")
@Test void
signUpFailByDuplicatedEmail() throws Exception {
        
        //given
        final SignUpDTO signUpDTO = signUpDTO();
        
        doReturn(true)
        .when(userService)
        .isEmailDuplicated(signUpDTO.getEmail());

        //when
        final ResultActions resultActions =
        mockMvc
        .perform(MockMvcRequestBuilders
        .post("/user/signUp")
        .contentType(MediaType.APPLICATION_JSON)
        .content(new Gson().toJson(signUpDTO)));
        
        //then
        resultActions.andExpect(status().isBadRequest());
    }
```

<br>

추가로 이메일이 중복된 경우에는 UserService의 signUp 메소드가 호출되지 않는다. 그렇기 때문에 SignUp에 대한 Stub은 불필요해졌으므로 제거해주어야 한다. (테스트는 Stub에 대해 엄격하기 때문에 불필요한 Stub이 있으면 테스트가 실패한다.)

<br>

### 3. 사용자 목록 조회

<br>

사용자 목록 조회의 given 단계에서는 UserService의 findAll에 대한 Stub이 필요하다. 그리고 when단계에서는 호출하는 HTTP 메소드를 GET으로, URL을 "/user/list"로 작성해주어야 한다. 그리고 마지막으로 then 단계에서는 HTTP Status가 OK이며, 주어진 Json 데이터를 객체로 변환하여 확인해보아야 한다.

<br>

```java

    @DisplayName("사용자 목록 조회")
    @Test
    void getUserList() throws Exception {
        //given
        doReturn(userList()).when(userService).findAll();

        //when
        final ResultActions resultActions =
        mockMvc.perform(MockMvcRequestBuilders.get("/user/list"));

        //then

        final MvcResult mvcResult =
        resultActions.andExpect(status().isOk()).andReturn();
        
        final UserListResponseDTO response =
        new Gson().
        fromJson(mvcResult.getResponse()
        .getContentAsString()
        , UserListResponseDTO.class);
        
        assertThat(response.getUserList().size()).isEqualTo(5);


    }

        private List<User> userList() {
        final List<User> userList = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            userList.add(new User("test@test.test"
            , "test", UserRole.ROLE_USER));
        }
        return userList;
    }


```

<br>

## 3. Spring 비지니스 로직 단위 테스트 작성 예시
---
<br>

### 사용자 회원가입/목록 조회 비지니스 로직 코드

<br>

사용자 회원가입과 목록 조회를 위해서는 다음과 같은 비지니스 로직 레이어(Service Layer)가 필요하다.

<br>

```java
    @RequiredArgsConstructor
    @Service
    public class UserServiceImpl implements UserService {
        private final UserRepository userRepository;
        private final BCryptPasswordEncoder passwordEncoder;

        @Override
        public User signUp(final SignUpDTO signUpDTO) {
            final User user =
            User.builder()
            .email(signUpDTO.getEmail())
            .pw(passwordEncoder
            .encode(signUpDTO.getPw()))
            .role(UserRole.ROLE_USER).build();
            return userRepository.save(user);
        }

        @Override
        public boolean isEmailDuplicated(final String email) {
            return userRepository.existsByEmail(email);
        }

        @Override
        public List<User> findAll() {
            return userRepository.findAll();
        }
    }
```

<br>

이번에는 다음과 같은 테스트 코드를 작성해보자.

<br>

1. 회원가입 성공
2. 이메일이 중복 여부
3. 사용자 목록 조회

<br>

### 단위 테스트 작성 준비

<br>

앞서 설명하였듯 @ExtendWith(MockitoExtension.class)와 가짜 객체 주입을 사용해 다음과 같은 테스트 클래스를 작성할 수 있다.

<br>

```java
    @ExtendWith(MockitoExtension.class)
    class UserServiceTest {
        @InjectMocks
        private UserService userService;
        @Mock
        private UserRepository userRepository;
        @Spy
        private BCryptPasswordEncoder passwordEncoder;
    }
```

<br>

그리고 이번에는 BCryptPasswordEncoder에 @Spy가 사용되었다. 앞서 설명하였듯 Spy는 Mock되지 않은 메소드는 실제 메소드로 동작하는 어노테이션이라고 하였다. 위의 예제에서 우리는 실제로 사용자 비밀번호를 암호화해야 하므로, @Spy를 사용해주었다.

<br>

### 1. 회원가입 성공

<br>

```java
    @DisplayName("회원 가입")
    @Test
    void signUp() {
        //given
        final BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        final SignUpDTO signUpDTO = signUpDTO();
        final String encryptedPw = encoder.encode(signUpDTO.getPw());

        //when
        doReturn(new User(signUpDTO.getEmail(),
        encryptedPw, UserRole.ROLE_USER))
        .when(userRepository)
        .save(any(User.class));
        
        final User user = userService.signUp(signUpDTO);

        //then
        assertThat(user.getEmail()).isEqualTo(signUpDTO.getEmail()); 
        assertThat(encoder.matches(signUpDTO.getPw(), user.getPw()))
        .isTrue();

        //verify
        verify(userRepository, times(1)).save(any(User.class));
        verify(passwordEncoder, times(1)).encode(any(String.class));

    }
```
<br>

이번 테스트 코드에서는 추가적으로 mockito의 verify()를 사용해보았다. verify는 Mock된 객체의 해당 메소드가 몇 번 호출되었는지를 검증하는데 도와준다. 위의 예제에서는 passwordEncoder의 encode 메소드와 userRepository의 save 메소드가 각각 1번씩만 호출되었는지를 검증하기 위해 사용되었다.

<br>

### 2. 이메일 중복 여부

```java
    @DisplayName("이메일 중복 여부")
    @Test
    void isEmailDuplicated() {
        //given
        final SignUpDTO signUpDTO = signUpDTO();
        
        doReturn(true).when(userRepository).
        existsByEmail(signUpDTO.getEmail());

        //when
        final boolean isDuplicated = 
        userService.isEmailDuplicated(signUpDTO.getEmail());

        //then
        assertThat(isDuplicated).isTrue();
    }
```

<br>

### 3. 사용자 목록 조회

<br>

```java

    @DisplayName("사용자 목록 조회")
    @Test
    void findAll() {
        //given
        doReturn(userList()).when(userRepository).findAll();

        //when
        final List<User> userList = userService.findAll();

        //then
        assertThat(userList.size()).isEqualTo(5);
    }

    private List<User> userList() {
        final List<User> userList = new ArrayList<>();
        
        for (int i = 0; i < 5; i++) {
            userList
            .add(new User("test@test.test",
            "test", UserRole.ROLE_USER));
        }

        return userList;
    }

```

<br>

지금까지 Spring 기반의 애플리케이션 코드에 대해 단위 테스를 작성하는 방법을 알아보았다.


<br>

### 출처

https://mangkyu.tistory.com/145