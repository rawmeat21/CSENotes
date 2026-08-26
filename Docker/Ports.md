## Allowing external connections into containers

- **Sending messages:** Programs can send messages to [URL](https://en.wikipedia.org/wiki/URL) addresses such as this: http://127.0.0.1:3000 where HTTP is the [_protocol_](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol), 127.0.0.1 is an IP address, and 3000 is a [_port_](https://en.wikipedia.org/wiki/Port_\(computer_networking\)). Note the IP part could also be a [_hostname_](https://en.wikipedia.org/wiki/Hostname): 127.0.0.1 is also called [_localhost_ ](https://en.wikipedia.org/wiki/Localhost) so instead you could use http://localhost:3000.

- **Receiving messages:** Programs can be assigned to listen to any available port. If a program is listening for traffic on port 3000, and a message is sent to that port, the program will receive and possibly process it.

The address _127.0.0.1_ and hostname _localhost_ are special ones, they refer to the machine or container itself, so if you are on a container and send a message to _localhost_, the target is the same container. Similarly, if you are sending the request from outside of a container to _localhost_, the target is your machine.

It is possible to **map a port on your host machine to a container port**

For example, if you map port 1000 on your host machine to port 2000 in the container, and then send a message to http://localhost:1000 on your computer, the container will receive that message if it's listening to its port 2000.


### How to open connections?

Opening a connection from the outside world to a Docker container happens in two steps:

- Exposing port
- Publishing port

**Exposing a container port** means informing Docker that the container **listens to a certain port**. This doesn't do much, except it helps humans with the configuration.

To expose a port, add the line `EXPOSE <port>` in your Dockerfile.


**Publishing a port** means that Docker will map **host ports to the container ports**.

To publish a port, run the container with `-p <host-port>:<container-port>`


If you leave out the host port and only specify the container port, Docker will automatically choose a free port as the host port.

```bash
docker run -p 4567 app-in-port
```

We could also limit connections to a certain protocol only, e.g. UDP by adding the protocol at the end: `EXPOSE <port>/udp` and `-p <host-port>:<container-port>/udp`.


**Solution to Assignment 1.9**

```bash
docker run -p 127.0.0.1:8080:8080 web-server:latest
```


