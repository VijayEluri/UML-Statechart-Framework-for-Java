**Table of contents:**
* What is it?
* Introduction to statecharts
* Getting started - an example
* References

# What is it?

This framework implements the semantics and syntax of UML statecharts in Java. It gives developers an easy to use API for integration of this diagramm-type into their own code. It is released under the GNU Lesser General Public License.

## What are statecharts?

Statecharts were initially introduced by David Harel in 1987 and are based on a generalisation of the concepts of finite state machines [1]. They are directed graphs and used to describe the behaviour of an object instead of Sequence- or Collaboration-Diagrams which describe the interaction between objects. The Object Management Group added this type of diagram into the UML specification with a slightly modified semantics. The main enhancement of statecharts is the ability to use hierarchy and concurrency in the modelling: If you are new to statecharts you can read a short introduction to the syntax and semantics below.

## Why using it? Comparison to alternatives

The problem with statecharts is that the semantics is quite hard to understand and difficult to implement because several elements are not able to map directly to current programming languages. Therefore the developer is in an awkward position: He can model the object behaviour in his CASE tool and then has the problem that the implementation is hard to realize and error-prone.

One possibility is the usage of the state-pattern, introduced by Erich Gamma et. al [2]. But there are several disadvantages with this approach:

* It just supports finite state machines (FSMs) and no hierarchy or concurrency.
* The elements (like e.g. actions and transitions) are not represented as real objects. Instead they are hidden in the classes implementing a state which makes it hard to understand, change and maintain the software.
* The infrastructure of the FSM and the runtime-configuration cannot be separated. Instead many instances of the FSM must occur in the memory if it should be used more than once at the time.

This project addresses these issues and uses a different approach. Originally the framework was based on my diploma thesis in computer science at the University of Dortmund, Germany, and was created to run statecharts on LEGO Mindstorms [3]. Later it was optimized and is now used in server side 24/7-applications to implement application logic and several protocol layers.

## Features

The framework uses version 1.5 of the UML as a basis for the implementation of the statechart semantics. The list below shows the supported elements and important features:

* Very easy to use API (See documentation below for details)
* Implemented in Java
* Does require only a very small amount of memory
* Complete object-oriented design (all elements are real objects)
* Parallel usage of the statechart infrastructure. Runtime specific data is encapsulated in a small object.
* Support for many elements of statecharts
  * Simple, hierarchical and concurrent states
  * Start and final states
  * History and deep-history pseudostates in hierarchical states
  * Fork- and join pseudostates for concurrent states
  * Segmented transitions using junction points
  * Transitions can cross borders of composite states (implicit entry/exit)
  * Entry, do and exit actions in states
  * Events, guards and actions for transitions
  * Asynchronous event queues for signal events including a thread-pool

If you like this software, have any questions, feature requests or find bugs feel free to write me an email.

# Introduction to statecharts

If you are new to the statecharts it may be hard to understand the syntax and especially the semantics. This short introduction should help you to get into statecharts and learn how to use them. All elements which are supported by this framework are described here. The original text can be found in [3] (german only).

## States and Actions

Statecharts are represented as directed graphs and therefore have vertices and (directed) edges. The vertices are the states an object can reach. Edges are changes of the state, the so-called _transitions_. States are represented through a rectangle with rounded edges and can have a name and actions. Three kind of state-actions are available:

* The _entry-action_ is executed when the state is activated.
* The _do-action_ is executed after finishing the entry-action.
* The _exit-action_ is executed when the state is deactivated.

A special type of state are the _pseudo-states_. They exists only for modelling and are not real states of the object. The statechart **must not** stay or can execute actions in these pseudostates. The main entry activated when the statechart begins the event-handling is the _start-state_ which is a pseudostate and only have outgoing transitions. The _final-state_ instead is a real state and only have incoming transitions.

![g1.png](/images/g1.png)

## Transitions

### Events

