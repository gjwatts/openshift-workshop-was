# Introduction to Containerization

## Background

If you are expecting a lab about `docker`, you are at the right place.
This lab will introduce you to the basic concepts of containerization, including:

- What are containers and container images
- How to start, stop, and remove containers.
- How to create container images

Note: This lab uses `docker`. 
However, We encourage you to use `podman` instead of `docker` in your own environment. By default, Openshift 4.x uses `podman`.
The reason is that `podman` is more secure.
It runs in user space, and does not require a daemon.
However, if you only have `docker` in your environment, you may use it. 
Otherwise, substitute `docker` with `podman` below.

- `podman` command line parameters are compatible with `docker`.
- Both `podman` and `docker` conform to the Open Container Initiative specifications, and they support the same image format and registry APIs.

The reason we are not yet using `podman` for this lab is that it does not yet work in the web terminal environment.
The web terminal runs in a container. 
In order to run this lab within the web terminal, we need a container that runs in a container.
We have not yet been able to configure `podman` to run inside a container.

## Prerequisites

- You have `podman` or `docker` installed.
- You have access to the internet.
- You have cloned this lab from github.

## What is a Container

Compared to virtual machines, containers supports virtualization at the process level. 
Think of them as virtual processes.
The isolation abstraction provided by the operating system makes the process think that it's running in its own virtual machine.
As processes, containers may be created, started, and stopped much more quickly than virtual machines.

Everything you need to run your application, from the operating system and up, is stored in a special file called a container image.  
Container images are self contained and portable. 
You may run one or more instances anywhere. 
And you don't have to worry about missing prerequisites, because all prerequisites are stored in the image.

Container images are created via tools such as `docker` or `podman`. 
Existing images are hosted in container registries. 
For example, docker hub, or registry.access.redhat.com, or your own internal registry. 


If you need more background on containers: https://www.docker.com/resources/what-container


## Check your environment

1. List version of docker: `docker --version`

## Run a pre-built image

1. List available local images: `docker images`

    ```
    REPOSITORY   TAG   IMAGE ID   CREATED   SIZE
    ```

1. Pull an test image from docker hub:  `docker pull openshift/hello-openshift`

    ```
    Trying to pull registry.access.redhat.com/openshift/hello-openshift...
      name unknown: Repo not found
    Trying to pull docker.io/openshift/hello-openshift...
    doGetting image source signatures
    Copying config 7af3297a3f done
    Writing manifest to image destination
    Storing signatures
    7af3297a3fb4487b740ed6798163f618e6eddea1ee5fa0ba340329fcae31c8f6
    ```

1. List available local images again: `docker images`

    ```
    REPOSITORY                            TAG      IMAGE ID       CREATED         SIZE
    docker.io/openshift/hello-openshift   latest   7af3297a3fb4   21 months ago   6.1 MB
    ```

1. Inspect the image: `docker inspect openshift/hello-openshift` and note that:
    - It exposes two ports: 8080 and  8088
    - It runs as user 1001
    - The entry point executable is /hello-openshift
  
    ```
    [
        {
            "Id": "7af3297a3fb4487b740ed6798163f618e6eddea1ee5fa0ba340329fcae31c8f6",
            "Digest": "sha256:aaea76ff622d2f8bcb32e538e7b3cd0ef6d291953f3e7c9f556c1ba5baf47e2e",
            ...
            "Config": {
                "User": "1001",
                "ExposedPorts": {
                    "8080/tcp": {},
                    "8888/tcp": {}
                },
                "Env": [
                    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
                ],
                "Entrypoint": [
                    "/hello-openshift"
                ]
                ...
        }
    ]
    ```

1. Run the image: `docker run --name hello1 -d -p 8080:8080 -p 8888:8888 openshift/hello-openshift`.  Note that:
    - The `--name` option gives the container a name.
    - The `-d` option runs the command in the background as a daemon
    - The `-p` command maps the host port to the port in the container. Through virtual network, the port in the container may be fixed, When you run the container, you may assign a new port on the host dynamically.
  
