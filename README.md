# Docker Compose Monitoring Stack prometheus-grafana

This repository provides a Docker Compose configuration for setting up a monitoring stack consisting of Prometheus, Grafana, Node Exporter, and cAdvisor. This stack is designed for monitoring and visualizing metrics from your Docker containers and host system.

## Services

### 1. Prometheus
- **Image**: `prom/prometheus:v2.37.9`
- **Container Name**: `monitor-prometheus`
- **Volumes**:
  - `./prometheus.yml:/etc/prometheus/prometheus.yml`
  - `./data:/prometheus`
- **Command**:
  - `--config.file=/etc/prometheus/prometheus.yml`
  - `--storage.tsdb.path=/prometheus`
  - `--web.console.libraries=/etc/prometheus/console_libraries`
  - `--web.console.templates=/etc/prometheus/consoles`
  - `--web.enable-lifecycle`
- **Networks**: `internal`, `web`
- **Restart Policy**: `unless-stopped`
- **Traefik Labels**: Configured for Traefik with HTTPS support.

### 2. Grafana
- **Image**: `grafana/grafana-oss:latest`
- **Container Name**: `monitor-grafana`
- **Volumes**:
  - `./grafana-data:/var/lib/grafana`
- **Networks**: `internal`, `web`
- **Restart Policy**: `unless-stopped`
- **Traefik Labels**: Configured for Traefik with HTTPS support.

### 3. Node Exporter
- **Image**: `prom/node-exporter:latest`
- **Container Name**: `monitor-node-exporter`
- **Volumes**:
  - `/proc:/host/proc:ro`
  - `/sys:/host/sys:ro`
  - `/:/rootfs:ro`
- **Command**:
  - `--path.procfs=/host/proc`
  - `--path.rootfs=/rootfs`
  - `--path.sysfs=/host/sys`
  - `--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)`
- **Networks**: `internal`, `web`
- **Restart Policy**: `unless-stopped`

### 4. cAdvisor
- **Image**: `gcr.io/cadvisor/cadvisor:v0.47.0`
- **Container Name**: `monitor-cadvisor`
- **Volumes**:
  - `/:/rootfs:ro`
  - `/var/run:/var/run:ro`
  - `/sys:/sys:ro`
  - `/var/lib/docker/:/var/lib/docker:ro`
  - `/dev/disk/:/dev/disk:ro`
- **Devices**:
  - `/dev/kmsg`
- **Privileged**: `true`
- **Restart Policy**: `unless-stopped`
- **Networks**: `internal`, `web`

## Networks

- **internal**: Internal network for communication between containers.
- **web**: External network for access via Traefik.

## Usage

1. **Clone this repository**:
   ```sh
   git clone <repository-url>
   cd <repository-directory>
   ```

2. **Create and start the containers**:
   ```sh
   docker-compose up -d
   ```

3. **Access the services**:
   - **Prometheus**: `http://<traefik-router-domain>/`
   - **Grafana**: `http://<traefik-router-domain>/`
   - **Node Exporter**: Exposed through Prometheus metrics.
   - **cAdvisor**: Exposed through Prometheus metrics.

## Configuration

- **Prometheus**: Modify `prometheus.yml` for Prometheus configuration.
- **Grafana**: Data will be persisted in `./grafana-data`.

## Traefik Integration

This stack uses Traefik for reverse proxy and HTTPS. Ensure that Traefik is properly configured and running in your environment.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
