•	Create a shell script to download the student-ui, built it, and host it using tomcat

1.	Shell Script is used to automate are the things that we repeat again and again. So in this task we will create a script to download not only the file that we have to host but also send it and automate in a way that we just need to run and check.
2.	In this we are not connecting to our database so the data would not be stored.
```
#!/bin/bash
sudo yum update
sudo yum install java-1.8.0-amazon-corretto-devel.x86_64 maven git -y
rm -rf student-ui
git clone https://github.com/Pritam-Khergade/student-ui.git
cd student-ui
mvn clean package
cd target
mv studentapp-2.2-SNAPSHOT ~
cd ~
rm -rf /mnt/apache-tomcat-8.5.93
curl -O https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.93/bin/apache-tomcat-8.5.93.zip
unzip apache-tomcat-8.5.93.zip
sudo mv apache-tomcat-8.5.93 /mnt/tomcat
mv studentapp-2.2-SNAPSHOT /mnt/tomcat/webapps/student
bash /mnt/tomcat/bin/catalina.sh start
```
3.	Now our script is ready we can now just need to attach it while creating the instance and done.


create RDS
launch EC2 instance, install package mariadb105 
connect to rds --> mysql -h hostname -u username -p 
create database create table --> 

CREATE TABLE students (
student_id int NOT NULL AUTO_INCREMENT PRIMARY KEY ,
student_name VARCHAR(100) NOT NULL, 
student_addr VARCHAR(100) NOT NULL, 
student_age VARCHAR(3) NOT NULL, 
student_qual VARCHAR(20) NOT NULL, 
student_percent VARCHAR(10) NOT NULL,
student_year_passed VARCHAR(10) NOT NULL
);


download tomcat8 --> copy link address -> curl -O link 
instal java-11 devel
unzip tomcat8, move to /mnt 
try to access the default tomcat page on port 8080
open s3 bucket link -> https://s3-us-west-2.amazonaws.com/studentapi-cit
search for student.war 
mv student.war to /mnt/tomcat/webapps 
cd /mnt/tomcat/bin
bash catalina.sh start 

access webpage on  ip:8080/student

create user in mysql 
assign permission and flush 

move to conf and vim context.xml 

<Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
           maxTotal="500" maxIdle="30" maxWaitMillis="1000"
           username="arjun" password="qwertyuiop1234567890#" driverClassName="com.mysql.jdbc.Driver"
           url="jdbc:mysql://database-1.cxtqpl5bdh0s.us-east-1.rds.amazonaws.com:3306/studentapp?useUnicode=yes&amp;characterEncoding=utf8"/>

replace the user , passowrd and endpoint of database

download mysql connecter from the same  s3 link and mv to /mnt/tomcat/lib
