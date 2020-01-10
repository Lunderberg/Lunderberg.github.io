---
layout: post
title: Midgard, Closing Socket Cleanly
tags: [c++, midgard]
---

Using Ctrl-C to send SIGINT is the easiest and most habitual way to
kill a program.  When SIGINT is received by a program, unless
explicitly handled, the program will be interrupted and close.  In
this case, the socket we are listening on is not closed cleanly.
While not particularly necessary for future development, this becomes
obnoxious, as it takes up to a minute for the OS to clean up the open
socket, during which a new instance of the program cannot be started.

The simplest way to adjust behavior on receiving signals is by
registering a signal handler.  If registered, a program will jump to
the signal handler on, then resume normal execution once completed.
While this allows user code to be run when a signal is received, there
are significant limitations to it.  The signal handler must be a
C-style function, and so it cannot have any context passed through a
`std::function`.  Any communication with the main program needs to be
done through global variables of type `sig_atomic_t`.

In this case, the resource held by the `WebServer` class is the open
socket.  According to [the websocketpp
documentation](https://docs.websocketpp.org/faq.html), in order to
cleanly exit, we need to call `stop_listening()`, call `close()` on
all existing connections, then wait for the call to
`io_service::run()` to finish.  This last one is the hard part, since
by the time the object is being destroyed, we have already exited out
of the `io_service::run()` call.

We can work around this by adding another thread.  The `WebServer`
object is constructed on the main thread, and owns a server thread.
The server thread runs `io_service::run()`, and the main thread waits
for the server thread to finish.  If SIGINT is received, the main
thread, in the destructor of `WebServer`, closes the connections, then
waits for the server thread to finish.

Lastly, even though the socket has been closed cleanly, it is possible
that the browser has just sent a packet.  If another program binds to
the same port, it could receive that packet instead.  For security
reasons, therefore, the default is to wait until any sent packet would
have times out before allowing another program to bind to the port.
To disable this behavior, the `SO_REUSEADDR` flag can be set.  In a
system with untrusted users, we would not want to set this, but on a
single-user machine, it is okay.

With that, we can test more quickly, since we don't need to wait a few
minutes to restart the server.