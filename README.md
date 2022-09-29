# IRIS-Docker-HugePages
This is an example on how to make use of  Linux HugePages in IRIS2022 from within a docker container.



## Introduction

For performance, IRIS needs to allocate enough disk Cache, mainly in the form of "global buffers", and to a lesser extent, "routine Buffers", which are parts of the iris shared memory segment. This memory segment is allocated on iris start, and uses, if available, linux  large/huge pages.

For better performance, (lower CPU usage, lower per IRIS process memory usage), it is important that the shared memory segment uses Linux Huge Pages. 

Configuring an instance of IRIS2022.1 to use Linux Huge pages is a 3 Step process:

*  Reserve enough HugePages in the host operating system. (Dynamic HugePages are **not** recommended)
* Modify the dockerfile o build a modified iris image from the InterSystems base image, with the added capabilities assigned to the iriscp executable.
* apply the add_cap IPC_LOCK to the container when running it
* Optionally, force IRIS to use HugePages through an iris.cpf configuration change.



## Directories

| Directory          | Content                                       |
| ------------------ | --------------------------------------------- |
| ./communityEdition | demo using the IRIS Community edition         |
| ./IRISWithLicense  | demo that required an IRIS with a License Key |



## Demo QuickStart

### Requirements

To run this sample, you need following software:

* a Linux host (also tested from Windows WSL2 using Ubuntu 22.04 )

* docker

* docker-compose

### Prepare Host System

Reserve some  HugePages in the Host OS (temporary, lost on next boot)

* Verify that hugepages are enabled or enable if necessary

  ```
  cat /sys/kernel/mm/transparent_hugepage/enabled
  ```

  Output: `[always] madvise never`

  * Enable:

    ```
    sudo echo always > /sys/kernel/mm/transparent_hugepage/enabled
    ```

* Verify HugePages Size (2 MB in this example)

  ```
  grep Hugepagesize /proc/meminfo
  ```

  Output: `Hugepagesize:       2048 kB`

  

* Verify how many HugePages allocated

  ```
  cat /proc/sys/vm/nr_hugepages
  or 
  sysctl vm.nr_hugepages
  ```

  

* Allocate some HugePages (350MB)

  ```
  sudo sysctl -w vm.nr_hugepages=175
  ```



### Community Edition Demo

Now to run the community container:

```
cd ./communityEdition
docker-compose up
```

And search for the line that says:

> 09/28/22-12:07:29:340 (1149) 0 [Generic.Event] Allocated 306MB shared memory using **Huge Pages**

Note: If the container fails to start with the error:

> irislpcom  | An error was encountered while initializing the system.
> irislpcom  | Please see the clone.log and messages.log files in
> irislpcom  | /usr/irissys/mgr/ and /dur/config/mgr.
> irislpcom  | [ERROR] Command "iris start IRIS quietly" exited with status 256
> irislpcom  | 09/29/22-09:00:14:196 (391) 3 [Utility.Event] Error while moving data directories ERROR **#5001: Cannot create target: /dur/config/**

you can run following, to make the host durable directory writeable by iris:

```
sudo chown 51773:51773 ./durable
```

### Licensed Edition Demo

* Copy an IRIS License as "iris.key" to the ./IRISWithLicense/shared directory.

* Login with a web browser to "containers.interystem.com" using your Intersystems WRC password and get the login string.

* Execute the docker login with the provided token:

  ```
  docker login -u="yourUsername" -p="YourProvidedLoginToken" containers.intersystems.com
  ```

* Run the Licensed edition demo

  ```
  cd ./IRISWithLicense/
  docker-compose up
  ```

  And search for the line that says:

  > 09/28/22-12:07:29:340 (1149) 0 [Generic.Event] Allocated 306MB shared memory using **Huge Pages**

  Note: If the container fails to start with the error:

  > irislpcom  | An error was encountered while initializing the system.
  > irislpcom  | Please see the clone.log and messages.log files in
  > irislpcom  | /usr/irissys/mgr/ and /dur/config/mgr.
  > irislpcom  | [ERROR] Command "iris start IRIS quietly" exited with status 256
  > irislpcom  | 09/29/22-09:00:14:196 (391) 3 [Utility.Event] Error while moving data directories ERROR **#5001: Cannot create target: /dur/config/**

  you can run following, to make the host durable directory writeable by iris:

  ```
  sudo chown 51773:51773 ./durable
  ```



## How to use Huge Pages from IRIS in a container

As of IRIS 2021.2, the iris process runs as user irisowner (id 51773). In order to make use of linux hugepages, it needs to have the linux capabilities flag IPC_LOCK.

The first step is to add following at the **end**  as the final step (after stopping iris quietly) of the Dockerfile when building the container:

```
USER 51773
RUN cp /usr/irissys/bin/irisdb /usr/irissys/bin/iriscp
USER root
RUN setcap cap_ipc_lock+ep /usr/irissys/bin/iriscp
USER 51773
```

Next, to allow the use of this functionality, the docker run needs to contain the options "cap-add CAP_IPC_LOCK". 

Thus, the docker run command looks like this **(replacing the intersystems official image with the one you just built)**:

```
docker run --publish 52773:52773 --name iris22 --cap-add IPC_LOCK containers.intersystems.com/intersystems/irishealth:2022.1.0.209.0 --check-caps false
```

This looks as following in the docker-compose.yaml:

```
services:
  irislpcom:
    #privileged: true
    cap_add:
    - IPC_LOCK
```

With the --check-caps false added as a command parameter to the iris-main:

```
command:  --check-caps false  #--key /shared/iris.key
```

