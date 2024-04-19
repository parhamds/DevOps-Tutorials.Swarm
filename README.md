# Docker Swarm Operations and Best Practices

This README provides a comprehensive guide to working with Docker Swarm, covering initialization, service management, secrets, zero-downtime upgrades, health checks, and service placement constraints.

## Docker Swarm

### Initializing Swarm

1. Check Swarm status:
    ```
    docker info
    ```

2. Initialize Swarm manager:
    ```
    docker swarm init --advertise-addr <ip_of_an_interface>
    ```

3. Join nodes to Swarm:
    ```
    docker swarm join --token SWMTKN-1-4q0p8tctmqmf4fcm68drobej2qcr8gmsgr0056md5hm5oj4i72-ek7pgwmi50spmipz7hs4g8l7o 192.168.251.32:2377
    ```

### Service Management

- Create a service:
    ```
    docker service create alpine ping google.com
    ```

- List services:
    ```
    docker service ls
    ```

- Inspect a service:
    ```
    docker service inspect <service_id>
    ```

- Show containers of a service:
    ```
    docker service ps happy_yonath
    ```

- Update service replicas:
    ```
    docker service update <service_name> --replicas 4
    ```

- Show join command for manager:
    ```
    docker swarm join-token manager
    ```

- List Swarm nodes:
    ```
    docker node ls
    ```

- Leave Swarm:
    ```
    docker swarm leave -f
    ```

### Networking

- List networks:
    ```
    docker network ls
    ```

- Create an overlay network:
    ```
    docker network create -d overlay <net_name>
    ```

- Create a service connected to the overlay network:
    ```
    docker service create --name postgres --network my_overlay -e POSTGRES_PASSWORD=mypassword postgres
    ```

### Stack Deployment

Deploy multiple services at once using a stack file:
```
docker stack deploy -c <stack_yaml_file> <stack_name>
```

## Swarm Secrets

### Creating Secrets

1. Create a secret from a file:
    ```
    docker secret create <secret_name> <secret_txt_file>
    ```

2. Create a secret using CLI:
    ```
    echo "<secret>" | docker secret create <secret_name> -
    ```

### Managing Secrets

- List secrets:
    ```
    docker secret ls
    ```

- Inspect a secret:
    ```
    docker secret inspect <secret_name>
    ```

### Using Secrets in Services

- Assign a secret to a service and use it:
    ```
    docker service create --name <service_name> --secret <secret_name> -e <env_var_of_pass>=run/secrets/<secret_name> <image_name>
    ```

### Example:

```
docker service create --name postgres --secret newdb --secret db -e POSTGRES_PASSWORD_FILE=run/secrets/db -e POSTGRES_USER_FILE=run/secrets/newdb postgres
```

- View secret inside a container:
    ```
    docker exec -it <cont_name> bash
    cat /run/secrets/<secret_name>
    ```

## Zero Downtime Upgrade

Ensure smooth service upgrades with zero downtime.

1. Create and scale service:
    ```
    docker service create -p 80:8080 --name web_server nginx:1.24
    docker service scale web_server=10
    ```

2. Update service image:
    ```
    docker service update --image nginx:1.25 web_server
    ```

3. Modify service port:
    ```
    docker service update --publish-rm 8080 --publish-add 9090:80 web_server
    ```

## Health Checks

Configure health checks for your services.

- Options:
    - `--interval`
    - `--timeout`
    - `--retries`
    - `--start-period`

Example health check on a PostgreSQL container:

1. Check health inside the container:
    ```
    docker run --name postgres1 -d -e POSTGRES_PASSWORD=mysecretpassword postgres
    docker exec -it <cont_id> bash
    pg_isready -U postgres
    ```

2. Health check on container creation:
    ```
    docker run --name postgres2 -d -e POSTGRES_PASSWORD=mysecretpassword --health-cmd="pg_isready -U postgres || exit 1" postgres
    ```

3. Health check on service creation:
    ```
    docker service create --name postgresservice2 -e POSTGRES_PASSWORD=mysecretpassword --health-cmd="pg_isready -U postgres || exit 1" postgres
    ```

## Placement with Service Constraint

Ensure services are placed on specific nodes.

1. Add node labels:
    ```
    docker node update --label-add=region=east-1-d <node_id>
    ```

2. Create service with constraints:
    ```
    docker service create --name postgress --constraint node.labels.region==east-1-d -e POSTGRES_PASSWORD=mysecretpassword postgres
    ```

3. Update constraints:
    ```
    docker service update --constraint-rm <constraint> --constraint-add <constraint> <service_name>
    ```

Feel free to modify and enhance this README according to your project's needs!