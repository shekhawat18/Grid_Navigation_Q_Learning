# Intelligent Grid Navigation Agent using Q-Learning

Bio-Inspired Computing hackathon, Problem 23.

## Problem statement
Develop an agent that learns to navigate a grid from a starting point to a destination while avoiding obstacles, using Q-learning. The agent receives a positive reward for reaching the destination and penalties for collisions and unnecessary moves. The agent is never given the route: it learns only from rewards.

## Algorithm: tabular Q-learning
- **State:** the agent's grid cell, encoded as `row * 8 + col` (64 states).
- **Actions:** 0 = up, 1 = down, 2 = left, 3 = right.
- **Rewards:** goal `+100` (terminal), each step `-1`, collision with an obstacle or the wall `-10` (the agent stays in place).
- **Update rule (Bellman):**

  `Q(s,a) <- Q(s,a) + alpha * [ r + gamma * max_a' Q(s',a') - Q(s,a) ]`

  At the terminal goal state the target is just `r`.
- **Exploration:** epsilon-greedy, epsilon decays exponentially from 1.0 to 0.05 over the first 70% of episodes. Ties between equal Q-values are broken randomly.
- **Hyperparameters:** alpha = 0.1, gamma = 0.95, 2000 training episodes, seed 42.

## Implementation details
- 8x8 grid, start `(0,0)`, goal `(7,7)`, 15 fixed obstacles. Moves outside the grid or into obstacles are blocked and penalised.
- Every episode is capped at 100 steps, so training and evaluation always terminate.
- 50% of training episodes start from a random free cell (exploring starts) so values are learned across the whole grid; the other 50% start at `S`.
- Evaluation uses a **separate random stream**, `epsilon = 0` (pure greedy), and no learning, over 200 episodes from random free start cells.
- BFS is used **only for validation** (is the goal reachable, what is the optimal length). It never chooses actions or trains the agent.

## Results (from an actual run of the notebook, seed 42)
| Metric | Value |
|---|---|
| Mean training reward, first 100 episodes | -283.7 |
| Mean training reward, last 100 episodes | 88.6 |
| Evaluation success rate (200 greedy episodes) | 100.0% |
| Average path length in evaluation | 7.28 steps |
| Average evaluation reward | 93.72 |
| Learned route from `S` to `G` | 14 steps (BFS optimal: 14), reward 87 |

Validation tests in Cell 5 check legal/invalid moves, obstacle collision, goal reward and termination, the Q-update (non-terminal and terminal), obstacle-free route, and an unreachable-goal map where the agent never succeeds and episodes still terminate.

## Limitations
- Tabular Q-learning does not scale to large or continuous state spaces.
- The policy is tied to one fixed map; a new layout requires retraining.
- Transitions are deterministic and the agent sees only its own position.
- Results depend on the seed and hyperparameters.
