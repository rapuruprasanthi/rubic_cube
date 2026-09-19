RUBIK'S_CUBE_SOLVER
RUBIK'S_CUBE_SOLVER

# Rubik's Cube Solver

A C++ command-line project that models a 3x3 Rubik's Cube and solves it using several search algorithms (DFS, BFS, IDDFS, IDA*). IDA* is accelerated by a **corner pattern database** (a precomputed lookup table of the minimum moves needed to solve the cube's corners).

> Note: this README was drafted from the repository's file list, `main.cpp` and `CMakeLists.txt`. Items marked **(suggested)** are recommendations, not features that exist in the code today.

---

## 1. What is this project?

A Rubik's Cube is scrambled with a sequence of random moves, then a solver searches for a move sequence that returns it to the solved state. The project compares different cube representations and search strategies, and shows how a heuristic (pattern database) makes deep searches practical.

For the full explanation (why it exists, how it works, interview answers), see sections 17-19 at the end.

## 2. Tech Stack

| Area | Technology |
|---|---|
| Language | C++ (C++14 standard) |
| Build system | CMake 3.20+ |
| IDE | JetBrains CLion (`.idea/` folder present) |
| Libraries | C++ standard library only (`<bits/stdc++.h>`) |
| Data storage | Plain-text pattern database file (e.g. `cornerDepth5V1.txt`) |
| Version control | Git / GitHub |

## 3. Functional Requirements

- Represent a 3x3 cube state and apply the 18 standard face moves.
- Print the cube to the console.
- Shuffle the cube with N random moves and return the moves used.
- Solve a scrambled cube using selectable algorithms:
  - **DFS**: depth-first search
  - **BFS**: breadth-first search
  - **IDDFS**: iterative-deepening DFS
  - **IDA\***: iterative-deepening A* guided by the corner pattern database
- Build a corner pattern database with `CornerDBMaker` and load it for IDA*.
- Output the scramble sequence and the solution sequence in move notation.

## 4. Expected Behavior

1. The program creates a solved cube.
2. It scrambles the cube with 13 random moves (the current demo in `main.cpp`).
3. It prints the scrambled cube and the scramble moves.
4. It loads the corner database and runs the IDA* solver.
5. It prints the solved cube and the list of moves that solved it.

Example flow:

```
<scrambled cube>
R U' F2 L ...            <- scramble
<solved cube>
F2 U R' ...              <- solution
```

## 5. Backend / Core Engine

This is a standalone C++ program with no web backend, API or database server. The "backend" is the core engine:

- **Model layer**: cube representations (see structure below).
- **Solver layer**: search algorithms, templated over the cube type and its hash function so any representation can be plugged in.
- **Pattern database layer**: compact storage and lookup of heuristic values.

## 6. Content

- **Cube representations**: 3D array, 1D array and **bitboard** (the most compact and fastest).
- **Search algorithms**: DFS, BFS, IDDFS, IDA*.
- **Heuristic data**: corner pattern database stored with a nibble array (4 bits per entry) and a permutation indexer that maps a corner arrangement to a unique index.

## 7. Bonus (Optional Enhancements) (suggested)

- Add edge pattern databases and combine them with the corner database using `max()` for a stronger heuristic.
- Add a command-line menu to pick the solver and the scramble length.
- Add timing and node-count statistics per algorithm.
- Accept a cube state from user input or a file.
- Add a simple GUI or 3D visualization.

## 8. Pagination

Not applicable. This project has no list or API endpoints. **(suggested)** If you later log many solve runs, output could be paged in the console or stored in a file.

## 9. Code Quality Expectations

- Keep the base `RubiksCube` interface separate from its implementations.
- Use meaningful names and small, single-purpose functions and classes.
- Avoid hard-coded absolute paths. The current `main.cpp` uses a Windows path (`C:\Users\...\cornerDepth5V1.txt`); replace it with a relative path or a command-line argument.
- Replace `#include <bits/stdc++.h>` with specific standard headers for portability (it is GCC-only).
- Avoid `using namespace std;` in headers.
- Add comments for non-obvious parts (bitboard operations, permutation indexing).
- Keep `CMakeLists.txt` tidy (e.g. list sources by directory or use `file(GLOB ...)`).

## 10. Testing (suggested)

There are no tests in the repository today. Recommended:

- **Unit tests** (GoogleTest or Catch2):
  - Each move followed by its inverse returns the cube to its original state.
  - Applying a move four times returns the original state.
  - The three representations stay consistent after the same move sequence.
  - `isSolved()` is true for a fresh cube and false after a move.
- **Solver tests**: scramble by 1-7 moves, then verify each solver returns a sequence that actually solves the cube.
- **Pattern database tests**: nibble array read/write, permutation indexer round trip, database values never exceed the true distance.
- **Performance checks**: compare time and nodes explored across solvers.

