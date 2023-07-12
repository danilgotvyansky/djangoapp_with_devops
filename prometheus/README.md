# Prometheus #

Prometheus is used to scrape all hosts performance metrics using the exporters. Then, these metrics are used by [Grafana](/grafana/README.md) for monitoring visualization and alerting.

**Used exporters:**
* [Node exporter](https://github.com/prometheus/node_exporter) - to scrape main machine metrics (CPU usage, Disk I/O, MEM usage, etc.). Launched on **all nodes**.
* [MySQLd exporter](https://github.com/prometheus/mysqld_exporter) - to scrape database-related metrics from our MariaDB containers located on **node1** and **node2**.
* [HAProxy exporter](https://github.com/prometheus/haproxy_exporter) - to scrape HAProxy load balancer related metrics from located on the **control2** server.
* [cAdvisor](https://github.com/google/cadvisor) - to scrape Docker/container-related metrics. Launched on **all nodes**
* Also, **Prometheus** scrape its own monitoring-related metrics.

### Installation ###

1. Create the [prometheus.yml](prometheus.yml) and [mariadb_exp_rec.yml](mariadb_exp_rec.yml) files and insert their contents from the repository.

`vagrant@control2:~$`
```shell
mkdir prom && cd prom
nano prometheus.yml
nano mariadb_exp_rec.yml 
```

[mariadb_exp_rec.yml](mariadb_exp_rec.yml) file is required for the proper working of the **MySQLd exporter**.

2. Launch Prometheus as a **docker container** ([image](https://hub.docker.com/r/prom/prometheus/)) with the additional options to mount our created config file and the recording rule file. Also, indicate the name of the container and assign the *unless-stopped* restart policy.
```commandline
sudo docker run -d -p 9090:9090 -v /home/vagrant/prom/prometheus.yml:/etc/prometheus/prometheus.yml -v /home/vagrant/prom/mariadb_exp_rec.yml:/prometheus/mariadb_exp_rec.yml --name djangoapp_prom --restart unless-stopped prom/prometheus
```

**Prometheus** should be working properly on [192.168.6.5:9090](http://192.168.6.5:9090)

3. Reconnect to the **node1** server and create the *exporter* user inside our MariaDB database with the appropriate permissions. Repeat the same steps on **node2**.

`vagrant@node1:~$` then `vagrant@node2:~$`
```shell
sudo docker exec -it mariadb bash
mysql -u root --password="<your-mysqlroot-pass>"
```

```sql
CREATE USER 'exporter'@'<your-machine-ip>' IDENTIFIED BY '<your-password>' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'<your-machine-ip>';
GRANT REPLICATION SLAVE ON *.* TO 'exporter'@'<your-machine-ip>';
FLUSH PRIVILEGES;
```

**NOTE:** DON'T USE `@` or `?` in PASSWORD - [KNOWN BUG](https://github.com/prometheus/mysqld_exporter/issues/303).

4. Create the [my.cnf](my.cnf) file and insert the content from the repository updating the credentials with the used ones on **Step 3**. Repeat the same steps on **node2**.

`vagrant@node1:~$` then `vagrant@node2:~$`
```shell
mkdir mariadb-exporter && cd mariadb-exporter
nano my.cnf
```

5. Launch MySQLd exporter as a **docker container** ([image](https://hub.docker.com/r/prom/mysqld-exporter)) on both nodes with the additional options to mount and activate our created config file, assign the *unless-stopped* restart policy. `--collect.*` parameters activate the exact scrapers on the exporter. 

`vagrant@node1:~$` then `vagrant@node2:~$`
```commandline
sudo docker run -d --name mariadb-exporter -p 9104:9104 -v /home/vagrant/mariadb-exporter/my.cnf:/cfg/.my.cnf --restart unless-stopped prom/mysqld-exporter:latest --config.my-cnf=/cfg/.my.cnf --collect.info_schema.processlist --collect.info_schema.innodb_metrics --collect.info_schema.tablestats --collect.info_schema.tables --collect.info_schema.userstats --collect.engine_innodb_status --collect.slave_status --collect.slave_hosts
```

After that, you should see the `mariadb-exporter 2/2 UP` in your **Prometheus UI** >> **Status** >> **Targets** menu. 

6. Launch Node exporter as a **docker container** ([image](https://hub.docker.com/r/prom/node-exporter)) on all machines.

`vagrant@*:~$`
```commandline
sudo docker run -d -p 9100:9100 --name node-exporter --restart unless-stopped prom/node-exporter
```

After that, you should see the `node-exporter 4/4 UP` in your **Prometheus UI** >> **Status** >> **Targets** menu. 

7. Launch cAdvisor as a **docker container** ([image](https://console.cloud.google.com/gcr/images/cadvisor/GLOBAL/cadvisor?pli=1)) on all machines with the additional options to run container in a *privileged* mode and to mount the appropriate for this service volume.

`vagrant@*:~$`
```commandline
sudo docker run -d -p 8080:8080 --restart unless-stopped  --name cadvisor --privileged --volume=/var/run/docker.sock:/var/run/docker.sock gcr.io/cadvisor/cadvisor:latest
```

After that, you should see the `cadvisor 4/4 UP` in your **Prometheus UI** >> **Status** >> **Targets** menu and be able to see the containers' statistics on *http://<your-machine-ip>:8080/containers*.

8. Launch HAProxy exporter as a **docker container** ([image](https://hub.docker.com/r/prom/haproxy-exporter)) on the **control2** machine with the additional options to indicate the scrape URI.

`vagrant@control2:~$`
```commandline
sudo docker run -d -p 9101:9101 --restart unless-stopped --name haproxy-exporter quay.io/prometheus/haproxy-exporter:latest --haproxy.scrape-uri="http://haproxy:root@192.168.6.5:1935/stats?stats;csv"
```

After that, you should see all exporters `UP` in your **Prometheus UI** >> **Status** >> **Targets** menu.

Now, let's install [Grafana](/grafana/README.md) to visualize our metrics.