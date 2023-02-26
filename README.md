# Django application with DevOps practices #
Danylo Hotvianskyi entry-level portfolio project. Stack: Django, Docker, Ansible, Vagrant, Nagios, Github actions.

**Project is not finished. In progress...**

## Description of the project: ##
The idea of this project is to visualize the DevOps practices deploying the Django simple application using 3 different servers.

### Application ###
[Application](djangoapp/README.md) in the project is the Django To Do list application with the MariaDB database. The project idea is not about the application itself, it is just used as an example. 

Application on-line demo: [django.burava.com](https://django.burava.com)

### Plan: ###
* [Vagrant](vagrant/README.md) is used to launch 3 VM servers in VirtualBox:
  * *control* server is used to operate the other two servers. On this server, we will install Ansible and Nagios to monitor our project application.
  * on the *nodeapp* server we will deploy our application.
  * on the *nodedb* server we will deploy our application database.
* [Ansible](vagrant/ansible/README.md) is installed on the *control* server. Playbooks will install and configure **Nagios** on the *control* server, install **Docker** on the other two servers, and deploy the different images+containers on the application server and the database one.
* **Docker** will be used to deploy images and containers on the application server and the database one.
* **Nagios** will be used to monitor both the application and database performance.
* **GitHub Actions** will be used to represent the *CI/CD* practices to automatically build, test, and deploy the application whenever changes are made to the repository.