Changing a state normally is triggered with events. An event is the result of an arbitrary action from the system or the environment around the system. Events are not defined in a special way. In the UML specification are four event-types described, the most important ones are:

* The _signal-event_ is triggered by the surrounding environment.
* The _time-event_ is triggered by the statechart itself.

Every state can have a completion-transition which is a transition without an event. If the state has no such transition (all transitions are triggered by events) the object stays in this state until an appropriate event ca be handled by an outgoing transition. All events are only handled by the current active state. If the event cannot be handled by the state, the event is discarded. For the end-transition the finishing of the states exit-action is interpreted as the trigger.

### Guards and Actions

A transition can have a guard and an action besides the event. The first is marked as `[guard]` the latter as `/action`. Guards are used to make sure that the transition only can trigger if the evaluation of the guard is true. Therefore it is possible  to create transitions with the same event and different guards. Depending on the guard different actions can be executed or target states reached.

![g2.png](/images/g2.png)

While modelling you have to make sure that the model is deterministic. So it is not allowed to add outgoing transitions to the state which can trigger both on an event.

### Time-triggered transitions

The time event is marked at the transition with the keyword after `(x)`. This defines a special kind of transition, the so-called time-triggered transition. `x` is the time the source state must be active before the transition can trigger (if the optional guard evaluation is true). Counting the time begins when an state is activated.

![g3.png](/images/g3.png)

### Segmented transitions

Another aspect of statecharts are the segmented transitions. They are used if you want to concatenate several transitions e.g. to execute an action with each subtransition. The other usefull commitment of segmented transitions is the modelling of branches. All subtransitions are concatenated with the junction point pseudostate which is displayed as a small black circle in the diagram.

One important property of segmented transitions is that they are always atomic. Therefore the transition triggers either complete or not at all. That means before a statechange occurs the segmented transition must check if there exists a permitted path to the next real state. But there is another noteworthiness to mention: Only the first segment of the transition can contain an event as a trigger. Otherwise all segments can contain guards and actions.

![g4.png](/images/g4.png)

## Hierarchical states

Complex systems often requires abstraction of an actual situation and specify the concrete behaviour in a separate step later on. In statecharts the usage of hierarchical states, also known as or-state, can be used to decompose a complex state into substates which describe the behaviour in detail. Once the object reaches this composite or-state, exactly one substate is activated automatically and only one substate is active at the time (which explains the name or-state). This interlocking implies new particularities which will be described now.

### Start- and final-states

Inside an or-state it is essential to know which substate to activate when the parent state is activated and when the or-state is finished respectivly. This information is modelled with the already known start- and final-states. The existence of these pseudostates is not mandatory in every case. Why will be cleared in the next paragraph.

### Entering and leaving the hierarchical state

Two types can be described how a complex state may be activated:

* An incoming transition ends at the border of the or-state. In this case the existence of the start-state is mandatory.
* An incoming transition ends at a substate, it crosses the border of the or-state. Therefore the start-state is not mandatory because the hierarchical state knows the first active substate by the target of the transition.

Leaving a hierarchical state is analogue:

* The state can be deactivated through an outgoing (end-)transition. If this transition has no trigger the finishing of the composite state is the signal to trigger the end-transition. In such a case a final-state is mandatory.
* The state can be deactivated through an outgoing transition which source is a substate and target is a state outside the or-state. In such a case a final-state is not mandatory.

### State transitions

Transitions within hierarchies brings up two questions we need a semantics for. At first, what happens with outgoing event-triggered transitions from the or-state? The semantics is that the object is in exactly one substate when the or-state is active. So it is nessecary that every substate can handle this event as well. To realize this semantics every event-triggered outgoing transition of the hierarchical-state is inherited by all substates. In other words, every substate handles all outgoing  event-triggered transitions of its parent states which are on the path from the state to the root node of the hierarchy-tree.

