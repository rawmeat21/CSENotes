Docker Compose is designed to simplify running multi-container applications using a single command:

`docker compose [-f <arg>...] [options] [COMMAND] [ARGS...]`


![Pasted image 20260913130345](../assets/Pasted%20image%2020260913130345.png)


It is basically used to provide options to docker commands.

### Single container example

Create a new file called: `docker-compose.yaml`. 

An example docker compose to run a mysql image will look like:

```yaml
services:
	mysqldb:
		image: 'mysql:latest'
		environment: 
		- MY_SQL_ROOT_PASSWORD="root"
		- MY_SQL_DATABASE="userinfo"
		container_name: "mysqldb"
```

Now run:

```
$ docker-compose up
```
This runs the container. You need to run this command in the same folder as `docker-compose.yaml`

To stop and remove:

```
$ docker-compose down
```

The image is still there, but the container will be stopped and deleted. 

To run in detached mode:

```
$ docker-compose up -d
```

### What about multiple containers?

![Pasted image 20260913132111](../assets/Pasted%20image%2020260913132111.png)

You should first refer a bit to `Communication to and from containers.md`. Glance over the whole thing to get an idea of what is being worked on.

```yaml
services:
	mysqldb:
		image: 'mysql:latest'
		environment: 
		- MY_SQL_ROOT_PASSWORD=root
		- MY_SQL_DATABASE=userinfo
		container_name: "mysqldb"
		healthcheck:
			test: ['CMD', 'msqladmin', 'ping', '-h', 'localhost']
			timeout: 20s
			retries: 10
	mypy:
		build: ./
		container_name: "mypyapp"
		depends_on:
			mysqldb:
				condition: service_healthy
		stdin_open: true
		tty: true
```

Now, we are first building our python app. 

`build`: Where is the Dockerfile. You can provide a path relative to where `docker-compose.yaml` is.

`depends_on`: Our python file needs the mysql server up and running. It cannot start before the mysql container. So, we add the service `mysqldb` as a dependency. The condition we check is `service_healthy`, which is basically a successful `health_check`. 

`health_check`: It is simply a command used to check if mysql software is healthy (ready to accept queries / is running)

`stdin_open`: Required for input
`tty`: for `-t` option


But doing `docker_compose up` doesn't work (input is not taken).

![Pasted image 20260913134010](../assets/Pasted%20image%2020260913134010.png)


Well you can start the services one by one:

```bash
$ docker-compose run mysqldb -d
$ docker-compose run mypy
```

Even better, you can just start `mypy`, and it will start `mysqldb` as a dependency:

```bash
$ docker-compose run mypy
```
![Pasted image 20260913134335](../assets/Pasted%20image%2020260913134335.png)


But, the question comes, **how does this even work? Where's the network?**

Answer: All service in a `docker-compose.yaml` file operate in the same network!

To add your own network:

```yaml
services:
	mysqldb:
		image: 'mysql:latest'
		environment: 
		- MY_SQL_ROOT_PASSWORD=root
		- MY_SQL_DATABASE=userinfo
		container_name: "mysqldb"
		networks:
		- my-network
		healthcheck:
			test: ['CMD', 'msqladmin', 'ping', '-h', 'localhost']
			timeout: 20s
			retries: 10
	mypy:
		build: ./
		container_name: "mypyapp"
		networks:
		- my-network
		depends_on:
			mysqldb:
				condition: service_healthy
		stdin_open: true
		tty: true
		
networks:
	- my-network:
```


**Tip:** When you do `docker-compose down`, by default, only the containers are removed. The binds or networks aren't. To make sure they do get deleted:

```bash
docker-compose down -v
```

#### Using bind mounts with docker compose

```yaml
services:
	mysqldb:
		image: 'mysql:latest'
		environment: 
		- MY_SQL_ROOT_PASSWORD=root
		- MY_SQL_DATABASE=userinfo
		container_name: "mysqldb"
		networks:
		- my-network
		healthcheck:
			test: ['CMD', 'msqladmin', 'ping', '-h', 'localhost']
			timeout: 20s
			retries: 10
	mypy:
		build: ./
		container_name: "mypyapp"
		networks:
		- my-network
		volumes:
		- ./servers.txt:/myapp/servers.txt
		depends_on:
			mysqldb:
				condition: service_healthy
		stdin_open: true
		tty: true
		
networks:
	- my-network:
```

#### Ports

The syntax is pretty damn simple:

```yaml
services:
	service_name:
		...
		ports:
		- 8080:3000
		...
```

Its always host_port:container_port


