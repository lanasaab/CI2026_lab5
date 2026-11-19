## Lab 5 Overview

This project implements an autonomous agent for the classic **Wumpus World** environment. The agent explores a partially observable 4 × 4 cave, collects gold, and returns to its starting position while avoiding pits and a living Wumpus.

The solution combines **condition–action rules**, **knowledge-based safety inference**, and **breadth-first search (BFS)**. It prioritizes moves that can be established as safe from previously observed percepts and uses its single arrow when it can uniquely identify the Wumpus and no safe exploration route remains.

> **Approach:** Classical rule-based AI with logical inference and search. This implementation does **not** use Q-learning, train a model, or implement a Learning Classifier System (LCS).

## Environment

- **Grid:** 4 × 4, with coordinates from `(1, 1)` to `(4, 4)`.
- **Start and exit:** `(1, 1)`.
- **Objective:** Find the gold, return to `(1, 1)`, and climb out.
- **Hazards:** Entering a pit or the square occupied by a living Wumpus ends the run.
- **Arrow:** The agent has one arrow, which travels straight ahead until it hits the Wumpus or reaches the cave boundary.

The environment supplies these percepts:

| Percept | Meaning |
| --- | --- |
| `breeze` | At least one adjacent square contains a pit. |
| `stench` | An adjacent square contains the living Wumpus. |
| `glitter` | The current square contains the gold. |
| `scream` | The arrow has killed the Wumpus. |

## How the Agent Works

### 1. Knowledge and safety inference

The agent stores observations in `Explorer.history`. Visited squares are treated as safe because the agent is alive there. A square with no `breeze` establishes that all its neighbors are pit-free; a square with no `stench` establishes that all its neighbors are Wumpus-free. The intersection of these sets forms the known-safe region.

When the agent hears a `scream`, it marks the Wumpus as dead and removes the Wumpus hazard from all grid cells. Pit safety is still assessed separately.

### 2. Path planning

`Explorer.route()` uses BFS over **(position, heading)** states. The planner considers three primitive actions: move forward, turn left, and turn right. It only permits forward movement into known-safe cells, and returns a shortest sequence of these actions to a target under this action model.

### 3. Rule-based action selection

The immutable `Rule` objects represent condition–action mappings. The agent checks the rules in priority order to climb when home with gold, grab glittering gold, shoot when a shot is planned, or execute the first movement action of its plan.

### 4. Shooting

If no safe unexplored square is reachable, the agent considers shooting. It intersects candidate Wumpus locations with the neighbors of previously observed `stench` squares and removes known Wumpus-free locations. It shoots only when exactly one candidate remains and a known-safe adjacent firing square is reachable. It first turns to face the target if needed.

### 5. Returning home

After collecting gold, the planner targets `(1, 1)` using known-safe squares. Reaching the starting square with gold activates the `climb` rule.

## Main Components

| Component | Responsibility |
| --- | --- |
| `adjacent(cell)` | Computes in-bounds neighboring squares. |
| `Rule` and `RULES` | Define fixed condition–action rules. |
| `Explorer.knowledge()` | Infers pit-free and Wumpus-free squares. |
| `Explorer.route()` | Finds action sequences with BFS. |
| `Explorer.aim_or_approach()` | Plans a shot at a uniquely inferred Wumpus location. |
| `Explorer.step()` | Processes percepts and performs one agent action. |
| `simulate()` | Runs a cave and reports success, action count, and arrows used. |

## Requirements

- Python 3.9 or later (for built-in generic type annotations).
- No third-party packages. The code uses only `collections` and `dataclasses` from the Python standard library.

## Running the Project

Save the implementation as `wumpus_world.py`, then run:

```bash
python wumpus_world.py
```

If using the Jupyter Notebook version, open the `.ipynb` file and select **Run All**.

To test another cave programmatically:

```python
result = simulate(
    pits={(3, 1), (3, 3), (4, 4)},
    monster=(1, 3),
    treasure=(2, 3),
    verbose=True,
)
print(result)
```

`simulate()` returns `(success, steps, arrows_used)`. The printed trace shows the action taken at each turn.

## Included Test Scenarios

The script includes two deterministic scenarios and assertions:

| Scenario | Pits | Wumpus | Gold | Expected arrows |
| --- | --- | --- | --- | --- |
| Original cave | `(3,1)`, `(3,3)`, `(4,4)` | `(1,3)` | `(2,3)` | 0 |
| Shooting cave | `(3,1)`, `(3,2)`, `(3,3)` | `(1,3)` | `(1,4)` | 1 |

For both scenarios, the assertions require successful retrieval and exit, with the indicated number of shots. These are **scenario-specific tests**, not proof that the agent solves every possible cave.

## Design Considerations and Limitations

- **Conservative exploration:** The agent enters only squares that it can establish as safe. It may stop if the gold is inaccessible through currently proven-safe cells.
- **Incomplete inference:** The safety rules do not implement a complete propositional-logic solver or enumerate all possible pit configurations.
- **Limited shooting strategy:** The agent shoots only when it uniquely infers the Wumpus's location and can reach a neighboring safe firing square. Other useful long-range shots are not considered.
- **Fixed rules:** The rules do not evolve or learn from repeated episodes; this is not reinforcement learning.
- **Simplified simulator:** The environment assumes one Wumpus, static pits and gold, and deterministic actions and percepts.

## Possible Extensions

Potential improvements include stronger logical inference, probabilistic risk assessment, more flexible line-of-sight shooting, broader randomized test coverage, and comparison against reinforcement-learning agents.

## Summary

The project demonstrates how an intelligent agent can combine **symbolic rules, percept-driven knowledge representation, and BFS planning** to act in a partially observable environment. Its main design priority is safe, explainable behavior rather than learning a policy from training data.
