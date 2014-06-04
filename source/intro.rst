Introduction
=================

In this chapter, we study parallel and distributed computing topics from a
user-centric software development angle, focusing on the authors' research and
experience in concurrency. While the focus of this chapter is largely within
the device itself, the online source code for our examples goes beyond the on-device
experience by providing versions that connect to RESTful web services
(optionally hosted in the cloud). We've deliberately focused this chapter
around the on-device experience, consistent with  "mobile first" thinking,
which more generally is the way the "internet of things" also works. This
thinking results in proper separation of concerns when it comes to the user
experience, local computation, and remote interactions (mediated using
web services).

It is worth taking a few moments to ponder why mobile platforms are
interesting from the standpoint of parallel and distributed computing, even if
at first glance it is obvious. From an architectural point of view, the
landscape of mobile devices has followed a similar trajectory to that of
traditional multiprocessing systems. The early mobile device offerings,
especially when it came to smartphones, were single core. At the time of
writing, the typical smartphone  or tablet is equipped with at least two cores
but four are becoming common. In this vein, it almost goes without saying that
today's devices are well on their way to being serious parallel systems in
their own right. (In fact, in the embedded space, there has been a corresponding
emergence of parallel boards, similar to the Raspberry Pi.)

The state of parallel computing today largely requires the mastery of two
styles, often appearing in a hybrid form. task parallelism and data
parallelism. The emerging mobile devices are following desktop and server
architecture by supporting both of these. In the case of task parallelism, to
get good performance, especially when it comes to the user experience,
concurrency must be *disciplined*. An additional constaint placed on mobile
devices, compared to parallel computing, is that unbounded concurrency
(threading) makes the device unusable/unresponsive, even to a greater  extent
than on desktops and servers (where there is better I/O performance  in
general). We posit that learning to program concurrency in a resource-constrained
environment (e.g. Android smartphones) can be greatly helpful for
writing better concurrent, parallel, and distributed code in general. More 
importantly, today's students really want to learn emerging platforms, so 
it is a great way to develop new talent in languages/systems that are likely 
to be used in future parallel/distributed programming environments.

In this chapter, we begin by examining with a simple interactive behavior and
explore how to implement this using the Android mobile application development
framework. To the end of making our presentation relevant to problem solvers,
our running example is a bounded click counter application (much more
interactive and exciting than the examples commonly found in concurrency
textbooks, e.g., atomic counters, bounded buffers, dining philosophers) that
can be used to keep track of the capacity of, say, a movie theater.

We then focus on providing a richer experience by introducing timers and
internal events,  which allow for more interesting scenarios to be explored. For
example, a countdown timer can be used for notification of elapsed time, a
concept that has almost uniquely emerged in the mobile space but has
applications in embedded and parallel computing in general, where asynchronous
paradigms have been present for some time, dating to  job scheduling,
especially for longer-running jobs.

We close by exploring longer-running, CPU-bound activities. In mobile, a
crucial design goal is to ensure UI responsiveness and appropriate progress
reporting. We demonstrate strategies for ensuring computation proceeds but can
be interrupted. The methods shown here are generalized to offline code (beyond
the scope of this chapter) for interfacing with cloud-hosted web services.

