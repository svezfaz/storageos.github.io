# StorageOS Docker Volume Plugin

The StorageOS volume plugin turns your Docker node into a hyper-converged storage platform. Each node that runs the StorageOS plugin can contribute available local or attached storage into a distributed pool, which is then available to all cluster members via a global namespace.

Volumes are available across the cluster so if a container gets moved to another node it still has access to its data. Data can be protected with synchronous replication. Compression, caching, and QoS are enabled by default, and all volumes are thin-provisioned.

No other hardware or software is required.

During beta, StorageOS is freely available for testing and experimentation. _DO NOT USE FOR PRODUCTION DATA_. A Developer edition will be free forever.

Full documentation is available at <https://docs.storageos.com>. To stay informed about new features and production-ready releases, sign up on our [customer portal](https://my.storageos.com).

## Quick Start

To install a single StorageOS node for testing:

```bash
$ docker plugin install --alias storageos storageos/plugin ADVERTISE_IP=$ADVERTISE_IP CLUSTER_ID=$CLUSTER_ID
```

That's it - you can now start containers with StorageOS-backed volumes:

```bash
$ docker run --name postgres01 \
   -e PGDATA=/var/lib/postgresql/data/db \
   -v postgres01:/var/lib/postgresql/data \
   --volume-driver=storageos -d postgres
```

Or pre-create and manage StorageOS volumes using the `docker volume` command:

```bash
$ docker volume create \
  --driver storageos --opt size=20 --opt storageos.feature.replicas=2 vol01
```

To install multiple StorageOS nodes in a clustered configuration, see the installation section below.

### Next Steps

To get the most out of StorageOS, try:

1. Running the CLI to manage volumes, rules, and cluster configuration

## Requirements

### Docker Version

The `docker plugin install` method requires Docker 1.13.0 or above. Older versions (from Docker 1.10.0 onwards) can use the [node container install](../node) method.

## Installation

The plugin (or node container) should be installed on each Docker node where you want to consume StorageOS volumes or to present capacity to other nodes.

### Network Block Device (NBD)

(Optional) NBD is a default Linux kernel module that allows block devices to be run in userspace. Enabling NBD is recommended as it will increase performance for some workloads. To enable the module and increase the number of allowable devices, you must either run:

```bash
$ sudo modprobe nbd nbds_max=1024
```

**To ensure the NBD module is loaded on reboot.**

1. Add the following line to `/etc/modules`

   ```bash
   nbd
   ```

1. Add the following module configuration line in `/etc/modprobe.d/nbd.conf`

   ```bash
   options nbd nbds_max=1024
   ```

### StorageOS Plugin Installation (Docker 1.13+)

If Consul is running locally, the defaults will work:

```bash
$ docker plugin install --alias storageos storageos/plugin

Plugin "storageos/plugin" is requesting the following privileges:
- network: [host]
- mount: [/var/lib]
- mount: [/dev]
- device: [/dev/fuse]
- allow-all-devices: [true]
- capabilities: [CAP_SYS_ADMIN]
Do you grant the above permissions? [y/N]
```

Other configuration parameters (see Configuration Reference below) may be set in a similar way. For most environments, only the KV_ADDR will need to be set if Consul is not running locally on the node.

**NOTE**: In order to make plugin upgrades easier, install the plugin using `--alias storageos`. This ensures that volumes provisioned with a previous version of the plugin continue to function after the upgrade. By default, Docker ties volumes to the plugin version. Volumes should then be provisioned using the alias e.g. `--volume-driver storageos`.

## Configuration Parameters

Although the default settings should work for most environments, a number of settings are configurable:

* `HOSTNAME`: Hostname of the Docker node, only if you wish to override it.
* `ADVERTISE_IP`: IP address of the Docker node, for incoming connections. Defaults to first non-loopback address.
* `USERNAME`: Username to authenticate to the API with. Defaults to `storageos`.
* `PASSWORD`: Password to authenticate to the API with. Defaults to `storageos`.
* `CLUSTER_ID`: Cluster ID for the node to join an existing cluster previously created through 'storageos cluster create' command
* `INITIAL_CLUSTER`: Static list of pre-existing cluster, supplied as comma separated list of `<hostname>=<url>:5707`
* `API_PORT`: Port for the API to listen on. Defaults to `5705` ([IANA Registered](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=5705)).
* `NATS_PORT`: Port for NATS messaging to listen on. Defaults to `4222`.
* `NATS_CLUSTER_PORT`: Port for the NATS cluster service to listen on. Defaults to `8222`.
* `SERF_PORT`: Port for the Serf protocol to listen on. Defaults to `13700`.
* `DFS_PORT`: Port for DirectFS to listen on. Defaults to `17100`.
* `KV_ADDR`: IP address/port of an external Key/Vaue store.  Must be specified with `KV_BACKEND=etcd`.
* `KV_BACKEND`: Type of KV store to use. Defaults to `embedded`. `etcd` is supported with `KV_ADDR` set to an external etcd instance.
* `LOG_LEVEL`: One of `debug`, `info`, `warning` or `error`. Defaults to `info`.
* `LOG_FORMAT`: Logging output format, one of `text` or `json`. Defaults to `json`.
* `DISABLE_TELEMETRY`: To disable anonymous usage reporting across the cluster, set to `true`. Defaults to `false`. To help improve the product, data such as API usage and StorageOS configuration information is collected.

