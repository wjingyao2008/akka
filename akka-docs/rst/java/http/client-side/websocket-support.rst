.. _client-side-websocket-support-java:

Client-Side WebSocket Support
=============================

Client side WebSocket support is available through ``Http.singleWebSocketRequest`` ,
``Http.webSocketClientFlow`` and ``Http.webSocketClientLayer``.

A WebSocket consists of two streams of messages, incoming messages (a :class:`Sink`) and outgoing messages
(a :class:`Source`) where either may be signalled first; or even be the only direction in which messages flow
during the lifetime of the connection. Therefore a WebSocket connection is modelled as either something you connect a
``Flow<Message, Message, Mat>`` to or a ``Flow<Message, Message, Mat>`` that you connect a ``Source<Message, Mat>``
and a ``Sink<Message, Mat>`` to.

A WebSocket request starts with a regular HTTP request which contains an ``Upgrade`` header (and possibly
other regular HTTP request properties), so in addition to the flow of messages there also is an initial response
from the server, this is modelled with :class:`WebSocketUpgradeResponse`.

The methods of the WebSocket client API handle the upgrade to WebSocket on connection success and materializes
the connected WebSocket stream. If the connection fails, for example with a ``404 NotFound`` error, this regular
HTTP result can be found in ``WebSocketUpgradeResponse.response``


Message
-------
Messages sent and received over a WebSocket can be either :class:`TextMessage` s or :class:`BinaryMessage` s and each
of those can be either strict (all data in one chunk) or streaming. In typical applications messages will be strict as
WebSockets are usually deployed to communicate using small messages not stream data, the protocol does however
allow this (by not marking the first fragment as final, as described in `rfc 6455 section 5.2`__).

__ https://tools.ietf.org/html/rfc6455#section-5.2

The strict text is available from ``TextMessage.getStrictText`` and strict binary data from
``BinaryMessage.getStrictData``.

For streaming messages ``BinaryMessage.getStreamedData`` and ``TextMessage.getStreamedText`` is used to access the data.
In these cases the data is provided as a ``Source<ByteString, NotUsed>`` for binary and ``Source<String, NotUsed>``
for text messages.


singleWebSocketRequest
----------------------
``singleWebSocketRequest`` takes a :class:`WebSocketRequest` and a flow it will connect to the source and
sink of the WebSocket connection. It will trigger the request right away and returns a tuple containing a
``CompletionStage<WebSocketUpgradeResponse>`` and the materialized value from the flow passed to the method.

The future will succeed when the WebSocket connection has been established or the server returned a regular
HTTP response, or fail if the connection fails with an exception.

Simple example sending a message and printing any incoming message:

.. includecode:: ../../code/docs/http/javadsl/WebSocketClientExampleTest.java
   :include: single-WebSocket-request

The websocket request may also include additional headers, like in this example, HTTP Basic Auth:

.. includecode:: ../../code/docs/http/javadsl/WebSocketClientExampleTest.java
   :include: authorized-single-WebSocket-request


webSocketClientFlow
-------------------
``webSocketClientFlow`` takes a request, and returns a ``Flow<Message, Message, CompletionStage<WebSocketUpgradeResponse>>``.

The future that is materialized from the flow will succeed when the WebSocket connection has been established or
the server returned a regular HTTP response, or fail if the connection fails with an exception.

.. note::
   The :class:`Flow` that is returned by this method can only be materialized once. For each request a new
   flow must be acquired by calling the method again.

Simple example sending a message and printing any incoming message:


.. includecode:: ../../code/docs/http/javadsl/WebSocketClientExampleTest.java
   :include: WebSocket-client-flow


webSocketClientLayer
--------------------
Just like the :ref:`http-client-layer-java` for regular HTTP requests, the WebSocket layer can be used fully detached from the
underlying TCP interface. The same scenarios as described for regular HTTP requests apply here.

The returned layer forms a ``BidiFlow<Message, SslTlsOutbound, SslTlsInbound, Message, CompletionStage<WebSocketUpgradeResponse>>``.


