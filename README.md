# Contents

- Introduction
  - [Pre-requisites](#pre-requisites)
  - [Installation & Configuration](#installation--configuration)
    - [Add Datasources & Dashboards](#add-datasources-and-dashboards)
    - [Install Dashboards the Old Way](#install-dashboards-the-old-way)
  	- [Alerting](#alerting)
    - [Add additional Datasources](#add-additional-datasources)
	  - [Grafana SMTP](#grafana-smtp)
  - [Security Considerations](#security-considerations)
  	- [Production Security](#production-security)

# A Prometheus & Grafana docker-compose stack

## Pre-requisites
Before we get started installing the Prometheus stack. Ensure you install the latest version of docker and [docker swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/) on your Docker host machine. Docker Swarm is installed automatically when using Docker for Mac or Docker for Windows.

# Installation & Configuration
Clone the project locally to your Docker host.

If you would like to change which targets should be monitored or make configuration changes edit the [/prometheus/prometheus.yml](prometheus/prometheus.yml) file. The targets section is where you define what should be monitored by Prometheus. The names defined in this file are actually sourced from the service name in the docker-compose file. If you wish to change names of the services you can add the `container_name` parameter in the `prometheus-stack.yml` file.

Once configurations are done let's start it up. From the project directory run the following command:

    $ docker stack deploy -c prometheus-stack.yml prometheus


That's it the `docker stack deploy` command deploys the entire Prometheus and Grafana stack automagically to the Docker Swarm. By default cAdvisor and node-exporter are set to Global deployment which means they will propogate to every docker host attached to the Swarm.

The Grafana Dashboard is now accessible via: `http://<Host IP Address>:3000` for example http://192.168.10.1:3000

	username - admin
	password - admin (You will be asked to change the Password in the first login)

In order to check the status of the newly created stack:

    $ docker stack ps prometheus

View running services:

    $ docker service ls

View logs for a specific service

    $ docker service logs prometheus_<service_name>

## Add Datasources and Dashboards
Grafana version 5.0.0 has introduced the concept of provisioning. This allows us to automate the process of adding Datasources & Dashboards. The [/grafana/provisioning/](grafana/provisioning/) directory contains the `datasources` and `dashboards` directories. These directories contain YAML files which allow us to specify which datasource or dashboards should be installed. 

If you would like to automate the installation of additional dashboards just copy the Dashboard `JSON` file to [/grafana/provisioning/dashboards/](grafana/provisioning/dashboards/) and it will be provisioned next time you stop and start Grafana.

## Install Dashboards the old way

I created a Dashboard template which is available on [Grafana Docker Dashboard](https://grafana.com/grafana/dashboards/8321). Simply select Import from the Grafana menu -> Dashboards -> Import and provide the Dashboard ID [#8321](https://grafana.com/grafana/dashboards/8321)

This dashboard is intended to help you get started with monitoring. If you have any changes you would like to see in the Dashboard let me know so I can update Grafana site as well.

Here's the Dashboard Template

![Grafana Dashboard](https://raw.githubusercontent.com/flaviomrjr/prometheus/master/images/nodes-swarm.PNG)

## Alerting
Alerting has been added to the stack with Slack integration. 1 Alert have been added and are managed

Alerts              - [/prometheus/alert.rules](prometheus/alert.rules)
Slack configuration - [/alertmanager/config.yml](alertmanager/config.yml)

The Slack configuration requires to build a custom integration.
* Open your slack team in your browser `https://<your-slack-team>.slack.com/apps`
* Click build in the upper right corner
* Choose Incoming Web Hooks link under Send Messages
* Click on the "incoming webhook integration" link
* Select which channel
* Click on Add Incoming WebHooks integration
* Copy the Webhook URL into the [/alertmanager/config.yml](alertmanager/config.yml) URL section
* Fill in Slack channel

View Prometheus alerts `http://<Host IP Address>:9090/alerts`
View Alert Manager `http://<Host IP Address>:9093`

## Add Additional Datasources
The Prometheus Datasource is being provisioned automagically. The [/grafana/provisioning/datasources](grafana/provisioning/datasources) directory contains the `datasources` that will be added.

If you need add a new Datasource in order to connect Grafana to Prometheus

* Click the `Grafana` Menu at the top left corner (looks like a fireball)
* Click `Data Sources`
* Click the green button `Add Data Source`.

<img src="https://raw.githubusercontent.com/flaviomrjr/prometheus/master/images/datasource.PNG" width="400" heighth="400">

## Grafana SMTP

SMTP configs can be set in [/grafana/config.monitoring](/grafana/config.monitoring).

The following variables must be set:

| VARIABLES | VALUES |
|-----------|--------|
|GF_SMTP_ENABLED|true
|GF_SMTP_HOST|smtp.server.com:smtp_port
|GF_SMTP_USER|mail@domail.com
|GF_SMTP_FROM_ADDRESS|mail@domail.com
|GF_SMTP_PASSWORD|password
|GF_SERVER_ROOT_URL|http://url.domain.com:3000
|GF_ROOT_URL|http://url.domain.com:3000

# Security Considerations
This project is intended to be a quick-start to get up and running with Docker and Prometheus. Security has not been implemented in this project. It is the users responsability to implement Firewall/IpTables and SSL.

Since this is a template to get started Prometheus and Alerting services are exposing their ports to allow for easy troubleshooting and understanding of how the stack works.

## Production Security:

Here are just a couple security considerations for this stack to help you get started.
* Remove the published ports from Prometheus and Alerting servicesi and only allow Grafana to be accessed
* Enable SSL for Grafana with a Proxy such as [jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/) Let's Encrypt
* Add user authentication via a Reverse Proxy [jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/) for services cAdvisor, Prometheus, & Alerting as they don't support user authenticaiton
* Terminate all services/containers via HTTPS/SSL/TLS
