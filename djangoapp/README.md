# Django To-Do list application # 
Application on-line demo: [django.burava.com](https://django.burava.com)

Application uses **MariaDB**(MySQL) database. Other requirements can be found [here](requirements.txt).
## Installation process ##
1. Connect to your application server (nodeapp) Docker container.
2. Clone the git repository:
```
git clone https://github.com/danilgotvyansky/djangoapp_with_devops
```
3. Check if *python* is installed:
```
python --version
```
If not, run these commands:
```
sudo apt update
sudo apt install python
```
4. cd to the **djangoapp/todo_project** directory and create the *.env* file. Then, fill in the file with your information:
```
SECRET_KEY=<django_secret_key>
DEBUG=<True> or <False>
DB_NAME=<todo_app> #if you want to use the example database
DB_USER=<database_user>
DB_PASSWORD=<database_user_password>
DB_HOST=<database_server_ip_address>
DB_PORT=3306
```
<database_server_ip_address> corresponds to the **nodedb** server IP address.
5. If you want to use the example database, cd to the **djangoapp** directory and run:
```
scp -P 21099 todo_app.sql vagrant@192.168.6.4:home/vagrant
```
6. Connect to your database server.
7. Install MariaDB:
```
sudo apt update
sudo apt install mariadb-server
```
8. Configure MariaDB:
```
sudo mysql_secure_installation
sudo systemctl status mariadb
sudo mysqladmin version
```
9. Login to your MariaDB server:
```
sudo mariadb -u root -p
```
10. Create administrative database user:
```
MariaDB [(none)]> CREATE USER 'admin'@'localhost' IDENTIFIED BY 'yourpassword';
MariaDB [(none)]> GRANT ALL ON *.* TO 'admin'@'localhost’;
MariaDB [(none)]> CREATE USER 'admin'@'%' IDENTIFIED BY 'yourpassword';
MariaDB [(none)]> GRANT ALL ON *.* TO 'admin'@'%’;
```
11. Create database for the application:
```
MariaDB [(none)]> CREATE DATABASE <database-name> CHARACTER SET UTF8 COLLATE UTF8_BIN;
```
Use the **<database-name>** from the **step 4**.
12. Create the database user:
```
MariaDB [(none)]> CREATE USER ‘<database_user_name>'@'localhost' IDENTIFIED BY 'yourpassword';
MariaDB [(none)]> GRANT ALL ON <database-name>.* to ‘<database_user_name>'@'localhost' IDENTIFIED BY 'yourpassword' WITH GRANT OPTION;
MariaDB [(none)]> CREATE USER ‘<database_user_name>'@'%' IDENTIFIED BY 'yourpassword';
MariaDB [(none)]> GRANT ALL ON <database-name>.* to ‘<database_user_name>'@'%' IDENTIFIED BY 'yourpassword' WITH GRANT OPTION;
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> EXIT;
```
Use the **<database-name>**, **<database_user_name>** from the **step 4**.
13. Run:
```
sudo ufw allow 3306 & ufw reload
```
14. If you want to use the example database, run:
```
sudo mariadb -u root -p < /vagrant/todo_app.sql
```
You can check if it has been successfully restored by running:
```
sudo mariadb -u root -p
MariaDB [(none)]> use todo_app
MariaDB [(todo_app)]> show tables
```
15. Re-connect to the application server and cd back to the **djangoapp** folder to enter into your python venv:
```
python -m venv venv
venv\Scripts\activate.bat
```
16. Install all the dependencies from **requirements.txt**:
```
pip install -r requirements.txt
```
17. If required, run:
```
python manage.py makemigrations
python manage.py migrate
```
18. Activate your application server:
```
python manage.py runserver
```