2.7.1 Socket Programming with UDP
In this subsection, we’ll write simple client-server programs that use UDP; in the following section, we’ll
write similar programs that use TCP.
Recall from Section 2.1 that processes running on different machines communicate with each other by
sending messages into sockets. We said that each process is analogous to a house and the process’s
socket is analogous to a door. The application resides on one side of the door in the house; the
transport-layer protocol resides on the other side of the door in the outside world. The application
developer has control of everything on the application-layer side of the socket; however, it has little
control of the transport-layer side.
Now let’s take a closer look at the interaction between two communicating processes that use UDP
sockets. Before the sending process can push a packet of data out the socket door, when using UDP, it
must first attach a destination address to the packet. After the packet passes through the sender’s
socket, the Internet will use this destination address to route the packet through the Internet to the
socket in the receiving process. When the packet arrives at the receiving socket, the receiving process
will retrieve the packet through the socket, and then inspect the packet’s contents and take appropriate
action.
So you may be now wondering, what goes into the destination address that is attached to the packet?
As you might expect, the destination host’s IP address is part of the destination address. By including
the destination IP address in the packet, the routers in the Internet will be able to route the packet
through the Internet to the destination host. But because a host may be running many network
application processes, each with one or more sockets, it is also necessary to identify the particular
socket in the destination host. When a socket is created, an identifier, called a port number, is assigned
to it. So, as you might expect, the packet’s destination address also includes the socket’s port number.
In summary, the sending process attaches to the packet a destination address, which consists of the
destination host’s IP address and the destination socket’s port number. Moreover, as we shall soon see,
the sender’s source address—consisting of the IP address of the source host and the port number of the
source socket—are also attached to the packet. However, attaching the source address to the packet is
typically not done by the UDP application code; instead it is automatically done by the underlying
operating system.
We’ll use the following simple client-server application to demonstrate socket programming for both
UDP and TCP:
1. The client reads a line of characters (data) from its keyboard and sends the data to the server.
2. The server receives the data and converts the characters to uppercase.
3. The server sends the modified data to the client.
4. The client receives the modified data and displays the line on its screen.
Figure 2.27 highlights the main socket-related activity of the client and server that communicate over
the UDP transport service.
Now let’s get our hands dirty and take a look at the client-server program pair for a UDP implementation
of this simple application. We also provide a detailed, line-by-line analysis after each program. We’ll
begin with the UDP client, which will send a simple application-level message to the server. In order for
Figure 2.27 The client-server application using UDP
the server to be able to receive and reply to the client’s message, it must be ready and running—that is,
it must be running as a process before the client sends its message.
The client program is called UDPClient.py, and the server program is called UDPServer.py. In order to
emphasize the key issues, we intentionally provide code that is minimal. “Good code” would certainly
have a few more auxiliary lines, in particular for handling error cases. For this application, we have
arbitrarily chosen 12000 for the server port number.
UDPClient.py
Here is the code for the client side of the application:
from socket import *
serverName = ’hostname’
serverPort = 12000
clientSocket = socket(AF_INET, SOCK_DGRAM)
message = raw_input(’Input lowercase sentence:’)
clientSocket.sendto(message.encode(),(serverName, serverPort))
modifiedMessage, serverAddress = clientSocket.recvfrom(2048)
print(modifiedMessage.decode())
clientSocket.close()
Now let’s take a look at the various lines of code in UDPClient.py.
from socket import *
The socket module forms the basis of all network communications in Python. By including this line, we
will be able to create sockets within our program.
serverName = ’hostname’
serverPort = 12000
The first line sets the variable serverName to the string ‘hostname’. Here, we provide a string
containing either the IP address of the server (e.g., “128.138.32.126”) or the hostname of the server
(e.g., “cis.poly.edu”). If we use the hostname, then a DNS lookup will automatically be performed to get
the IP address.) The second line sets the integer variable serverPort to 12000.
clientSocket = socket(AF_INET, SOCK_DGRAM)
This line creates the client’s socket, called clientSocket . The first parameter indicates the address
family; in particular, AF_INET indicates that the underlying network is using IPv4. (Do not worry about
this now—we will discuss IPv4 in Chapter 4.) The second parameter indicates that the socket is of type
SOCK_DGRAM , which means it is a UDP socket (rather than a TCP socket). Note that we are not
specifying the port number of the client socket when we create it; we are instead letting the operating
system do this for us. Now that the client process’s door has been created, we will want to create a
message to send through the door.
message = raw_input(’Input lowercase sentence:’)
raw_input() is a built-in function in Python. When this command is executed, the user at the client is
prompted with the words “Input lowercase sentence:” The user then uses her keyboard to input a line,
which is put into the variable message . Now that we have a socket and a message, we will want to
send the message through the socket to the destination host.
clientSocket.sendto(message.encode(),(serverName, serverPort))
In the above line, we first convert the message from string type to byte type, as we need to send bytes
into a socket; this is done with the encode() method. The method sendto() attaches the destination
address ( serverName, serverPort ) to the message and sends the resulting packet into the
process’s socket, clientSocket . (As mentioned earlier, the source address is also attached to the
packet, although this is done automatically rather than explicitly by the code.) Sending a client-to-server
message via a UDP socket is that simple! After sending the packet, the client waits to receive data from
the server.
modifiedMessage, serverAddress = clientSocket.recvfrom(2048)
With the above line, when a packet arrives from the Internet at the client’s socket, the packet’s data is
put into the variable modifiedMessage and the packet’s source address is put into the variable
serverAddress . The variable serverAddress contains both the server’s IP address and the
server’s port number. The program UDPClient doesn’t actually need this server address information,
since it already knows the server address from the outset; but this line of Python provides the server
address nevertheless. The method recvfrom also takes the buffer size 2048 as input. (This buffer size
works for most purposes.)
print(modifiedMessage.decode())
This line prints out modifiedMessage on the user’s display, after converting the message from bytes to
string. It should be the original line that the user typed, but now capitalized.
clientSocket.close()
This line closes the socket. The process then terminates.
UDPServer.py
Let’s now take a look at the server side of the application:
from socket import *
serverPort = 12000
serverSocket = socket(AF_INET, SOCK_DGRAM)
serverSocket.bind((’’, serverPort))
print(”The server is ready to receive”)
while True:
 message, clientAddress = serverSocket.recvfrom(2048)
 modifiedMessage = message.decode().upper()
 serverSocket.sendto(modifiedMessage.encode(), clientAddress)
