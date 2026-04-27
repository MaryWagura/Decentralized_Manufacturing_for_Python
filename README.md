# Decentralized Manufacturing System

A multi-agent simulation system built with **AgentPy** that implements a decentralized manufacturing network using the **Contract Net Protocol**. This project demonstrates how independent machine agents and order agents can communicate, negotiate, and autonomously reach agreements without centralized control.

## Overview

This repository contains a progressive implementation of a distributed manufacturing environment where:
- **OrderAgents** submit job requests for specific manufacturing operations
- **MachineAgents** compete by bidding on available work based on their capabilities
- A negotiation mechanism distributes jobs to the most efficient machines

The implementation is broken down into 5 iterative phases, building complexity and functionality with each step.

---

## Project Phases

### Phase 1: Defining the Agents
**Objective**: Create the foundational agent classes

- **MachineAgent**: Represents manufacturing equipment with a specific capability (e.g., "Drilling")
  - Has a unique ID and a single capability
  - Maintains an inbox for incoming messages
  - Prints startup confirmation with registered capability

- **OrderAgent**: Represents manufacturing orders that require specific operations
  - Has a unique ID and a required operation
  - Maintains an inbox for incoming messages
  - Prints startup confirmation with required operation

**Key Concept**: Both agents are initialized with constructor-like `setup()` methods using AgentPy's framework.

---

### Phase 2: Building the Model Environment
**Objective**: Create a centralized orchestration environment

