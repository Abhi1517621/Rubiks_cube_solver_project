# Rubik's Cube Solver: High-Performance Pathfinding

## Project Overview

This project features a high-performance Rubik's Cube solver engineered in C++17. Designed to explore graph theory, state-space search algorithms, and low-level system optimization, the solver effectively navigates a state space of over 43 quintillion possible configurations. Finding the optimal solution path requires managing this massive space without exhausting system memory or CPU time. To achieve this, the architecture decouples the physical representation of the cube from the pathfinding logic, enabling the benchmarking and comparison of multiple data structures and search strategies.

## Technical Stack

* Language: C++17
* Build System: CMake
* Core Algorithms: Breadth-First Search (BFS), Depth-First Search (DFS), Iterative Deepening DFS (IDDFS), Iterative Deepening A* (IDA*)

## Core Architecture and Engineering Decisions

### 1. State Representation and Optimization
Three distinct models represent the cube in memory, progressively optimizing for computational speed:
* 3D Array Model: An intuitive 6x3x3 character matrix.
* 1D Array Model: A flattened 54-character array that improves cache locality and calculates spatial positions using index mapping formulas.
* Bitboard Model: The most highly optimized representation. It compresses the 8 moving pieces of each face into a single 64-bit integer (`uint64_t`). Face rotations are executed using hardware-level circular bitwise shifts (`<<`, `>>`) and masks, reducing a complex 3D rotation to just 3 CPU clock cycles.

### 2. Heuristics and Pattern Databases
To intelligently guide the IDA* solver, an admissible heuristic is utilized to estimate the distance to the solved state.
* Problem Relaxation: The solver precomputes the exact number of moves required to solve just the 8 corner pieces.
* Perfect Hashing: Lehmer codes and base-3 conversions are utilized to assign a unique, collision-free integer index to all 88,179,840 possible corner configurations.
* Memory Compression: Storing this database traditionally would require approximately 88 MB of RAM. Since corner solutions never exceed 11 moves, a custom `NibbleArray` was implemented to pack two 4-bit values into a single byte. This effectively reduces the database memory footprint by 50%.

### 3. Pathfinding Algorithms
The solver implements multiple algorithms to demonstrate the inherent trade-offs between time and space complexity in graph search:
* BFS: Guarantees the shortest path but causes memory exhaustion (O(18^d) space) on deep scrambles due to exponential queue size growth.
* DFS: Highly memory-efficient (O(d) space) but returns non-optimal, excessively long solution sequences.
* IDDFS: Combines the memory efficiency of DFS with the optimality of BFS, but suffers from exponential time complexity due to redundant top-level searches.
* IDA*: The final and most computationally efficient solver. It utilizes the precomputed Pattern Database to aggressively prune search branches that mathematically cannot reach the solution within the current depth threshold, solving highly scrambled cubes in milliseconds.

## Detailed Project Structure

* Model/: Contains the abstract `RubiksCube` interface and concrete models implementing the interface (`3dArray`, `1dArray`, `Bitboard`).
* Pattern_Databases/: Handles the generation, indexing, and serialization of the heuristic lookup tables. Includes the `NibbleArray` implementation for memory compression and `math.h` for mathematical operations.
* Solvers/: Contains the distinct search algorithm implementations, including the optimized `IDAstar_solver.h`.
* main.cpp: The main execution entry point. Handles cube initialization, scrambling, solver instantiation, and result output.
* CMakeLists.txt: Configuration directives for the CMake build system.

## Build and Run Instructions

### Prerequisites
* A C++17 compliant compiler (GCC, Clang, or MSVC)
* CMake (version 3.20 or higher)

### Setup
1. Clone the repository to the local machine.
2. Open `main.cpp` and ensure the `fileName` variable points to a valid local path for the database file generation and loading.

### Compilation (Linux / macOS / Git Bash)
Navigate to the project root directory and execute the following commands in the terminal:

    mkdir build
    cd build
    cmake ..
    make

### Execution
Run the compiled executable:

    ./rubiks_cube_solver