1. Access the application in the container.
    - If you are running in a server, or the web terminal, try `curl http://localhost:8080`.
    - If you are running on a desktop, point your browser to `http://localhost:8080`
    - Also try port 8888

1. Run another instance of the same image: `docker run --name hello2 -d -p 8081:8080 -p 8889:8888 openshift/hello-openshift`. 
Question: how does this compare to the time it takes to start a new virtual machine?

1. Access the application in the new container the same way. However, use port `8081` and `8889` instead.

1. Verify there are two containers running in the same host: `docker ps`:

    ```
    CONTAINER ID  IMAGE                                       COMMAND  CREATED             STATUS             PORTS
                                      NAMES
    5d2c82fe3f7c  docker.io/openshift/hello-openshift:latest           5 seconds ago       Up 5 seconds ago   0.0.0.0:8081->
    8080/tcp, 0.0.0.0:8889->8888/tcp  hello2
    43c70b723756  docker.io/openshift/hello-openshift:latest           About a minute ago  Up 57 seconds ago  0.0.0.0:8080->
    8080/tcp                          hello1```
    ```

1. View the logs: `docker logs hello1`

    ```
    serving on 8888
    serving on 8080
    ```

1. View the logs on the second container: `docker logs hello2`. 

    ```
    serving on 8888
    serving on 8080
    ```

    Note: they each think they have their own machine, and have opened the same ports.

1. To export the file system of a running container: `docker export hello1 > hello1.tar`

1. List the files on the file system: `tar -tvf hello1.tar`. Note that this is a very small image.

    ```
    drwxr-xr-x root/root         0 2020-01-31 20:31 dev/
    drwxr-xr-x root/root         0 2020-01-31 20:31 etc/
    -rwxr-xr-x root/root         0 2020-01-31 20:31 etc/hostname
    -rwxr-xr-x root/root         0 2020-01-31 20:31 etc/hosts
    -rwxr-xr-x root/root         0 2020-01-31 20:31 etc/passwd
    -rwxr-xr-x root/root         0 2020-01-31 20:31 etc/resolv.conf
    -rwxr-xr-x root/root   6089990 2018-04-18 10:22 hello-openshift
    drwxr-xr-x root/root         0 2020-01-31 20:31 proc/
    drwxr-xr-x root/root         0 2020-01-31 20:31 run/
    -rwxr-xr-x root/root         0 2020-01-31 20:31 run/.containerenv
    drwxr-xr-x root/root         0 2020-01-31 20:31 run/secrets/
    drwxr-xr-x root/root         0 2020-01-31 20:31 sys/
    ```

1. Run another command in the running container. 
You can reach into the running container to run another command. 
The typical use case is to run a shell command, so you can use the shell to navigate within the container and run other commands.
However, our image is tiny, and there is no built-in shell.
This makes it safer, but also more difficult to debug.
For the purpose of this lab, we'll execute the same command again: `docker exec -ti hello1 /hello-openshift`. 
Running this command again in the same container results in an error, 
because there is already another copy running in the background that is bound to the ports 8080 and 8888:

    ```
    serving on 8888
    serving on 8080
    panic: ListenAndServe: listen tcp :8888: bind: address already in use
    ...
    ```

1. Stop the containers:
   - `docker stop hello1`
   - `docker stop hello2`

1. List running containers: `docker ps`

1. List all containers, including stopped ones: `docker ps -a`

    ```
    CONTAINER ID  IMAGE                                       COMMAND  CREATED         STATUS                    PORTS
                                         NAMES
    2b1181079e03  docker.io/openshift/hello-openshift:latest           16 seconds ago  Exited (2) 4 seconds ago  0.0.0.0:808
    1->8080/tcp, 0.0.0.0:8889->8888/tcp  hello2
    5bfd89e698e7  docker.io/openshift/hello-openshift:latest           32 seconds ago  Exited (2) 7 seconds ago  0.0.0.0:808
    0->8080/tcp                          hello1
    ```
  
1. restart a stopped container: `docker restart hello1`

1. List running containers: `docker ps`

    ```
    CONTAINER ID  IMAGE                                       COMMAND  CREATED        STATUS             PORTS
         NAMES
    5bfd89e698e7  docker.io/openshift/hello-openshift:latest           2 minutes ago  Up 29 seconds ago  0.0.0.0:8080->8080/
    tcp  hello1
    ```
1. Stop the container: `docker stop hello1`

1. Remove stopped containers, and note that there are no more containers:
    - `docker rm hello1`
    - `docker rm hello2`
    - `docker ps -a`

1. Remove the image from local cache:

    ```
    docker images
    docker rmi openshift/hello-openshift
    docker images
    ```
 

## Build and Run Your Own Image

We use a `Containerfile`, which contains the instructions to create the new layers of your image.
Recall an image contains the entire file system that you want to use to run your virtual process in a container.

1. Review the provided `Containerfile`:

    ```
    FROM ibmcom/websphere-liberty:kernel-java8-ibmjava-ubi
    COPY server.xml  /config
    COPY ServletApp.war /config/dropins/app.war
    RUN /liberty/wlp/bin/installUtility install --acceptLicense /config/server.xml 
    ```

   - The first line `FROM` specifies the existing image to be used as the base.  If this is not in the local registry, it will be pulled from a remote registry such as docker hub.

   - The second line `COPY`  is a straight copy of the file `server.xml` from the local directory to `/config/server.xml` in the image. This adds a new layer to the image with the actual server configuration to be used.
   
   - The third line, another `COPY`, copies `ServletApp.war` from the current directory into a new layer in the image you are creating, at the location `/config/dropins/app.war`.

   - The last line `RUN` runs he `installUtility` command within the image to install additional features required to run the server as specified in `server.xml`. You can use the `RUN` command to run any command that is available within the image to customize the image itself.

1. Run the build: 
    ```
    docker build -t app -f Containerfile .
    ```
    The `-t` option tags the name of the image as `app`.  The `-f` option specifies the name of the `Containerfile`. The build command runs the commands in `Containerfile` to build a new image called `app`.

    ```
    STEP 1: FROM ibmcom/websphere-liberty:kernel-java8-ibmjava-ubi
    STEP 2: COPY server.xml  /config
    eac7bbb2c9f3b010d57744094e7bb11528b22cec5a3f914a7f1e7c9b657859f8
    STEP 3: COPY ServletApp.war /config/dropins/app.war
    d8c6c1b3b3b1ab810bf1d4d2a1fea9399a33b714264ae610143c0c6c15222dd9
    STEP 4: RUN /liberty/wlp/bin/installUtility install --acceptLicense /config/server.xml
    Checking for missing features required by the server ...
    The server requires the following additional features: servlet-3.0.  Installing features from the repository ...
    Establishing a connection to the configured repositories ...
    This process might take several minutes to complete.
    
    Successfully connected to all configured repositories.
    
    Preparing assets for installation. This process might take several minutes to complete.
    
    Additional Liberty features must be installed for this server.
    
    To install the additional features, review and accept the feature license agreement:
    The --acceptLicense argument was found. This indicates that you have
    accepted the terms of the license agreement.
    
    
    Step 1 of 4: Downloading servlet-3.0 ...
    Step 2 of 4: Installing servlet-3.0 ...
    Step 3 of 4: Validating installed fixes ...
    Step 4 of 4: Cleaning up temporary files ...
    
    
    All assets were successfully installed.
    
    Start product validation...
    Product validation completed successfully.
    STEP 5: COMMIT app
    3f9c0085cca1fc11ecb918451b054bd60a5da6911b559c54b18551283a4e784f
    ```

1. List the images to see that the new image `app` is built: `docker images`. Note that the base image, `docker.io/ibmcom/websphere-liberty` has also be pulled into the local registry.

 ```
 REPOSITORY                            TAG                        IMAGE ID       CREATED         SIZE
