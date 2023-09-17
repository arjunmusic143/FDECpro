•	Create a cloudwatch that will upload automatically logs in cloudwatch. Use Nginx.

1.	For this we will use a new instance.
2.	OPEN EC2.
3.	We will create a normal instance with HTTP active cause we are using nginx service.
4.	Install nginx and clowdwatch. Nginx will work as a web server and clowdwatch is our agent from aws that will send data logs to services.
i.	sudo yum install
ii.	sudo yum install nginx -y
iii.	sudo yum install amazon-cloudwatch-agent.x86_64 -y
5.	Now go back to your EC2 page and select the instance on which you’re working on and then go to Actions>Security>Modify IAM Security.
6.	Now Select a role that has cloudwatch policy or just click on `Create a new IAM role`. It would it in the new page.
7.	In IAM role page, click on create a new role.
8.	Select AWS service and for use case select Ec2 and click o Next.
9.	Search for cloudwatchfullaccess and add it in the policy and click on create the role. Now you can close this tab.
10.	We’re back at Modify IAM role page for the instance, click on the refresh and check the drop down that you have created it would shown and select the role and click on ‘update IAM role’ and it will add the role in your instance.
11.	 Go back to your instance and open aws cloudwatch demon and create a json file.
i.	cd /opt/aws/amazon-cloudwatch-agent/etc/
ii.	vim amazon-cloudwatch-agent.json
```
"agent": {
    "metrics_collection_interval": 10,
    "run_as_user": "cwagent"
    },
      "logs": {
        "logs_collected": {
          "files": {
            "collect_list": [
              {
                "file_path": "/var/log/nginx/access.log",
                "log_group_name": "access.log",
                "log_stream_name": "access.log",
                "timezone": "UTC"
              },
              {
                "file_path": "/var/log/nginx/error.log",
                "log_group_name": "error.log",
                "log_stream_name": "error.log",
                "timezone": "UTC"
              }
            ]
          }
        },
        "log_stream_name": "my_log_stream_name",
        "force_flush_interval" : 15
        }
    }
```
12.	save the file and go to cloudwatch service. Check if the two files are present if not check the previous steps again.