## 11. Constraints

- Search space is huge (about 4.3 x 10^19 states), so BFS and DFS only work for shallow scrambles.
- IDA* needs the pattern database file to exist beforehand; it must be generated first with `CornerDBMaker`.
- Database generation takes time and disk space; the depth-5 corner database is a compromise.
- Only the corner heuristic is used, so long scrambles may still be slow.
- The build depends on a GCC-compatible compiler because of `<bits/stdc++.h>`.

## 12. Interview Discussion Points

- Why are there three cube representations, and what are the trade-offs in memory and speed?
- Why is BFS memory-heavy and DFS not guaranteed to find the shortest solution?
- How does IDDFS combine the strengths of BFS and DFS?
- What makes IDA* better, and what is an admissible heuristic? Why is a pattern database admissible?
- Why use a nibble array and a permutation indexer (Lehmer code)?
- Why do templates for the cube and hash type help the solver design?
- How would you add edge databases, and why does `max()` of heuristics stay admissible?
- How would you make the project testable and portable?

## 13. Expected Scope

**In scope:** cube model, move engine, four search solvers, corner pattern database generation and use, console output.

**Out of scope (for now):** GUI, web interface, human-style solving methods, optimal solving for arbitrary scrambles, network features.

## 14. Final Goal

A clean, well-structured and tested C++ solver that can take a scrambled 3x3 cube and reliably return a valid, reasonably short solution, while clearly demonstrating how cube representation, search strategy and heuristics affect performance.

## 15. Project Structure

Current structure (from the repository and `CMakeLists.txt`):

```
Solver_RubiksCube/
├── .idea/                      # CLion project settings
├── Model/
│   ├── RubiksCube.h / .cpp     # Base cube interface / shared logic
│   ├── RubiksCube3dArray.cpp   # 3D array representation
│   ├── RubiksCube1dArray.cpp   # 1D array representation
│   └── RubiksCubeBitboard.cpp  # Bitboard representation
├── Solver/
│   ├── DFSSolver.h
│   ├── BFSSolver.h
│   ├── IDDFSSolver.h
│   └── IDAstarSolver.h
├── PatternDatabases/
│   ├── NibbleArray.h / .cpp
│   ├── PatternDatabase.h / .cpp
│   ├── PermutationIndexer.h
│   ├── CornerPatternDatabase.h / .cpp
│   ├── CornerDBMaker.h / .cpp
│   └── math.h / .cpp
├── main.cpp                    # Demo: shuffle, then solve with IDA*
├── CMakeLists.txt
├── .gitignore
└── README.md
```

Suggested additions:

```
├── Databases/                  # Generated pattern DB files (referenced by main.cpp)
├── tests/                      # Unit tests
└── docs/                       # Notes and diagrams
```

## 16. Build & Run

```bash
mkdir build && cd build
cmake ..
cmake --build .
./rubiks_cube_solver
```

Before running, generate or place the corner database file and update the path in `main.cpp`.


---

# Deep Dive: Explain the Project (Interview Preparation)

> Where I describe internals (bitboard layout, database defaults), I am describing the standard approach for this kind of project. Open your own files and confirm the details before an interview, and answer in your own words.

## 17. Overall Explanation of the Project

### 17.1 What is it?
A program that takes a scrambled 3x3 Rubik's Cube and finds a sequence of moves that solves it. It is not a "human method" solver (layer by layer). It is a **search-based solver**: it treats the cube as a graph problem and searches for a path from the scrambled state to the solved state.

