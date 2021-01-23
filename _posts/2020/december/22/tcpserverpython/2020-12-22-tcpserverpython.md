---
layout: post
title: Building a TCP server @ Python for pentesting
categories: projects, python
permalink: /tcp-server-python/
published: true
---

In this blog post we will be looking at how to create a TCP server with Python. We need to understand how sockets work for our upcoming projects and creating a TCP server is a good exemple. In this post we will be focusing on creating the server, and in the next we will be creating the client. We will also see how they actually communicate with each other.

### But what is a socket anyway?

Sockets are very important in penetration testing because they initialize connection. They allow the sending and receiving of data. In terms of technicality a socket is an internal endpoint for sending and receiving data. In simpler terms, a socket is like an outlet: it gives you connection and power from the grid to the transformer so that you can use the power. The thing with outlets though is that you cannot send the power. Network sockets on the other hand allow you to send *and* receive data. It's an internal endpoint which means it works locally. 
The socket module in the Python standard library allows you to implement sockets using differents protocols. Today we will be looking at TCP but we also have UDP connections available.

### Writing the code

Since we are running this on Linux we need to specify the working directory for Python 3. On Windows it's automatic and you don't have to do this. Also since we will be working with sockets we need to import the socket module.


```python
#!/usr/bin/python3

import socket
```

Now we need to create the socket object. We will call it ```serversocket```. We then call the socket function, specifying as arguments the socket family and the socket type. ```AF_INET``` stands for IPv4, and ```SOCK_STREAM``` means we are using a TCP connection.

```python
#!/usr/bin/python3

import socket

serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```

Having specify our socket family and type, we now need to create another object that will store the host name by using the function ```gethostbyname```. This function we'll get the address and any other information that we will specify.
I chose 999 as the port.

```python
#!/usr/bin/python3

import socket

serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

host = socket.gethostbyname()
port = 999
```

Our next step is to bind the values ```host``` and ```port``` to the object that we created which is ```serversocket```.

```python
#!/usr/bin/python3

import socket

serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

host = socket.gethostbyname()
port = 999

serversocket.bind((host, port))
```

So far everything makes sense. Now we need to set up a TCP listener that listens for TCP connections from the client. Our listener we'll be listening for 3 connections at a time.

```python
#!/usr/bin/python3

import socket

serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

host = socket.gethostbyname()
port = 999

serversocket.bind((host, port))

serversocket.listen(3)
```

Our last step will require writing a while loop. We create a new object ```clientsocket``` that we will use in a next post, and well as ```address``` which are going to countain the TCP informations coming from the client.
We can also print some data which is always useful for readability. I love priting when writing code. As for the client socket information, we need to convert that into a string. The message is for our TCP client, and is sent by the function ```send``` on the next line.
Finally, we can close the socket that we created. 

```python
#!/usr/bin/python3

import socket

serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

host = socket.gethostbyname()
port = 999

serversocket.bind((host, port))

serversocket.listen(3)

while True:
	clientsocket, address = serversocket.accept()

	print(f"Received connection from " % str(address))

	message = "Thank you for connecting to the server." + "\r\n"
	clientsocket.send(message)

	clientsocket.close()
```

### Conclusion

We've imported the socket module from the python standard library. We then created a server socket object, which then called the server socket function. We specified the socket family and the socket type. We give that value to host and also specified the port number.
Next we binded the address (which is host & port) to the socket. We then set up a TCP listener which will listen to TCP connections up to a maximum of 3.
While all of this is true, the client socket and the address are going to be the informations we accept from our clients. We also specify that we want to convert the address to a string, and wrote a thank you message. Finishing all of this by closing the socket.