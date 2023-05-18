### High level design
The server listens for incoming connections on a port 8080. and processes client requests in separate threads. This allows the server to handle multiple concurrent connections.

Upon receiving a client request, the server checks if it is a valid GET request and extracts the requested file name. It then decodes the URL-encoded file name and determines the file's extension to identify the appropriate MIME type for the response.

Using a case-insensitive file search, the server checks if the requested file exists in the current directory. If it does, the server constructs an HTTP response with the appropriate headers, including the determined MIME type, and sends the file's contents to the client. If the file is not found, the server sends a 404 Not Found response.

The server continues to accept and process incoming connections until it is manually terminated.

### Problems encountered
1. After re-compiling and restarting server after every change, I got error `bind failed: address already in use`. I was confused because I've already killed every process that uses port 8080. After some [research](https://stackoverflow.com/questions/15198834/bind-failed-address-already-in-use), I found out that "the server still owns the socket when it starts and terminates quickly". So I add `SO_REUSEADDR` to the socket config, which "tells the server that even if this port is busy, go ahead and reuse it anyway".

2. After increasing buffer to 10MiB, I got error `bus error` every time I started the server. After some research, I found out that the stack memory allocated for each thread is usually limited. So I allocate the buffers on the heap instead of the stack by using `malloc()` and `free()`.
