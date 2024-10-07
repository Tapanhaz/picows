.. image:: https://raw.githubusercontent.com/tarasko/picows/master/docs/source/_static/banner.png
    :align: center

Introduction
============

.. image:: https://badge.fury.io/py/picows.svg
    :target: https://pypi.org/project/picows
    :alt: Latest PyPI package version

.. image:: https://img.shields.io/pypi/dm/picows
    :target: https://pypistats.org/packages/picows
    :alt: Downloads count

.. image:: https://readthedocs.org/projects/picows/badge/?version=latest
    :target: https://picows.readthedocs.io/en/latest/
    :alt: Latest Read The Docs

**picows** is a high-performance python library designed for building asyncio WebSocket clients and servers.
Implemented in Cython, it offers exceptional speed and efficiency, surpassing other popular WebSocket python libraries.

.. image:: https://raw.githubusercontent.com/tarasko/picows/master/docs/source/_static/picows_benchmark.png
    :target: https://github.com/tarasko/picows/blob/master/docs/source/_static/picows_benchmark.png?raw=true
    :align: center


The above chart shows the performance of echo clients communicating with a server through a loopback interface using popular Python libraries.
`boost.beast client <https://www.boost.org/doc/libs/1_85_0/libs/beast/example/websocket/client/sync/websocket_client_sync.cpp>`_
is also included for reference. Typically, picows is ~1.5-2 times faster than aiohttp. All Python clients use uvloop. Please find the benchmark sources
`here <https://github.com/tarasko/picows/blob/master/examples/echo_client_benchmark.py>`_.

Installation
============

picows requires Python 3.8 or greater and is available on PyPI.
Use pip to install it::

    $ pip install picows


Getting started
===============

Echo client
-----------
Connects to an echo server, sends a message and disconnect upon reply.

.. code-block:: python

    import asyncio
    import uvloop
    from picows import ws_connect, WSFrame, WSTransport, WSListener, WSMsgType, WSCloseCode

    class ClientListener(WSListener):
        def on_ws_connected(self, transport: WSTransport):
            self.transport = transport
            transport.send(WSMsgType.TEXT, b"Hello world")

        def on_ws_frame(self, transport: WSTransport, frame: WSFrame):
            print(f"Echo reply: {frame.get_payload_as_ascii_text()}")
            transport.send_close(WSCloseCode.OK)
            transport.disconnect()


    async def main(url):
        (_, client) = await ws_connect(ClientListener, url)
        await client.transport.wait_disconnected()


    if __name__ == '__main__':
        asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())
        asyncio.run(main("ws://127.0.0.1:9001"))

This prints:

.. code-block::

    Echo reply: Hello world

Echo server
-----------

.. code-block:: python

    import asyncio
    import uvloop
    from picows import ws_create_server, WSFrame, WSTransport, WSListener, WSMsgType, WSUpgradeRequest

    class ServerClientListener(WSListener):
        def on_ws_connected(self, transport: WSTransport):
            print("New client connected")

        def on_ws_frame(self, transport: WSTransport, frame: WSFrame):
            if frame.msg_type == WSMsgType.PING:
                transport.send_pong(frame.get_payload_as_bytes())
            elif frame.msg_type == WSMsgType.CLOSE:
                transport.send_close(frame.get_close_code(), frame.get_close_message())
                transport.disconnect()
            else:
                transport.send(frame.msg_type, frame.get_payload_as_bytes())

    async def main():
        def listener_factory(r: WSUpgradeRequest):
            # Routing can be implemented here by analyzing request content
            return ServerClientListener()

        server: asyncio.Server = await ws_create_server(listener_factory, "127.0.0.1", 9001)
        for s in server.sockets:
            print(f"Server started on {s.getsockname()}")

        await server.serve_forever()

    if __name__ == '__main__':
      asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())
      asyncio.run(main())