This semantics will cause the next question: What happens if a substate "overrides" an outgoing transition with the same event? In this case more than one transition is able to fire. That means a rule is nessecary to prioritize transitions: `t1` is an outgoing transition of the state `s1` and `s1` is a transitively reachable substate of `s2`. `t2` is an outgoing transition of `s2` and handles the same event as `t1`. In this case the inner transition `t1` has a higher priority as `t2`. This rule assures that always the lowest transition in the hierarchy tree triggers.

![g5.png](/images/g5.png)

### History states

Many Systems require to know the last configuration of a complex state when it was deactivated. With this information it is possible to continue in the same configuration when reactivating the complex-state. This information is modelled using history-pseudostates in hierarchical states. History is shown as `H` or `H*` in the diagram and are categorized as follow:

* The shallow history `H` stores the last active substate.
* The deep history `H*` stores all active substates on the path from the node to the leaf in the hierarchy tree.

One difference between classic statecharts by David Harel and UML statecharts is that in latter you must specify a start-transition from the history-state to the state which should be activated if no history is available. This is the case when the or-state has never been active before. If a history is available, the stored substate is activated.

![g6.png](/images/g6.png)

## Concurrent states

The second improvement of statecharts is the possibility to model concurrency. A concurrent state must have at least two parallel active substates. These are the regions of the and-state. In the diagram these regions are separated with a dashed line. Concurrency implies new conceptual consequences which are described now.

### Regions

Regions are described with hierarchical states, therefore the semantics introduced above applies here as well. Basically all regions are active as soon as the and-state is activated. They are some kind of processes which are running inside the concurrent state.

It not possible that several regions are "deactivated" while others aren´t. Incoming events are always handled by all regions which means that perhaps more than one transition are able to fire. Namely up to region count transitions. Like in or-states event triggered outgoing transitions are inherited by all regions (and thereby all substates). The priority rule for fireing transitions is applied here as well.

![g7.png](/images/g7.png)

### Entering and leaving the concurrent state

When entering a concurrent state all regions must be activated and vice versa. As in hierarchical states two cases can be described.

* The incoming transition ends at the and-state. In this case all regions must have a start-state.
* The incoming transition ends at a substate of exatly one region. Then the and-state is activated imlicit at this substate. All other regions are activated at there start-states. Therefore n-1 start-states are nessecary.

Deactivating is analogue:

* If all regions have activated their end-state the completion-transition can trigger. The end-states are synching the regions.
* If a transition of a substate fires and the target state is outside the and-state, the concurrent state is deactivated implicit. All other regions must be deactivated immediately as well. This semantics applies also for inherited event-triggered transition.

![g8.png](/images/g8.png)

### Complex transitions

Two cases of activating a concurrent-state were described above: implicit and explicit. The first all n regions need start-states where latter `n-1` start-states are needed. But what happens when `n-m` regions should be activated (where `1 < m < n`). E.g. this is nessecary if the start-states should describe the default entry behaviour and for some special case the configuration of the and-state should be different when activating.

To model this behaviour complex transitions were introduced. They split the control flow. One incoming transition is split into maximum one transition per region with a target substate in this region. An analogue case can be described for leaving the and-state. A complex transition can have these two semantics:

* Split one transition into at least two transitions. All regions which are not allowed for the incoming transitions are activated at their start-states automatically.
* Synchronising different regions for leaving the and-state when some substates are activated. All regions which are not allowed for the outgoing transitions are deactivated automatically.

The diagram provides two kind of pseudo-states for this behaviour. Both are shown as a small black bar. This bar represents the fork-state if it is a `1:n`-mapping, so one transition is split into n. If it is `n:1`-mapping, that means `n` incoming transitions are synchronised into one. This kind is called the join-state.

![g9.png](/images/g9.png)

# Getting started - an example

This introduction should help you to understand how to use the framework. Mainly the following steps are necessary:

* Model the diagram with your favorite CASE-Tool.
* If needed create a derivation of the Metadata-class to store your runtime-specific data.
* Create your implementation of the used actions, guards and events.
* Code your diagram using the framework classes.
* Dispatch the events on the statechart.

