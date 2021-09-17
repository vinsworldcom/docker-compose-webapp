# Load Balancing Sample

## Overview

This uses `docker-compose`, Docker Swarm and `docker stack` to create a small 
network of containers:

```
               (5000) |--> App (Flask) --| (6379)
    :8080 NGINX ------|                  |------- redis (counter)
               (5000) |--> App (Flask) --|
                            .            
                            .            
                            .            
```

The 'NGINX' front-end listens on `localhost:8080` and load balances requests
to the Python Flask application servers, which connect to a single Python
'Redis' counter container.  The displayed web pages is a simple message:

```
Hello from App running on 172.26.0.X!

I have been seen # times.
```

Where 'X' is the server's IP address and '#' is the number of times the page 
has been loaded.  Note this is a universal counter, not individual for each 
application.  This simulates a load balancer to front-end many application 
servers all connecting to a single back-end database server.

## Build (`docker-compose`)

To launch, just open a terminal, go to the same level with 
'docker-compose.yml', and execute:

`docker-compose up -d --scale app=X`

Where 'X' is the number of application servers wanted - 2 is recommended at 
least.

This will spin up the Docker architecture.

### Test

Open a browser:

`http://localhost:8080`

Press `F5` or refresh / reload the page and you should see the IP address 
('X') change as well as the counter ('#') for number of times the page is 
loaded.

### Debug

The 'docker-compose.yml' file specifies:

```
    environment:
      - FLASK_ENV
```

for each application container.  The `docker-compose` command looks at the 
'.env' file in this directory and sets the value to 'development'.  This 
allows changes to be made to the `app/app.py` files in real-time and have 
them reflected in subsequent page reloads without having to stop and restart 
any services.

### Stop

To quit (`CTRL+C` in the terminal if not called with `-d` option)

`docker-compose down`

## Deploy (`docker swarm / stack`)

To deploy, ensure a Docker Swarm is running:

`dccker swarm init`

Then deploy:

`docker stack deploy -c docker-compose.yml myweb`

Verify it is running:

`docker stack ls`

### Test

Open a browser:

`http://localhost:8080`

Press `F5` or refresh / reload the page and you should see the counter ('#') 
for number of times the page is loaded change, but not the IP address ('X').
There is only 1 Python Flask application running.

### Scale

Scaling the service is adding more application nodes, similar to the `--scale` 
argument to `docker-compose` from the 'Build' phase.  To make 2 Python Flask 
applications:

`docker service scale myweb_app=2`

Verify the scaling worked by looking at the 'REPLICAS' in the following 
command:

```
C:\> docker service ls
ID           NAME        MODE       REPLICAS IMAGE              PORTS
m69zrbz3bbzx myweb_app   replicated 2/2      app_myweb:latest   
p8lc1l6oszjy myweb_nginx replicated 1/1      nginx_myweb:latest *:8080->80/tcp
uob8mlz006tz myweb_redis replicated 1/1      redis:alpine
```

Now, open a browser:

`http://localhost:8080`

Press `F5` or refresh / reload the page and you should see the counter ('#') 
for number of times the page is loaded change, as well as the IP address ('X').

### Stop

To stop:

`docker stack rm myweb`

## Cleanup

If you started the Docker Swarm on the computer specifically for this test, 
you can clean up with:

```
docker swarm leave --force
docker network prune -f
```
