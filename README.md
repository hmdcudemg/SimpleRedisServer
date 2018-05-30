# SimpleRedisServer
## Autor
[charlesleifer.com](http://charlesleifer.com/blog/building-a-simple-redis-server-with-python/)

## Description
The other day the idea occurred to me that it would be neat to write a simple Redis-like database server. While I've had plenty of experience with WSGI applications, a database server presented a novel challenge and proved to be a nice practical way of learning how to work with sockets in Python. In this post I'll share what I learned along the way.

The goal of my project was to write a simple server that I could use with a task queue project of mine called huey. Huey uses Redis as the default storage engine for tracking enqueued jobs, results of finished jobs, and other things. For the purposes of this post, I've reduced the scope of the original project even further so as not to muddy the waters with code you could very easily write yourself, but if you're curious, you can check out the end result here (documentation).

The server we'll be building will be able to respond to the following commands:
- GET <key>
- SET <key> <value>
- DELETE <key>
- FLUSH
- MGET <key1> ... <keyn>
- MSET <key1> <value1> ... <keyn> <valuen>

We'll support the following data-types as well:
- Strings and Binary Data
- Numbers
- NULL
- Arrays (which may be nested)
- Dictionaries (which may be nested)
- Error messages

To handle multiple clients asynchronously, we'll be using gevent, but you could also use the standard library's SocketServer module with either the ForkingMixin or the ThreadingMixin.

## Testing the Server
To test the server, just execute the server's Python module from the command line. In another terminal, open up a Python interpreter and import the Client class from the server's module. Instantiating the client will open a connection and you can start running commands!

```
>>> from server_ex import Client
>>> client = Client()
>>> client.mset('k1', 'v1', 'k2', ['v2-0', 1, 'v2-2'], 'k3', 'v3')
3
>>> client.get('k2')
['v2-0', 1, 'v2-2']
>>> client.mget('k3', 'k1')
['v3', 'v1']
>>> client.delete('k1')
1
>>> client.get('k1')
>>> client.delete('k1')
0
>>> client.set('kx', {'vx': {'vy': 0, 'vz': [1, 2, 3]}})
1
>>> client.get('kx')
{'vx': {'vy': 0, 'vz': [1, 2, 3]}}
>>> client.flush()
2
```
The code presented in this post is absolutely for demonstration purposes only. I hope you enjoyed reading about this project as much as I enjoyed writing about it. You can find a original copy of the code [here](https://gist.github.com/coleifer/dbbedc287605dcc22990a6e549de9f36). To see a more complete example, check out [simpledb (documentation)](https://github.com/coleifer/simpledb).

## To extend the project, you might consider:

- Add more commands!
- Use the protocol handler to implement an append-only command log
- More robust error handling
- Allow client to close connection and re-connect
- Logging
- Re-write to use the standard library's SocketServer and ThreadingMixin