These steps are described in detail in the next sections. As a basis the following statechart is used to create the code for:

![gettingstarted.png](/images/gettingstarted.png)

## Creating your metadata, actions, events and guards

After the diagram is modelled we need to recognize the runtime-specific data we are using. In this example we need to store an integer-value which is decremented every time the state `F` is activated.

All your data is stored in a separate object to make sure the infrastructure of the statechart can be used by more than one thread. This object must be a derivation of the base class Metadata:

```java
public class MyMetadata extends Metadata {
  public int value = 0;
}
```

**And again the warning:** Make sure you never store data which will be modified in any way in your actions and guards because they are accessed concurrently!

### Events

The next step is to create all events used in the statechart. In this example we have two events `anEvent` and `anotherEvent`. Events are represented via the base class Event. Creating your events is simply done by deriving a class. A default implementation is given so you normaly don´t need to implement something.

```java
public class anEvent extends Event {}
public class anotherEvent extends Event {}
```

Every event can store a string as an _id_ and can get with the `toString`-method:

```java
public class anEvent extends Event {
  public anEvent() {
    super("anEvent");
  }
}
```

You may noticed that we did not specify the time event used in the concurrent-state. As you will see later, this is not necessary.

### Actions

Transitions and states will trigger an action by invoking the `execute`-method from the interface. So for each action specified in the statechart we need a class implementing this interface and overwrite the execute-method. In our example we have the actions `SetValue`, `DecrementValue` and `print`. The implementation is pretty simple:

```java
public class SetValue extends Action {
  private int value;

  public SetValue(int value) {
    this.value = value;
  }

  public void execute(Metadata data, Parameter parameter) {
    data.value = value;
  }
}

public class DecrementValue extends Action {
  public void execute(Metadata data, Parameter parameter) {
    data.value--;
  }
}

public class Print extends Action {
  private string value;

  public Print(string value) {
    this.value = value;
  }

  public void execute(Metadata data, Parameter parameter) {
    System.out.println(value);
  }
}
```

When dispatching an event you can use the parameter class (or exactly a derivation of it) to send call-parameter to the action. Another important thing is that an action must never dispatch a synchronous call event on the statechart. This may lead to unspecified behaviour! That means never call the dispatch method from the statechart in the execute method. If you need your action to trigger an event do this using the event queue to dispatch an asynchronous signal-event. In the implementation you will use `statechart.dispatchAsynchron` instead of `statechart.dispatch`.

### Guards

Guards are implemented - I think you guess it - by implementing the interface `Guard`. When using guards you must make sure two things:

* The guard must never change any data in the metadata object.
* The evaluation must always be a boolean expression.

Our only guard in the example is a simple check of the integer value stored in the data object:

```java
public class ValueEquals extends Guard {
  private int value;

  public ValueEquals(int value) {
   this.value = value;
  }

  public boolean check(Metadata data, Parameter parameter) {
    return data.value == value;
  }
}
```

The `else`-guard is just a `meta`-guard, that means it is only shown in the diagram but not represented in the statechart implementation. Every transition with an `else`-guard does not have any kind of guard at all in the implementation.

## Building the statechart

After we have created the specific elements we can start building the statechart itself. The main idea is to start with the statechart object and then create all substates in a top-down way. Every state you create will need to know its parent state in the constructor.

When you allocate the statechart you can specify the number of worker threads for the `event`- and `timeout-event`-queue. The main difference is that the normal event-queue implements a fifo handling and makes sure that only one event per metadata is dispatched with the worker threads at a time while the timeout-event-queue is only internally used for implementing the timeout-event semantics. If a state is activated it will automatically trigger timeout-events for each outgoing transition associated with this kind of event and removes it from the queue when the state is deactivated. The following code represents the states of the diagram:

