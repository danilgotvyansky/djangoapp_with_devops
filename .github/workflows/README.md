# GitHub Actions #
## Description ##
In our application **GitHub Actions** is used to automatically build, test and deploy our [Django application](https://github.com/danilgotvyansky/djangoapp_with_devops/blob/main/vagrant/djangoapp/README.md) to our servers.

CI/CD practices are all represented in one file - [djangoapp.yml](djangoapp.yml)

The workflow is being triggered by the pull requests and push (only when changes are made within the *vagrant/djangoapp/*** path) on the **main** branch.

There **3 jobs** inside:
1. *build:* It actually builds the application on the *control1* server, tests it and then push the image to the **GitHub Registry**.
2. *deploynode1:* It pulls the application image from the **GitHub Registry** and builds the containers using the database and application images on the *node1*.
3. *deploynode2:* It pulls the application image from the **GitHub Registry** and builds the containers using the database and application images on the *node2*.

All three nodes are registered as the *self-hosted runners*. 

Possibly, it is a temporary solution that I will remake later by changing the way of the image deployment to the servers.

##  Pre-running steps ##
1. Register all three machines as the *self-hosted runners*.
2. Make sure that the **gnupg2** is installed and configured on all three servers (*recommended to do from the ~/actions-runner folder*):
```
sudo apt-get install pass gnupg2
gpg2 --gen-key
pass init $gpg_id
```
3. If you have followed the steps from the [Ansible tutorial](https://github.com/danilgotvyansky/djangoapp_with_devops/blob/main/vagrant/ansible/README.md), you should already have **Docker** and **Docker-compose** installed on your nodes.
4. Make sure to create all the required **secrets** from [djangoapp.yml](djangoapp.yml) in the GitHub repository. Also, make sure to change the **DB_HOST** variable values to the ones corresponding to your servers' IPs.

## After-running steps ##
Don't forget to create your **Django Admin SuperUser**:

1. Find your docker container ID:
```
sudo docker container ls
```

Copy the ID of your *application_1* Docker container here.
2. Enter your container bash interactively:
```
sudo docker exec -it <your container id> bash
```
3. Create the Django Admin SuperUser:
```
python manage.py createsuperuser
```

You can check if the application works properly on:
http://{your_node_IP}:8000/

P.S: docker-compose.yaml is not used in the workflow because there is an issue when the node tries authenticating to Docker.