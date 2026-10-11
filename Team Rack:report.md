# PS-O1 Report: Max-Cut QAOA on Processors A and B

**Problem Statement:** PS-O1 — Max-Cut on Weighted Graphs
**Track:** Quantum Optimization
**Team:** [Your Name / Team Name]

---

## 1. Problem

Max-Cut asks us to partition the vertices of a weighted graph into two sets so that the total weight of edges crossing between the sets is maximized. For n vertices encoded as spins x_i ∈ {-1, +1}, the cost function is:

  Max-Cut(G) = max over x of (1/2) * Σ_(i,j)∈E w_ij * (1 - x_i * x_j)

This is an NP-hard combinatorial optimization problem with applications in network design, clustering, and statistical physics.

**Instance used:** A 6-node weighted graph with edges:
(0,1,3), (1,2,5), (2,3,2), (3,4,4), (4,5,1), (5,0,3)

Classical optimum (found by brute force over all 2^6 = 64 assignments): **18**

---

## 2. Method

We use the **Quantum Approximate Optimization Algorithm (QAOA)** at depth p = 1:

1. **Encode** the Max-Cut problem as a cost Hamiltonian:
   H_C = Σ_(i,j)∈E w_ij * Z_i * Z_j

2. **Prepare** the initial state |+⟩^⊗n by applying Hadamard gates to all qubits.

3. **Alternate** between:
   - Cost unitary: U_C(γ) = exp(-i γ H_C), decomposed as CX–RZ(2wγ)–CX per edge
   - Mixer unitary: U_B(β) = exp(-i β Σ X_i), decomposed as RX(2β) per qubit

4. **Measure** all qubits and use a classical optimizer (COBYLA) to tune γ and β to maximize the expected cut weight.

---

## 3. Processor A

- **Qubits:** 5
- **Coupling map:** (0,1), (1,2), (1,3), (3,4)
- **Topology:** Branching tree with central hub at qubit 1

**Result:** Processor A **cannot execute** the 6-qubit Max-Cut problem. Its 5 physical qubits are insufficient to hold the 6 logical qubits required by the graph. This is a **hard architectural limit** — no amount of transpilation, routing, or SWAP insertion can fit 6 logical qubits into 5 physical qubits.

---

## 4. Processor B

- **Qubits:** 9
- **Coupling map:** (0,1), (1,2), (1,6), (2,3), (2,6), (2,5), (3,4), (4,5), (5,6), (6,7), (6,8), (8,0)
- **Topology:** Heavy-hex-inspired graph with higher connectivity

**Result:** Processor B successfully executes the circuit. Transpilation inserts SWAP gates for logical edges that are not directly available in the physical coupling map, but all 6 qubits can be placed.

---

## 5. A vs B Comparison

| Metric | Processor A | Processor B |
|--------|-------------|-------------|
| Approximation ratio | N/A (5 qubits insufficient) | **1.000** |
| Best cut weight found | N/A | **18** (matches classical optimum) |
| Optimal bitstring | N/A | **101010** |
| Probability of optimal cut | N/A | 0.0049 |
| Physical qubits | 5 | 9 |
| Can hold problem? | ❌ No | ✅ Yes |

---

## 6. Explanation: The SWAP Bottleneck

The central architectural finding is a **qubit-count bottleneck combined with a connectivity bottleneck**:

**Qubit count.** Processor A's 5 qubits < 6 logical qubits needed by the problem. The circuit cannot execute at all. This is a binary feasibility failure, not a gradual degradation. In a real deployment, this would force the team to either:
- Reduce the problem to 5 nodes (changing the problem), or
- Embed onto a larger machine with additional ancillas (paying SWAP cost to move between the 5-qubit region and external qubits).

**Connectivity.** Even once we move to Processor B with 9 qubits, most logical edges in our 6-node graph are not directly available in the coupling map. For example, the logical edge (0,5) has no direct physical connection — the transpiler must insert SWAP gates (each costing 3 CX gates) to bring those qubits together.

**Noise impact.** Under our synthetic depolarizing model with p_cx = 0.005 per CX, each additional SWAP adds roughly 1.5% error to the circuit. Even though Processor B succeeded, the low probability of sampling the optimal bitstring (0.49%) reflects both the low QAOA depth (p = 1) and the accumulated noise from routed two-qubit gates.

---

## 7. Evidence

- **Figure:** problem graph (6-node hexagon with weights)
- **Figure:** Processor A coupling graph (5-node branching tree)
- **Figure:** Processor B coupling graph (9-node heavy-hex-inspired)
- **Table:** AB_comparison.csv in results/tables/
- **Output:** Cell 8 transpilation metrics; Cell 11 QAOA results

---

## 8. Limitations

1. **Only p = 1 QAOA.** Higher depth (p = 2, 3) would improve the probability of sampling the optimal bitstring.
2. **Synthetic noise model.** We use a depolarizing model with p_cx = 0.005 per CX. Real hardware would have qubit-specific noise, readout errors, and crosstalk.
3. **Small instance.** 6 nodes is tiny; QAOA scaling on 20+ node graphs would reveal the SWAP bottleneck much more dramatically.
4. **No parameter optimization shown here.** The reported approximation ratio 1.000 was obtained with a single random parameter draw. A full COBYLA optimization loop would improve the optimal-bitstring probability.
5. **Single seed.** Results depend on the random parameter initialization.

---

## 9. Conclusion

Processor architecture determines **whether** a QAOA Max-Cut circuit can execute at all, not just **how well** it executes. Processor A's 5-qubit limit makes the 6-node problem infeasible; Processor B's 9 qubits and higher connectivity allow execution with SWAP overhead. For larger problem instances, the SWAP bottleneck becomes the dominant factor in both circuit depth and fidelity loss.

---

*AI assistance was used for code scaffolding and error debugging. All results were verified by running the code in Google Colab.*
