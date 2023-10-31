•	Using Jenkins pipline & image from amazon Elastic Container and registry run yaml file directly from github

1.	Use t3.medium instance type so that we can run everything smooth. 
2.	Launch Instance give it some proper name in instance type select t3.medium, select key pair. Scroll down and click n Advanced details. From IAM instance profile select the profile that has full admin access and launch the instance.
3.	Select the Instance that you have created and select the security tab and open the security group and at the inbound rules. Add rule select All Traffic in Type and in source select Anywhere-IPv4 and click on save rules.
4.	Now open the instance in the environment you like using ssh.
``
a.	ssh -i KEY_PAIR ec2-instance@public-ip
``
5.	Now run the code to install and run the shebang code to install kubectl, eksctl and Jenkins also create the cluster.
a.	jenkins.yaml
```
#!/bin/bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade
sudo yum install fontconfig java-17* -y
sudo yum install jenkins -y
sudo systemctl daemon-reload
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status Jenkins
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
b. kubectl_eksctl_cluster.yaml
```
#!/bin/bash
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.5/2023-09-14/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin
# Below code will start creating cluster for 1 nodes
eksctl create cluster --name=ukki --node-type=t3.medium --nodes=1 --region=ap-south-1
```
6.	 In end after all process is done we will see the password, now open the browser and input the address.
``
a.	Public-ip:8080
``
7.	Now copy the password of the Jenkins(it would be shown I the environment) and press next.
8.	Click on “Install Suggested Plugins” button and wait for all the necessary plugins to be install.
9.	Now enter the details what is suggested and then press save and continue button. Set the URL if you have any and press save and finish button and press start using Jenkins.
10.	Open Github website and login to your profile.
11.	Click on create new repository and add the backend.yaml and frontend.yaml file to your new repository.
a.	backend.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
  labels:
    app.kubernetes.io/name: proxy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: tomcat
  template:
    metadata:
      labels:
        app: tomcat
    spec:
      containers:
      - name: tomcat
        image: 203772634753.dkr.ecr.ap-south-1.amazonaws.com/tomcat-student:latest
        ports:
        - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: proxy
  name: tomcat-deployment
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: tomcat
  type: LoadBalancer
```
b.	frontend.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app.kubernetes.io/name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: pritamkhergade/frontend:latest
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: frontend
  name: nginx-deployment
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: ClusterIP
```
12.	Go to github settings page, scroll down and select Developer settings option from the left side.
13.	Click on Personal access tokens and generate classic token, in this give some name to token and select after how many days this token should be expired and in scope we will only select repo and click on Generate Token. Copy the generated token and save it somewhere save as we gonna need it in future steps.
14.	Go back to the Jenkins page (public-ip:8080) and in this Click on Manage Jenkins from left side bar.
15.	From the System Configure click o Plugins, at left select the Available plugins option. In search we gonna need amazon service to communicate so in this we will search for aws and then at top it would show “aws credentials” select the package and the install button would be clickable. Click on the install button, now you can see the package is been installed as you clicked on it now scroll down and click Go back to the page, as you click the hyperlink you will be thrown the main page of the Jenkins.
16.	Now again click on Manage Jenkins and now select the Credentials from the Security tab. You can see the System in the Stores scoped to Jenkins. Click on the System in it click on Global credentials, I would be open and now click on Add Credentials. In Kind it would already selected Userame and password.
17.	Enter Username of your Github profile and in password bar add the token you have saved and press on create button. 
18.	Click on Add Credentials button again and in Kind select AWS Credentials add Access Key in Access Key ID and add the Secret Access Key in Access key ID and click on the create button.
19.	Now we have the Two credentials set for aws and github.
20.	Now click on the Jenkins icon it would send us to the home page. Click on the New Item Give some proper name to it and select the pipeline from the option and click on the o button that is now been activated.
21.	In general search for Github project option and select it. In this add the repo link that you have created, scroll down and add the script. 
22.	For the first half the code we gonna need to communicate with github so that it can fetch the files from the repo and build it and to write the code in right format we will have to use the pipeline syntax, click on the link and it would open in the new page. In Sample Step click on the drop-down menu and search for the “Checkout: Check out from version control” and the SCM the Git option would already be selected. Add the Repository URL (the link we gnna provide is the link that we use to clone any repo the one with .git in it) in the credentials drop down button you will see the username with the hidden password in it (there would be more then one if you have added more then one github profile it) select it. Scroll down and in Branch Specifier update it with the branch you’re working on (by default the branch is created in github repository is the main branch). Scroll down and click on the button that says “Generate Pipeline Script” and it would create the script. Copy the script and go back to the previous tab.
23.	In this we gonna write the first block.
```
pipeline {
    agent any

    stages {
        stage('git checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '020b0cb8-cf97-4a23-b75b-9817fcea697d', url: ‘https://github.com/Vaibhav-Shewale/kuber-jen.git’]])
            }
        }
   }
}
```
25.	Run the code and see the code is working or not, if not check what is breaking the code. Now we will try to build the code that we have I the github repo directly.
```
stage('deploy') {
            steps {
                sh '''
                aws eks update-kubeconfig --region ap-south-1 --name ukki
                kubectl apply -f .
                '''
            }
        }
```
26.	In the above code we have said it to create the cluster in the specific region and add the name of the cluster.
27.	Below is the master code by combing the two block in one.
```
pipeline {
    agent any

    stages {
        stage('git checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], 
                    userRemoteConfigs: [[
                        credentialsId: '4a42babb-2c26-4919-91df-8002e85f1980', 
                        url: 'https://github.com/Vaibhav-Shewale/kuber-jen'
                    ]]
                ])
            }
        }

        stage('deploy') {
            steps {
                sh '''
                aws eks update-kubeconfig --region ap-south-1 --name ukki
                kubectl apply -f .
                '''
            }
        }
    }
}
```
29.	Now click on the Apply button then save the pipeline code and it would open the pipeline page now click on the build option from the left bar and wait until is build and deploy.
30.	If you faced the error, check the output or the log of the tab where the error has been occurred. But now you have faced the error of kubectl command.
31.	Open the environment again where the you have run the shebang code earlier and now, we would like to copy the kubectl in the bin directory.
``
a.	cp -rf kubectl usr/local/bin
``
33.	Go back to the Jenkins page and again try to run the code and see if it ran properly or not.  
