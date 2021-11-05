
# NEM Docker

This Docker file and the helper scripts are the fastest and easiest way deploy a [NEM](https://nemproject.github.io/nem-docs/pages/) node.

## Prerequisites

- You need [Docker](https://docs.docker.com/get-docker/) installed.

## Installation

Just clone [this repository](https://github.com/NemProject/nem-docker) and move into it:

```bash
$ git clone https://github.com/NemProject/nem-docker.git
$ cd nem-docker
```

## How to run

Depending on the way you have configured Docker, you might need to run these commands as root (or equivalently prefixed by ``sudo``):

```bash
$ ./boot.sh
```

The first time you will be asked for a node name:

```text
No config file was found. We need to generate one, even if you run only NCC
Enter the name you want to assign to your node:
```

You will also be asked for a boot key. This is optional and one will be generated for you if you just press ENTER:

```text
Enter the boot key of your node. If you do not know what this is, press enter and one will be generated for you

The config file generated is at custom-configs/nis.config-user.properties
```

This will start the NIS process in a Docker container named ``mynem_container``:

```text
Starting NIS
nis: started
All done, here are the services running:

ncc                              STOPPED   Not started
nis                              RUNNING   pid 19, uptime 0:00:01
perms                            RUNNING   pid 16, uptime 0:00:01
servant                          STOPPED   Not started
```

## How to stop

To stop the container and all the running services, simply execute:

```bash
$ ./stop.sh
```

## Controlling the process in the container

Services running in the container are controlled with ``supervisord``. You can easily control them with the provided ``service.sh`` script.

- To **check** which services are running:

  ```bash
  $ ./service.sh status
  ncc                              STOPPED   Not started
  nis                              RUNNING   pid 18, uptime 0:01:19
  perms                            RUNNING   pid 15, uptime 0:01:19
  servant                          STOPPED   Not started
  ```

- To **stop** the NIS service (without stopping the container):

  ```bash
  $ ./service.sh stop nis
  nis: stopped
  ```

- To **start** the NIS service again:

  ```bash
  $ ./service.sh start nis
  nis: started
  ```

- To **restart** the NIS service in a single command:

  ```bash
  $ ./service.sh restart nis
  nis: stopped
  nis: started
  ```

## Enabling the Servant process

[NEM Supernodes](https://nemproject.github.io/nem-docs/pages/Guides/supernode-program/docs.en.html) require a servant process to be running to monitor and report the node's health.

To enable the servant in the container:

- Stop the container if you had it running.
- **Copy** `custom-configs/servant.config.properties.sample` to `custom-configs/servant.config.properties` and [edit it to your liking](https://nemproject.github.io/nem-docs/pages/Guides/supernode-program/docs.en.html).
- Start the container and wait for NIS to synchronize:

  ```bash
  $ ./boot.sh
  ```

- Start the servant:

  ```bash
  $ ./service.sh start servant
  ```

## Tweaking the config

The ``boot.sh`` script checks if a file named ``custom-configs/config-user.properties``
exists when running NIS, and if it doesn't, it prompts the user for
information. It then generates the file with a ``bootName`` and a ``bootKey``. If you
want to tweak the configuration of your node, this is the file to edit.

Once the config file is generated the script builds and runs the image with these commands, naming the container ``mynem_container``:

```bash
sudo docker build -t mynem_image  .
sudo docker run --name mynem_container -v ${PWD}/nem:/root/nem $config_mounts -t -d  -p 7777:7777 -p 7880:7880 -p 7890:7890 -p 8989:8989 mynem_image "$@"
```

This runs the container and makes the necessary ports available on your host.
``$config_mounts`` passes the necessary arguments to use the custom config file located in ``custom-configs``. Currently handled files are `supervisord.conf`, `nis.config-user.properties` and `servant.config.properties`.

Here is an example to customize the supervisor config. First copy the sample config file to get started, then edit it, e.g. to set some services as automatically started at boot. After that stop and reboot the container and the new configuration will be applied:

```bash
cp custom-configs/supervisord.conf.sample custom-configs/supervisord.conf
vim custom-configs/supervisord.conf
./stop.sh
./boot.sh
```

## The ``nem`` directory

All blockchain data used by NIS is saved in the ``nem`` directory, so it is persisted across restarts of the container.

It contains the blocks (``nem/nis/data``), but also the logs of NIS (``nem/nis/logs``).
