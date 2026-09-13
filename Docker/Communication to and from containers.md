![[Pasted image 20260913122545.png]]

Container talks to a remote server / API.

![[Pasted image 20260913122937.png]]
![[Pasted image 20260913122923.png]]

You have this python program which performs a GET request to a remote server.

Our Dockerfile:

```Dockerfile
from python
workdir /myapp
copy ./requester.py .
run pip install requests
cmd ["python", "requester.py"]
```

Build an image using `docker build .`

Now if we do: 

```
$ docker run 7342fd2
```

We get a random cat fact. Nice!


![[Pasted image 20260913122622.png]]

Container talks to a software on local machine.


![[Pasted image 20260913123618.png]]
![[Pasted image 20260913123651.png]]
![[Pasted image 20260913123700.png]]

Build and run:

```bash
$ docker build .
$ docker run -it 248gfw7t
```

You get an error like:

![[Pasted image 20260913123949.png]]

What's the problem? The file in your container cannot talk to the MySQL software on your local machine. It interprets `localhost` as it's own localhost, which gives error.

To fix:

![[Pasted image 20260913124133.png]]

Change host to `host.docker.internal`. (Target the machine where docker is installed)


![[Pasted image 20260913124419.png]]

Container talks to another container.

Do get a MySQL image:

```
$ docker pull mysql
$ docker run -d --env MY_SQL_ROOT_PASSWORD="root" --env MY-SQL_DATABASE="userinfo" --name mysqldb mysql
```

Now do:

```
$ docker inspect mysqldb
```

You will see a json output. Look at the Networks part:

![[Pasted image 20260913124912.png]]

Just copy that IP address.

Make changes:

![[Pasted image 20260913124953.png]]


### Docker network

![[Pasted image 20260913125725.png]]

The containers run in the same network, making communication between them a lot easier.

To create one:

```
$ docker network create my-net
```

You can check using `docker network ls`. Example:

![[Pasted image 20260913125922.png]]


Now start your mysql container:

```bash
$ docker run -d --env MY_SQL_ROOT_PASSWORD="root" --env MY-SQL_DATABASE="userinfo" --name mysqldb --network my-net mysql
```

Make changes to python program:

![[Pasted image 20260913130110.png]]

You just need the **name of the container**.

Now run the container:

```bash
$ docker run -it --rm --network my-net 7gui34rt3
```








