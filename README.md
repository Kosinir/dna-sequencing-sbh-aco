# DNA Sequencing by Hybridization (SBH) using Ant Colony Optimization

Project implementing reconstruction of DNA sequences using the **Sequencing by Hybridization (SBH)** approach combined with metaheuristic optimization.

## Overview

The goal of the project is to reconstruct an original DNA sequence from a spectrum of oligonucleotides (k-mers).

The biological problem is transformed into a **graph optimization problem**, where:
- vertices represent k-mers,
- edges represent overlaps between sequences,
- edge weights correspond to the number of added nucleotides.

The solution combines:

- Ant Colony Optimization (ACO)
- Dijkstra shortest-path recovery
- local 2-opt optimization
- pheromone evaporation and elitist reinforcement

The algorithm is designed to work even in the presence of:
- missing k-mers (negative errors),
- false k-mers (positive errors),
- repeated fragments.

---

## Input Data

Program reads a text file containing:

- `n` — target DNA sequence length
- `k` — k-mer length (7 ≤ k ≤ 10)
- `start_oligo` — starting fragment
- `num_neg_errors` — missing k-mers
- `num_pos_errors` — false k-mers
- `has_repeats` — repeated fragments flag
- list of k-mers

---

## Algorithm Description

### Graph Construction
The k-mer spectrum is converted into a directed graph based on sequence overlaps.

### ACO Phase
Each ant builds a candidate reconstruction path guided by:
- pheromone levels,
- overlap quality,
- exploration vs exploitation parameters.

### Trap Recovery
When ants get stuck, the algorithm applies:
- Dijkstra shortest path search,
- virtual jumps to escape local dead ends.

### Local Optimization
Completed solutions are improved using a **2-opt optimization** strategy.

### Pheromone Update
- evaporation factor (ρ)
- solution-based deposition
- elitist reinforcement

---

## Experiments

Two categories of experiments were performed:

1. Algorithm parameter analysis
2. Instance parameter analysis

Metrics:
- average results over 50 instances
- Levenshtein distance used as reconstruction quality measure
- tested for n = 300, 500, 700

Key observations:

- Increasing runtime improves convergence.
- Higher β favors optimal overlaps.
- Positive errors degrade performance more than negative errors.
- Larger k values significantly improve reconstruction quality.

---


## Technologies

- Python
- Graph algorithms
- Metaheuristics (ACO)
- Bioinformatics

---

## Author

Szymon Warguła
