Keeping the User Interface Responsive with Background Activities
================================================================

**TODO** refactor back to doInBackground calling isPrime and invoke isPrime in foreground

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

The running example for this and the next section is a prime number
checking app. 

.. figure:: images/PrimecheckerAndroid.png
   :alt: Prime checker Android app
   :scale: 50%

   asdf adf

We can check whether a number is prime using this brute-force
algorithm. While not a good prime checker implementation, the fact
that it hogs the processor makes it an effective running example for
discussing whether to move processor-bound activities to the
background (or remote servers).  **TODO** footnote with forward
reference to future work

.. literalinclude:: ../examples/primenumbers-android-java/PrimeNumbers/src/main/java/edu/luc/etl/cs313/android/primechecker/android/PrimeCheckerTask.java
   :start-after: begin-method-doInBackground
   :end-before: end-method-doInBackground 
   :language: java 
   :linenos:

The auxiliary methods ``isCancelled`` and ``publishProgress`` are
for checking whether the user has tried to cancel the ongoing check
and updating the progress bar, respectively. They are connected to the
``AsyncTask`` lifecycle methods ``onCancelled`` and
``onProgressUpdate``, respectively. These and several other
lifecycle methods are implemented here:

.. literalinclude:: ../examples/primenumbers-android-java/PrimeNumbers/src/main/java/edu/luc/etl/cs313/android/primechecker/android/PrimeCheckerTask.java
   :start-after: begin-methods-asyncTask
   :end-before: end-methods-asyncTask
   :language: java 
   :linenos:

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

**TODO** consider Q&A-style sidebar on single-threaded GUI model (with brief intro to race conditions, explain how component repaint events are queued, etc.)

We can now run the prime number checker from within our event listener:

.. literalinclude:: ../examples/primenumbers-android-java/PrimeNumbers/src/main/java/edu/luc/etl/cs313/android/primechecker/android/PrimeCheckerAdapter.java
   :start-after: begin-fragment-executeForeground
   :end-before: end-fragment-executeForeground
   :language: java 
   :linenos:

The methods ``onPreExecute`` and ``onPostExecute`` are for
resetting the user interface and displaying the result. Note that we
are running the method ``doInBackground`` directly in the
foreground---despite its name---because we are simply invoking it in
the current thread of execution (the main GUI thread).

If we do this, however, the user interface freezes while the prime
number check is going on, so it does not respond to pressing the
cancel button. There is no progress reporting either: The progress bar
jumps from zero to 100 when the check finishes.

Background tasks to the rescue
------------------------------

- explain concept, want these:
  - async background task with interface
  - progress reporting 
  - cancelation
- describe Android AsyncTask and how it supports the concept
- example: PrimeCheckerTask

.. code-block:: xml
   :linenos:

   public abstract class AsyncTask<Params, Progress, Result> {
       
   }


The three types used by an asynchronous task are the following:

- Params, the type of the parameters sent to the task upon execution.
- Progress, the type of the progress units published during the background computation.
- Result, the type of the result of the background computation.

Not all types are always used by an asynchronous task. To mark a type
as unused, simply use the type Void.


When an asynchronous task is executed, the task goes through 4 steps:

- onPreExecute(), invoked on the UI thread before the task is executed. This step is normally used to setup the task, for instance by showing a progress bar in the user interface.
- doInBackground(Params...), invoked on the background thread immediately after onPreExecute() finishes executing. This step is used to perform background computation that can take a long time. The parameters of the asynchronous task are passed to this step. The result of the computation must be returned by this step and will be passed back to the last step. This step can also use publishProgress(Progress...) to publish one or more units of progress. These values are published on the UI thread, in the onProgressUpdate(Progress...) step.
- onProgressUpdate(Progress...), invoked on the UI thread after a call to publishProgress(Progress...). The timing of the execution is undefined. This method is used to display any form of progress in the user interface while the background computation is still executing. For instance, it can be used to animate a progress bar or show logs in a text field.
- onPostExecute(Result), invoked on the UI thread after the background computation finishes. The result of the background computation is passed to this step as a parameter.




The solution is to run ``doInBackground`` really in the background:

.. literalinclude:: ../examples/primenumbers-android-java/PrimeNumbers/src/main/java/edu/luc/etl/cs313/android/primechecker/android/PrimeCheckerAdapter.java
   :start-after: begin-fragment-executeBackground
   :end-before: end-fragment-executeBackground
   :language: java 
   :linenos:


**TODO** end of section summaries, Q&A/FAQ, questions to ponder/exercises
