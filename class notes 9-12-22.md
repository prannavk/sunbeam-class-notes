# Class Notes

# Agenda Taught

- [] REST
- [] Hibernate
- [] JPA
- [] Spring Hibernate integration

### Spring RestTemplate client

- RestTemplate is used to create applications that consume RESTful Web Services. The application can be console application, web application or another REST service.

  - RestTemplate can be created directly OR using RestTemplateBuilder.

- Creating RestTemplate using builder is a good practice. It can be used to customize restTemplate with custom interceptors. It also enables application metrics and work with distributed tracing.
- RestTemplate sends GET, POST, PUT or DELETE request to the REST resource/api.
- Important RestTemplate methods:

  ```Java
      T getForObject(String url, Class<T> responseType, Object... uriVariables);
      ResponseEntity<T> getForEntity(String url, Class<T> responseType, Object... uriVariables);
      T postForObject(String url, Object request, Class<T> responseType, Object... uriVariables);
      ResponseEntity<T> postForEntity(String url, Object request, Class<T> responseType, Object... uriVariables);
  ```

- Set the various URI components through the respective methods (scheme(String), userInfo(String), host(String), port(int), path(String) , pathSegment(String...), queryParam(String, Object...), and fragment(String).
- Build the UriComponents instance with the build() method.

### [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

- Cross-Origin Resource Sharing (CORS) is an HTTP-header based mechanism that allows a server to indicate any origins (domain, scheme, or port) other than its own from which a browser should permit loading resources.
- The CORS mechanism supports secure cross-origin requests and data transfers between browsers and servers.
- Modern browsers use CORS in APIs such as XMLHttpRequest to mitigate the risks of cross-origin HTTP requests.

- [Server applicable may restrict resources/APIs to be accessed only from certain resources Access-Control-Allow-Origin](https://myapp-frontend.com)
- Spring boot application handle CORS with @CrossOrigin annotation on @Controller and/or @RestController. It can also be used on individual request handler methods in controllers.
  ```Java
  @CrossOrigin("*") // default is to allow all origins "*".
  @CrossOrigin(origins = "https://myapp-front-end.com")
  ```

### ORM

- Converting Java objects into RDBMS rows and vice-versa is done manually in JDBC code.
- This can be automated using Object Relational Mapping.
- Java Class is mapped to RDBMS Table, while it's field are mapped to Column.
- It also map table relations into entities associations/inheritance and auto-generates SQL queries

#### Hibernate

    > Hibernate is the most popular ORM tool

#### Hibernate Elements:

##### SessionFactory

- One SessionFactory per application (per db).
- Its Heavy-weight. Not recommended to create multiple instances in single application.
- ItsThread-safe. Can be accessed from multiple threads (synchronization is built-in).
- Typical practice is to create singleton utility class for that.

##### Session

- Created by SessionFactory & it encapsulate JDBC connection.
- All hibernate (CRUD) operations are done on hibernate session.
- Not thread-safe. Should not access same session from multiple threads.
- Light-weight. Can be created and destroyed as per need.

##### Transaction

- In hibernate, autocommit is false by default.
- DML operations should be performed using transaction

##### Session Flush

- session.flush(): Forcibly synchronize in-memory state of hibernate session with database.
- Each tx.commit() automatically flush the state of session.
- Manually calling flush() will result in executing appropriate SQL queries into database.
- Note that flush() will not commit the data into the RDBMS tables.
- The flush mode can be set using session.setHibernateFlushMode(mode).
- ALWAYS, AUTO, COMMIT, MANUAL

##### Dialect

- RDBMS have specific features like data types, stored procedures, primary key generation, etc.
- Hibernate support all RDBMS.
- Most of code base of Hibernate is common.
- Database level changes are to be handled specifically and appropriate queries should be generated. This is done by Dialect.
- Hibernate have dialects for all popular RDBMS (org.hibernate.dialect.\*). Programmer should configure appropriate dialect to utilize full features of RDBMS

#### Hibernate CRUD methods

get()/find() -- select/retrieve entity from db for given id.
persist() -- insert new entity in db.
update() -- update existing entity in db.
delete() -- delete entity from db

#### Hibernate 3 Bootstrapping

- Bootstrapping refers to the process of building and initializing a SessionFactory.
- SessionFactory is typically initialized with following properties.

  ```PHP
      hibernate.connection.driver_class
      hibernate.connection.url
      hibernate.connection.username
      hibernate.connection.password
      hibernate.dialect
      hibernate.show_sql

  ```

- SessionFactory can be configured using hibernate.cfg.xml or Java code to initialize Configuration object.

---

- User Object has a concept of named query
- using which you can add a name to the query and you want to write an actual query
- Either write '?' or the parameter's name
- You can write as many queries as you want.

  - In the User class we write:

    ```Java
        @NamedQuery(name="byMobile", query="SELECT u FROM User u WHERE u.mobile=?1")
        @NamedQuery(name="updateMobile", query="UPDATE User u SET u.mobile=?1 WHERE u.id=?2")
        @Entity
        @Table(name="users")
        public class User {
        ....
        }
    ```

  - In UserDaoImpl we write:

    ```Java
        public User findByMobile(String mobile) {
          Query<User> q = session.getNamedQuery("byMobile");
          q.setParameter(1, mobile);
          User user = q.getSingleResult();
          return user;
      }
    ```

- Another example of using namedQuery

  ```Java
      public int changeMobile(int userId, String mobile) {
  	    Query q = session.getNamedQuery("updateMobile");
  	    q.setParameter(1, mobile);
  	    q.setParameter(2, userId);
  	    return q.executeUpdate(); // DML query
    }
  ```

- **Note** : Since its an Update Operation, we need to check the transactions, hence we write the same function as:

  ```Java
  public int changeMobile(int userId, String mobile) {
  	try {
  		session.getTransaction().begin();
  		Query q = session.getNamedQuery("updateMobile");
  		q.setParameter(1, mobile);
  		q.setParameter(2, userId);
  		int cnt = q.executeUpdate(); // DML query
  		session.getTransaction().commit();
  		return cnt;
  	}
  	catch (RuntimeException e) {
  		session.getTransaction().rollback();
  		throw e;
  	}
  }

  ```

- Now, How do we call it..???

  - In Demo14Main, write:

  ```Java
  try(UserDaoImpl dao = new UserDaoImpl()) {
  		int cnt = dao.changeMobile(13, "777777777");
  		System.out.println("Updated rows: " + cnt);
  	}
  	catch (Exception e) {
  		e.printStackTrace();
  	}
  ```

- **createNatievQuery**

  - it takes care of creating the actual native queries, if you at all have the interest to create the native queries.
    <sub>Things behind the curtain, since till now Spring Data was doing a lot for you</sub>

- Till now:
  - Bootstrapping ka code Hibernate 3 ka tha, and dependency jo add kiya tha, woh Hibernate 5 ka bhi tha
- Note: ye tha HIbernate 3 ka code tha

##### Hibernate 5 mein:

- Configuration is divided into 2 parts
  > hibernate.cfg.Xml mein configs are Given to Service Registry
  > Orm ka config is given to the metadata
- Then the session Factory ka object banta hai
  - [First Service Registry ka obj, then Metadata ka obj banta hai, and then jaake session Factory ka object banta hai]
- Pehle ek ORM class likho

  - we take a basic user class first
  - we copy paste config from hibernate.cf.xml from demo14-hibernate project

- ServiceRegistry

  - In Hibernate 5.0, a Service is a type of functionality represented by the interface org.hibernate.service.Service.
  - ServiceRegistry is interface and represent a standard to add, manage hibernate services.

- Metadata

  - Represents application's domain model & its database mapping

- wkt we have to:

  - [First Service Registry ka obj, then Metadata ka obj banta hai, and then jaake session Factory ka object banta hai]
  - nOw for Service Registry object, we use:

  ```Java
  // Hibernate 5 Bootstrapping
  private static SessionFactory buildSessionFactory() {
  	StandardServiceRegistryBuilder builder = new StandardServiceRegistryBuilder();
  	StandardServiceRegistry serviceRegistry = builder
  		.configure() // load settings from hibernate.cfg.xml
  		.build();

  	return null;
  }
  ```

- Jitna hum Hibernate ke andar setting likhe the, un sabko, applySetting() {multiple times} karte hai.
- Spring Boot mein ye automatic hota hai
- Filhal mein readymade .configure() method use karta hun

  - <sub>this method load settings from hibernate.cfg.xml</sub>

- Abhi HUme metadeta ka object build karna hai

  - Uske liye -> hum jitne mapping ke tag likhe the, from hibernate.cf.xml, jo upar load hue the, hum isme load kar rahe hai

  ```Java
  Metadata metadata = new MetadataSources(serviceRegistry) // load <mapping> from hibernate.cfg.xml
  		.buildMetadata();
  ```

- Abhi iske andar sara mapping ka info hai
- Abhi isse use karke hum finally Session Factory object banate hai

- **Question: Agar hume without xml karna hai toh??**

  > Spring Boot internally classes rakhta hai metadata kai
  > uske liye we use : addAnnotatedClass(Class annotatedClass) : MetadataSources metadatasources method
  > aap log chaho toh pura package bhi add kar sakte hai
  > Par yaha, since hum hibernate.cfg.xml bana chuke hai, we will continue with using xml

- Abhi Demo15Main ke andar:

  - main mein:

  ```Java
      SessionFactory factory = HbUtil.getSessionFactory();
  		Session session1 = factory.openSession();
  		User user1 = session1.find(User.class, 1);
  		System.out.println("User1 Found: " + user1);
  		session1.close();

  		HbUtil.shutdown();
  ```

- **Note**: Factory ko close karna mat bhoolna, since its a heavyweight object, resources will get leaked
  <sub>Bhool Mat Bhosdike, yaad kar, yaad rakh</sub>

  - Toh, ye tha Demo15 - Hibernate 5 bootstrapping

- Abhi agar hume ek aur session banana hai toh??

  - Aisa likho:

  ```Java
  public class Demo15Main {

    public static void main(String[] args) {
  	    SessionFactory factory = HbUtil.getSessionFactory();

  		    Session session1 = factory.openSession();
  		    User user1 = session1.find(User.class, 1); // L1 cache of session1
  		    System.out.println("User1 Found: " + user1);

  	    Session session2 = factory.openSession();
  		    User user2 = session2.find(User.class, 1); // L1 cache of session2
  		    System.out.println("User2 Found: " + user2);

  		    // Each call to openSession() returns a new session -- encapsulating JDBC connection

  		    session1.close();
  		    session2.close();
  		    HbUtil.shutdown();
    }

  }
  ```

- Abh ye dono same sessions hai, ya different sessions??

  - Ye karke dekho:

  ```Java
  System.out.println("Same sessions: " + (session1 == session2));
  ```

  - **iska output 'false' aayega **

- **Question: Agar mein ek hi session mein 2 baar userno 1 ko dhundata hun, toh are they the same user objects?**

  - L1 cache of the session returns the same object

  ```Java
      SessionFactory factory = HbUtil.getSessionFactory();

  		Session session = factory.openSession();
  		User user1 = session.find(User.class, 1); // L1 cache of session
  		System.out.println("User1 Found: " + user1);

  		User user2 = session.find(User.class, 1); // L1 cache of session -- returns same object
  		System.out.println("User1 Found: " + user1);

  		System.out.println("Same User objects: " + (user1 == user2));
  		session.close();

  ```

- Now point to be noted is:
  > If obj is there in L1 cache, db is not queried again, the object is just returned

There is a method called getCurrentSession();

- Each call to getCurrentSession() returns the same session, if called in the same context.

  - Agar mein ye execute karta hun:

  ```Java
      SessionFactory factory = HbUtil.getSessionFactory();

  		Session session1 = factory.getCurrentSession();
  		//User user1 = session1.find(User.class, 1);
  		//System.out.println("User1 Found: " + user1);

  	Session session2 = factory.getCurrentSession();
  		//User user2 = session2.find(User.class, 1);
  		//System.out.println("User2 Found: " + user2);

  		// Each call to getCurrentSession() returns same session if called in same context
  		System.out.println("Same sessions: " + (session1 == session2));
  ```

- Ye error dega

  - Kyunki aapne SessionContext define hi nahi kiya
  - Hence uske liye

- We can have 3 type of hibernate.current_session_context_class contexts

  1. Thread Context
  2. JPA Context
  3. Custom Context

- Samjho humne hibernate.current_session_context ko Thread set kiya
- Question: us main function mein kitne thread chalrahe hai?

  > Answer: 1

  - Hence if I call that method again and again, it will return the same session (in the same thread)

  ```Java
      // hibernate.cfg.xml -- current_session_context_class=thread
  	SessionFactory factory = HbUtil.getSessionFactory();

  		Session session1 = factory.getCurrentSession(); // when called 1st time in a thread, create new session and return it.
  	Session session2 = factory.getCurrentSession(); // for all subsequent calls in same thread, returns same session
  ```

- Agar....., Hum kuch aise likhte.....

  ```Java
      SessionFactory factory = HbUtil.getSessionFactory();

  	new Thread(() -> {
  		Session session1 = factory.getCurrentSession(); // when called 1st time in a thread, create new session and return it.
  	}).start();

  	new Thread(() -> {
  		Session session2 = factory.getCurrentSession(); // when called 1st time in a thread, create new session and return it.
  	}).start();
  ```

- <sub>Core Java</sub>

  - **Note**: Jabhi aap anonymous inner class use kar rahe ho, then aap only the final members of the enclosing class ko access kar sakte ho.

  ```Java
      SessionFactory factory = HbUtil.getSessionFactory();
  	Session[] arr = new Session[2];

  	new Thread(() -> {
  		Session session1 = factory.getCurrentSession(); // when called 1st time in a thread, create new session and return it.
  		arr[0] = session1;
  		// session is auto-closed when thread terminates
  	}).start();

  	new Thread(() -> {
  		Session session2 = factory.getCurrentSession(); // when called 1st time in a thread, create new session and return it.
  		arr[1] = session2;
  		// session is auto-closed when thread terminates
  	}).start();

  		// session1 and session2 are different
  		System.out.println("Same sessions: " + (arr[0] == arr[1]));

  	HbUtil.shutdown();
  ```

- Abhi yahaan par, session1 aur session2, dono alag hai
- Note: jab aaplog session open karte hai, toh session ko close karna aapki responsibility hai
- Par jab woh session create karta hai, session close karna uska responsibility hai

- when thread is terminated, session is automatically closed.
- Hum woh session ko 5 seconds tak marne nahi de rahe hai, so that we can compare
- Hence ye dono session alag alag hai since dono alag alag threads mein call kiye hue hai.
- In same thread - same return Karta hai, different thread - different hota hai.

- Agar alag alag dbs mein agar transaction karna hai, toh hum JTA use karta hai. [JTA - Java ka technology hai, which stands for: Java Transaction API]
- [Transaction concept - Agar dono succes hona hai ya, dono fail hona chahiye]

- Agar hum JTA, Hibernate ke sath use karte hai, toh usko currentSessionContext dete hai
- Jab hum Spring, Hibernate ke sath use karte hai, Spring custom context banake de deta hai

  ````Java
  SessionFactory factory = HbUtil.getSessionFactory();
  Session[] arr = new Session[2];
  new Thread(() -> {
  Session session1 = factory.getCurrentSession(); // when called 1st time in a thread, create new session and return it.
  arr[0] = session1;
  delay(5000);
  // session is auto-closed when thread terminates
  }).start();
  new Thread(() -> {
  Session session2 = factory.getCurrentSession(); // when called 1st time in a thread, create new session and return it.
  arr[1] = session2;
  delay(5000);
  // session is auto-closed when thread terminates
  }).start();
  // session1 and session2 are different
  delay(2000);
  System.out.println("Same sessions: " + (arr[0] == arr[1]));

      	HbUtil.shutdown();
      }

      public static void delay(int milli) {
      	try {
      		Thread.sleep(milli);
      	} catch (InterruptedException e) {
      		e.printStackTrace();
      	}
      }
      ```

  ````

- When ure making call to getCurrentsession for the 1st time

  - It internally checks if any sesion is already present in its thread local storage
  - [har ek thread ka ek internal local storage hota hai]
  - Agar hai, toh uska pointer return hojata hai
  - Agr nahi hai, toh ek naya session create karta hai thread local storage mein, aur uska pointer return karta hai

  <sub>[Thread Local Storage, OS ka concept hai]</sub>

- [agar current thread ka thread local storage access Karna hai Core Java mein, uske liye Thread Local class hota hai]

- CustomCOntext khud ko banana hi, toh hum Ek class likhna hai which extends SessionCOntext class

  <sub>[Google mein search karo: How to implement Custom Hibernate COntext?]</sub>

- OpenSession mein aapko Select hojata tha, par DML ke liye transaction lagta tha
- Par yahaan pe saare db operations ke liye transaction chahiye. [Not even SELECT will work]

- Agar Aapko work karvana hai, toh aisa kuch likho

  ```Java
  Session session = factory.getCurrentSession(); // returns session -- but no tx/connection associated with it -- so cannot perform any db operation
  session.getTransaction().begin(); // start tx context -- attach a db connection internally
  User user = session.find(User.class, 1); // L1 cache of session
  System.out.println("User Found: " + user);
  session.getTransaction().commit(); // end tx context
  session.close();
  HbUtil.shutdown();
  ```

- Ab chaljaayega

#### Open Session vs GetCurrentSession
<sub> imp CCEE concept </sub>

- Each call to OS creates a new seesion on every new call
    > First call returns a new session, next calls returns the same session

- Read only transactions already allowed hai, DML ke liye u need transaction context
    > getCurrentSession() mein - no transaction is associated with that, u cant perform any operation without explicitly creating a transaction

- OS ka session needs to be closed explicitly by the programmer
    > getCurrentSession() ka session is automatically closed by the programmer at the end of the context

---

- I want to use Hibernate with Spring (Only Spring not Spring Boot)
- Toh ab, kaam asaan hoga, ya muskhil hoga?

Note:
5.6.14 - Hibernate ka version
5.3.24 - Spring ORM ka version

<sub>Agr hum yaha Spring BOOT use karte, toh version note karne ki zarurat na hoti, since Spring BOOt eliminates the jar conflicts of Spring</sub>

- New Spring Bean Definition xml file:
- XSD namespaces chunne hai:
  - beans - 4.3
  - context -4.3
  - tx - 4.3
- Next >> Finish dabao

- Lets See How Spring Simplifies the Hibernate

<!-- database configurations are to be done for the data source -->

- datasource is a connection pool
- JDBC ke sath connect karne ke liye
- DriverManager khud java.sql.DataSource interface se inherit hota

- Abh beans.xml mein ye likho: [Spring HIbernate Helper beans config]

  ```PHP
  <!-- database config for DataSource -->
  <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
  	<property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
  	<property name="url" value="jdbc:mysql://localhost:3306/mobiles_db"/>
  	<property name="username" value="nilesh"/>
  	<property name="password" value="nilesh"/>
  </bean>
  ```

- **Note**: LocalSessionFactoryBean is a wrapper on our SessionFactory

  ```PHP
  <!-- session factory config -->
  <bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
  	<property name="dataSource" ref="dataSrc"/>
  	<property name="packagesToScan" value="com.sunbeam"/>

  </bean>
  ```

- yahan packagesToScan property Spring BOOT internally kardeta hai

- Abhi hum tx i.e. Transaction Manager ko configure karte hai:

  ```php
  <!-- tx manager config -->
  <bean id="txManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
  	<property name="sessionFactory" ref="sessionFactory"/>
  </bean>
  ```

- Here, dataSrc Bean is injected into SessionFactory Bean, which is injected into txManager Bean
  - <sub>[Dependency Injection concept]</sub>
  ```php
  <tx:annotation-driven transaction-manager="txManager"/>
  ```
- Agar ye likhe, toh: Spring manages all transactions using annotation --@Transactional internally use TransactionManager bean

- Note: autowiring un beans mein hota hai, jo khud beans hote hai

- Hum Simple JPA Repository ue kar rahe the aaj tak, abhi hum apne khudke @Repository banaya hai

- Abhi ye annotations chalne ke liye, we need to write:

  - [beans.xml mein, sabse upar likhte hai:]

  ```php
  <!-- auto-detect all spring beans with stereo-type annotations -->
  <context:component-scan base-package="com.sunbeam"/>
  ```

- beans.xml mein agar hum componentscan nahi likhe, toh woh @Repository ka usko bean hi nahi milega

- <sub>Read about: [Hibernate LOAD VS GET KA Difference]</sub>
