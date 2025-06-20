This is a design document for the [Ribbon](https://ribbon-engine.com) realtime software engine, a [Tiny Bow](https://tinybow.org) project.

*   **Compiled Document:** [TODO: Add link to compiled documentation]
*   **Source Format:** The source files are written in [Obsidian](https://obsidian.md) flavored Markdown. For the best browsing experience with working links and formatting, it is recommend to clone the repository and open it as an Obsidian vault.

---
### **Guiding Principles**

The Ribbon project is engineered around two core principles:

*   **Performance:** From cache-friendly data layouts and a multi-threaded scheduler to a unified, AOT-compiled language, every architectural decision is made to maximize raw computational throughput and ensure smooth, high-framerate presentation.
*   **Control:** Ribbon provides developers with absolute control over their work. This is achieved through a deterministic simulation model that guarantees predictable, repeatable results, and an open, extensible ecosystem where every tool can be modified and even shipped to end-users.

### **Overview**

The Ribbon project is an integrated ecosystem comprising a **realtime software engine** and a companion **systems programming language**. It is designed to be a high-performance, deterministic, and extensible foundation for a wide array of applications, from complex data analysis and scientific visualization to deterministic network-based games and interactive media.

#### **Core Architecture: Data-Oriented ECS**

The foundation of Ribbon is a highly data-oriented Entity Component System (ECS) architecture. This design prioritizes cache-friendly memory layouts and efficient data access patterns, allowing systems to process large batches of component data in tight, predictable loops.

To manage scene complexity and relationships, the engine complements the flat data model of the ECS with a separate, globally-accessible graph data structure representing a spatial hierarchy. Entities are assigned a place in this hierarchy via a simple component containing their unique ID. This approach maintains the performance of a pure ECS while providing a natural, intuitive structure for grouping objects and defining the parent-child relationships that the simulation scheduler uses for parallelization.

#### **Simulation Model: Determinism and Predictability**

Ribbon's simulation model is engineered for absolute consistency, a critical requirement for verifiable and network-enabled applications.

Ribbon's simulation model is engineered for absolute consistency, a critical requirement for verifiable and network-enabled applications.

* **Tick-Based and Event-Driven:** Ribbon's simulation architecture is designed for a combination of predictable timing and efficient, dynamic execution. This is achieved by combining a fixed timestep with an event-driven model.
	- **Fixed Timestep Execution**
	    The simulation advances at a hard, fixed tick rate, with a default of 60 ticks per second (T/s). This tick acts as a metronome for the entire system, ensuring that the simulation's state progresses in discrete, predictable intervals. This timing is completely decoupled from variable factors like rendering speed or hardware performance, providing the stable foundation required for deterministic behavior.
	- **Event-Driven Logic**
	    Instead of wastefully updating every system on every tick, the simulation loop is fundamentally an **event processor**. During each discrete interval, the loop processes queues of events that dictate what logic needs to run. This approach is more flexible and performant, as work is only done when it is explicitly required. Systems operate based on two primary categories of events:
	    - **Scheduled Events:** These are events tied directly to the simulation clock. They can be configured to occur at specific ticks or on a recurring basis (e.g., "evaluate this AI behavior tree every 10 ticks"), handling all predictable, time-based logic.
	    - **Reactive Events:** These are generated dynamically by systems as a result of their computations (e.g., a physics system emits an `OnCollision` event, a user input system emits a `ButtonClicked` event). These events are placed into a queue and processed in a deterministic order during the tick, allowing for immediate and reactive behaviors without sacrificing predictability.
	* **State Interpolation for Smooth Presentation**  
	    The tick-based, event-driven model enables a complete decoupling of the core simulation from the presentation layers (i.e., rendering and UI).
		* **Zero-stutter presentation:** This avoids the visual stutter common in engines where rendering is locked to the simulation rate, allowing Ribbon to produce exceptionally smooth visuals. Instead of displaying a new state only when a tick occurs, the rendering and UI layers can update at the maximum available refresh rate by interpolating data between two known simulation states.
		* **Example:** If the simulation produces a world state at Tick A and Tick B, a frame rendered 75% of the way between them can show an object at a position interpolated to be 75% of the way between its position at A and B. This same principle extends to the UI, where expensive operations like recalculating a layout are not performed on every frame. Instead, they are only triggered by events indicating that relevant data has changed, ensuring the entire presentation layer remains responsive and efficient.

*   **Locally-Deterministic Simulation:** A Ribbon simulation is strictly deterministic on a consistent hardware and software architecture. Given an identical initial state and input sequence, every step of the simulation will produce the exact same output. The engine does not enforce strict cross-platform determinism (e.g., guaranteeing identical floating-point results between x86 and ARM architectures). This local determinism is the foundation for two powerful capabilities:
    *   **High-Fidelity Prediction for Distributed Systems:** This model is ideal for authoritative client-server architectures. Clients run the same deterministic simulation locally to instantly **predict** the outcome of their inputs, providing immediate visual feedback. The server runs the definitive "source of truth" simulation. When clients receive state updates from the authoritative server, they smoothly interpolate their predicted state toward the server's state of record. This process corrects for any minor drift that inevitably occurs between different machine architectures, providing a responsive experience while retaining the robustness of a server-authoritative design.
    *   **Reliable Simulation Recording & Playback:** Enables byte-perfect replication of any simulation run when replayed on the same or an architecturally identical machine. This offers a powerful and low-overhead tool for precise bug reproduction, performance analysis, and review, as we can recreate the exact conditions of a simulation with a simple file containing the initial state and inputs.

#### **Concurrency and Data Management**

Ribbon employs a novel spatial partitioning and scheduling model to manage complexity and scale performance across modern multi-core processors.

*   **Spatial Hierarchy:** All entities can exist within a strict parent-child hierarchy, creating a natural structure for grouping and managing related objects and their data.
*   **Spatial Threading:** The simulation is parallelized by partitioning the workload based on **developer-designated top-level items in the spatial hierarchy, which serve as Points of Interest (POIs).** Systems primarily operate on data within a single POI, minimizing data contention and enabling a high degree of parallelism.
*   **Universal Scheduler:** A global scheduler oversees the entire simulation. It manages the execution of systems within their respective spatial threads (POIs) and orchestrates **deterministic message passing** between them. **By design, all inter-POI communication is subject to a one-tick delay and is sorted by a deterministic heuristic. This guarantees a consistent message delivery order and transparently manages the overhead of entities transitioning between large-scale simulation zones.**

### **The Ribbon Ecosystem: Language and Tooling**

A core tenet of the Ribbon project is the deep, synergistic relationship between the engine and its native language.

*   **The Ribbon Language:** The Ribbon Language is a single, unified language that operates in multiple modes: a highly-optimized, ahead-of-time (AOT) compiled mode for the core engine, and a dynamic, just-in-time (JIT) compiled mode for scripting and live tooling. This unified approach eliminates the performance penalties and data marshalling overhead typically found at the boundary between a C++ engine core and a scripting layer like Lua or C#. All code, whether for low-level systems or high-level gameplay scripts, benefits from a robust static analysis and type inference system, providing both safety and ease of use.
*   **A Self-Hosting Engine:** The Ribbon Engine is written entirely in the Ribbon systems language, ensuring maximum performance and a cohesive development experience.
*   **Extensible, Shippable Tools:** All official tools and pipeline software are written in the scriptable dialect of Ribbon, running on the engine itself. This unique architecture means developers can easily customize, extend, or even **ship these tools with their applications**, empowering end-users with sophisticated modification and extensibility capabilities.

### **Licensing and Philosophy**

Ribbon is committed to a transparent and developer-forward open-source model designed to foster unrestricted creativity.

*   **Engine Core License:** The core engine, like the Ribbon programming language, is licensed under the permissive **Apache 2.0 License**. This allows developers to use the engine in commercial, closed-source products, with the only primary obligation being to provide attribution.
*   **Tooling Dedicated to the Public Domain:** To maximize freedom and remove all barriers to adoption and modification, all official tooling and its source code are dedicated to the public domain via **CC0 (Creative Commons Zero)**. This allows developers and modders alike to use, modify, and distribute the tool scripts with their applications or extensions, without any requirement for attribution or inclusion of license files.

