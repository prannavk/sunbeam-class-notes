# Class Notes

### Revision of concepts taught on 5th and 6th December

#### @ModelAttribute

- View --> Request params --> Controller

  - FrontController accepts req params and fill it in an object and send as an argument to the request handler method.
  - example 1:

  ```Java
  @RequestMapping("/login")
  String validate(Credentials cred, Model model){
      model.addAttribute("message", "login successful");
  }
  ```

  - example 2:

  ```Java
  @RequestMapping("/login")
  String validate(@ModelAttribute("cr") Credentials cred, Model model){
      model.addAttribute("message", "login successful");
  }
  ```

- Agar aapko ek hi model attribute bhejna hai, we can have ModelAndView as return type, and...
  ```Java
  @RequestMapping("/url")
  ModelAndView method(){
      Pojo obj = new Pojo();
      return new ModelANdView("viewName", "command", obj);
  }
  ```
- Note, here:
  ```Java
      return new ModelANdView("viewName", "command", obj);
  ```
  - is equivalent to:
  ```Java
      model.addAttribute("command", obj);
      return "viewName"
  ```

#### Spring from tag

- A spring form - must have a spring form backing bean/pojo to initialize the form -- given as attribute
  ```JSP
  <sf: form modelAttribute="beanName" action="...">
  ```
  - In this case, the controller must send a bean/pojo with the name as "beanName" in request/model scope
  - There are several ways a controller can send this: (Controller can send this using)
    - model.addAttribute("command", obj);
    - @ModelAttribute Pojo obj -- in argument or return value
    - return ModelView from req handler method

## Today's Agenda

- [ ] MVC Validation
- [ ] Session scoped Beans
- [ ] REST Services
- [ ] Image Upload/Download

<sub>_Android Mobiles App karke rakhlo, kyun ki sir said, it will be used to connect with our backend application which we will be building_</sub>

#### Note

- Spring REST is a subset of Spring MVC

### Client Side Validations:

- Client side validations are faster and gives better user experience
- Client side validations can be compromized by the end user
- Hence we might go for Server side validations
- It uses standard annotations for data validations (JSR 303 - Java Specification Request) [Standard Annotations hai Spring ke nahi]
  - @NotBlank -- shouldnt be empty (primitives and Strings)
  - @NotEmpty -- shouldnt be empty - used with Collections
  - @NotNull
  - @Min and @Max
  - @size -- string length
    -@DateTimeFormat -- jis pojo object mein aap date collect karne vale ho, vahaan pe ye mention karna
- To process validations use @Valid on command/pojo object -- req handler method arg

#### Note:

- Pojo ke andar, field ke upar @NotBlank lagane ke pehle, we need a started validator
- Add this dependency (copy this) (add new dependency/starter in existing project)

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

- and then inside the POJO

  ```Java
  @NotBlank
  private String name;
  @NotBlank
  @Email
  @pattern() //iske andar hum apne regular expression daal sakte
  private String emailid;
  @size(min = 4, max = 12)
  @NotBlank
  private String password;
  ```

- command object ke baad next argument should be binding result.

- res.hasErrors() -> result ke andar saare errors ko catch karega //If any validation error return back to same page
  ```Java
  if(res.hasErrors()){
      System.out.println(res.toString());
      return newUser;
  }
  ```
- yaha pe jo errors arahe hai, uske error codes note karo Spring console mein. It will be like:
- It looks like **Annotation.command.field**
- Iske baad JSP ke andar jaao aur, email ke baaju email ka error aur password ke baaju password ka error dikhao. For that hume:

```JSP
    Name: <sf: input path="name"> <sf: errors path="name" cssClass="error"/>
    <%-- sf: input tag ke baad wala tag add karna padhega --%>
```

- Now, to display all the validation error messages at once

```JSP
   <sf: errors path="*">
```

- To display error messages

```Java
  @NotBlank
  private String name;
  @NotBlank(message = "*")
  @Email
  @pattern() //iske andar hum apne regular expression daal sakte
  private String emailid;
  @size(min = 4, max = 12)
  @NotBlank((message = "*"))
  private String password;
```

