Basic Event-Based User Interaction
==================================

In this section, we'll study a simple interactive behavior and how
this maps to an implementation based on the Android mobile application
development framework. Our running example will be a bounded click
counter application that can be used to keep track of the capacity of,
say, a movie theater.

The bounded counter abstraction
-------------------------------

A *bounded counter* [REF], the concept underlying this application, is an
integer counter that is guaranteed to stay between a preconfigured
minimum and maximum value. This is called the *data invariant* of the
bounded counter.

.. math::

  min <= counter <= max

We can represent this abstraction as a simple, passive object with,
say, the following interface:

.. literalinclude:: ../examples/clickcounter-android-java/ClickCounter/src/main/java/edu/luc/etl/cs313/misc/boundedcounter/cli/BoundedCounter.java
   :start-after: begin-interface-BoundedCounter
   :end-before: end-interface-BoundedCounter
   :language: java 
   :linenos:

In following a test-driven mindset [REF], we would test implementations of
this interface using methods such as this one, which ensures that
incrementing the counter works properly:

.. literalinclude:: ../examples/clickcounter-android-java/ClickCounter/src/main/java/edu/luc/etl/cs313/misc/boundedcounter/model/AbstractCounterTest.java
   :start-after: begin-method-testIncrement
   :end-before: end-method-testIncrement
   :language: java 
   :linenos:


In the remainder of this section, we'll put this abstraction to good
use by building an interactive application on top of it.

The interactive behavior of a click counter device
--------------------------------------------------

Next, let's imagine a device based on top of this bounded counter
concept. For example, a bouncer positioned at the door of a movie
theater to prevent overcrowding, could benefit from such a device with
the following behavior:

- The device is preconfigured to the capacity of the venue.

- The device always displays the current counter value, initially
  zero.

- Whenever a person enters the movie theater, the bouncer presses the
  *increment* button; if there is still capacity, the counter value
  goes up by one.

- Whenever a person leaves the theater, the bouncer presses the
  *decrement* button; the counter value goes down by one (but not
  below zero).

- If the maximum has been reached, the *increment* button either
  becomes unavailable (or, as an alternative design choice, attempts
  to press it cause an error). This behavior continues until the
  counter value falls below the maximum again.

- There is a *reset* button for resetting the counter value directly
  to zero.

A simple graphical user interface (GUI) for a click counter
-----------------------------------------------------------

Let's now flesh out the user interface of this click counter
device. In the case of a dedicated hardware device, the interface
could have tactile inputs and visual outputs, along with, say, audio
and haptic outputs.

As a minimum, we would require these interface elements:

- Three buttons, for incrementing and decrementing the counter value
  and for resetting it to zero.

- A numeric display of the current counter value.

Optionally, we would benefit from different types of feedback:

- Beep and/or vibrate when reaching the maximum counter value.

- Show the percentage of capacity as a numeric percentage or color
  thermometer.

Instead of a hardware device, we'll now implement this behavior as a
mobile software app, so let's focus first on the minimum interface
elements. In addition, we'll make the design choice that operations
that would violate the counter's data invariant are disabled.

These decisions lead to the three *view states* shown in the following
images. Note how the reset button is always enabled, while the
decrement and increment buttons are disabled in the minimum and
maximum states, respectively.

.. figure:: images/BoundedCounterViewStateMin.png
   :alt: Bounded Counter Min State
   :scale: 50%

   The initial (minimum) view state of the bounded click counter
   Android [REF] app with the decrement button disabled.

.. figure:: images/BoundedCounterViewStateCounting.png 
   :alt: Bounded Counter Counting State
   :scale: 50%

   The counting view state of the bounded click counter Android app
   with all buttons enabled.

.. figure:: images/BoundedCounterViewStateMax.png 
   :alt: Bounded Counter Max State
   :scale: 50%

   The maximum view state of the bounded click counter Android app
   with the increment button disabled (assuming a maximum value of
   10).

Understanding user interaction as events
----------------------------------------

It was fairly easy to express the familiar bounded counter abstraction
and to envision a possible user interface for putting this abstraction
to practical use. The remaining challenge is to tie the two together
in a meaningful way, such that the interface uses the abstraction to
provide the required behavior. In this section, we'll work on bridging
this gap.

Revisiting the interactive behavior
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As a first step, let's abstract away the concrete aspects of the user
interface: 

- Instead of touch buttons, we'll have *input events*.

- Instead of setting a visual display, we'll *modify a counter value*.

After we take this step, we can use a *finite state machine* [REF] to
model the dynamic behavior we described at the beginning of this
section more formally. Note how the touch buttons correspond to
*events* (triggers of *transitions*, i.e., arrows) with the matching
names.

The behavior starts with the *initial pseudostate* represented by the
black circle. From there, the counter value gets its initial value,
and we start in the minimum state. Assuming that the minimum and
maximum values are at least two apart, we can increment
unconditionally and reach the counting state. As we keep incrementing,
we stay here as long as we are at least two away from the maximum
state. As soon as we are exactly one away from the maximum state, the
next increment takes us to that state, and now we can no longer
increment, just decrement. The system mirrors this behavior in
response to the decrement event. There is a surrounding global state
to support a single reset transition back to the minimum state.

.. figure:: images/BoundedCounterStates.png
   :alt: Bounded Counter FSM

   The finite state machine of the bounded counter model.

**TODO** junction for initial and reset with action counter = min

As you can see, the three model states map directly to the view states
from the previous subsection, and the transitions enabled in each
model state map to the buttons enabled in each view state.



GUI widgets as event sources
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. figure:: images/AndroidStudioGUIEditor.png
   :alt: Android Studio GUI editor
   :scale: 50%

   The visual interface of the application in the Android Studio GUI editor.

.. figure:: images/ComponentTree.png
   :alt: Click Counter View Component Tree
   :scale: 50%

   The view component tree of the bounded click counter Android app.


Event listeners and the Observer pattern
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. figure:: images/AndroidStudioButtonProperties.png
   :alt: Android Studio view component editor
   :scale: 50%

   The increment button in the Android Studio view component editor.

asdf

.. code-block:: xml
   :linenos:
   :emphasize-lines: 5

    <Button
        android:id="@+id/button_increment"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:onClick="onIncrement"
        android:text="@string/label_increment" />

asdf asdf

asdf
 
asdf

.. figure:: images/ModelViewAdapter.png
   :alt: Model-View-Adapter Architecture
   :scale: 100%

   The Model-View-Adapter (MVA) architecture of the bounded click
   counter Android app.


.. literalinclude:: ../examples/clickcounter-android-java/ClickCounter/src/main/java/edu/luc/etl/cs313/android/clickcounter/ClickCounterActivity.java
   :start-after: begin-method-onIncrement
   :end-before: end-method-onIncrement
   :language: java 
   :linenos:

.. literalinclude:: ../examples/clickcounter-android-java/ClickCounter/src/main/java/edu/luc/etl/cs313/android/clickcounter/ClickCounterActivity.java
   :start-after: begin-method-updateView
   :end-before: end-method-updateView
   :language: java 
   :linenos:



Reference: https://www.palantir.com/2009/04/model-view-adapter/

asdf
