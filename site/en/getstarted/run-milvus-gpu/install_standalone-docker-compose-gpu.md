---
id: install_standalone-docker-compose-gpu.md
label: Standalone (Docker Compose)
related_key: Kubernetes
summary: Learn how to install Milvus cluster on Kubernetes.
title: Run Milvus with GPU Support Using Docker Compose
---

# Run Milvus with GPU Support Using Docker Compose

This page illustrates how to start a Milvus instance with GPU support using Docker Compose.

## Prerequisites

- [Install Docker](https://docs.docker.com/get-docker/).
- [Check the requirements for hardware and software](prerequisite-gpu.md) prior to your installation.

## Install Milvus

To install Milvus with GPU support using Docker Compose, follow these steps.

### 1. Download and configure the YAML file

Download [`milvus-standalone-docker-compose-gpu.yml`](https://github.com/milvus-io/milvus/releases/download/v{{var.milvus_release_version}}/milvus-standalone-docker-compose-gpu.yml) and save it as docker-compose.yml manually, or with the following command.

```shell
$ wget https://github.com/milvus-io/milvus/releases/download/v{{var.milvus_release_version}}/milvus-standalone-docker-compose-gpu.yml -O docker-compose.yml
```

You need to make some changes to the environment variables of the standalone service in the YAML file as follows:

- To assign a specific GPU device to Milvus, locate the `deploy.resources.reservations.devices[0].devices_ids` field in the definition of the `standalone` service and replace its value with the ID of the desired GPU. You can use the `nvidia-smi` tool, included with NVIDIA GPU display drivers, to determine the ID of a GPU device. Milvus supports multiple GPU devices.

- Set the size of the memory pool assigned for GPU indexing, where `initialSize` represents the initial size of the memory pool and `maximumSize` represents its maximum size. Both values should be integers set in MB. Milvus uses these fields to allocate display memory to each process.

Assign a single GPU device to Milvus:

```yaml
...
standalone:
  gpu:
    initMemSize: 0
    maxMemSize: 1024
  ...
  deploy:
    resources:
      reservations:
        devices:
          - driver: nvidia
            capabilities: ["gpu"]
            device_ids: ["0"]
...
```

Assign multiple GPU devices to Milvus:

```yaml
...
standalone:
  gpu:
    initMemSize: 0
    maxMemSize: 1024
  ...
  deploy:
    resources:
      reservations:
        devices:
          - driver: nvidia
            capabilities: ["gpu"]
            device_ids: ['0', '1']
...
```

### 2. Start Milvus

In the directory that holds docker-compose.yml, start Milvus by running:

```shell
$ sudo docker compose up -d

Creating milvus-etcd  ... done
Creating milvus-minio ... done
Creating milvus-standalone ... done
```

<div class="alert note">

If you failed to run the above command, check whether your system has Docker Compose V1 installed. If this is the case, you are advised to migrate to Docker Compose V2 due to the notes on [this page](https://docs.docker.com/compose/).

</div>

After starting up Milvus,

- Containers named **milvus-standalone**, **milvus-minio**, and **milvus-etcd** are up.
  - The **milvus-etcd** container does not expose any ports to the host and maps its data to **volumes/etcd** in the current folder.
  - The **milvus-minio** container serves ports **9090** and **9091** locally with the default authentication credentials and maps its data to **volumes/minio** in the current folder.
  - The **milvus-standalone** container serves ports **19530** locally with the default settings and maps its data to **volumes/milvus** in the current folder.

You can check if the containers are up and running using the following command:

```shell
$ sudo docker compose ps

      Name                     Command                  State                            Ports
--------------------------------------------------------------------------------------------------------------------
milvus-etcd         etcd -advertise-client-url ...   Up             2379/tcp, 2380/tcp
milvus-minio        /usr/bin/docker-entrypoint ...   Up (healthy)   9000/tcp
milvus-standalone   /tini -- milvus run standalone   Up             0.0.0.0:19530->19530/tcp, 0.0.0.0:9091->9091/tcp
```

If you have assigned multiple GPU devices to Milvus in docker-compose.yml, you can specify which GPU device is visible or available for use.

Make GPU device `0` visible to Milvus:

```shell
$ CUDA_VISIBLE_DEVICES=0 ./milvus run standalone
```

Make GPU devices `0` and `1` visible to Milvus:

```shell
$ CUDA_VISIBLE_DEVICES=0,1 ./milvus run standalone
```

You can stop and delete this container as follows.

```shell
# Stop Milvus
$ sudo docker compose down

# Delete service data
$ sudo rm -rf volumes
```

## What's next

Having installed Milvus in Docker, you can:

- Check [Quickstart](quickstart.md) to see what Milvus can do.

- Learn the basic operations of Milvus:
  - [Manage Databases](manage_databases.md)
  - [Manage Collections](manage-collections.md)
  - [Manage Partitions](manage-partitions.md)
  - [Insert, Upsert & Delete](insert-update-delete.md)
  - [Single-Vector Search](single-vector-search.md)
  - [Multi-Vector Search](multi-vector-search.md)

- [Upgrade Milvus Using Helm Chart](upgrade_milvus_cluster-helm.md).
- [Scale your Milvus cluster](scaleout.md).
- Deploy your Milvu cluster on clouds:
  - [Amazon EC2](aws.md)
  - [Amazon EKS](eks.md)
  - [Google Cloud](gcp.md)
  - [Google Cloud Storage](gcs.md)
  - [Microsoft Azure](azure.md)
  - [Microsoft Azure Blob Storage](abs.md)
- Explore [Milvus Backup](milvus_backup_overview.md), an open-source tool for Milvus data backups.
- Explore [Birdwatcher](birdwatcher_overview.md), an open-source tool for debugging Milvus and dynamic configuration updates.
- Explore [Attu](https://milvus.io/docs/attu.md), an open-source GUI tool for intuitive Milvus management.
- [Monitor Milvus with Prometheus](monitor.md).

 
