# Grafana on Wodby

What Wodby sets up for Grafana on this service. It runs from the official `grafana/grafana` image and listens on port 3000. Configuration is passed as `GF_<SECTION>_<KEY>` environment variables, which override the configuration file.

## What the manifest sets

- `GF_SERVER_ROOT_URL` is the service's URL in the environment.
- `GF_SECURITY_ADMIN_USER` and `GF_SECURITY_ADMIN_PASSWORD` come from the service tokens `admin_username` and `admin_password`. Grafana applies them once, on the first start with an empty data volume. Changing the tokens later does not change the existing account.
- Usage reporting and update checks are off (`GF_ANALYTICS_*`).

## Data sources

The service can be linked to Grafana Loki, Prometheus and VictoriaMetrics. The links set no variables and create no data source by themselves.

Data sources are declared in the config file "Grafana datasources", mounted at `/etc/grafana/provisioning/datasources/wodby.yml`. It is empty by default (`datasources: []`) and has `prune: true`: a data source removed from the file is deleted from Grafana on the next start. A linked service is reached at `http://<its service name>:<its port>`, for example port 3100 for Loki and 8428 for VictoriaMetrics. Grafana reads the file when it starts, so a change needs a deployment of the service.

## Changing configuration

Add or change `GF_*` environment variables on the service, or edit the data sources config file, then deploy. Do not edit `grafana.ini` in the container: it is not on a volume.

## Data

The `data` volume is mounted at `/var/lib/grafana`: the built-in SQLite database with dashboards, users and settings, and installed plugins. The manifest declares no backup or import.

## Check the result

`/api/health` on port 3000 reports whether Grafana and its database are up.
