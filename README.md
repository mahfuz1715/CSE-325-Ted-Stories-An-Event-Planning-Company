# Ted's Stories — Event Planning Simulation

A C-based Operating Systems project that simulates an event planning company where multiple planners handle concurrent events while sharing limited resources.

## Project Overview

**Ted's Stories** models an event planning environment with:

- 20 event requests
- 5 concurrent planner processes
- 3 proposal locations
- 3 decoration themes
- 3 AV equipment setups
- Spontaneous idea-sharing between planners

The project focuses on practical implementation of **process management, inter-process communication, synchronization, and resource allocation** using Linux System V mechanisms.

## Key Concepts Implemented

### Process Management
- `fork()` is used to create 5 independent planner processes.
- Each planner continuously waits for and handles event assignments.
- `kill()` is used to terminate planner processes after the simulation.

### Shared Memory
System V shared memory is used to maintain common data between planner processes, including:

- Activity log buffer
- Resource usage status
- Idea-sharing state
- Idea counter

### Semaphores
Five semaphores coordinate access to planners and shared resources:

| Semaphore | Purpose |
|---|---|
| 0 | Planner availability |
| 1 | Proposal locations |
| 2 | Decoration themes |
| 3 | AV equipment |
| 4 | Idea-sharing mutex |

Semaphores ensure that limited resources are accessed safely and help prevent race conditions and deadlock situations.

### Resource Allocation

For each event, a planner:

1. Acquires an available planner slot.
2. Reserves a proposal location.
3. Selects a decoration theme.
4. Reserves AV equipment.
5. Simulates event-planning work.
6. Releases the resources.
7. Completes the event.

Resources are released after use, allowing other planner processes to access them.

### Spontaneous Idea Sharing

Planners have a 50% chance of initiating a spontaneous idea-sharing session. A binary semaphore protects this critical section so that idea sharing does not occur concurrently.

### Activity Logging

The simulation produces real-time console logs and stores activities in shared memory. At the end of the simulation, the accumulated log is written to:

```text
event_log.txt
```

## Technologies Used

- **C**
- Linux / POSIX environment
- System V Shared Memory
- System V Semaphores
- Process Management with `fork()` and `kill()`
- Standard UNIX system calls

## How to Run

### Requirements

- Linux environment or Linux-based virtual machine
- GCC compiler

### Compile

```bash
gcc project.c -o project
```

### Run

```bash
./project
```

The program simulates 20 event requests, coordinates multiple planner processes, manages shared resources, and generates an activity log.

## Expected Output

The terminal displays events such as:

```text
Event Planner 1 is ready for new assignments!
Planner 1 is assigned to a new event!
Planner 1 reserves proposal spot: Empire State Building
Planner 1 chooses decoration theme: Vintage Romance
Planner 1 reserves AV equipment: Projector & Screen
Planner 1 is working on the event...
Planner 1 has completed the event!
```

The exact order varies because the simulation uses multiple processes and randomized delays.

## Project Outcome

The project demonstrates how an operating system can coordinate multiple concurrent processes competing for limited shared resources. It applies **process creation, inter-process communication, semaphore-based synchronization, resource allocation, critical-section protection, logging, and cleanup** in a single practical simulation.

## Repository Contents

```text
.
├── project.c
├── README.md
└── CSE325_Final_Project_Report.pdf
```

## Academic Information

**Course:** CSE325 — Operating Systems  
**Project:** Ted's Stories — An Event Planning Company  
**Institution:** East West University  
**Department:** Computer Science and Engineering
