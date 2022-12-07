# Class Notes

#### There are 2 approaches in writing code for projects

1. CodeFirst Approach
   > pehle code likho (java ke entities) -> phir usko jaake Db mein table banane ko bolo
2. DatabaseFirst Approach
   > App pehle DB design karo, normalize karo, etc -> and jaisa db bana hai, uske hisab se code banao

#### For CodeFirst Approach we use:

- In properties.xml file type:
  ```PHP
  spring.jpa.show-sql=true
  spring.jpa.hibernate.ddl-auto=create
  ```
- Note: By default the above's value is none
  ```PHP
  spring.jpa.hibernate.ddl-auto=create-drop
  ```
- Application shuru hote hi table create karo, application khatam hote hi table uda do (I don't want to keep the table permanently in my DB)
  ```PHP
  spring.jpa.hibernate.ddl-auto=update
  ```
- It will check the schema of the table with the schema of the entity class (of the table)
- If they are exactly matching it will work, if not it will only perform the relevant changes and stop

- Now if we add another field in our class

  > It will perform ALTER TABLE ... query and add the table

  ```php
  spring.jpa.hibernate.ddl-auto=validate
  ```

- It will check the schema of the table with the schema of the entity class (of the table)
- If they are exactly matching it will work, if not it will THROW AN EXCEPTION terminate program

- Note: validate doesn't touch the db
  ```php
  spring.jpa.hibernate.ddl-auto=none
  ```
- It will do nothing -> The Database First Approach

- While creating , if we want to set length, datatype and add the unique constraint:
  ```Java
  @Column(name="mname", length = 60, unique = true)
  ```
  ```php
  spring.jpa.hibernate.ddl-auto=update
  spring.jpa.properties.hibernate.hbm2ddl.import_files=db.sql
  ```
- SQL file can be executed here after table creation to populate the data

- Now, for HAS A fields:
  ```Java
  @Embedable
  ```
  - Put on the property entity class for a list of columns which we want to embed in another table
  ```Java
  @Embedded
  ```
  - Put on top of the field of the @Embedable entity class type

##### NOTE:

- Jab bhi aap koi class composite primary key ke liye alag se class banarahe ho, it MUST IMPLEMENT SERIALIZABLE INTERFACE.

- Its a good practise to override equals() and hashCode() methods inside the composite primary key class i.e. the @Embeddable class.

- Oracle mein Primary Key generation ke liye uska SEQUENCE feature use hota hai.
- Identity is a feature attached with the table.
  ```SQL
  CREATE TRIGGER ...;
  CREATE SEQUENCE ...;
  SELECT Nextval FROM sequenceName;
  ```
- MySQL doesn't have the SEQUENCE feature, for which it internally created a TABLE called bookIdSeq with only 1 column, called NEXT_VAL

#### For images:

- In Mobile class, dont declare images (fields) with String type declare it with an embedded type along with:
- <sub>[since our requirement = Har ek mobile mein ek mobile image hai]</sub>
- Keep
  ```Java
  @OneToOne
  ```
- Main mobile ke sath kya kiya, toh andar mobile image ke sath woh hua hai -> (Cascade={------})
- ek save() se dono table mein insert hota hai (2 insert queries).

[Composition]

- n+1 problem

  > n queries for n child and 1 query for the main parent (If you had 1000 mobiles)

- standard naming convention hai java ke andhar camel case

---

- JAB JAB MEIN ORDER KO JPA KE SATH ATTACH KAR RAHA HUN, TAB TAB MUJHE MOBILE KO BHI ATTACH KARNA PADHEGA

- id main 0 deta hu, taki woh generate karlega

- Order ko hum JPA ke sath add kar rahe hai

- MySQL Specific : How to know last inserted id.

- Bidirectional relationship:

  > Jis field se aapne apne order mein User ka field banaya hai, usi ke naam User mein dena hai annotation ke andar

- Note: **Pehla wala fetch eagerly fetch karo**

- Remember:
  > FOR ONETOMANY DEFAULT FETCH TYPE IS LAZY

#### Fetch type LAZY vs EAGER:

##### EAGER :

    - Have a single query on a db, and fetch both of them in one query (join)
    - [Ek connection mein sara ka sara data fetch karo]
    - [Multiple times db se connect karne ki zarurat nahi hai]

- Provisions sare configuration mein karte hai. In properties file, enter:
  - Agar koi problem aaya, toh connection open karne ki permission hai
  ```php
  spring.jpa.open-in-view=true
  #Agar koi problem aaya, toh connection open karne ki permission hai
  ```

##### LAZY:

    - Abhi kya maang raha hai? User? Toh phir User lelo. Jab Order maanga jaayega, tab dekhlengey

- Question
  > MANY TO MANY KONSE CONTEXT MEIN AATA HAI???

<sub>
[Internals jab pata karna hai toh Seekho: Reflection, Annotation and proxies]
</sub> <br/>

<sub>Apne DB ka design ko padho - aur one-to-one, many-to-one relations figure it out.</sub>

- ONE TO ONE
  > KISI TABLE KA PK, FK KEY SATH MAP KARNA PADTHA HAI
- MANY TO MANY

  > Jab bhi banta hai, woh ek auxilary table ke sahara banta hai!!!

- MANY TO ONE -> F TO P
- ONE TO MANY -> P TO F

---

<sub>
SQL Notes
Java Notes
BookShop till 4
Java File IO
Java Collection
SQL Normalisation - in new notebook

lombok java
postgresql tutorial
odd odes
Making a text editor using java
</sub>