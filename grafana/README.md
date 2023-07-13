# Grafana #

Grafana is used to visualize our monitoring metrics scraped by [Prometheus](/prometheus/README.md) and alert us in case of any incidents in our cluster.

### Installation ###

1. Launch Grafana as a **docker container** ([image](https://hub.docker.com/r/grafana/grafana)) on the **control1** machine with an additional option to install the *Grafana clock* plugin:

```commandline
sudo docker run -d -p 3000:3000 --name grafana -e GF_INSTALL_PLUGINS=grafana-clock-panel --restart unless-stopped grafana/grafana
```
2. Make sure **Prometheus** data source is properly configured by checking if the correct IP is specified in *Administration >> Data Sources >> the Prometheus server* URL field.
3. Install some basic dashboards to monitor our services:
   * [Node Exporter Full](https://grafana.com/grafana/dashboards/1860-node-exporter-full/)
   * [MySQL Overview](https://grafana.com/grafana/dashboards/7362-mysql-overview/)
   * [HAProxy](https://grafana.com/grafana/dashboards/2428-haproxy/)
   * [Docker Container & Host Metrics](https://grafana.com/grafana/dashboards/10619-docker-host-container-overview/)
   * [Prometheus 2.0 Overview](https://grafana.com/grafana/dashboards/3662-prometheus-2-0-overview/)

It is planned that here will appear dashboards created by me precisely for this project and the alerting will be managed by Grafana too.