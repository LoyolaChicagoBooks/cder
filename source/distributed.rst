Offloading Resource-Intensive Computation to the Cloud
======================================================

Learning objectives
-------------------

- Show how CPU-intensive computation can be off-loaded from a mobile app to the cloud, by comparison to a mobile device, an unlimited resource for computation and storage.
- synchronous local foreground tasks in Android (running directly in an event listener)
- asynchronous local background tasks in Android using AsyncTask
- asynchronous access of remote resources (usually RESTful web services) in Android using AsyncHttpClient

Introduction
------------

The goal of this example is to demonstrate the tradeoffs found in the
mobile + cloud architecture, where one has a choice between doing work
locally (on the mobile device) versus remotely (in the cloud) with
different performance considerations in each case.

The RESTful prime checker web service
-------------------------------------

We can implement a prime checker web service using the same code as in
the previous section. We can expose this functionality as a RESTful
service at a URL like this one::

    http://host[:port]/number

for example::

    http://laufer-primechecker.herokuapp.com/1000003 

Implementation
--------------

Our app can now invoke this service via the HTTP protocol instead of
performing the check locally. We use
``com.loopj.android.http.AsyncHttpClient`` to submit the remote
request asynchronously: Instead of waiting for completion, we can
perform other work---such as responding to user input---while the
request is in progress and receive notifications for events in the
request lifecycle, such as completion. 

.. literalinclude:: ../examples/primenumbers-android-java/PrimeNumbers/src/main/java/edu/luc/etl/cs313/android/primechecker/android/PrimeCheckerRemoteTask.java
   :start-after: begin-method-remoteStart
   :end-before: end-method-remoteStart
   :language: java 
   :linenos:

Cancelation of pending requests is supported as well.

.. literalinclude:: ../examples/primenumbers-android-java/PrimeNumbers/src/main/java/edu/luc/etl/cs313/android/primechecker/android/PrimeCheckerRemoteTask.java
   :start-after: begin-method-remoteCancel
   :end-before: end-method-remoteCancel
   :language: java 
   :linenos:

Fortunately, because ``AsyncHttpClient`` is already asynchronous,
there is no need to use ``AsyncTask`` anymore. This is how we invoke
the remote task:

.. literalinclude:: ../examples/primenumbers-android-java/PrimeNumbers/src/main/java/edu/luc/etl/cs313/android/primechecker/android/PrimeCheckerAdapter.java
   :start-after: begin-fragment-executeRemote
   :end-before: end-fragment-executeRemote
   :language: java 
   :linenos:
 
The tradeoff
------------

The most widely used performance metrics in distributed computing are

- throughput: average number of requests handled per time unit
- latency: average time to handle a request
- network latency: the portion of latency attributable to the network
- jitter: variance of latency

Assuming a reasonably server performance and straightforward
improvements such as memoization and caching, latency should not be
more than a second or two. Most of this observed latency comes from
the network and does not depend on the number we are checking for
primeness.

If network latency is :math:`l`, the mobile device takes time
:math:`t(n)` to check the number :math:`n`, and the server takes time
:math:`t(n) / k`, then we should use remoting if

.. math::

   t(n) > l + t(n) / k

or

.. math::

   t(n) (1 - 1/k) > l

Assuming k is large, we can ignore it, so we should use remoting if
local processing time exceeds network latency.

Other considerations that might influence the decision to remote or not:

- security
- reliability
- connectedness
