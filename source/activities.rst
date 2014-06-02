Keeping the User Interface Responsive with Background Activities
================================================================

Learning objectives
-------------------

- Designing responsive interactive applications (A)
- Design choices for user interface toolkits such as Android (C)
- The difference between events and activities (C)
- Using asynchronous tasks for long-running activities (A)
- Reporting progress to the user (A)
- Responding to cancelation requests (A)

Introduction
------------

The running example for this section is a simple app for checking
whether a number is prime.

.. figure:: images/PrimecheckerAndroid.png
   :alt: Prime checker Android app
   :scale: 50%

   Screenshot of an Android app for checking prime numbers

The functional requirements for this app are as follows: *The app
allows us to enter a number in a text field. When we press the "check"
button, the app checks whether the number we entered is prime. When we
press the "cancel" button, the ongoing check(s) are discontinued.*

To check whether a number is prime, we can use this brute-force
algorithm. 

.. literalinclude:: ../examples/primenumbers-android-java/PrimeNumbers/src/main/java/edu/luc/etl/cs313/android/primechecker/android/PrimeCheckerTask.java
   :start-after: begin-method-isPrime
   :end-before: end-method-isPrime
   :language: java 
   :linenos:

For now, let's ignore the ``isCancelled`` and ``updateProgress``
methods and agree to discuss their significance later in this section.

While this is not a good prime checker implementation, the app will
allow us to explore and discuss different ways to run one or more such
checks. In particular, the fact that the algorithm hogs the processor
makes it an effective running example for discussing whether to move
processor-bound activities to the background (or remote servers).

.. note:: In this chapter, we focus on tasks that run locally in the
	  foreground or background. In a future submission, we will
	  explore the possibility of offloading compute- and
	  storage-intensive tasks to remote web services.


The problem with foreground tasks
---------------------------------

As a first attempt, we now can run the ``isPrime`` method from within
our event listener in the current thread of execution (the main GUI
thread).

.. literalinclude:: ../examples/primenumbers-android-java/PrimeNumbers/src/main/java/edu/luc/etl/cs313/android/primechecker/android/PrimeCheckerAdapter.java
   :start-after: begin-fragment-executeForeground
   :end-before: end-fragment-executeForeground
   :language: java 
   :linenos:

The methods ``onPreExecute`` and ``onPostExecute`` are for resetting
the user interface and displaying the result.

This approach works for very small numbers. For larger numbers,
however, the user interface freezes noticeably while the prime number
check is going on, so it does not respond to pressing the cancel
button. There is no progress reporting either: The progress bar jumps
from zero to 100 when the check finishes.

The single-threaded user interface model
----------------------------------------

The behavior we are observing is a consequence of Android's design
decision to keep the user interface single-threaded. In this design,
all UI events, including user inputs such as button presses and mouse
moves, outputs such as changes to text fields, progress bar updates,
and other component repaints, and internal timers, are processed
sequentially by a single thread, known in Android as the *main thread*
(or UI thread). We will say *UI thread* for greater clarity.

To process an event completely, the UI thread needs to dispatch the
event to any event listener(s) attached to the event source
component. Accordingly, single-threaded UI designs typically come with
two rules:

#. To ensure responsiveness, code running on the UI thread must never
   block.

#. To ensure thread-safety, only code running on the UI thread is
   allowed to access the UI components.

In interactive applications, running for a long time is almost as bad
as blocking indefinitely on, say, user input. To understand exactly
what is happening, let's focus on the point that events are processed
sequentially in our scenario of entering a number and attempting to
cancel the ongoing check.

- The user enters the number to be checked.
- The user presses the check button.
- To process this event, the UI thread runs the attached listener,
  which checks whether the number is prime. 
- While the UI thread running the listener, all other incoming UI
  events---pressing the cancel button, updating the progress bar,
  changing the background color of the input field, etc.---are
  *queued* sequentially.
- Once the UI thread is done running the listener, it will process the
  remaining events on the queue. At this point, the cancel button has
  no effect anymore, and we will instantly see the progress bar jump
  to 100% and the background color of the input field change according
  to the result of the check.

So why doesn't Android simply handle incoming events concurrently,
say, each in its own thread? The main reason not to do this is that it
greatly complicates the design while at the same time sending us back
to square one in most scenarios: Because the UI components are a
shared resource, to ensure thread safety in the presence of race
conditions to access the UI, we would now have to use mutual exclusion
in every event listener that accesses a UI component. Because that is
what event listener typically do, in practice, mutual exclusion would
amount to bringing back a sequential order. So we would have greatly
complicated the whole model without achieving significantly greater
concurrency in our system.

.. note:: Many other GUI toolkits, such as Java Swing, are also based
	  on a single-threaded design for similar reasons as
	  Android. There are also single-threaded server designs. The
	  software design pattern underlying these designs is known as
	  *Reactor* [REF: Schmidt].


There are two main approaches to keeping the UI from freezing while a
long-running activity is going on. 

Breaking up an activity into small units of work
------------------------------------------------

