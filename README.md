# NGINX Prometheus Exporter

NGINX Prometheus exporter makes it possible to monitor NGINX or NGINX Plus using Prometheus.

## Overview

[NGINX](http://nginx.org) exposes a handful of metrics via the [stub_status page](http://nginx.org/en/docs/http/ngx_http_stub_status_module.html#stub_status). [NGINX Plus](https://www.nginx.com/products/nginx/) provides a richer set of metrics via the [API](https://nginx.org/en/docs/http/ngx_http_api_module.html) and the [monitoring dashboard](https://www.nginx.com/products/nginx/live-activity-monitoring/). NGINX Prometheus exporter fetches the metrics from a single NGINX or NGINX Plus, converts the metrics into appropriate Prometheus metrics types and finally exposes them via an HTTP server to be collected by [Prometheus](https://prometheus.io/).

**Note**: Currently it is only supported to run NGINX Prometheus Exporter in a Docker container.

## Getting Started

In this section, we show how to quickly run NGINX Prometheus Exporter in a container for NGINX or NGINX Plus.

### A Note about NGINX Ingress Controller

If you’d like to use the NGINX Prometheus Exporter with [NGINX Ingress Controller](https://github.com/nginxinc/kubernetes-ingress/) for Kubernetes, see [this doc](https://github.com/nginxinc/kubernetes-ingress/blob/master/docs/installation.md#5-access-the-live-activity-monitoring-dashboard) for the installation instructions.

### Prerequisites

We assume that you have already installed Prometheus and NGINX or NGINX Plus. Additionally, you need to:
* Expose the built-in metrics in NGINX/NGINX Plus:
    * For NGINX, expose the [stub_status page](http://nginx.org/en/docs/http/ngx_http_stub_status_module.html#stub_status) at `/stub_status` on port `8080`.
    * For NGINX Plus, expose the [API](https://nginx.org/en/docs/http/ngx_http_api_module.html#api) at `/api` on port `8080`.
* Install Docker on the server where you’re planning to run the exporter.
* Configure Prometheus to scrape metrics from the server with the exporter. Note that the default scrape port of the exporter is `9113` and the default metrics path -- `/metrics`.

### Running the Exporter in a Container

To start the exporter we use the [docker run](https://docs.docker.com/engine/reference/run/) command.

* To export NGINX metrics, run:
    ```
    $ docker run -p 9113:9113 nginx/nginx-prometheus-exporter:0.1.0 -nginx.scrape-uri http://<nginx>:8080/api
    ```
    where `<nginx>` is the IP address/DNS name, through which NGINX is available.

* To export NGINX Plus metrics, run:
    ```
    $ docker run -p 9113:9113 nginx/nginx-prometheus-exporter:0.1.0 -nginx.plus -nginx.scrape-uri http://<nginx-plus>:8080/api
    ```
    where `<nginx-plus>` is the IP address/DNS name, through which NGINX Plus is available.

## Usage

### Command-line Arguments 

```
Usage of ./nginx-prometheus-exporter:
  -nginx.plus
        Start the exporter for NGINX Plus. By default, the exporter is started for NGINX.
  -nginx.scrape-uri string
        A URI for scraping NGINX or NGINX Plus metrics.
        For NGINX, the stub_status page must be available through the URI. For NGINX Plus -- the API. (default "http://127.0.0.1:8080/stub_status")
  -web.listen-address string
        An address to listen on for web interface and telemetry. (default ":9113")
  -web.telemetry-path string
        A path under which to expose metrics. (default "/metrics")
```

### Exported Metrics 

* For NGINX, all stub_status metrics are exported. Connect to the `/metrics` page of the running exporter to see the complete list of metrics along with their descriptions.
* For NGINX Plus, the following metrics are exported:
    * [Connections](http://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_connections).
    * [HTTP](http://nginx.org/en/docs/http/ngx_http_api_module.html#http_).
    * [SSL](http://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_ssl_object).
    * [HTTP Server Zones](http://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_http_server_zone).
    * [HTTP Upsteams](http://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_http_upstream). Note: for the `state` metric, the string values are converted to float64 using the following rule: `"up"` -> `1.0`, `"draining"` -> `2.0`, `"down"` -> `3.0`, `"unavail"` –> `4.0`, `"checking"` –> `5.0`, `"unhealthy"` -> `6.0`.
    
    Connect to the `/metrics` page of the running exporter to see the complete list of metrics along with their descriptions. Note: to see server zones related metrics you must configure [status zones](https://nginx.org/en/docs/http/ngx_http_status_module.html#status_zone) and to see upstream related metrics you must configure upstreams with a [shared memory zone](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone).

### Troubleshooting

The exporter logs errors to the standard output. If the exporter doesn’t work as expected, check its logs using [docker logs](https://docs.docker.com/engine/reference/commandline/logs/) command.

## Releases

For each release, we publish the corresponding Docker image at `nginx/nginx-prometheus-exporter` [DockerHub repo](https://hub.docker.com/r/nginx/nginx-prometheus-exporter/). See the GitHub [releases page](https://github.com/nginxinc/nginx-prometheus-exporter/releases) for the list of all releases and the [CHANGELOG.md](CHANGELOG.md) for the changelog.

## Building the Container Image

You can build the exporter image using the provided Makefile. Before building the image, make sure the following software is installed on your machine:
* make
* Docker
* git
 
To build the image, run:
```
$ make container 
```

Note:
* golang is not required, as the exporter binary is built in a Docker container. See the [Dockerfile](Dockerfile).
* The Makefile supports a few variables which can override its default behaviour. See [its content](Makefile).

## Support

The commercial support is available for NGINX Plus customers when the NGINX Prometheus Exporter is used with NGINX Ingress Controller.

## License

[Apache License, Version 2.0](LICENSE).
