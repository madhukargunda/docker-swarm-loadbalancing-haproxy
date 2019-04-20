# docker-swarm-loadbalancing-haproxy
Docker Swarm loadbalancing with haproxy


This Artice i am going to describes loadbalncing with haproxy and distributing the containers into differnt nodes
using docker swarm

##

```
Step1: 

Create the docker-machine Loadbalancer node
  docker-machine create --driver virtualbox loadbalancer

Step2 : 

Create the docker-machine Manager node
   docker-machine create --driver virtualbox manager1

Step3 : 

Create the docker-machine Worker-1 and Worker-2 node
   docker-machine create --driver virtualbox worker1
   docker-machine create --driver virtualbox worker2

Step4 : 

ssh into manager-1 node
eval $(docker-machine env manager1)

Step5 : 

docker swarm init --advertise-addr $(docker-machine ip manager1)

Swarm initialized: current node (b4etd4utcgn9xnpawfs2tf5qe) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0hbyhpos06fwf85r35319wyhjpfysk9l88emio2av1qsr04alu-8th0xvea2f0yw3voiqc5er32c 192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

Step6: 

Go to the other worker nodes and join the worker nodes to swaram manager
        eval $(docker-machine env worker-1)

docker swarm join --token SWMTKN-1-0hbyhpos06fwf85r35319wyhjpfysk9l88emio2av1qsr04alu-8th0xvea2f0yw3voiqc5er32c 192.168.99.100:2377

Step7 : 

eval $(docker-machine env worker-2)
docker swarm join --token SWMTKN-1-0hbyhpos06fwf85r35319wyhjpfysk9l88emio2av1qsr04alu-8th0xvea2f0yw3voiqc5er32c 192.168.99.100:2377

Step8: execute docker-machine ls

NAME            ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER     ERRORS
load-balancer   -        virtualbox   Running   tcp://192.168.99.101:2376           v18.09.5   
manager1        -        virtualbox   Running   tcp://192.168.99.100:2376           v18.09.5   
worker-1        -        virtualbox   Running   tcp://192.168.99.102:2376           v18.09.5   
worker-2        *        virtualbox   Running   tcp://192.168.99.103:2376           v18.09.5   

Step8 : 

Go to loadbalancer node
eval $(docker-machine env load-balancer)

Step9: 

docker-compose up -d
Creating network "docker-swarm-loadbalancing-haproxy_default" with the default driver
Creating lb ... done
madhu@Admins-MacBook-Pro:~/dockerfiles/docker-swarm-loadbalancing-haproxy$ docker-compose ps
Name              Command               State                     Ports                   
------------------------------------------------------------------------------------------
lb     /docker-entrypoint.sh hapr ...   Up      0.0.0.0:80->80/tcp, 0.0.0.0:8404->8404/tcp

Step10: 

madhu@Admins-MacBook-Pro:~/dockerfiles/docker-swarm-loadbalancing-haproxy$ eval $(docker-machine env manager1)
madhu@Admins-MacBook-Pro:~/dockerfiles/docker-swarm-loadbalancing-haproxy$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
b4etd4utcgn9xnpawfs2tf5qe *   manager1            Ready               Active              Leader              18.09.5
c7gnmmmvoalmb4vowhxcmgs58     worker-1            Ready               Active                                  18.09.5
eu7lv79v468fl6z3apok9a4u1     worker-2            Ready               Active                                  18.09.5


Step 11: 

docker stack deploy -c docker-compose-microservices.yml account-stack
Ignoring unsupported options: restart

Ignoring deprecated options:

container_name: Setting the container name is not supported.

Creating network account-stack_back-end
Creating service account-stack_visualizer
Creating service account-stack_account-service
madhu@Admins-MacBook-Pro:~/dockerfiles/docker-swarm-loadbalancing-haproxy$ 

Step 12:

docker stack ls

NAME                SERVICES            ORCHESTRATOR
account-stack       2                   Swarm
madhu@Admins-MacBook-Pro:~/dockerfiles/docker-swarm-loadbalancing-haproxy$ docker service ls
ID                  NAME                            MODE                REPLICAS            IMAGE                                  PORTS
oh76u5b6lyz3        account-stack_account-service   replicated          1/1                 madhukargunda/account-service:latest   *:80->2222/tcp
mxin1o40530h        account-stack_visualizer        replicated          1/1                 dockersamples/visualizer:stable        *:7001->8080/tcp


docker service ps account-stack

docker stack ps account-stack
ID                  NAME                              IMAGE                                  NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
g7iud46vufo6        account-stack_account-service.1   madhukargunda/account-service:latest   worker-1            Running             Running 8 minutes ago                       
vyvxhfub1hv4        account-stack_visualizer.1        dockersamples/visualizer:stable        manager1            Running             Running 9 minutes ago 


docker stack services account-stack

ID                  NAME                            MODE                REPLICAS            IMAGE                                  PORTS
mxin1o40530h        account-stack_visualizer        replicated          1/1                 dockersamples/visualizer:stable        *:7001->8080/tcp
oh76u5b6lyz3        account-stack_account-service   replicated          1/1                 madhukargunda/account-service:latest   *:80->2222/tcp

docker-machine ls

NAME            ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER     ERRORS
load-balancer   -        virtualbox   Running   tcp://192.168.99.101:2376           v18.09.5   
manager1        *        virtualbox   Running   tcp://192.168.99.100:2376           v18.09.5   
worker-1        -        virtualbox   Running   tcp://192.168.99.102:2376           v18.09.5   
worker-2        -        virtualbox   Running   tcp://192.168.99.103:2376           v18.09.5


Step13 : Use the loadbalancer IP and Port access the swagger-ui.html

Step14 : Scale the account-service

docker service scale oh7=3
oh7 scaled to 3
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>] 
verify: Service converged 
madhu@Admins-MacBook-Pro:~/dockerfiles/docker-swarm-loadbalancing-haproxy$ docker stack services account-stack
ID                  NAME                            MODE                REPLICAS            IMAGE                                  PORTS
mxin1o40530h        account-stack_visualizer        replicated          1/1                 dockersamples/visualizer:stable        *:7001->8080/tcp
oh76u5b6lyz3        account-stack_account-service   replicated          3/3                 madhukargunda/account-service:latest   *:80->2222/tcp

Step15 : Open the docker swarm visualizer to verify \

http://192.168.99.100:7001/

```

<img width="1643" alt="docker-swarm-visualizer" src="https://user-images.githubusercontent.com/5623861/56455453-15e13800-6391-11e9-9367-b1324b1437d7.png">

```
Step16 : tail the docker service logs

docker service logs oh7 -f

Step17 : Launch the http://192.168.99.101/swagger-ui.html and invoke the rest api URL 
         and we can see each request routed to one service.
         
```
<img width="1675" alt="load-balancing-request" src="https://user-images.githubusercontent.com/5623861/56455726-c3a21600-6394-11e9-89cc-830a90354257.png">