The first approach is still single-threaded: We break up the
long-running activity into very small units of work to be executed
directly by the event-handling thread. When the current chunk is about
to finish, it schedules the next unit of work for execution on the
event-handling thread. Once the next unit of work runs, it first
checks whether a cancelation request has come in. If so, it simply
will not continue, otherwise it will do its work and then schedule its
successor. This approach allows other events, such as reporting
progress or pressing the cancel button, to get in between two
consecutive units of work and will keep the UI responsive as long as
each unit executes fast enough.

Now, in the same scenario as above---entering a number and attempting
to cancel the ongoing check---the behavior will be much more
responsive:

- The user enters the number to be checked.
- The user presses the check button.
- To process this event, the UI thread runs the attached listener,
  which makes a little bit of progress toward checking whether the
  number is prime and then schedules the next unit of work on the
  event queue.
- Meanwhile, the user has pressed the cancel button, so this event is
  on the event queue *before* the next unit of work toward checking
  the number.
- Once the UI thread is done running the first (very short) unit of
  work, it will run the event listener attached to the cancel button,
  which will prevent further units of work from running.


Asynchronous tasks to the rescue
--------------------------------

The second approach is typically multi-threaded: We represent the
entire activity as a separate asynchronous task. Because this is such
a common scenario, Android provides the abstract class ``AsyncTask``
for this purpose.

.. code-block:: java
   :linenos:

   public abstract class AsyncTask<Params, Progress, Result> {
       protected void onPreExecute() { }
       protected abstract Result doInBackground(Params... params);
       protected void onProgressUpdate(Progress... values) { }
       protected void onPostExecute(Result result) { }
       protected void onCancelled(Result result) { }
       protected final void publishProgress(Progress... values) { ... }
       public final boolean isCancelled() { ... }
       public final AsyncTask<...> executeOnExecutor(Executor exec, Params... params) { ... }
       public final boolean cancel(boolean mayInterruptIfRunning) { ... }
   }

The three generic type parameters are ``Params``, the type of the
arguments of the activity; ``Progress``, they type of the progress
values reported while the activity runs in the background, and
``Result``, the result type of the background activity. Not all three
type parameters have to be used, and we can use the type ``Void`` to
mark a type parameter as unused.

When an asynchronous task is executed, the task goes through the
following lifecycle:

   - ``onPreExecute`` runs on the UI thread and is used to set up
     the task in a thread-safe manner.
   - ``doInBackground(Params...)`` performs the actual task. Within
     this method, we can report progress using
     ``publishProgress(Progress...)`` and check for cancelation using ``isCancelled()``.
   - ``onProgressUpdate(Progress...)`` is scheduled on the UI thread
     whenever the background task reports progress and runs whenever
     the UI thread gets to this event. Typically, we use this method
     to advance the progress bar or display progress to the user in
     some other form.
   - ``onPostExecute(Result)``, receives the result of the background
     task as an argument and runs on the UI thread after the
     background task finishes.

Using ``AsyncTask`` in the prime number checker
-----------------------------------------------

We set up the corresponding asynchronous task with an input of type
``Long``, progress of type ``Integer``, and result of type
``Boolean``.  In addition, the task has access to the progress bar and
input text field in the Android GUI for reporting progress and
results, respectively.

.. literalinclude:: ../examples/primenumbers-android-java/PrimeNumbers/src/main/java/edu/luc/etl/cs313/android/primechecker/android/PrimeCheckerTask.java 
   :start-after: begin-fragment-PrimeCheckerTaskSETUP 
   :end-before: end-fragment-PrimeCheckerTaskSETUP 
   :language: java 
   :linenos:

The centerpiece of our solution is to invoke the ``isPrime`` method
from the main method of the task, ``doInBackground``.  The auxiliary
methods ``isCancelled`` and ``publishProgress`` we saw earlier in the
implementation of ``isPrime`` are for checking for requests to cancel
the current task and updating the progress bar,
respectively. ``doInBackground`` and the other lifecycle methods are
implemented here:

.. literalinclude:: ../examples/primenumbers-android-java/PrimeNumbers/src/main/java/edu/luc/etl/cs313/android/primechecker/android/PrimeCheckerTask.java
   :start-after: begin-methods-asyncTask
   :end-before: end-methods-asyncTask
   :language: java 
   :linenos:

When the user presses the cancel button in the UI, any currently
running tasks are canceled using the control method
``cancel(boolean)``, and subsequent invocations of ``isCancelled``
return false; as a result, the ``isPrime`` method returns on the next
iteration.

How often to check for cancelation attempts is a matter of
experimentation: Typically, it is sufficient to check only every so
many iterations to ensure that the task can make progress on the
actual computation. Note how this design decision is closely related
to the granularity of the units of work in the single-threaded design
discussed above [REF].

What remains is to schedule the resulting ``AsyncTask`` on a suitable
executor object:

.. literalinclude:: ../examples/primenumbers-android-java/PrimeNumbers/src/main/java/edu/luc/etl/cs313/android/primechecker/android/PrimeCheckerAdapter.java
   :start-after: begin-fragment-executeBackground
   :end-before: end-fragment-executeBackground
   :language: java 
   :linenos:

This completes the picture of moving long-running activities out of
the UI thread but in a way that they can still be controlled by the UI
thread.