- To show custom created error messages

  - create a messages.properties file in the src/main/resources folder and in it, type

  <!-- Note: Main yahan pe PHP aise hi lagaya hai text mein colour laane ke liye, the enclosed statements aren't PHP codes-->

  ```PHP
  Size.command.password=Password length must be from {2} to {1}
  NotBlank.command.email = Email cannot be balnk
  Email.command.email = Email is not valid
  ```

  - Go to application.properties and type:

  ```PHP
  spring.messages.basename=messages
  ```

  - Ye by deafult basename "messages" hota hai, agar hame custom naam dena padhega
  - Jo spelling aapne diya tha, woh spelling wahan pe likho
  - properties file basename (without extension) -- default is "messages"
  - It MUST be given when file name is other than messages.properties
  - example:

  ```PHP
  spring.messages.basename=errors
  ```

  - There can be some type of validations where hume server side hi karna padhega (For example in case of duplicate email)
  - When we need to do something out of the scope of basic Validation, we need to write a custom Validator

  - Create a new Class as... "UserExistValidatorImpl" (Apko Spring ka Validator interface chahiye)
  - Yeh na to service hai, na toh Controller hai, isliye hum @Component lagate hai

  ```Java
  @Component
  class UserExistValidatorImpl implements Validator{

    @Autowired
    private UserDao userDao;

    //Check karte hai kon konse pojos ka validation support karta hai
    //Agar User class ka hi, then only hum support kar rahe hai
    //called by Spring if POJO validation is supported by this Validator
    @Override
    public boolean supports(Class<?> clazz){
        //This validator validates User POJO
        return clazz = User.class;
    }

    //Jo user se aaya hua pojo object hai, wo yaha pe diya jaayega
    //Agr user already present hai, toh isko reject karte hai using Errors object use karke, "email" naam ke field pe
    //This is called by Spring to actually validate the pojo, which is input from the user - request parameters
    @Override
    public void validate(Object target, Errors errors){
        User user = (User) target;
        User dbUser = userDao.findByEmail(user.getEmail());
        if(dbUser != null){
            errors.reject("email", "UserExists", "User already exists");
            // arguments hai: field, error code and default message
            // This default message can be changed in application.properties
        }
    }

  }
  ```

  - ye mujhe apne LoginController ke sath attach karna hai toh LoginController class ke andar likho...
  - (Attach)

  ```Java
  //Access Validator in controller
  //Note that validator class is a spring bean -> we used @Component annotation
  @Autowired
  private UserExistValidatorImpl userExistValidator;

  //Agar user ka object hai toh hi isko validate karvao
  //This validator is applicable only for "User" validation
  //Called by Spring for each request before each data binding (request param ka data aap pojo mein daal rahe ho)
  @InitBinder
  public void init(WebDataBinder binder){
    //if binding user object then add custom validator
    if(binder.getTarget() instanceOf User)
        binder.addValidators(userExistValidator);
  }
  ```

  - and then In registerUser() method add @Valid before the "command" @ModelAttribute

  ```Java
  @RequestMapping("/register")
  public String registerUser(@Valid @ModelAttribute("command") User user, ...)
  ```

  - Here the validator will be auto invoked because of thie @Valid

  - To make Strings in our code to not be Hardcoded
    - In properties file add this, then in the jsp file do the next snippet
    ```PHP
        app.title = Sunbeam BlogSite
    ```
    ```JSP
        <h2><s:message code="app.title"/></h2>
    ```
  - Hum ek software ko localise karte hai international languages ke liye
  - Its a standard practise to not harcode the strings
  - In android we use the Strings.xml file

### Note:

- Hindi ka locale ka shortform hai : hi
- messages-hi.properties
- i18n - internationalization ka likhne ka tareeka (i aur n ke beech mein 18 letter hote hai)
- l10n - localization
- A SW company bolta hai ki aise Software banao jo kisi bhi locale ko apply hosake
- If we have a devanagri script word, we can't add it in our properties.xml file since its by default utf-8 format, but its requires utf-16 to save it. (Note: utf stands for Unicode Text Format).
- Agar hum locale ko -> 5 letters mein likte hai yoh kuch aise likhte hai:

  - **_en_US_** - (Langauge_countrycode), - Here its (english_USA)
  - **_en_GB_** - its english_Great Britain
  - **_fr_FR_** - its french_FRANCE
  - **_hi_IN_** - its hindi_India

- Make a messages_hi.properties (basename_languagecode.properties) file and add the tags with values in Hindi equivalent unicode characters. It will look somethin g like this:

  ```PHP
      app.title = \u0938\u0928\u092C\u0940\u092e\u092c...
  ```

- In .jsp change the charset to 'UTF-8' or 'UTF-16' (remove the ISO-).

---

## Session Scoped Beans

- Session Scoped Bean is a wrapper on HttpSession
- Session Scoped Bean mere multiple hosakte hai, joh unke respective HttpSession se connect hone chahiye.
- Session Scoped Bean per user alag alag create hona chahiye, but BlogController ek hi hai jiske paas only ek hi UserInfo ka pointer object. - Toh jab Kabhi Rohan ka browser se request aaya hai toh usemein us ki hi object ka address rakhsakte hai, aur agar Nilesh ke browser se bhi request aaya hai toh..???

  > Answer : Toh yeh kehta hai I will keep an object of UserInfo Proxy (Its a proxy object and not a main object), aur woh uss object ko access deta jis object ka current session deraha hai. (UserInfo ka proxy singleton hota hai)
  > Iss proxy ke andar code hota hai jo current HttpSession se UserInfo uthata hai.

- Proxies 2 types ke hote hai

  1. Class Based Proxy
  2. Interface Based Proxy

- Agar aap Interface likh ke Bean banate then 2 type use karsakte hai or agar interface nahi banaya then directly 1st type
- Ab wahi proxy likhne ke liye, @Scope ke andar proxyMode add karo:

  ```Java
    @Scope(value = "session", proxyMode=ScopedProxyMode.TARGET_CLASS)
    @Component
    public class UserInfo{

    }
  ```

#### Note:

