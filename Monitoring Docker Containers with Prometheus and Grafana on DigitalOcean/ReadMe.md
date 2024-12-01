
# Docker Container Monitoring with Prometheus and Grafana

This project sets up monitoring for Docker containers using Prometheus and Grafana on a DigitalOcean Droplet. Prometheus collects metrics from the host and Docker containers, and Grafana provides a powerful interface for visualizing these metrics.

## Prerequisites

- DigitalOcean account
- Basic knowledge of Docker, Prometheus, and Grafana

## Project Setup

### 1. Create a DigitalOcean Droplet

1. Create a new Droplet on DigitalOcean with Ubuntu as the operating system.
2. SSH into the Droplet:
   ```
   ssh root@your_droplet_ip
   ```

### 2. Install Docker

1. Update the package database:
   ```
   sudo apt update
   ```
2. Install Docker:
   ```
   sudo apt install -y docker.io
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

### 3. Set Up Prometheus

1. Create a directory for Prometheus configuration:

   ```
   mkdir -p /opt/prometheus
   cd /opt/prometheus
   ```
2. Create a Prometheus configuration file (`prometheus.yml`):
   
   ```
   global:
     scrape_interval: 15s

   scrape_configs:
     - job_name: 'prometheus'
       static_configs:
         - targets: ['localhost:9090']

     - job_name: 'docker'
       static_configs:
         - targets: ['node_exporter:9100', 'cadvisor:8080']
   ```

3. Create a Docker network for Prometheus and its exporters:
   ```
   sudo docker network create monitoring
   ```

4. Run Prometheus in a Docker container:
   ```
   sudo docker run -d --name=prometheus --network=monitoring -p 9090:9090 -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus

   ```

### 4. Set Up Node Exporter

1. Run Node Exporter in a Docker container:
   ```
   sudo docker run -d --name=node_exporter --network=monitoring -p 9100:9100 prom/node-exporter
   ```

### 5. Set Up cAdvisor

1. Run cAdvisor in a Docker container:
   ```
   sudo docker run -d --name=cadvisor --network=monitoring -p 8080:8080 --volume=/:/rootfs:ro --volume=/var/run:/var/run:ro --volume=/sys:/sys:ro --volume=/var/lib/docker/:/var/lib/docker:ro google/cadvisor:latest
   ```

### 6. Set Up Grafana

1. Create a directory for Grafana data:
   ```
   mkdir -p /opt/grafana
   ```
2. Ensure the directory has the correct permissions:
   ```
   sudo chown -R 472:472 /opt/grafana
   ```
3. Run Grafana in a Docker container:
   ```
   sudo docker run -d --name=grafana --network=monitoring -p 3000:3000 -v /opt/grafana:/var/lib/grafana grafana/grafana
   ```

### 7. Configure Grafana

1. Access Grafana at `http://your_droplet_ip:3000` and log in with the default credentials (admin/admin). Change the password when prompted.
2. Add Prometheus as a data source:
   - Go to **Configuration** > **Data Sources**.
   - Click **Add data source**.
   - Select **Prometheus**.
   - Set the URL to `http://prometheus:9090`.
   - Click **Save & Test**.
3. Import a Grafana dashboard for Docker monitoring:
   - Go to **Create** > **Import**.
   - Use a dashboard ID from Grafana's [dashboard repository](https://grafana.com/grafana/dashboards) (e.g., 893 for Docker and system monitoring).
   - Click **Load**.
   - Select the Prometheus data source.
   - Click **Import**.

## Maintenance Tips

- Regularly update your Docker containers to ensure you have the latest features and security patches.
- Backup your Grafana configuration and dashboards to avoid data loss.
- Monitor the performance of Prometheus and Grafana themselves, as they can become resource-intensive.

## Next Steps

- Explore creating custom Grafana dashboards tailored to your specific monitoring needs.
- Set up alerting rules in Prometheus and configure Grafana to send alerts to your preferred notification channels (e.g., Slack, email).

## References

- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana Documentation](https://grafana.com/docs/grafana/latest/)
- [Docker Documentation](https://docs.docker.com/)
- [Node Exporter Documentation](https://github.com/prometheus/node_exporter)
- [cAdvisor Documentation](https://github.com/google/cadvisor)

---

Feel free to contribute to this project by submitting issues or pull requests. Your feedback and improvements are welcome!
