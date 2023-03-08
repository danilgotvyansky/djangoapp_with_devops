# Multi-master MariaDB replication #

### What do we have: ###
**2 servers:**
* node1 (application container + database container on *192.168.6.3:3306*)
* node2 (application container + database container on *192.168.6.4:3306*)

### What do we need: ###
Multi-master replication on **node1**&**node2**.

## Step-by-step guide on how to configure Multi-master MariaDB replication: ##
1. Enter your mariadb docker container interactively on both servers:
```
sudo docker exec -it <container-id> bash
```
2. Install nano for both servers:
```
apt update && apt install nano
```
3. Edit the *etc/mysql/my.cnf* file with the values:
```
nano etc/mysql/my.cnf
```
For **node1**:
```
# under the [mysqld] part 
server-id               = 1
log_bin                 = /var/log/mysql/mariadb-bin
```
For **node2**:
```
# under the [mysqld] part 
server-id               = 2
log_bin                 = /var/log/mysql/mariadb-bin
```
4. Logout from the containers and restart them:
```
exit
sudo docker restart <container-id> 
```
5. Enter your mariadb container again on the **node1** and log in to the MariaDB server:
```
sudo docker exec -it <container-id> bash
mysql -u root --password="<your-mysqlroot-pass>"
```

6. Create a replication user for **node2** on **node1** and grant replication slave for it:
```
create user 'repl'@'192.168.6.4' identified by '<your-pass>';
grant replication slave on *.* to 'repl'@'192.168.6.4';
```
7. Run this query and make sure to save the values:
```
show master status  \g
```

You should see something like that:
```
MariaDB [(none)]> show master status \g
+--------------------+----------+--------------+------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+--------------------+----------+--------------+------------------+
| mariadb-bin.000001 |      330 |              |                  |
+--------------------+----------+--------------+------------------+
```

NOTE: Make sure not to quit the MariaDB server to prevent these values from being updated.
8. Enter your mariadb container again on the **node2** and log in to the MariaDB server:
```
sudo docker exec -it <container-id> bash
mysql -u root --password="<your-mysqlroot-pass>"
```
9. Set your **node2** replication source values:
```
change master to master_host='192.168.6.3', master_user='repl', master_password='<your-pass>', master_port=3306, master_log_file='<your-saved-file-on-step7>', master_log_pos=<your-saved-pos-on-step7>, master_connect_retry=60;
```

If you are doing that not for the first time, you might need to stop the slaves **prior** this:
```
stop slave;
```
10. Start **master**(node1)-**slave**(node2) and check its status:
```
start slave;
show slave status \G
```

If replication is working correctly, both the values of *Slave_IO_Running* and *Slave_SQL_Running* should be Yes.
11. Create a replication user for **node1** on **node2** and grant replication slave for it:
```
create user 'repl'@'192.168.6.3' identified by '<your-pass>';
grant replication slave on *.* to 'repl'@'192.168.6.3';
```
12. Run this query and make sure to save the values:
```
show master status  \g
```
13. Use your **node1** MariaDB shell and set your **node1** replication source values:
```
change master to master_host='192.168.6.4', master_user='repl', master_password='<your-pass>', master_port=3306, master_log_file='<your-saved-file-on-step12>', master_log_pos=<your-saved-pos-on-step12>, master_connect_retry=60;
```
14. Start **master**(node2)-**slave**(node1) and check its status:
```
start slave;
show slave status \G
```

If replication is working correctly, both the values of *Slave_IO_Running* and *Slave_SQL_Running* should be Yes.

If your application container was not running during this process, it might be required to rebuild the application container because of this error:
```
Error response from daemon: Cannot link to a non running container: /mariadb AS /djangoapp/db
Error: failed to start containers: <container-id>
```
You can rebuild the container by running these commands:
```
sudo docker rm -f djangoapp
echo "<git-PAT>" | sudo docker login ghcr.io -u "<git-username>" --password-stdin
sudo docker pull ghcr.io/danilgotvyansky/djangoapp_with_devops:latest
sudo docker run --name djangoapp -d \
>   -e DB_NAME=<db-name-from-secrets> \
>   -e DB_USER=<db-user-from-secrets> \
>   -e DB_PASSWORD="<db-pass-from-secrets>" \
>   -e DB_HOST=<your-node-IP> \ 
>   -e DB_PORT=3306 \
>   -e DJANGO_SECRET_KEY="<django-secret-from-secrets>" \
>   -e DEBUG=TRUE \
>   -p 8000:8000 \
>   --link mariadb:db \
>   ghcr.io/danilgotvyansky/djangoapp_with_devops:latest \
>   sh -c "python manage.py migrate &&
>          python manage.py collectstatic --noinput &&
>          python manage.py runserver 0.0.0.0:8000"
```

After all these steps, the Multi-master MariaDB replication should work properly for you. 