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

The goal of this example is to demonstrate the tradeoffs found in the mobile + cloud architecture, where one has a choice between doing work locally (on the mobile device) versus remotely (in the cloud) with different performance considerations in each case.
This is a very rough initial attempt and still needs significant work.


- com.loopj.android.http.AsyncHttpClient
- PrimeCheckerRemoteTask
