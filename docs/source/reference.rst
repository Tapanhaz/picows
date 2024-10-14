API reference
====================

.. automodule:: picows

Functions
---------

.. autofunction:: ws_connect
.. autofunction:: ws_create_server

Classes
-------

.. autoclass:: WSError
    :members:

.. autoclass:: WSFrame
    :members:

    .. py:attribute:: msg_type
        :type: WSMsgType

        Message type

    .. py:attribute:: fin
        :type: bool

        Indicates whether this is the last frame of the message.
        Websocket messages MAY consist of multiple frames.

        Unfragmented message::

            WSFrame(msg_type=WSMsgType.<actual message type>, fin=True)

        Fragmented message::

            WSFrame(msg_type=WSMsgType.<actual message type>, fin=False)
            WSFrame(msg_type=WSMsgType.CONTINUATION, fin=False)
            ...
            # the last frame of the message
            WSFrame(msg_type=WSMsgType.CONTINUATION, fin=True)

    .. py:attribute:: rsv1
        :type: bool

        Indicates whether rsv1 flag is set in the frame.
        Some protocol extensions use this flag to indicated that the frame data
        is compressed.
        For example in `permessage_deflate extension <https://www.rfc-editor.org/rfc/rfc7692#section-7>`_

        .. note::
            Currently, picows forbids any protocol extensions during upgrade phase.
            You may still check that it is set to False to verify behaviour of the
            remote side.

    .. py:attribute:: last_in_buffer
        :type: bool

        Indicates whether this is the last available frame in the receiving buffer.
        The receiving buffer may contain more available data, but not the full frame yet.

    .. py:attribute:: tail_size
        :type: int

        Indicates how many bytes are in the receiving buffer after the current frame.

    .. py:attribute:: payload_ptr
        :type: char*

        **Available only from Cython.**

        Raw pointer to the beginning of the frame payload in the receiving buffer.

    .. py:attribute:: payload_size
        :type: size_t

        **Available only from Cython.**

        Size of the payload.

.. autoclass:: WSUpgradeRequest
    :members:

    .. py:attribute:: method
        :type: bytes

        Request method. b"GET", b"POST", etc

    .. py:attribute:: path
        :type: bytes

        Request path. For example b"/ws"

    .. py:attribute:: version
        :type: bytes

        HTTP version. For example b"HTTP/1.1"

    .. py:attribute:: headers
        :type: CIMultiDict[str, str]

        Request headers. Keys are case insensitive

.. autoclass:: WSListener
    :members:

.. autoclass:: WSTransport
    :members:

    .. py:attribute:: underlying_transport
        :type: asyncio.Transport

        Underlying TCP or SSL transport. Can be used to set buffer limits, check connection state, etc.

        .. note::

            Please don't use it to send data. Use only WSTransport.send_* methods to send frames.

    .. py:method:: send_reuse_external_buffer(WSMsgType msg_type, char* msg_ptr, size_t msg_size, bint fin=True, bint rsv1=False)

        **Available only from Cython.**

        Send a frame over websocket with a message as its payload.
        This function does not copy message to prepare websocket frames.
        It reuses message's memory and append websocket header at the front.

        .. attention::

            Message's buffer should have at least 10 bytes in front of the message pointer available for writing.

        :param msg_type: Message type
        :param msg_ptr: Pointer to a message payload
        :param msg_size: Size of the message payload
        :param fin: fin bit in websocket frame.
            Indicate that the frame is the last one in the message.
        :param rsv1: first reserved bit in websocket frame.
            Some protocol extensions use it to indicate that payload
            is compressed.

Enums
-----

.. autoenum:: WSMsgType
.. autoenum:: WSCloseCode
.. autoenum:: WSAutoPingStrategy
