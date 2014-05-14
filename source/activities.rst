Keeping the User Interface Responsive with Background Activities
================================================================

Learning objectives
-------------------

- synchronous local foreground tasks in Android (running directly in
  an event listener) -> bad unless really short
- asynchronous local background tasks in Android using AsyncTask ->
  good but need to handle
  - progress reporting
  - cancelation

Introduction
------------

asdf

The problem with foreground tasks
---------------------------------

- explain freezing UI, cannot cancel
- explain that even attempted UI *updates* are queued and don't happen
  until foreground task is done
  - this is a consequence of the single-threaded event model
  - why not make event handling asynchronous? huge can of worms, UI is
    shared resource, would have to use mutual exclusion in every event
    handler that accesses GUI; explain a bit more in a sidebar
    (including race conditions etc.)

Background tasks to the rescue
------------------------------

- explain concept
  - async background task
  - progress reporting 
  - cancelation
- describe Android AsyncTask and how it supports the concept
- example: PrimeCheckerTask

