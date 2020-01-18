# Monitoring Swarm Ansible

This repository shows how to setup a monitoring stack running in Docker Swarm using Ansible to automate the task. This stack consists of the following well known technologies:

+ [Prometheus](https://prometheus.io/)
+ [AlertManager](https://prometheus.io/docs/alerting/alertmanager/)
+ [cAdvisor](https://github.com/google/cadvisor)
+ [Node Exporter](https://github.com/prometheus/node_exporter)
+ [Grafana](https://grafana.com/)

The services in this stack are also configured to work with the Traefik reverse proxy. The [proxy-swarm-ansible](https://github.com/KeithWilliamsGMIT/proxy-swarm-ansible) repository can be used to setup Traefik.

## Getting Started

The first step to getting started is to clone the repository:

```bash
git clone https://github.com/KeithWilliamsGMIT/monitoring-swarm-ansible.git
```

Next, install Ansible and then we can use two existing Ansible roles to install Docker and initialise a Docker Swarm. These roles are defined in the `requirements.yml` file and will be installed to `~/.ansible/roles/` by default using the below commands:

```bash
cd playbooks/requirements
ansible-galaxy install -r requirements.yml
```

We can install [Docker](https://docs.docker.com/install/) on all the hosts now using the `install-docker.yml` playbook as shown below:

```bash
cd playbooks
ansible-playbook -i ./inventories/localhost install-docker.yml
```

Similarly, to initialise a Docker Swarm between the hosts we can run the `initialize-swarm.yml` playbook. Note that we set extra variables to override some defaults in the playbook. We want to skip everything except the actual initialisation of Docker Swarm.

```bash
ansible-playbook -i ./inventories/localhost initialize-swarm.yml --extra-vars="{'skip_engine': 'True', 'skip_group': 'True', 'skip_docker_py': 'True'}"
```

Finally, we can deploy the stack containing all of our monitoring services to Docker Swarm using the `deploy-monitoring.yml` playbook.

```bash
ansible-playbook -i ./inventories/localhost deploy-monitoring.yml --ask-vault-pass --ask-become-pass
```

The `--ask-vault-pass` flag will prompt you for the vault password to access sensitive data. A default password used in this case, for demostraion purposes, is `password`.

## Does it work?

Once you followed the above steps we can check if all the services are running as expected on the docker manager.

```bash
ubuntu:~/monitoring-swarm-ansible/playbooks$ sudo docker service ls
ID                  NAME                       MODE                REPLICAS            IMAGE                       PORTS
oxmy6ipal9zg        monitoring_alertmanager    replicated          1/1                 prom/alertmanager:latest    *:9093->9093/tcp
szmfb4ptmdjz        monitoring_cadvisor        global              1/1                 google/cadvisor:latest      *:9080->8080/tcp
zcm0n2e2z5t4        monitoring_grafana         replicated          1/1                 grafana/grafana:latest      *:3000->3000/tcp
vtq70cdl1s0v        monitoring_node-exporter   global              1/1                 prom/node-exporter:latest   *:9100->9100/tcp
uvoj1vw3jvyo        monitoring_prometheus      replicated          1/1                 prom/prometheus:latest      *:9090->9090/tcp
```

We can now check these services by navigating to `http://localhost:port/` in a browser. For example, to navigate to cAdvisor go to [http://localhost:9080/](http://localhost:9080/).

## Further configuration

### Configuration files

This repository comes with basic configuration files for the services. However, you will likely need to edit these files to suit the needs of your application. The configuration files for each service can be found in `monitoring-swarm-ansible/roles/deploy-monitoring/templates`. Each subdirectory contains the configuration files for the service of the same name, for example, `alertmanager`, `grafana` and `prometheus`.

#### AlertManager

+ **alertmanager.yml** - The file defines the receivers for alerts. This repository sends all alerts to a Slack channel by default.

#### Grafana

+ **dashboard.yml** - This file defines the path to the dashboards. This repository contains one dashboard by default. More dashboard JSON files can be added to `monitoring-swarm-ansible/roles/deploy-monitoring/files/grafana/dashboards`.
+ **datasource.yml** - This file defines the datasources for Grafana. This repository only uses Prometheus by default.

#### Prometheus

+ **alert.rules.yml** - This file defines any alert rules. This repository does not have any alerts by default.
+ **prometheus.yml** - This file defines the services to scape metrics from. This repository scapes the Prometheus, cAdvisor and node-exporter services by default.

### Ansible variables

There are also a number of Ansible variables that can be overridden. These can be found in the table below:

| Variable Name | Description | Default Value |
|---------------|-------------|---------------|
| temporary_directory | A path to a directory where temporary files such as the compose file can be written to | "/tmp" |
| volume_directory | A path to a directory where service data can be persisted | "/tmp" |
| docker_network_name | The name of the new Docker overlay network | "monitoring_network" |
| prometheus_version | The tag of the Prometheus Docker image to use | latest |
| alertmanager_version | The tag of the AlertManager Docker image to use | latest |
| cadvisor_version | The tag of the cAdvisor Docker image to use | latest |
| node_exporter_version | The tag of the node exporter Docker image to use | latest |
| grafana_version | The tag of the Grafana Docker image to use | latest |
| slack_user | The name of the Slack user used to send alerts | 'AlertManager' |
| slack_channel | The name of the Slack channel to send alerts to | '#alerts' |

## Contributing

Any contribution to this repository is appreciated, whether it is a pull request, bug report, feature request, feedback or even starring the repository. Some potential areas that need further refinement are:

+ Hardening of playbooks
+ Making playbooks completely idempotent
+ Publish to Ansible Galaxy
+ Documentation

## Conclusion

This repository demonstates a number of well known technologies used to provide monitoring for web applications. Running these technologies together within Docker Swarm was an interesting experiment and seems to work as expected. Finally, running all of these commands manually everytime we need to deploy a monitoring stack would be extremely time consuming and so Ansible is quite useful to automate this process.
