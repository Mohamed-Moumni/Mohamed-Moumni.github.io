# Cadvisor

**What is Cadvisor?**

> cAdvisor (short for "container Advisor") is an open-source tool developed by Google for monitoring resource usage and performance of containerized applications. It runs as a daemon on a host machine, collecting data about the resource usage and performance of the running containers.
> 

> It can be used to monitor the usage of CPU, memory, network, and storage for each running container.
> 

> cAdvisor also provides a web-based user interface, which provides an easy way to visualize the data and access the data it exposes an HTTP endpoint that can be queried to retrieve the data in JSON format.
> 

The dockerfile of Cadvisor

```docker
# base image
FROM    debian:buster

# install wget
RUN apt update && apt install -y wget

# download the cadvisor with wget
RUN wget https://github.com/google/cadvisor/releases/download/v0.47.0/cadvisor-v0.47.0-linux-amd64

# change the name of cadvisor executable
RUN mv cadvisor-v0.47.0-linux-amd64 cadvisor

# change the permission for cadvisor
RUN chmod +x cadvisor

# running ther cadvisor programm
CMD [ "./cadvisor" ]
```

> In order to run Cadvisor properly with Docker Compose, you'll need to set up a volume mapping in your `docker-compose.yml` file that allows the Cadvisor container to access the host's filesystem. This is typically done using the `volumes`
 key in the service configuration for Cadvisor.
> 

![Cadvisor section in docker-compose.yml](Cadvisor%201448c0a9dfe048be852acffa1242ffce/Screen_Shot_2023-01-11_at_9.27.37_AM.png)

Cadvisor section in docker-compose.yml

Then you can connect to [http://localhost:8080](http://localhost:8080) and you can see the visualization of all the running containers.

![Screen Shot 2023-01-11 at 9.32.36 AM.png](Cadvisor%201448c0a9dfe048be852acffa1242ffce/Screen_Shot_2023-01-11_at_9.32.36_AM.png)

![Screen Shot 2023-01-11 at 9.32.47 AM.png](Cadvisor%201448c0a9dfe048be852acffa1242ffce/Screen_Shot_2023-01-11_at_9.32.47_AM.png)