localhost/app                         latest                     baa6bb9ad29d   2 minutes ago   544 MB
ibmcom/websphere-liberty    kernel-java8-ibmjava-ubi   7ea3d0a2b3fe   4 hours ago     544 MB
 ```

- Start the container. Note that you are running with both http and https ports: `docker run -d -p 9080:9080 -p 9443:9443 --name=app-instance app`

- Access the application running in the container:
  - If you are running in a server, or using the web terminal, use `curl --insecure https://localhost:9443/app` and ensure you have output that looks like: `<html><h1><font color=green>Simple Servlet ran successfully</font></h1>Powered by WebSphere Liberty  <html>`.
  - If you are running on a desktop with browser
    -  point your browser to `http://localhost:9080/app` and check that renders a page showing `Simple Servlet ran successfully`.
    - Also point your browser to `https://localhost:9443/app`


1. List the running containers: `docker ps`

    ```
      CONTAINER ID  IMAGE                 COMMAND               CREATED             STATUS                 PORTS NAMES
    45e82c9cd416  localhost/app:latest  /opt/ibm/wlp/bin/...  About a minute ago  Up About a minute ago  0.0.0.0:9081->9080/ tcp, 0.0.0.0:9444->9443/tcp  app-instance1
    ```

1. Access the logs to your container: `docker logs -f app-instance`. Use `Ctrl-C` To exit.