- **ManufacturingModel**: Central environment that houses all agents
  - Uses `ap.AgentList()` to spawn and manage multiple agents
  - Implements **Service Discovery**: Manually provides OrderAgents with a list of available MachineAgents
  - Eliminates the need for complex discovery mechanisms (unlike JADE's Directory Facilitator)

**Key Advantage**: AgentPy's Model-based approach is simpler and more intuitive than traditional agent frameworks—all agents can easily access each other through the central Model.

---

### Phase 3: Enabling Agent Communication
**Objective**: Implement time-stepped behavior and state management

- **MachineAgent Updates**:
  - Implements `step()` method called every simulation tick
  - Listens for work passively

- **OrderAgent Updates**:
  - Introduces **state management** with `self.state` tracking
  - `'seeking_machine'` state: Broadcasts initial request
  - `'waiting_for_bids'` state: Waits for responses
  - Changes state after first action to prevent repeated broadcasts

- **ManufacturingModel Updates**:
  - Adds `step()` method that calls `step()` on all agent lists
  - Simulation runs for a configurable number of time steps

**Key Concept**: State machines control agent behavior without rigid one-shot behaviors; agents intelligently decide what to do each tick based on their current state.

---

### Phase 4: Agent Communication (Simulated Messaging)
**Objective**: Implement peer-to-peer message passing

- **Message Format**: Python dictionaries with FIPA-ACL-like structure:
  ```python
  {
    "performative": "CFP",        # Message type (intent)
    "sender": agent_reference,    # Reply-to address
    "content": job_description    # Message payload
  }
  ```

- **MachineAgent Updates**:
  - Reads inbox messages in `step()`
  - Checks message performative (type)
  - Removes messages after processing to prevent double-processing

- **OrderAgent Updates**:
  - Sends CFP (Call for Proposals) messages to all capable machines
  - Broadcasts to `self.target_machines` by appending to their inboxes

- **Model Setup**: Now spawns 2 machines to demonstrate broadcasting capability

**Key Concept**: Messages are passed by appending dictionaries to agent inboxes—simple but effective.

---

### Phase 5: Contract Net Protocol Challenge
**Objective**: Implement full contract negotiation with competitive bidding

This is the complete multi-step auction protocol:

#### Step 1: CFP Broadcasting
- OrderAgent broadcasts a "Call for Proposals" with the required operation (e.g., "Drilling")

#### Step 2: Competitive Bidding
- MachineAgents receive the CFP
- Check if their capability matches the required operation
- If they can do the job: Generate a random processing time (10-50 units) and send a PROPOSE message back to the OrderAgent
- If they can't: Ignore the request (capability mismatch)

#### Step 3: Bid Evaluation & Award
- OrderAgent collects all PROPOSE messages
- Finds the proposal with the **lowest processing time**
- Sends ACCEPT_PROPOSAL to the winning bidder
- Sends REJECT_PROPOSAL to all others

#### Step 4: Notification
- MachineAgents receive their results:
  - **Winners**: Acknowledge job acceptance and begin manufacturing
  - **Losers**: Return to waiting state

#### Final Model Configuration:
- **2 Drilling machines** (competing agents)
- **1 Milling machine** (tests capability filtering)
- **1 OrderAgent** requesting a Drilling operation
- **5-step simulation** to allow complete negotiation cycle

---

## Key Features

✅ **Decentralized Decision Making**: No central controller; agents autonomously negotiate  
✅ **Contract Net Protocol**: Industry-standard negotiation mechanism  
✅ **Capability Matching**: Machines only bid on jobs they can perform  
✅ **State-Based Behavior**: Agents use state machines instead of rigid behaviors  
✅ **Dynamic Bidding**: Competitive process to find the optimal contractor  
✅ **Simple Messaging**: Dictionary-based inbox system (FIPA-ACL inspired)  
✅ **Scalable Architecture**: Easy to extend with more agents or operation types

---

## How It Works

### Execution Flow

```
Initialization
    ↓
[Model Setup: Create agents and configure service discovery]
    ↓
Simulation Tick (Step 1-5)
    ├─ OrderAgent.step()
    │  └─ If seeking_machine: Broadcast CFP to all machines
    │     └─ Change state to waiting_for_bids
    │
    ├─ MachineAgent.step() (exec for each machine)
    │  ├─ If received CFP with matching capability → Send PROPOSE
    │  ├─ If received ACCEPT_PROPOSAL → Print winner message
    │  └─ If received REJECT_PROPOSAL → Print loser message
    │
    └─ If proposals received and waiting_for_bids
       └─ Find best bid → Send ACCEPT/REJECT to all bidders
```

### Communication Sequence

```
OrderAgent                                    MachineAgent (Drilling #1)    MachineAgent (Drilling #2)
   |                                                 |                               |
   |-- CFP (Drilling) -------------------->        |                               |
   |                                                 | (Check capability: Match!)     |
   |                                                 |-- PROPOSE (Time: 42) -------->|
   |                                                 |                          (Check: Match!)
   |<--- PROPOSE (Time: 42) ----                   |     |-- PROPOSE (Time: 28) -->|
   |                                                 |     |                          |
   |-- ACCEPT_PROPOSAL (to #2) ----------->        |     |                          |
   |-- REJECT_PROPOSAL (to #1) ---------->        |     |-- Award Winner |
   |                                                 |     (Continue waiting)
```

---

## Technical Stack

- **Language**: Python 3.x
- **Framework**: [AgentPy](https://agentpy.readthedocs.io/) - A lightweight agent-based modeling framework
- **Paradigm**: Multi-agent systems (MAS)
- **Protocol**: Contract Net Protocol (FIPA standard)

---

## Installation & Usage

### Requirements
```bash
pip install agentpy
```

### Running the Simulation
```python
python notebook.ipynb
# or run the final execution block in Jupyter/IPython
```

### Expected Output
```
--- Model Setup Complete ---

<<< Order 0 broadcasting CFP for Drilling...

<<< Machine 1 (Drilling) proposing time: 42
<<< Machine 2 (Drilling) proposing time: 28
--- Machine 3 ignoring request (Needs Drilling, but I do Milling)

--- Order 0 evaluating 3 bids ---
<<< Awarding job to Machine 2 (Time: 28)

WINNER! >>> Machine 2 got the job! Starting manufacturing...
LOSER   >>> Machine 1 lost the bid. Back to waiting.

--- Negotiation Complete ---
```

---

## Architecture Insights

### Why AgentPy Over Other Frameworks?

Unlike complex frameworks like JADE (Java Agent DEvelopment Framework), AgentPy provides:
- **Simpler agent discovery**: Centralized Model instead of Directory Facilitators
- **Pythonic API**: Natural integration with Python ecosystem
- **Lightweight**: Minimal overhead while maintaining functionality
- **Export capabilities**: Built-in data collection and analysis

### Messaging System

This implementation uses a simplified but effective messaging system:
- **No complex ACL parsers**: Plain Python dictionaries
- **Direct inbox access**: Agents append directly to targets' inboxes
- **One-tick delivery**: Messages are processed immediately next tick

---

## Future Enhancements

- 🔄 Multiple simultaneous orders competing for machines
- 🔄 Different machine capability combinations and specialization
- 🔄 Dynamic machine availability (maintenance scheduling)
- 🔄 Cost-based bidding in addition to time-based
- 🔄 Machine learning for improved bid estimation
- 🔄 Visualization of negotiation network
- 🔄 Data collection and performance analytics

---

## Project Structure

```
DecentralizedManufacturing/
├── notebook.ipynb          # Main Jupyter notebook with all 5 phases
├── README.md              # This file
├── data/                  # Future: Data collection and results
└── models/                # Future: Trained modules or advanced configurations
```

---

## Author Notes

This project demonstrates the power of agent-based modeling for solving real-world distributed systems problems. The Contract Net Protocol used here is applicable to many domains:
- Supply chain optimization
- Cloud resource allocation
- IoT device coordination
- Autonomous vehicle fleet management
- Distributed manufacturing (like Industry 4.0)

The beauty of this approach is its **scalability and resilience**—the system continues to work even if individual agents fail, and new agents can join without re-programming the entire system.