Note that the beginning of UDPServer is similar to UDPClient. It also imports the socket module, also
sets the integer variable serverPort to 12000, and also creates a socket of type SOCK_DGRAM (a
UDP socket). The first line of code that is significantly different from UDPClient is:
serverSocket.bind((’’, serverPort))
The above line binds (that is, assigns) the port number 12000 to the server’s socket. Thus in
UDPServer, the code (written by the application developer) is explicitly assigning a port number to the
socket. In this manner, when anyone sends a packet to port 12000 at the IP address of the server, that
packet will be directed to this socket. UDPServer then enters a while loop; the while loop will allow
UDPServer to receive and process packets from clients indefinitely. In the while loop, UDPServer waits
for a packet to arrive.
message, clientAddress = serverSocket.recvfrom(2048)
This line of code is similar to what we saw in UDPClient. When a packet arrives at the server’s socket,
the packet’s data is put into the variable message and the packet’s source address is put into the
variable clientAddress . The variable clientAddress contains both the client’s IP address and the
client’s port number. Here, UDPServer will make use of this address information, as it provides a return
address, similar to the return address with ordinary postal mail. With this source address information,
the server now knows to where it should direct its reply.
modifiedMessage = message.decode().upper()
This line is the heart of our simple application. It takes the line sent by the client and, after converting the
message to a string, uses the method upper() to capitalize it.
serverSocket.sendto(modifiedMessage.encode(), clientAddress)
This last line attaches the client’s address (IP address and port number) to the capitalized message
(after converting the string to bytes), and sends the resulting packet into the server’s socket. (As
mentioned earlier, the server address is also attached to the packet, although this is done automatically
rather than explicitly by the code.) The Internet will then deliver the packet to this client address. After
the server sends the packet, it remains in the while loop, waiting for another UDP packet to arrive (from
any client running on any host).
To test the pair of programs, you run UDPClient.py on one host and UDPServer.py on another host. Be
sure to include the proper hostname or IP address of the server in UDPClient.py. Next, you execute
UDPServer.py, the compiled server program, in the server host. This creates a process in the server
that idles until it is contacted by some client. Then you execute UDPClient.py, the compiled client
program, in the client. This creates a process in the client. Finally, to use the application at the client,
you type a sentence followed by a carriage return.
To develop your own UDP client-server application, you can begin by slightly modifying the client or
server programs. For example, instead of converting all the letters to uppercase, the server could count
the number of times the letter s appears and return this number. Or you can modify the client so that
after receiving a capitalized sentence, the user can continue to send more sentences to the server.
