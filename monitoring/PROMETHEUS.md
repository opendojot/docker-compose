# Prometheus services Deploy - Docker compose

## About

This readme contains the steps for deploying ``Prometheus`` monitoring application and ``Grafana`` which is used for visualize and analyze metrics generated by Prometheus.

## Disclaimer

As the inclusion of services (Prometheus and Grafana) is under development, some services are currently not being monitored. The following describes the services already implemented.

## Node-Exporter

Node-Exporter is used for get host metrics from the host itself. In other words, the metrics obtained by Prometheus are directly related to hardware and operating system.

Available metrics:

* Service uptime;
* Number of CPU cores;
* Total amount of RAM;
* Total amount of containers;
* Disk usage percentage;
* RAM usage percentage;
* Total amount of SWAP;
* Network traffic (Sent and Received);
* CPU usage percentage;
* Total, free and avaliable RAM;
* Amount of RAM for I/O processes;
* Used space of available disks.

## CAdvisor

CAdvisor is an exporter developed by Google that provides metrics regarding resource usage and performance characteristics of running containers.

Available metrics::

* Current status of the container: ``Running``, ``Stopped`` and ``Paused``;
* Network traffic received by container;
* Network traffic sent by container;
* Percentage of RAM usage per container;
* Percentage of CPU usage per container;
* RSS Memory Usage per Container;
* RAM consumption table per container;
* Available RAM for each conatainer;
* RAM limit established for each container.

## VerneMQ

As ``VerneMQ`` officially enables integration, the process becomes simpler. It is only necessary to declare the ``job`` in the Prometheus configuration file.

Available metrics:

* Summary;
* Clients;
* Queues;
* Subscriptions;
* Erlang VM;
* Bytes IN & OUT;
* Retain;
* TCP Sockets;
* Node to cluster communication;
* MQTT Connect;
* MQTT Subscribe;
* MQTT Publish;
* MQTT Ping;
* MISC.

## Apache Kafka Exporter

Kafka-Exporter is used to get metrics from the service ``Kafka``. In other words, the metrics obtained by Prometheus are directly related to high-performance data pipelines, streaming analytics, data integration, and mission-critical applications.

Available metrics:

* Message in per second;
* Lag by Consumer Group;
* Message in per minute;
* Message consume per minute;
* Partitions per Topic.

## Kong API Gateway Exporter

As ``Kong`` officially allows the integration, the process becomes simpler. It is only necessary to declare ``job`` in the Prometheus configuration file to get metrics regarding cloud services to manage, monitor and scale application programming interfaces and microservices.

Available metrics:

* Status codes;
* Latencies Histograms;
  * Request;
  * Kong;
  * Upstream;
* Bandwidth;
* DB reachability;
* Connections;
* Target Health;
* Dataplane Status;
* Enterprise License Information.

## Using the service

Since we are using Docker, it is natural that we will use its metrics. To set the Docker daemon as a Prometheus target, you need to specify the metrics-address in ``/etc/docker/daemon.json``.

The first step is editing the file ``/etc/docker/daemon.json``.

If the file does not exists, you need to create it.

If the file is empty, paste the following:

``
{
  "metrics-addr" : "0.0.0.0:9323",
  "experimental" : true
}
``

If the file is not empty, add these two keys, making sure the resulting file is a valid JSON. Be careful that all lines must end with a comma "," except the last line.

Since we are running Prometheus on localhost, in case we try to use the docker as localhost as well, Prometheus could understand that the metrics are inside the service itself (Prometheus) and then, could try to access port ``9323`` inside the Promethues container. To prevent this from happening we should use the default IP **172.17.0.1** of the **bridge** docker ``docker0``.

Before running make sure that the ``docker0`` IP is really the default (172.17.0.1) with the following command:

```
ip addr show docker0
```

The output will be similar to the one shown below, the *IP* will normally appear on the third line after the term ``inet``:
```
docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:a3:bf:97:21 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:a3ff:febf:9721/64 scope link
       valid_lft forever preferred_lft forever
```

If the IP is different, use the one that is on your ``docker0``.
## Changing prometheus.yml

If the port is not the default one, you should edit the jobs that prometheus will analyze in the [prometheus.yml](prometheus/prometheus.yml), changing the IP of the docker service as shown below:

Example job **docker**:

```
  - job_name: docker
    static_configs:
      - targets: ['<IP_DOCKER0>:9323']
```
## Starting services

As we've already done the configuration, we can start the service:

```
cd monitoring

docker-compose -f docker-compose-monitoring.yml up -d
```

When the process is complete, we can check whether the services have been started:

*   [Docker Metrics](http://localhost:9323/metrics).

*   [Prometheus Metrics](http://localhost:9090/metrics).

Visually, it is possible to check if everything is "UP" by [Prometheus Targets](http://localhost:9090/targets).

## Viewing metrics in grafana

As the service is declared in [Docker Compose Monitoring](docker-compose-monitoring.yml), it has already been instantiated. Then just access the service at the URL http://localhost:3000.

**USER**=``admin``

**SENHA**=``admin``

[Grafana](http://localhost:3000)

## Graphana Configuration

If you need to change or analyze the configuration files, you can find them in [Grafana Configuration Files](grafana/).

In the indicated folder, there are all dashboards (in json files), configuration files of the dashboards themselves and the data source.

## Viewing data

As the data and data source have already been imported automatically, you will be able to verify the data.

* **System Metrics** refers to exporting Node-Exporter;
* **Docker Metrics** refers to the CAdvisor exporter;
* **Overview** has the main panels of each dashboard.