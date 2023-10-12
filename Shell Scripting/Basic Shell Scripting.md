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
