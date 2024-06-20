---
title: "What happens when tcp listen to a port?"
date: 2020-06-29T18:24:09+08:00
draft: false
categories: [TCP/IP]
---

When reading the net/http code of golang, I was suddenly curious about the mechanism of tcp listen, then I googled and found this interesting Q&A - [What happens when we say "listen to a port"?](https://stackoverflow.com/questions/4530187/what-happens-when-we-say-listen-to-a-port) on StackOverflow.

> Q:When we start a server application, we always need to speicify the port number it listens to. But how is this "listening mechanism" implemented under the hood?
>
> My current _imagination_ is like this:
>
> The operating system associate the port number with some buffer. The server application's responsibiliy is to monitor this buffer. If there's no data in this buffer, the server application's listen operation will just **\*block\*** the application.
>
> When some data arrives from the wire, the operating system will **\*know\*** that and then check the data and see if it is targeted at this port number. And then it will fill the **\*corresponding\*** buffer. And then OS will notify the blocked server application and the server application will get the data and continue to run.
>
> Question is:
>
> - If the above scenario is correct, how could the opearting system **\*know\*** there's data arriving from wire? It cannot be a busy polling. Is it some kind of interrupt-based mechanism?
> - If there's too much data arriving and the buffer is not big enough, will there be data loss?
> - Is the "listen to a port" operation really a blocking operation?

The accepted answer is :

> There is _no buffer_ that the application monitors. Instead, the application calls listen() at some point, and the OS remembers from then on that this application is interested in new connections to that port number. Only one application can indicate interest in a certain port at any time.
>
> The listen operation does _not block_. Instead, it returns right away. What may block is [`accept()`](http://linux.die.net/man/2/accept). The system has a backlog of incoming connections (buffering the data that have been received), and returns one of the connections every time accept is called. accept doesn't transmit any data; the application must then do recv() calls on the accepted socket.
>
> As to your questions:
>
> - as others have said: hardware interrupts. The NIC takes the datagram completely off the wire, interrupts, and is assigned an address in memory to copy it to.
> - for TCP, there will be no data loss, as there will always be sufficient memory during the communication. TCP has flow control, and the sender will stop sending before the receiver has no more memory. For UDP and new TCP connections, there can be data loss; the sender will typically get an error indication (as the system reserves memory to accept just one more datagram).
> - see above: listen itself is not blocking; accept is.
