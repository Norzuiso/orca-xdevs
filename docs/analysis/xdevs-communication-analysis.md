# xDEVS Distributed Communication Analysis

## Sources
- [Devs Over MQTT to Enable Distributed Real-Time Simulation](https://ieeexplore.ieee.org/abstract/document/11118639)
- [DEVS/SOA: A Cross-Platform Framework for Net-centric Modeling and Simulation in DEVS Unified Process](https://doi.org/10.1177/0037549709340968)
- [DEVSML 3.0 stack: rapid deployment of DEVS farm in distributed cloud environment using microservices and containers](https://dl.acm.org/doi/10.5555/3108905.3108924)
- [DEVSML Studio: A Framework for Integrating Domain-Specific Languages for Discrete and Continuous Hybrid Systems into DEVS-Based M&S Environment](https://www.sce.carleton.ca/faculty/wainer/papers/SCSC49.pdf)
- [xdevs.java/examples/distributed](https://github.com/iscar-ucm/xdevs.java/tree/main/src/xdevs/core/examples/distributed/gpt)
- [xdevs.java/core/simulation/distributed](https://github.com/iscar-ucm/xdevs.java/blob/main/src/xdevs/core/simulation/distributed)


## 1. Communication mechanism

## 2. Distributed configuration process

## 3. Required network information

## 4. Communication relationships

## 5. Configuration responsibilities

## 6. Limitations and dependencies

## 7. Requirements derived for ORCA-xDEVS

## Raw Notes
---
### Code analysis
---
**Sources analyzed:**
- [xdevs.java/examples/distributed](https://github.com/iscar-ucm/xdevs.java/tree/main/src/xdevs/core/examples/distributed/gpt)
- [xdevs.java/core/simulation/distributed](https://github.com/iscar-ucm/xdevs.java/blob/main/src/xdevs/core/simulation/distributed)

**What information does distributed xDEVS need to locate each simulator and communicate with it?** For each simulation we need to declare the name of the simulator, the path of the class, the host IP, the main port and the auxiliary. Also we need to create an entry for each connection between atomic components, this connection needs to know from what component and port is the output, and what component and port will be the input.

#### **How to run a distributed simulation? What is need it to run it?**
##### ***src/xdevs/core/examples/distributed/gpt/RunDistributedSimulation.java***
- To run a distributed simulation you need to execute: `$ java -cp xdevs.jar xdevs.core.examples.distributed.gpt.RunDistributedSimulation` follow by the file name of the complete XML model as first argument
- The execution command should have two arguments to run a simulator; the second argument will be the name of the simulator that will run. 
	- The simulator is responsible for running a single atomic model.
- If only one argument is passed (the file name of the XML model) that will be the coordinator.
	- The coordinator is responsible for initializing the simulation and running it until the end.
	- The coordinator is responsible for the finalization of the simulation.
- All the simulation is orchestrated by passing messages between the coordinator and the simulators. What does that mean? Does the coordinator send messages to each simulator and the simulators response to the coordinator or the simulators are sending messages between them?
##### **Composition of the XML model file**
The XML model file is an XML file, that represents a coupled model.
- The coupled model has the follow fields:
	- name: Name singed to the model
	- class: lass path/route of the class of the coupled model
	- host: IP where the coupled model is hosted
	- mainPort: Main port of the coupled model
	- auxPort: Auxiliary port of the coupled model, why? 
- The model is composed of atomic components
	- Each atomic component is composed by the follow fields:
		- name: Name of the atomic con
		- class: Class path/route of the class of the atomic model
		- host: IP where the atomic model is hosted
		- mainPort: Main port of the atomic model
		- auxPort: Auxiliary port of the atomic model, why? 
- The model is also composed of connections between the atomic components
	- The connection label has the follow fields:
		- componentFrom: Name of the component where the connection starts
		- portFrom: Name of the output port of the From component 
		- componentTo: Name of the component that will receive the connection
		- portTo: Name of the input port of the To component
#### src/xdevs/core/simulation/distributed/CoordinatorDistributed.java
The coordinator for distributed simulation extends the Coordinator and implements methods to execute the simulation in a distributed way. This means that as the Coordinator, the Coordinator distributed has the same implemented methods but the behavior is focus on the distributed logic. These are the implemented methods:
- void Initialize(): For each component on the coordinator a new DistributedTask with the command **DS_INITIALIZE**.
- void exit(): For each component on the coordinator creates a MessageDistributed with the command **DS_EXIT**, and the string value of the time from the Clock as message, and ping the message to each connected component, after this, the executor calls shutdown()
- double ta(): Declares double var tn as INFINITY .Get tas from executing a DistributedTask with the command **DS_TA** for all components. For each ta from tas list if ta is less than tn and assigns ta value to tn, after do so, the returned value is obtained doing tn-clock.getTime(). The clock is access by hierarchy from AbstractSimulator. The Coordinator class extends from AbstractSimulator. According to DEVS formalism the TA function is the time advance function
- void lambda(): For all components executes a DistributedTask with the command DS_LAMBDA, and then executes the method propagateOutput()
- void propagateOutput(): For all components executes a DistributedTask with the command **DS_PROPAGATE_OUTPUT**.
- void deltfcn(): For all components executes a DistributedTask with the command **DS_DELTFCN**. Then updates the tN var (from AbstractSimulator) to tL (clock.getTime()) + ta()
- void clear():  For all components executes a DistributedTask with the command **DS_CLEAR**.
The distributed coordinator uses the method executeTasksList to run all their functionalities except on the exit method. Also, the use of the ExecutorService is used to run all the functions of the CoordinatorDistributed. 

##### MessageDistributed
The Messaage for distributed simulation is composed by:
- command: A int field that stores the command that will be sent
- message: A string field that stores the content of the message
- valuesPort: A Collection<\?> of the values on the port
##### DistributedTask
The distribute extends from Callable class and implements the call method to execute the task. This class as these fields:
- host: String field, stores the host for the task
- port: Integer field, stores the port for the task
- command: int field, the command is normally uses one of the project constants
- message: String field, stores the message send it on the task
The class DistributedTask also has the method `String call()` this method creates a new MessageDistributed and do a ping to a component using the host and port fields. then it returns the message get it from the ping.
##### PingMessage
The PingMessage class implements a message to be transmited to a remote host. It contains a message and a destination defined with host and port. It also contains a method to send the message and receive the response. The class has the follow fields:
- DEFAULT_DESTINATION_HOST: String constant (static final String), "127.0.0.1" (localhost)
- DEFAULT_DESTINATION_PORT: int constant (static final int), 5000
- message: MessageDistributed field
- destinationHost: String field of the destination host
- destinationPort: int field of the destination port
The class holds the *ping()* is the method to send message and receive the response. The method uses Sockets, to do so it opens an ObjectOutputStream using the field message as a parameter on *writeObject(this.message)*. Also opens an ObjectInputStream to receive the response. After this the function closes the streams and the socket. The response is get it from the the ObjectInputStream and return it after the streams closes.

##### DistributedDaemon.java
Pending, I haven't found the place where is used.
#### Constants from xDEVS
[src/xdevs/core/util/Constants.java](github.com/iscar-ucm/xdevs.java/blob/main/src/xdevs/core/util/Constants.java)
```java
public class Constants {
    // Commmon constants
    public static final double INFINITY = Double.POSITIVE_INFINITY;
    public static final String PHASE_PASSIVE = "passive";
    public static final String PHASE_ACTIVE = "active";
    // Commands for the distributed simulation
    public static final int DS_NONE = 0; // Distrituted simulation: this does not do anything.   
    public static final int DS_INITIALIZE = 2; // Distrituted simulation: it allows to initialize the simulator.
    public static final int DS_TA = 3; // Distrituted simulation: it allows find the advance time.
    public static final int DS_LAMBDA = 4; // Distrituted simulation:: it allows to execute the lambda method.
    public static final int DS_PROPAGATE_OUTPUT = 5; // Distrituted simulation: it allows propagate the outputs for each simulator.    
    public static final int DS_PROPAGATE_OUTPUT_N2N = 6; // Distrituted simulation: it allows propagate the outputs of node to node.    
    public static final int DS_DELTFCN = 7; // Distrituted simulation: it allows to execute the delta methods.    
    public static final int DS_CLEAR = 8; // Distrituted simulation: it allows to refresh ports.    
    public static final int DS_EXIT = 15; // Distrituted simulation: it allows to exit of each simulator.    
}
```