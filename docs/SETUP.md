## Navigation

* [Requirements](#requirements)
* [Installation](#installation)
* [Run](#run)
    * [Standalone deployment](#standalone-deployment)
    * [Distributed deployment](#distributed-deployment)
* [See also](#see-also)

## Requirements
In order to run this Docker image, you need the following prerequisites and dependencies installed on each node you plan on deploying the container:
* Linux-based operating system, such as Debian, CentOS, and so on.
* Chipset: 
    * `splunk/splunk` image supports x86-64 chipsets
    * `splunk/universalforwarder` image supports both x86-64 and s390x chipsets
* Kernel version 4.0 or higher
* Docker engine:
    * Docker Enterprise Engine 17.06.2 or higher
    * Docker Community Engine 17.06.2 or higher
* [OverlayFS](https://docs.docker.com/storage/storagedriver/overlayfs-driver/) `overlay2` Docker daemon storage driver
    1. Edit `/etc/docker/daemon.json`. If it does not yet exist, create it.
    2. Assuming the file was empty, add the following contents:
        ```
        { "storage-driver": "overlay2" }
        ``` 
        **Note:** If you already have an existing JSON file, add only `"storage-driver": "overlay2"` as a key-value pair. Docker does not start if the `daemon.json` file contains badly-formed JSON.

For more details, see the official [supported architectures and platforms for containerized Splunk software environments](https://docs.splunk.com/Documentation/Splunk/latest/Installation/Systemrequirements#Containerized_computing_platforms) as well as [hardware and capacity recommendations](https://docs.splunk.com/Documentation/Splunk/latest/Installation/Systemrequirements). 

## Installation
Run the following commands to pull the latest images down from Docker Hub and into your local environment:
```
$ docker pull splunk/splunk:latest
$ docker pull splunk/universalforwarder:latest
```

## Run

This section explains how to start basic standalone and distributed deployments. See the [Examples](EXAMPLES.md) page for instructions on creating additional types of deployments.

### Standalone deployment

Start a single containerized instance of Splunk Enterprise with the command below, replacing `<password>` with a password string that conforms to the [Splunk Enterprise password requirements](https://docs.splunk.com/Documentation/Splunk/latest/Security/Configurepasswordsinspecfile).

```
$ docker run -it -p 8000:8000 -e "SPLUNK_PASSWORD=<password>" -e "SPLUNK_START_ARGS=--accept-license" splunk/splunk:latest
```

**You successfully created a standalone deployment with `docker-splunk`!**

After the container starts up, you can access Splunk Web at <http://localhost:8000> with `admin:<password>`.

### Distributed deployment

Start a Splunk Universal Forwarder running in a container to stream logs to a Splunk Enterprise standalone instance, also running in a container.

First, create a [network](https://docs.docker.com/engine/reference/commandline/network_create/) to enable communication between each of the services.

```
$ docker network create --driver bridge --attachable skynet
```

#### Splunk Enterprise
Start a single, standalone instance of Splunk Enterprise in the network created above, replacing `<password>` with a password string that conforms to the [Splunk Enterprise password requirements](https://docs.splunk.com/Documentation/Splunk/latest/Security/Configurepasswordsinspecfile).
```
$ docker run -it --network skynet --name so1 --hostname so1 -p 8000:8000 -e "SPLUNK_PASSWORD=<password>" -e "SPLUNK_START_ARGS=--accept-license" splunk/splunk:latest
```

This command does the following:
1. Starts a Docker container using the `splunk/splunk:latest` image.
2. Launches the container in the formerly-created bridge network `skynet`.
3. Names the container and the host as `so1`.
4. Exposes a port mapping from the host's `8000` port to the container's `8000` port
5. Specifies a custom `SPLUNK_PASSWORD`.
6. Accepts the license agreement with `SPLUNK_START_ARGS=--accept-license`. This agreement must be explicitly accepted on every container, or Splunk Enterprise doesn't start.

After the container starts up successfully, you can access Splunk Web at <http://localhost:8000> with `admin:<password>`.

#### Splunk Universal Forwarder
Start a single, standalone instance of Splunk Universal Forwarder in the network created above, replacing `<password>` with a password string that conforms to the [Splunk Enterprise password requirements](https://docs.splunk.com/Documentation/Splunk/latest/Security/Configurepasswordsinspecfile).
```
$ docker run -it --network skynet --name uf1 --hostname uf1 -e "SPLUNK_PASSWORD=<password>" -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_STANDALONE_URL=so1" splunk/universalforwarder:latest
```

This command does the following:
1. Starts a Docker container using the `splunk/universalforwarder:latest` image.
2. Launches the container in the formerly-created bridge network `skynet`.
3. Names the container and the host as `uf1`.
4. Specifies a custom `SPLUNK_PASSWORD`.
5. Accepts the license agreement with `SPLUNK_START_ARGS=--accept-license`. This agreement must be explicitly accepted on every container, otherwise Splunk Enterprise doesn't start.
6. Connects it to the standalone instance created earlier to automatically send logs to `so1`.

**NOTE:** The Splunk Universal Forwarder product does not have a web interface. If you require access to the Splunk installation in this particular container, please refer to the [REST API](https://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTprolog) documentation or use `docker exec` to access the [Splunk CLI](https://docs.splunk.com/Documentation/Splunk/latest/Admin/CLIadmincommands).

**You successfully created a distributed deployment with `docker-splunk`!**

If everything went smoothly, you can log in to your Splunk Enterprise instance at <http://localhost:8000>, and then run a search to confirm the logs are available. For example, a query such as `index=_internal` should return all the internal Splunk logs for both `host=so1` and `host=uf1`.

## See also

[More examples of standalone and distributed deployments](EXAMPLES.md)

[Design and architecture of docker-splunk](ARCHITECTURE.md)

[Adding advanced complexity to your containerized Splunk deployments](ADVANCED.md)
