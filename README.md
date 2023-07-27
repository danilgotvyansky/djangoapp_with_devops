123
# Django application with DevOps practices #
Danylo Hotvianskyi portfolio project. Stack: Django, Docker, Ansible, Vagrant, Prometheus, GitHub actions, HAProxy Load Balancer, Grafana.

## Description of the project: ##
The idea of this project is to visualize the DevOps practices by deploying the Django simple application using 4 different servers.

### Application ###
[Application](vagrant/djangoapp/README.md) in the project is the Django To Do list application with the MariaDB database. The project idea is not about the application itself, it is just used as an example. 

Application on-line demo: [django.burava.com](https://django.burava.com)

### Project Diagram ###
![Image1](plan1.svg)

### Plan: ###
[Vagrant](vagrant/README.md) is used to launch VM cluster of 4 servers in VirtualBox:

Operation functionalities are divided between 2 control servers.
* *control1* server is used to operate the other two servers. On this server, we will install Ansible and Grafana to visualize the scraped monitoring metrics.
* *control2* server is used to balance the load of our project using HAProxy Load Balancer and scrape the monitoring metrics from the exporters by Prometheus. 

Application is deployed on 2 servers:
* *node1* is used to deploy our application. 
* *node2* is a full copy of the first one to ensure Fault Tolerance.

This step above is done to imagine that we have 4 different physical servers in our company which all are completely empty and no services are installed on them. And yes, there is no actual sense in this step if you are going to manage your servers from your personal PC. Consider this step as *Creating a lab* or *VM cluster*.
* [Ansible](vagrant/ansible/README.md) is installed on the *control1* server. Playbook installs **Docker&Docker-compose** on the other servers.
* [Docker](vagrant/djangoapp/README.md) is used to deploy our application, database, exporters, HAProxy, Prometheus and Grafana as containers on our servers.
* [GitHub Actions](.github/workflows/README.md) is used to represent the *CI/CD* practices to automatically build, test, and deploy the application whenever changes are made to the repository. We will register our servers as the **GitHub self-hosted runners** to make them listen to our repository and once there is a new push/pull request on it, it will trigger the pipeline on the *control1*, *node1* and *node2* servers. 
 
This [pipeline](.github/workflows/djangoapp.yml) will test our application, build an image of it and push it to the *GitHub Registry*. Then the nodes will pull that image and build containers using it. 
* [Prometheus](prometheus/README.md) is used to scrape monitoring metrics from all deployed exporters. It is deployed as Docker container on the *control2* server.
* [Grafana](grafana/README.md) is used to visualize our monitoring metrics scraped by Prometheus and notify once there are any alerts. It is deployed as Docker container on the *control1* server.
* [HAProxy Load balancer](haproxy/README.md) is used to route the application traffic to node2 if node1 is down for some reason.
* To ensure the *database synchronization* between node1 and node2, the [Master-master Replication](replication.md) has been built.