1. Remote shell into your running container to poke around: `docker exec -it app-instance /bin/sh`
    - run `whoami` and note you're not running as root.
    - Note that this is a stripped down environment where many coammnds are not available. For example, try `which ps`.
    - cd `/logs` to find the log files
    - cd `/liberty/wlp` to find the location of the liberty install
    - cd `/liberty/wlp/usr/servers/defaultServer` to find the server configuration. 
    - cd `/opt/ibm/wlp/output` to find the workarea files required by the server runtime.
    - Exit from the container: `exit`

1. Managing image versions. 
There is no built-in versioning for container images. 
However, you may use a tagging convention to version your images. 
The convention is to use `major.minor.patch`, such as `1.3.5`.
The default tag for the most recently built image is `latest`. 

    ```
    docker tag app 1
    docker tag app 1.3
    docker tag app 1.3.5
    docker images
    ```

    And the output:
    ```
    REPOSITORY                 TAG                        IMAGE ID            CREATED             SIZE
    app                        1                          d98cbdf82a0d        21 hours ago        542MB
    app                        1.3                        d98cbdf82a0d        21 hours ago        542MB
    app                        1.3.5                      d98cbdf82a0d        21 hours ago        542MB
    app                        latest                     d98cbdf82a0d        21 hours ago        542MB
    ```

    Note that all the different tags are currently associated with the same image ID. 
    When you pull or run an image, you may specify the tag, based on your versioning requirements. For example, 
    - `docker run app:1 ...` matches the latest 1.x.x version.
    - `docker run app:1.3 ...` matches the latest 1.3.x version.
    - `docker run app:1.3.5 ...` matches only 1.3.5.

    After you build a new patch version of the image, you will tag the new version as follows:
    ```
    docker tag app 1
    docker tag app 1.3
    docker tag app 1.3.6
    ```

    This means that:
    - `docker run app:1 ...` matches the new version
    - `docker run app:1.3 ...` matches the new version
    - `docker run app:1.3.5 ...` uses the exact version.
    - `docker run app:1.3.6 ...` uses the exact version.

    When you build a new minor version of the image, you will tag the new version as follows:
    ```
    docker tag app 1
    docker tag app 1.4
    docker tag app 1.4.0
    ```

    This means that:
    - `docker run app:1 ...` matches the new version
    - `docker run app:1.3 ...` matches the old version
    - `docker run app:1.4 ...` uses the new version
    - `docker run app:1.3.5 ...` uses the exact version.
    - `docker run app:1.4.0 ...` uses the exact version.

1. Cleanup: 
    - `docker stop app-instance`
    - `docker rm app-instance`

1. For extra credit:
    - Start another instances of the image for vertical scaling, but with different port numbers on the host.
    - Point your browser to `hub.docker.com`, click "Explore" and explore the millions of available images.
    - Think about how you would tag a new image at a major version, `2.0.0`.
    - Think about what would be required to manage containers across multiple machines to support horizontal scaling.
    

Congratulations. You have completed the introduction to containerization lab.