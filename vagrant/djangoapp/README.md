# Django To-Do list application # 
Application on-line demo: [django.burava.com](https://django.burava.com)

Application uses **MariaDB**(MySQL) database. Other requirements can be found [here](requirements.txt).

## Docker-related files used in the application: ##
* [docker-compose.yaml](docker-compose.yaml)
* [Dockerfile](Dockerfile)
* [wait-for-it.sh](https://github.com/vishnubob/wait-for-it)

## Installation process ##
Our application will be deployed on 2 nodes (node1 & node2). Each node will contain 2 *Docker containers*: one for application and one for the database.

It is planned that later **GitHub actions** will automatically deploy our containers from our *control* machine (which will be set as the *self-hosted runner*) using the automatically built images which will be pushed to the GitHub Registry as well. 

For now, I will show how to deploy application without GitHub actions as it hasn't been configured yet.

1. Connect to your application server (*node1*).
```
vagrant ssh node1 # from your personal PC
ssh node1 # if you are already connected to control
```

For now, all the project files are already available in the *vagrant/* folder which won't be the same in real life when you need to deploy application on the remote server, so let's clone the git repository to make it *"more realistic"* 

2. Clone the git repository:
```
git clone https://github.com/danilgotvyansky/djangoapp_with_devops .
```

If you have followed the steps from the [Ansible tutorial](../ansible/README.md), you should already have **Docker** and **Docker-compose** installed on your nodes.
3. Create the *.env* file:
```
cd vagrant/djangoapp && nano .env
```
Add the following content to it:
```
MARIADB_DATABASE=<db_name>
MARIADB_USER=<db_user>
MARIADB_PASSWORD=<db_password>
MARIADB_ROOT_PASSWORD=<db_root_password>
DB_HOST=db
DB_PORT=3306 #or you can set a custom one 
DB_NAME=<db_name> #here and below same as for MARIADB_*
DB_USER=<db_user> 
DB_PASSWORD=<db_password>
DJANGO_SUPERUSER_PASSWORD=<django_admin_password>
DJANGO_SUPERUSER_USERNAME=<django_admin_username> 
DJANGO_SUPERUSER_EMAIL=<djanto_admin_email>
```
4. Build and launch docker containers:
```
cd vagrant/djangoapp
docker-compose build && docker-compose up
```
You should be able to open the application on the http://{your_node1_IP}:8000 link.

Later I will add the application image to the *GitHub Registry* and re-create the [docker-compose.yaml](docker-compose.yaml) file to build the container without building an image.

