=============
TCP Transport
=============

The tcp transport is an implementation of Salt's channels using raw tcp sockets.
Since this isn't using a pre-defined messaging library we will describe the wire
protocol, message semantics, etc. in this document.

The tcp transport is enabled by changing the :conf_minion:`transport` setting
to ``tcp`` on each Salt minion and Salt master.

.. code-block:: yaml

   transport: tcp

Wire Protocol
=============
This implementation over TCP focuses on flexibility over absolute efficiency.
This means we are okay to spend a couple of bytes of wire space for flexibility
in the future. That being said, the wire framing is quite efficient and looks
like:

.. code-block:: text

    msgpack({'head': SOMEHEADER, 'body': SOMEBODY})

Since msgpack is an iterably parsed serialization, we can simply write the serialized
payload to the wire. Within that payload we have two items "head" and "body".
Head contains header information (such as "message id"). The Body contains the
actual message that we are sending. With this flexible wire protocol we can
implement any message semantics that we'd like-- including multiplexed message
passing on a single socket.


Crypto
======
The current implementation uses the same crypto as the ``zeromq`` transport.


Pub Channel
===========
For the pub channel we send messages without "message ids" which the remote end
interprets as a one-way send.

.. note::

    As of today we send all publishes to all minions and rely on minion-side filtering.


Req Channel
===========
For the req channel we send messages with a "message id". This "message id" allows
us to multiplex messages across the socket.
