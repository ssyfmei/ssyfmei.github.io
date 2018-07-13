---
layout: post
comments: true
title:  "My Dumb App"
excerpt: "A mini CRUD web application"
date:   2018-07-13 12:00:00
mathjax: false
---
### 07/13/218

#### How to set up MySQL

1. Login as root. Create a database and table.

   ```mysql
   CREATE DATABASE web_student_tracker;
   USE web_student_tracker;
   CREATE TABLE student (
     id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, 
     first_name VARCHAR(45),
     last_name VARCHAR(45),
     email VARCHAR(45));
   ```

2. Create a new User

   ```mysql
   CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'password';
   ```

3. Set permission for the new user

   ```mysql
   GRANT ALL PRIVILEGES ON web_student_tracker . student TO 'newuser'@'localhost';
   ```

##### we have created a table for our web-app. The web-app will login as a new user who can only manipulate that specific table.



#### Use JDBC to Access Data from the Database

1. META-INF/context.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <Context path="">
   	<Resource
   		name="jdbc/web_student_tracker"
   		auth="Container"
   		type="javax.sql.DataSource"
   		maxActive="20" maxIdle="5"
   		maxWait="10000"
   		username="newuser"
   		password="password"
   		driverClassName="com.mysql.jdbc.Driver"
   		url="jdbc:mysql://localhost:3306/web_student_tracker?useSSL=false&amp;serverTimezone=UTC"/>
   </Context>
   ```

   The file contains login info for the new user to access the database. Still don't know how the path works.

2. The info above can be injected into a DataSource variable through annotation.

   ```java
   @Resource(name="jdbc/web_student_tracker")
   	private DataSource dataSource;
   ```

3. Establish connection and read data to a list.

   ```java
   List<String> students = new LinkedList<>();
   Connection myConn;
   Statement  myStmt;
   ResultSet  myRs;

   try{
     myConn = dataSource.getConnecton();
     myStmt = myConn.createStatment();
     myRs   = myStmt.query("select * from student order by last_name");
   }
   while(myRs.next()) {
     String firstName = myRs.getString("first_name");
     String lastName = myRs.getString("last_name");
     String email = myRs.getString("email");
     Student tempStudent = new Student(firstName, lastName, email);
     students.add(tempStudent);
   }
   ```

4. A servlet will pass the String list to a JSP page.

   ```java
   doGet (HttpServletRequest request, HttpServletResponse response) {
     request.setAttribute("STUDENT_LIST", students);
     RequsetDispatcher dispatcher = request.getRequestDispatch("/list.jsp");
     dispatcher.forward(request, response);
   }
   ```

5. Display data on a jsp page

   ```html
   <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
   <table>
   	<tr>
     		<th>First Name</th>
       	<th>Last Name</th>
         	<th>Email</th>
     	</tr>  
     	<c:forEach var="tempStudent" items="${STUDENT_LIST}">
     	<tr>
   		<td>${tempStudent.firstName}</td>
   		<td>${tempStudent.lastName}</td>
   		<td>${tempStudent.email}</td>
   	</tr>
       </c:forEach>
   </table>
   ```

#### Deployment

1. Start a new maven project. Add dependency to a pom.xml file

   ```xml
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <version>8.0.11</version>
       </dependency>
   ```

2. Add all the java files. (Better do this on eclipse.)

   ```java
   /src/main/java/com/yifeng/app/file1.java
   /src/main/java/com/yifeng/app/file2.java
   /src/main/java/com/yifeng/app/file3.java

   /src/main/webapp/list.jsp

   /src/main/META-INF/context.xml
   ```

3. Build maven project. (more commands need to be understand!!!!!)

   ```shell
   mvn clean build;
   ```

4. Deploy the war file on the Tomcat server

   ```shell
   #!/bin/bash
   cp /target/myapp.war /opt/tomcat/webapp
   ```

### Done
