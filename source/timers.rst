Interactive Behaviors and Implicit Concurrency with Internal Timers
===================================================================

Learning objectives
-------------------

* Internal timers as event sources (CS1/CS2: C, IOOD: A)
* Timeout events (CS1/CS2: C, IOOD: A)
* Recurring timers (CS1/CS2: C, IOOD: A)
* Modeling more complex application behavior with UML State Machine diagrams (IOOD: A)
* Structuring interactive applications (IOOD: A)
   * Model-View-Adapter (MVA) architecture
   * Façade pattern
   * Testability (CS2: C, IOOD: A)

Introduction
------------

asdf

.. figure:: images/AutonomousModelViewAdapter.png
   :alt: Autonomous Model-View-Adapter Architecture
   :scale: 100%

   The Model-View-Adapter (MVA) architecture of the countdown timer 
   Android app. Solid arrows represent (synchronous) method invocation, 
   and dashed arrows represent (asynchronous) events. Here, the model's
   autonomous timer behavior sends events to the adapter.

asdf

.. figure:: images/CountdownTimerStates.png
   :alt: Countdown Timer FSM

   The finite state machine of the countdown timer model.

asfd

**TODO** code examples

- listener interfaces
- adapter
  - receive view events
  - receive model events
- model
  - state pattern
  - implementation of timeouts
  - wiring pieces together
- testing?