### 17.2 Why is this problem interesting?
- A 3x3 cube has about **43 quintillion (4.3 x 10^19)** reachable states.
- Each state has **18 possible moves** (6 faces x clockwise, counter-clockwise, half-turn).
- Any cube can be solved in at most **20 moves** (known as God's number, in the half-turn metric), but finding a short solution by brute force is very expensive.
- So the project is a compact way to study how **state representation, search strategy and heuristics** decide whether a problem is solvable in practice.

### 17.3 What is it used for?
- **Learning and demonstration** of core AI/algorithm topics: graph search, BFS/DFS, iterative deepening, A*, heuristics, memory-efficient data structures.
- The same ideas apply to other problems: **path finding, route planning, robot planning, puzzle solving (15-puzzle, sliding tiles), game AI, scheduling**.
- It is a good **portfolio project** because it shows C++ skills, algorithm knowledge and performance thinking.

### 17.4 Why did I build it? (adapt this to your real reason)
Suggested honest answer:
"I wanted a project that goes beyond simple CRUD applications and forces me to think about algorithms and performance. The Rubik's Cube is a good test: the state space is enormous, so naive search fails and I had to learn why. I built four solvers to compare them, and added a pattern database to see how a good heuristic changes the result. I used C++ because speed and memory control matter here."

If you followed a tutorial or course, say so, then explain what you understood and changed. Interviewers respect that more than overclaiming.

### 17.5 How does it work? (step by step)
1. **Model the cube.** A class stores the sticker colours and applies moves (F, B, L, R, U, D and their inverses/doubles). Three implementations exist so they can be compared: 3D array, 1D array, bitboard.
2. **Scramble.** `randomShuffleCube(n)` applies n random moves and returns them, so the answer is known to exist within n moves.
3. **Pick a solver.** Each solver takes a cube and returns a list of moves.
4. **Search.** The solver expands states by applying the 18 moves, avoids repeating states, and stops when the cube is solved.
5. **Print.** The scramble and the solution are printed with `getMove()` as readable notation (e.g. R, U', F2).

### 17.6 Cube representations and trade-offs
| Representation | Idea | Pros | Cons |
|---|---|---|---|
| 3D array | 6 faces x 3 x 3 grid | Easy to read and debug | Slow, memory heavy |
| 1D array | 54 stickers in one array | Simpler, faster copies | Moves need index tables |
| Bitboard | Each face packed into a 64-bit integer (8 stickers x 8 bits, centre is fixed) | Very fast moves (bit shifts/rotations), tiny memory, easy hashing | Hardest to read and debug |

Same interface (`RubiksCube`), different internals. Because the solvers are templates over the cube type and a hash function (`IDAstarSolver<RubiksCubeBitboard, HashBitboard>`), any representation can be used without rewriting the solver.

### 17.7 The algorithms, compared
| Algorithm | Idea | Finds shortest? | Memory | Weakness here |
|---|---|---|---|---|
| **DFS** | Go deep first, backtrack | No | Low | Can wander deep, no shortest guarantee |
| **BFS** | Expand level by level | Yes | Very high, grows about 18^d | Runs out of memory at small depth |
| **IDDFS** | Repeated DFS with depth limit 1, 2, 3... | Yes | Low | Repeats work, still exponential |
| **IDA\*** | IDDFS but prunes with `g + h > limit` using a heuristic | Yes if h is admissible | Low | Needs a good heuristic |

Key words: **g** = moves made so far, **h** = estimated moves remaining, **f = g + h**.

### 17.8 The pattern database (the most important idea)
- A **heuristic** estimates the distance to the goal. It must be **admissible** (never overestimate) so IDA* stays optimal.
- A **pattern database** precomputes exact distances for a simplified version of the problem. Here, the simplification is: look at **only the 8 corner cubies** and ignore edges.
- Solving only the corners can never take more moves than solving the whole cube, so this lookup is a valid lower bound, hence admissible.
- There are 8! x 3^7 = **88,179,840** corner states. Each needs a distance 0-11, which fits in 4 bits, so a **nibble array** (two entries per byte) stores them compactly.
- A **permutation indexer** (Lehmer code / ranking) turns a corner arrangement into a unique array index.
- `CornerDBMaker` builds the table by BFS from the solved state. The file name `cornerDepth5V1.txt` suggests a limited-depth table (depth 5); states beyond the stored depth get a fallback estimate. Confirm this in `CornerDBMaker` and `CornerPatternDatabase`.
- During IDA*, `h` is a fast table lookup, which prunes most of the search tree.

### 17.9 What `main.cpp` does (walk through it in an interview)
```
RubiksCubeBitboard cube;                     // solved cube, bitboard representation
auto shuffleMoves = cube.randomShuffleCube(13);  // scramble with 13 random moves
cube.print();                                // show scrambled cube
IDAstarSolver<...> solver(cube, fileName);   // solver loads corner database file
auto moves = solver.solve();                 // run IDA*
solver.rubiksCube.print();                   // show solved cube
```
It scrambles with 13 moves because deeper scrambles get slow with only a corner heuristic.

### 17.10 Limitations (say these confidently)
- Uses only a corner heuristic, so long scrambles are slow.
- Pattern database file path is hard-coded to a Windows path.
- No automated tests, no command-line arguments, no error handling for a missing database.
- Not guaranteed to find the absolute shortest solution unless the heuristic is admissible and the search completes.

### 17.11 How I would improve it
Add edge pattern databases (two 6-edge tables) and use `max(h_corner, h_edge1, h_edge2)`; add unit tests; add a CLI to choose solver/scramble; replace the hard-coded path; add benchmarks; move toward Kociemba's two-phase algorithm for near-optimal solutions on any scramble.

## 18. Short Pitches

**30 seconds:**
"It's a C++ Rubik's Cube solver. It models the cube, scrambles it, and solves it using DFS, BFS, IDDFS and IDA*. The IDA* solver uses a corner pattern database as a heuristic, which makes searching the huge state space practical. I built it to learn how state representation and heuristics affect search performance."

**2 minutes:**
Cover in order: the problem (43 quintillion states, 18 moves), the three representations, why BFS/DFS fail, how IDDFS fixes memory, how IDA* with a pattern database fixes speed, the nibble array and indexing for compact storage, and finally what you would improve.

## 19. Interview Questions and Answers

**Q1. What is your project?**
A C++ program that solves a scrambled 3x3 Rubik's Cube using search algorithms, with a pattern database heuristic for speed.

**Q2. Why a Rubik's Cube?**
Large state space, clear rules, and easy to verify correctness. It shows why algorithm choice and heuristics matter.

**Q3. Why C++?**
Performance and memory control, which matter for millions of states, bit manipulation and compact tables.

**Q4. How is the cube represented?**
Three ways: 3D array, 1D array, bitboard. All share a common base class, so solvers work with any of them.

**Q5. Why a bitboard?**
Faces are packed into integers, so a move is a few bit operations, and copies and hashing are cheap. That matters when you generate millions of states.

**Q6. Why not just BFS?**
BFS finds the shortest path, but memory grows about 18^depth. It fails around depth 6-7 on ordinary hardware.

**Q7. Why is DFS not enough?**
It uses little memory but can go arbitrarily deep and does not guarantee the shortest solution.

**Q8. What is IDDFS?**
Depth-limited DFS run repeatedly with increasing limits. It gets BFS's shortest-path result with DFS's memory. The repeated work is small because the last level dominates.

**Q9. What is IDA*?**
IDDFS with pruning: a branch is cut when `g + h` exceeds the current limit. A good `h` removes most of the tree.

**Q10. What is an admissible heuristic?**
One that never overestimates the real remaining cost. It guarantees IDA* returns an optimal solution.

**Q11. What is a pattern database?**
A precomputed table of exact solution distances for a simplified problem (here, corners only), used as a heuristic lookup.

**Q12. Why is the corner database admissible?**
Solving the whole cube requires solving the corners, so corner-only distance is a lower bound on the total.

**Q13. How many corner states are there, and how are they stored?**
8! x 3^7 = 88,179,840. Each distance fits in 4 bits, so a nibble array stores two entries per byte, about 44 MB for the full table.

**Q14. Why 3^7 and not 3^8?**
Corner twists must sum to a multiple of 3, so once seven are fixed the eighth is determined.

**Q15. What is the permutation indexer for?**
It maps each corner permutation to a unique integer (factorial number system) to use as an array index.

**Q16. How was the database built?**
By BFS outward from the solved state, recording the depth at which each corner state is first reached. Adjust this answer to match `CornerDBMaker`.

**Q17. Time and space complexity?**
BFS: O(b^d) time and space. DFS: O(b^m) time, O(bm) space. IDDFS: O(b^d) time, O(bd) space. IDA*: much less than b^d in practice, O(bd) space, plus the database in memory. Here b is about 18.

**Q18. How do you avoid redundant moves?**
Skip a move that immediately undoes the previous one (R then R'), and avoid repeated same-face sequences. Check how your solvers do this and be ready to show it.

**Q19. How do you test correctness?**
Scramble with n moves, run the solver, apply its moves and check the cube is solved. Also check each move followed by its inverse restores the state. (Formal tests are a planned improvement.)

**Q20. What are the limitations?**
Only a corner heuristic, hard-coded database path, no tests, slow for long scrambles.

**Q21. How would you make it faster?**
Add edge databases and take the max of heuristics, prune symmetric/redundant moves, use multithreading, or implement Kociemba's two-phase algorithm.

**Q22. What was the hardest part?**
Suggested: getting the moves right on the compact bitboard, and making the database indexing correct. Replace with your real experience.

**Q23. What did you learn?**
How heuristic quality dominates search cost, how to trade readability for speed with data layout, and how to use templates to keep solvers independent of the cube representation.

**Q24. Is your solution optimal?**
With an admissible heuristic and a complete search, IDA* returns a shortest solution. With only the corner heuristic and limited depth table, it is practical mainly for moderate scrambles.

**Q25. How does this relate to real-world problems?**
Same techniques appear in navigation, robotics planning, game AI and solving sliding-tile puzzles.

### Tips for the interview
- Be able to draw the flow: scramble, search, heuristic lookup, solution.
- Know one algorithm (IDA*) well enough to write pseudocode on a whiteboard.
- Be honest about limitations; naming them shows understanding.
- Run the program once beforehand so you can describe real output.