```java
// Create the statechart and top-level states
Statechart chart = new Statechart();
PseudoState state_start = new PseudoState("start", chart, PseudoState.pseudostate_start);
FinalState state_final = new FinalState("final", chart);
HierarchicalState state_a = new HierarchicalState("a", chart, new SetValue(10), null, null);

// create substates of the or-state a
State state_b = new State("b", state_a);
PseudoState state_j = new PseudoState("j", state_a, PseudoState.pseudostate_junction);
FinalState state_a_final = new FinalState("a_final", state_a);
ConcurrentState state_c = new ConcurrentState("c", state_a);

// region 1 of the and-state
HierarchicalState state_c_r1 = new HierarchicalState("c_r1", state_c);
PseudoState state_c_r1_start = new PseudoState("c_r1_start", state_c_r1, PseudoState.pseudostate_start);
State state_d = new State("d", state_c_r1, new Print("c_r1 active"), null, new Print("c_r1 inactive"));

// region 2 of the and-state
HierarchicalState state_c_r2 = new HierarchicalState("c_r2", state_c, null, null, null);
PseudoState state_c_r2_start = new PseudoState("c_r2_start", state_c_r2, PseudoState.pseudostate_start);
State state_e = new State("e", state_c_r2, new Print("start timeout"), null, null);
State state_f = new State("f", state_c_r2, new DecrementValue(), null, null);
```

If you want you can give every state a name as an identifier. This is strongly recommended because it simplifies the debugging.

After all states are created we must create the transitions. It is important that you create the transitions after the states because the transition-constructor need the state-hierarchy.

```java
// create the transitions
new Transition(state_start, state_b);
new Transition(state_b, state_c);
new Transition(state_c_r1_start, state_d);
new Transition(state_c_r2_start, state_e);
new Transition(state_e, state_f, new TimeoutEvent(1000), new Print("Timeout"));
new Transition(state_f, state_e, new AnotherEvent());
new Transition(state_c, state_j, new AnEvent());
new Transition(state_j, state_b, new ValueEquals(0));
new Transition(state_j, state_a_final);
new Transition(state_a, state_final);
```

That is all you have to do to implement the statechart. The next section will explain how to use it.

## Usage

The created instance of the statechart can now be used for dispatching events. It is possible to use the same statechart instance with more than one metadata object. What we need to do is create a metadata object and then trigger events. You can choose between using the event queue or not. When a new Metadata object is created you must call the start-method before dispatching events.

```java
MyMetadata myData = new MyMetadata();
chart.start(myData);
```

A call of the dispatch- (and the start) method returns only if a state is reached where no outgoing transition can trigger. In our example this is the case with state `E`: You call start and the statechart runs to state `E` for this data. Then a timeout event is created internally and the `start`-method returns. When we now sleep at least 1 second `E` is de- and `F` activated. Then we trigger an event:

```java
Thread.sleep(1500);
chart.dispatch(myData, new AnEvent());
```

This is all we do for dispatching events. If you want to use the event queue just call the asynchronous dispatch and start methods:

```java
chart.startAsynchron(myData);
chart.dispatchAsynchron(myData, new AnEvent());
```

## Conclusion

I hope that this short introduction helps you implementing your own statecharts. If you have further questions, find bugs, have feature requests or just want to leave a comment feel free to write me an email or use the forum and tracker at the project page.

# References

[1] Harel, David: "Statecharts: A Visual Formalism for Complex Systems" http://www.wisdom.weizmann.ac.il/~dharel/SCANNED.PAPERS/Statecharts.pdf. In: Science of Computer Programming 8 (1987), June, Nr. 3, Pages 231-274

[2] Gamma, Erich ; Helm, Richard ; Johnson, Ralph ; Vlissides, John: Design Patterns - Elements of Reusable Object-Oriented Software. Addison-Wesley, 1995.

[3] Mocek, Christian: "Erweiterung des CASE-Werkzeugs DAVE um einen Code-Generator für LEGO Mindstorms" ftp://ls10-ftp.cs.uni-dortmund.de/pub/Diplom-Arbeiten/da_Christian_Mocek.pdf. Diploma-Thesis, 2006, Software-Technology, University of Dortmund, Germany.