Java Proxy ek design pattern which is an AOP. (Java EE mein Filters aur Core Java mein Proxies)

- Note Interviews mein asie pooch sakte hai:

  > Question : Hibernate ka Get Method aur Load Method mein kya farak hai?
  > Answer: Get method returns entity and Load Method returns proxy

- [Java Proxy Video Tutorial 1](https://www.youtube.com/watch?v=oXZMKD98TOE)
- [Java Proxy Video Tutorial 2](https://www.youtube.com/watch?v=AuINJdBPf8I)
- [Java Proxy Video Tutorial 3](https://www.youtube.com/watch?v=cHg5bWW4nUI)
- [Java Proxy Video Tutorial 4](https://www.youtube.com/watch?v=gDs-bpKv4mc)

- proxy directly on class created internally by CGLIB library
- Alternatively Proxy can be interface based and is created by java dynamic proxy (Present from Java 1.3)

- Examples of Proxies
  > Hibernate ka get vs load method -> uses proxy
  > @OneToMany jab karte ho aur fetch type lazy hai, toh wahaan pe proxy use hota hai

<sub>read about @Scope("session"), RequestScopeBean, SessionScopeBean, etc</sub>
<sub>NOTE: At 11:45 PM on 07-12-2022, Nilesh sir says: File upload aur download hum REST ke sath padhengey</sub>
<sub>Hum Thymeleaf use karte as an alternative to .jsp</sub>
<sub>Par industry mein thymeleaf mein hum React aur Angular</sub>

---

## REST

- **REST** stands for **RE**presentational **S**tate **T**ransfer
- When Applications communicate with each other its through Microservices
- REST service is a building block of Microservices

- A Web service doesn't have any UI.

  - Uses Http Protocol
  - Data transfer from client to server and vice-versa
  - platform independent (ye Tab hota hai jab data transfer is in a standard format)

- SOAP Web Services

  - Purane zamane mein hum jab hum web services banate the, toh hum XML uske karte the
  - those web services were called **_SOAP Web Services_**
  - XML data transfer over Http Protocol in a predefined grammer = **_SOAP Web Services_**
  - WSDL : Web Service Description Language
  - wsimport tool WSDL import padhke uska proxies banadeta hai (Ye proxy alag wala hai)
  - Server side ke upar ek Stub Hota hai
  - Aap proxy ka object banate ho. woh Proxy Object, packet bhejta hai aur stub ke pass recieve hota hai
  - Phir Stub, method execution (Processing) ke baad result ka packet, proxy ko bhejta hai
  - Drawbacks :
    > - Involves Heavy Processing.
    > - Not ideal for small embedded devices.
    > - Processing time was slow.

Hence to overcome those drawbacks we swicthed to REST Services

- **REST** stands for **RE**presentational **S**tate **T**ransfer
  - **RE**presentational - JSON (Most popular), XML, YAML
  - **S**tate - Values or Fields
  - **T**ransfer - Protocol (HTTP, TCP)

<sub>Objects ke state ka transfer ki baat horaha hai yahan</sub>

- _REST Request Methods_
  - **GET**
    > Get one/more records from Server - **SELECT**
  - **POST**
    > Post/Send new record to Server - **INSERT**
  - **PUT**
    > Send modified record to Server - **UPDATE**
  - **DELETE**
    > Delete a record in the Server - **DELETE**
  - **TRACE**
  - **OPTIONS**
  - **HEAD**
  - **PATCH**
    > Send **only changes** in record to the Server - **PATCH**

### Conventions

- Lets Consider out Mobiles Project
- http://localhost:8080/mobiles
  - GET: get All Mobiles
- http://localhost:8080/mobiles/{id}
  - GET: get Mobile of given id
- http://localhost:8080/mobiles
  - POST: insert new mobile
- http://localhost:8080/mobiles
  - DELETE: delete mobile of given id
- http://localhost:8080/mobiles/{id}
  - PUT: Update mobile of given id

<sub>Hum Dao Layer aur codes Demo11, i.e. Day11 se copy kar rahe hai</sub>

### mobilerest implementation:

- Open application.properties and type:

  ```PHP
    server.port=4000
  ```

  <sub>By default port hota hai 8080, we changed it to 4000</sub>

- Create a class UserControllerImpl and type:

  ```Java
    @RestController
    public class UserControllerImpl{
        @Autowired
        private UserDao userDao;

        //By default uska manna ye hai, ki jo bhi current view hai, usko model bana ke bhejna hai
        //Hum usko ye bata rahe hai ki hum response ke body mein response banana hai aur bhejna hai
        @RequestMapping(value="/users", method=RequestMethod.GET)
        public @ResponseBody List<User> getAllUsers(){
            List<User> list = userDao.findAll();
            return list;
        }

        @RequestMapping(value="/users/{id}", method=RequestMethod.GET)
        public @ResponseBody List<User> getUser(@PathVariable("id") int id){
            User user = userDao.findById(id);
            return user;
        }

        //Here we might get infinite loop due to recursion caused by BiDirectional recursion setup
    }
  ```

  <sub>Read about REST API versioning</sub><br/>
  <sub>Classwork and Assignment: RestController likho</sub>
