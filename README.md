# Django application with DevOps practices #
Danylo Hotvianskyi entry-level portfolio project. Stack: Django, Docker, Ansible, Vagrant, Nagios, GitHub actions, HAProxy Load Balancer.

## Description of the project: ##
The idea of this project is to visualize the DevOps practices by deploying the Django simple application using 4 different servers.

### Application ###
[Application](vagrant/djangoapp/README.md) in the project is the Django To Do list application with the MariaDB database. The project idea is not about the application itself, it is just used as an example. 

Application on-line demo: [django.burava.com](https://django.burava.com)

### Project Diagram ###
![Image1](plan.png)

### Plan: ###
* [Vagrant](vagrant/README.md) is used to launch 4 VM servers in VirtualBox:
  * *control* server is used to operate the other two servers. On this server, we will install Ansible and Nagios to monitor our project application.
  * *node1* is used to deploy our application. 
  * *node2* is a full copy of the first one to ensure High Availability.
  * *haproxy* server is used to balance the load of our project using HAProxy Load Balancer.

This step above is done to imagine that we have 4 different physical servers in our company which all are completely empty and no services are installed on them. And yes, there is no actual sense in this step if you are going to manage your servers from your personal PC. Consider this step as *Creating a lab*.
* [Ansible](vagrant/ansible/README.md) is installed on the *control* server. Playbooks will install and configure **Nagios** on the *control* server, and install **Docker&Docker-compose** on the other two servers.
* [Docker](vagrant/djangoapp/README.md) is used to deploy our application and database as two different containers on our both servers.
* [GitHub Actions](.github/workflows/README.md) is used to represent the *CI/CD* practices to automatically build, test, and deploy the application whenever changes are made to the repository. We will register our servers as the **GitHub self-hosted runners** to make them listen to our repository and once there is a new push/pull on it, it will trigger the pipeline on the *control*, *node1* and *node2* servers. 
 
This [pipeline](.github/workflows/djangoapp.yml) will test our application, build an image of it and push it to the *GitHub Registry*. Then the nodes will pull that image and build containers using it. 
* [Nagios](nagios/README.md) is used to monitor both servers' performance. Also, it will monitor our application demo and notify us when there is any issue.
* [HAProxy Load balancer](haproxy/README.md) is used to route the application traffic to node2 if node1 is down for some reason.
* To ensure the *database synchronization* between node1 and node2, the [Master-master Replication](replication.md) has been built.
