Interactive Behaviors and Implicit Concurrency with Internal Timers
===================================================================

Learning objectives
-------------------

**TODO** streamline

- Modeling
  - Distinguishing between view states and (behavioral) model states
- Semantics
  - Internal events from background timers
  - Concurrency issues: single-thread rule of accessing/updating the view in the GUI thread
- Architecture and Design (focus on relevant essentials here)
  - Distinguishing among dumb, reactive, and autonomous model components
  - Implementing state-dependent behavior using the State pattern
  - Command pattern for representing tasks as objects
  - Façade pattern for hiding complexity in the model from the adapter
  - Relevant class-level design principles
    - Dependency Inversion Principle (DIP)
    - Single Responsibility Principle (SRP)
    - Interface Segregation Principle (ISP)
  - Package-level architecture and relevant principles
    - Dependency graph (see also here)
    - Stable Dependencies Principle (SDP)
    - Acyclic Dependencies Principle (ADP)
  - Architectural journey
- Testing
  - Different types of testing
    - Component-level unit testing
    - System testing
    - Instrumentation testing
    - Mock-based testing
    - Testcase Superclass pattern
    - Test coverage



Introduction
------------

asdf

.. figure:: images/AutonomousModelViewAdapter.png
   :alt: Autonomous Model-View-Adapter Architecture
   :scale: 100%

   The Model-View-Adapter (MVA) architecture of the countdown timer 
   Android app. Solid arrows represent (synchronous) method invocation, 
   and dashed arrows represent (asynchronous) events. Here, both the 
   view components and the model's autonomous timer send events to
   the adapter.

asdf

.. figure:: images/CountdownTimerStates.png
   :alt: Countdown Timer FSM

   The finite state machine modeling the dynamic behavior of the countdown
   timer application.

asfd

.. figure:: images/CountdownTimerStoppedZero.png
   :alt: Countdown Timer stopped state with zero time
   :scale: 50%

   The initial stopped state of the countdown timer with zero time left.

.. figure:: images/CountdownTimerStoppedNonzero.png
   :alt: Countdown Timer stopped state with zero time
   :scale: 50%

   The countdown timer in the stopped state after adding some time.

.. figure:: images/CountdownTimerRunning.png
   :alt: Countdown Timer stopped state with zero time
   :scale: 50%

   The countdown timer in the running state.

.. figure:: images/CountdownTimerRinging.png
   :alt: Countdown Timer in the alarm state
   :scale: 50%

   The countdown timer in the alarm ringing state.


explain the two types of timers

- timeout - stopped state 
- recurring - running state

need sequence diagrams explaining both of these


explain possibility of custom app-specific events

explain JavaBeans event source/listener patterns


**TODO** improve view states:
- remove label
- label button: add time, cancel, silent


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
