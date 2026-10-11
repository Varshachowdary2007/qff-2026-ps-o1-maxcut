# QFF 2026 Phase 1 — PS-O1: Max-Cut on Weighted Graphs

**Track:** Quantum Optimization
**Problem Statement:** PS-O1 — Max-Cut using QAOA
**Team:** Varshachowdary2007

## What This Project Does

Solves the Max-Cut problem on a 6-node weighted graph using the Quantum Approximate Optimization Algorithm (QAOA) with p=1, then compares execution on two simulated quantum processors:

- **Processor A:** 5 qubits, branching topology — **cannot fit the 6-qubit problem**
- **Processor B:** 9 qubits, heavy-hex-inspired topology — **fits and solves the problem optimally**

## Key Result

QAOA found the classical optimum (cut weight 18) on Processor B with approximation ratio 1.000. Processor A is architecturally infeasible for this instance.

## How to Run

1. Open `main.ipynb` in Google Colab
2. Click **Runtime → Run all**
3. All results reproduce from a clean environment

## Requirements

See `requirements.txt`. Install with:

## Repository Structure

## AI Assistance Disclosure

Code scaffolding and debugging were assisted by an LLM. All physics reasoning, processor analysis, and results were verified by running the notebook.
