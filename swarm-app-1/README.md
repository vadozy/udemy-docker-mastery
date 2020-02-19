# Assignment: Create A Multi-Service Multi-Node Web App

## Goal: create networks, volumes, and services for a web-based "cats vs. dogs" voting app.
Here is a basic diagram of how the 5 services will work:

![diagram](./architecture.png)
- All images are on Docker Hub, so you should use editor to craft your commands locally, then paste them into swarm shell (at least that's how I'd do it)
- a `backend` and `frontend` overlay network are needed. Nothing different about them other then that backend will help protect database from the voting web app. (similar to how a VLAN setup might be in traditional architecture)
- The database server should use a named volume for preserving data. Use the new `--mount` format to do this: `--mount type=volume,source=db-data,target=/var/lib/postgresql/data`

### Services (names below should be service names)
- vote
    - bretfisher/examplevotingapp_vote
    - web front end for users to vote dog/cat
    - ideally published on TCP 80. Container listens on 80
    - on frontend network
    - 2+ replicas of this container

- redis
    - redis:3.2
    - key/value storage for incoming votes
    - no public ports
    - on frontend network
    - 1 replica NOTE VIDEO SAYS TWO BUT ONLY ONE NEEDED

- worker
    - bretfisher/examplevotingapp_worker:java
    - backend processor of redis and storing results in postgres
    - no public ports
    - on frontend and backend networks
    - 1 replica

- db
    - postgres:9.4
    - one named volume needed, pointing to /var/lib/postgresql/data
    - on backend network
    - 1 replica
    - remember set env for password-less connections -e POSTGRES_HOST_AUTH_METHOD=trust

- result
    - bretfisher/examplevotingapp_result
    - web app that shows results
    - runs on high port since just for admins (lets imagine)
    - so run on a high port of your choosing (I choose 5001), container listens on 80
    - on backend network
    - 1 replica


node1

ubuntu@ip-10-0-0-75:~$ docker swarm init
Swarm initialized: current node (7atfoqtrm77neg15disql72xe) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0whcvo2ouw7ritjc8skbxpp7t9koe36nwl4n3dx0e7k5fzlrp2-66da75lsbh32ung686hlqkj3j 10.0.0.75:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

ubuntu@ip-10-0-0-75:~$ 


node2

ubuntu@ip-10-0-0-248:~$ docker swarm join --token SWMTKN-1-0whcvo2ouw7ritjc8skbxpp7t9koe36nwl4n3dx0e7k5fzlrp2-66da75lsbh32ung686hlqkj3j 10.0.0.75:2377
This node joined a swarm as a worker.
ubuntu@ip-10-0-0-248:~$



ubuntu@ip-10-0-0-75:~$ docker network create -d overlay frontend
ubuntu@ip-10-0-0-75:~$ docker network create -d overlay backend

ubuntu@ip-10-0-0-75:~$ docker service create --name db --network backend --mount type=volume,source=db-data,target=/var/lib/postgresql/data --replicas 1 postgres:9.4

ubuntu@ip-10-0-0-75:~$ docker service create --name worker --network backend --network frontend --replicas 1 dockersamples/examplevotingapp_worker 

ubuntu@ip-10-0-0-75:~$ docker service create --name redis --network frontend --replicas 1 redis:3.2 

ubuntu@ip-10-0-0-75:~$ docker service create --name result --network backend --replicas 1 -p 5001:80 dockersamples/examplevotingapp_result:before 

ubuntu@ip-10-0-0-75:~$ docker service create --name vote --network frontend --replicas 3 -p 80:80 dockersamples/examplevotingapp_vote:before 