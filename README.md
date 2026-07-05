# Rubik's Cube Solver: High-Performance Pathfinding

## Project Overview

This project is a high-performance Rubik's Cube solver written in C++17[cite: 1]. I built this solver to deepen my understanding of graph theory, state-space search algorithms, and low-level system optimization. 

A Rubik's Cube has over 43 quintillion possible configurations. Finding the optimal solution path requires navigating this massive state space without exhausting system memory or CPU time. To achieve this, the project decouples the physical representation of the cube from the pathfinding logic, allowing for the comparison of multiple data structures and search strategies[cite: 1].

## Technical Stack

* Language: C++17
* Build System: CMake[cite: 1]
* Core Algorithms: Breadth-First Search (BFS), Depth-First Search (DFS), Iterative Deepening DFS (IDDFS), Iterative Deepening A* (IDA*)[cite: 1]

## Core Architecture and Engineering Decisions

### 1. State Representation and Optimization
I implemented three different models to represent the cube in memory, progressively optimizing for speed[cite: 1]:
* 3D Array Model: An intuitive 6x3x3 character matrix.
* 1D Array Model: A flattened 54-character array that improves cache locality and calculates positions using index mapping[cite: 1].
* Bitboard Model: The most optimized representation. It compresses the 8 moving pieces of each face into a single 64-bit integer (`uint64_t`). Face rotations are executed using hardware-level circular bitwise shifts (`<<`, `>>`) and masks, reducing a complex 3D rotation to just 3 CPU clock cycles.

### 2. Heuristics and Pattern Databases
To guide the IDA* solver, I needed an admissible heuristic to estimate the distance to the solved state.
* Problem Relaxation: The solver precomputes the exact number of moves required to solve just the 8 corner pieces[cite: 1].
* Perfect Hashing: I used Lehmer codes and base-3 conversions to assign a unique, collision-free integer index to all 88 million possible corner configurations[cite: 1].
* Memory Compression: Storing this database normally would require 88 MB of RAM. Since corner solutions never exceed 11 moves, I implemented a custom `NibbleArray` to pack two 4-bit values into a single byte. This effectively cut the database memory footprint in half[cite: 1].

### 3. Pathfinding Algorithms
The solver implements multiple algorithms to demonstrate the trade-offs between time and space complexity[cite: 1]:
* BFS: Guarantees the shortest path but causes memory exhaustion (O(18^d) space) on deep scrambles due to queue size[cite: 1].
* DFS: Highly memory-efficient (O(d) space) but returns non-optimal solutions[cite: 1].
* IDDFS: Combines DFS's memory efficiency with BFS's optimality, but suffers from exponential time complexity[cite: 1].
* IDA*: The final and most efficient solver. It uses the precomputed Pattern Database to aggressively prune search branches that mathematically cannot reach the solution within the current depth threshold, solving highly scrambled cubes in milliseconds[cite: 1].

## Project Structure

* /Model: Contains the abstract `RubiksCube` interface and concrete models (3D Array, 1D Array, Bitboard)[cite: 1].
* /PatternDatabases: Handles the generation, indexing, and serialization of the heuristic lookup tables (CornerDB)[cite: 1].
* /Solver: Contains the distinct search algorithm implementations[cite: 1].
* CMakeLists.txt: Configuration for the build system[cite: 1].

## Build and Run Instructions

### Prerequisites
* A C++17 compliant compiler (GCC, Clang, or MSVC)
* CMake (version 3.20 or higher)[cite: 1]

### Setup
1. Clone the repository to your local machine.
2. Open `main.cpp` and ensure the `fileName` variable points to a valid local path for the database file generation/loading[cite: 1].

### Compilation (Linux / macOS / Git Bash)
Navigate to the project root directory and run the following commands:

```bash
mkdir build
cd build
cmake ..
make
```

### Execution
Run the compiled executable:

```bash
./rubiks_cube_solver
```

## Key Takeaways

Building this project reinforced my ability to analyze time-space trade-offs. Transitioning from a standard array to a bitboard taught me the value of mechanical sympathy and cache-friendly data structures. Furthermore, designing the NibbleArray demonstrated how bit-level memory management can directly solve hardware constraints in algorithm design.
