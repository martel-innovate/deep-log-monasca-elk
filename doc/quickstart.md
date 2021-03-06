## Quick Start
Installing and running the Deep Log Inspection system requires [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) and [Docker Engine](https://www.docker.com/) in [Swarm mode](https://docs.docker.com/engine/swarm/):

1. Set up the [Docker Swarm cluster](https://docs.docker.com/get-started/part4/#create-a-cluster) where services will be deployed
2. On the cluster's manager node, download the project by cloning the git repository:

        git clone https://github.com/martel-innovate/deep-log-inspection.git

3. On all nodes in the cluster, set user limits in Docker [configuration file](https://github.com/martel-innovate/deep-log-inspection/blob/master/log-server/config/docker.service). For more details, refer to the [Docker configuration section](docker.md)
4. On the manager node, create the overlay network where the services will be deployed, and the storage volumes for Elasticsearch data and logs:

        docker network create -d overlay backend
        docker volume create esdata
        docker volume create eslogs

5. Let `deeplog` be the name of the stack where we deploy all services to, then:

        docker stack deploy -c docker-compose.yml deeplog

The compose file can be found [here](https://github.com/martel-innovate/deep-log-inspection/blob/master/log-server/docker-compose.yml).

## Services and DNS
The system is meant to be deployed on a multi-node cluster, in which the master node is accessible with three valid names. Let `example.com` be the main name and `elastic.example.com` and `kibana.example.com` two alternative names (in this page and throughout the full guide). Then, the following services should be exposed on the master node:

* monasca-log-api at `example.com`
* elasticsearch at `elastic.example.com`
* kibana at `kibana.example.com`
* traefik dashboard at `example.com:8080`

Optionally (for development/test):

* keystone at `example.com` on ports 5000 and 35357
    * see [`preload.yml`](https://github.com/martel-innovate/deep-log-inspection/blob/master/log-server/keystone/preload.yml) for full account info

To bring keystone up along with the other services, deploy it to the same stack:

    docker stack deploy -c docker-compose-keystone.yml deeplog

The compose file for keystone can be found [here](https://github.com/martel-innovate/deep-log-inspection/blob/master/log-server/docker-compose-keystone.yml).

## System is up
Once the system is up and running, you can check its status either by visiting Kibana or by querying Elasticsearch's Health API:

    curl https://elastic.example.com/_cluster/health

To visualize information about the running services, including name and ports:

    docker service ls

You also can apply later changes to a service, e.g.:

    docker service update deeplog_elasticsearch

or reload all services by redeploying the system, just like the first time:

    docker stack deploy -c docker-compose.yml deeplog

To completely clean the environment:

    docker stack rm deeplog

For the full CLI reference, please refer to the [Docker CLI reference guide](https://docs.docker.com/engine/reference/commandline/cli/).
