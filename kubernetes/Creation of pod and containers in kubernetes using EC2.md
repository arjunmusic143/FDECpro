# Creation of Pod in AWS using kubectl

1.	Search IAM service in aws search console
2.	Give a proper name, select IAM user and create a password and untick the option of creating new password when login and press Next button. In permission select the “Attach policies directly” and then search for “AdministratorAccess” select it and press Next button.
3.	Check the creation detail proper if any mistake is present and then press Create User button.
4.	Note down the Username, password and url for logging from this user. Click on Reurn User button, it would send you back you to users page and here you will see your new user that you have recently created. Click on it and click on “Create Access Key” link. Click the “Application running on an AWS compute service” option and tick mark the option of access key and press the Next button. Give a proper Description if you want and click on Create access Key button. Download .csv file of the Access Key and Secret access key somewhere safe and click on Done button.
5.	 Open EC2 service and create an instance with default security group and details and click on Create Instance.
6.	Open your Instance in your desired environment.
7.	Now we need to create aws config file and then create a folder where the file would be shif
8.	`
      a.	aws configure
          i.	Enter the Access Key Id
          ii.	Enter the Secret Access key 
          iii.	Enter the region (for this I have put `ap-south-1`)
          iv.	-Let this be default-
      b.	mkdir aws
      c.	cp -r .aws aws
  	`
10.	 Now we need to Install ‘kubectl’. We gonna need install n-1 version means we need to install one previous version from the release version
      a.	curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.5/2023-09-14/bin/linux/amd64/kubectl  # There is a script that do everyhing from step 8 to step 17 --> [Script](https://github.com/Vaibhav-Shewale/fdec/blob/main/Shell%20Scripting/Kubect_eksctl%20%26%20cluster%20creation.md)
11.	Now check the permission of the kubectl, for that we gonna use long list.
      a.	ll
12.	It would show the permission allowed to the newly downloaded file. Now we need to add the execution permission to the newly downloaded directory “Kubectl”.
      a.	chmod +x ./kubectl
      b.	ll
13.	Now you can see that kubectl is in execution mode but if you try to write kubectl it would show bash error cause we need to add in binary file.
      a.	mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
      b.	kubectl
14.	Now it would not show any bash error and show the list of commands that can be used using kubectl.
15.	Now we need to create cluster and the cluster is been created using ‘eksctl’. For this we can find the installation guide from the official website https://eksctl.io/installation/. To install we gonna make a shell script file.
      a.	vim shell.sh
      b.	# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
            ARCH=amd64
            PLATFORM=$(uname -s)_$ARCH
            curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
            # (Optional) Verify checksum
            curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
            tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
            sudo mv /tmp/eksctl /usr/local/bin
16.	Now we gonna need to create the cluster.
      a.	eksctl create cluster --node-type=t3.medium --nodes=2
17.	It would take time to build, open aws and check cloudformation and check the cluster is been created or not.
18.	As the cluster is been created now we will create a yaml file 
      a.	vim pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```
17.	Now the yaml file has been created now we have to create the pod using it, to do it so we required this file to run
      a.	kubectl apply -f pod.yaml       # There is a script that do everyhing from step 8 step 17 --> [Script](https://github.com/Vaibhav-Shewale/fdec/blob/main/Shell%20Scripting/Kubect_eksctl%20%26%20cluster%20creation.md)
18.	It would show the message of pod creation, now to check the pod status so that to know if it is been created or now
      a.	kubectl get pod
19.	To know the details of the pod and different configuration added to it we will run
      a.	kubectl describe pod nginx nginx is the name of the pod
20.	To know the number of cluster 
      a.	eksctl get cluster 
21.	If you wanted to delete the cluster 
      a.	eksctl delete cluster --name cluster-name
