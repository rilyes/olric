# Multi-container environment with Docker Compose

We provide a multi-container environment to test, develop and deploy Olric clusters. This environment includes nginx as 
TCP reverse proxy and Consul for service discovery. 

## Usage

In this folder, simply run:

```
docker-compose up olric
```

To create a multi-node cluster:

```
docker-compose up --scale olric=2
```

Sample output:

```
docker-compose up olric
Creating docker_nginx_1  ... done
Creating docker_consul_1 ... done
Creating docker_olric_1  ... done
Creating docker_olric_2  ... done
Attaching to docker_olric_1
olric_1      | 2020/08/12 15:53:18 [olricd] pid: 1 has been started on 172.25.0.4:3320
olric_1      | 2020/08/12 15:53:18 [INFO] Service discovery plugin is enabled, provider: consul
olric_1      | 2020/08/12 15:53:18 [DEBUG] memberlist: Stream connection from=172.25.0.3:56830
olric_1      | 2020/08/12 15:53:19 [ERROR] Join attempt returned error: no peers found => olric.go:2
```

You can modify `olric-consul.yaml` file to try different configuration options. 

If Consul service works without any problem, you can visit [http://localhost:8500](http://localhost:8500) to monitor 
cluster health.

### Accessing to the cluster

`nginx` service exposes port `3320` to access to the cluster. 

### Develop & Test with Docker Compose

In order to test your local changes in a containerized environment, just run the following command:

```
docker-compose up olric-dev
```

`olric-dev` container includes Delve debugger.

## Service discovery

Olric provides a service discovery subsystem via a plugin interface. We currently have three service discovery plugins:

* [olric-consul-plugin](https://github.com/buraksezer/olric-consul-plugin): Consul-backed service discovery, 
* [olric-nats-plugin](https://github.com/justinfx/olric-nats-plugin): Nats-backed service discovery,
* [olric-cloud-plugin](https://github.com/buraksezer/olric-cloud-plugin): Service discovery plugin for cloud environments (AWS, GKE, Azure and Kubernetes)

We use Consul plugin in this document:

### Consul 

Consul is easy to use and proven way to discover nodes in a clustered environment. Olric discover the other nodes via 
[olric-consul-plugin](https://github.com/buraksezer/olric-consul-plugin). Here is a simple payload for this setup:

```json
{
  "Name": "olric-cluster",
  "Tags": [
    "primary",
    "v1"
  ],
  "Port": 3322,
  "EnableTagOverride": false,
  "Check": {
    "Name": "Olric node on 3322",
    "Interval": "1s",
    "Timeout": "10s"
  }
}
```

`3322` is used by [hashicorp/memberlist](https://github.com/hashicorp/memberlist) to maintain an eventually consistent view of the cluster. 
Consul dials this port to control the node. `Address`, `ID` and `Check.TCP` fields is being filled by the plugin. You can still 
give your own configuration values, if you know what you are doing.

