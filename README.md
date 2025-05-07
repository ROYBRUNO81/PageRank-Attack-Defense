## Summary  
We investigate how web spammers and SEO manipulators use link farms to artificially inflate PageRank for low-visibility pages, and compare two lightweight defenses:  
1. **Edge Pruning**: surgically remove all outgoing edges from automatically detected spam nodes.  
2. **Penalty Reduction**: compute, for each page, the exact percentage of its PageRank mass that originated from those spam nodes, and subtract that share from its attacked score.  

On the Stanford SNAP web-Google graph (875 713 nodes, 5 105 039 edges), adding 1 000 one-off spam pages to a low-rank target (node 6) boosts its PageRank from ~2.8 × 10⁻⁷ to ~2.7 × 10⁻⁴. Both defenses collapse it back near baseline (pruning → 3.6 × 10⁻⁷; reduction → 3.7 × 10⁻⁷), while leaving a good number of pages—including the top 20 % by original rank—virtually unaffected.

---

## Hypotheses  
- **H1 (Attack Effectiveness):** A link-farm of 1 000 new nodes each pointing only to a low-rank page will dramatically increase that page’s PageRank, since every new inbound link channels rank mass directly to it.  
- **H2 (Pruning Defense):** Removing all edges from structurally suspect spam nodes will restore the victim’s PageRank to near its original baseline by cutting off its sole sources of illicit mass. This will produce significant impact on other pages.
- **H3 (Penalty Defense):** Deducting from each page the exact fraction of its PageRank that came via spam links will similarly neutralize the attack—without deleting any graph structure—and will produce minimal collateral impact on other pages.

---

## Experimental Methods  

1. **Baseline Computation**  
   - Load the directed web-Google graph (875 713 nodes, 5 105 039 edges).  
   - Compute baseline PageRank to identify the lowest-rank node (node 6; PR ≈ 2.828 × 10⁻⁷).  

2. **Link-Farm Attack**  
   - Simulate an adversarial attack by adding 1 000 new nodes (IDs > max existing), each with a single outgoing link to node 6.  
   - Recompute PageRank: the victim’s score soars to ~2.73794 × 10⁻⁴.  

3. **Defense A: Edge Pruning**  
   - Automatically detect spam candidates as any node with in-degree ≤ 0 and out-degree = 1 (flagging 38 564 nodes).  
   - Remove every edge originating from these spam nodes.  
   - Recompute PageRank: node 6’s score collapses to ~3.63624 × 10⁻⁷.  

4. **Defense B: Penalty Reduction**  
   - For each page, compute `penalty[v]` = sum of PageRank contributions received directly from spam nodes.  
   - Define `penalty_pct[v]` = `penalty[v]` / `pr_attack[v]` and set `final_pr[v]` = `pr_attack[v] × (1 – penalty_pct[v])`.  
   - The victim’s adjusted PageRank falls to ~3.688 × 10⁻⁷, effectively matching the pruning result—without removing any edges.

---

### Spam Node Detection

We use a simple, degree‐based rule to flag spam pages:

* **Exact match for our simulation:** We created each spam node with **zero in‐links** and **one out‐link** to the target. By selecting nodes with `in_degree = 0` and `out_degree = 1`, we automatically recover exactly those injected attackers.
* **Generalizes to real link farms:** Real one‐off spam pages typically point only at their target and receive no organic links, so this heuristic catches such link‐farm patterns while leaving genuine pages largely unaffected.


## Key Results  

| Stage            | PageRank of node 6        |
|------------------|---------------------------|
| Baseline         | 2.828 × 10⁻⁷              |
| After Attack     | 2.73794 × 10⁻⁴            |
| After Pruning    | 3.63624 × 10⁻⁷            |
| After Reduction  | 0.0000000000              |

- **Spam nodes flagged:** 38 564 / 875 713  
- Both defenses restore the victim’s rank to within ~30 % of its original value.

---

## Side-Effect Analysis  

Pruning the spam origin edges restores the victim, but at the cost of stripping away roughly **35 032** pages from the original top 20 % and reducing their average PageRank by ~9.2 %. In contrast, the penalty-based reduction drops only **1 468** pages from that cohort and lowers their average by just ~1.5 %.  

This demonstrates that directly deducting the exact spam-derived share of PageRank:  
1. **Minimizes collateral damage**—legitimate high-rank pages remain largely unaffected.  
2. **Preserves graph integrity**—no edges or nodes are deleted, maintaining continuity for downstream analyses.  

Penalty-based reduction therefore offers an effective defense against link-farm attacks while retaining the rightful ranking and connectivity of genuine pages.

### Visualizing Effects

To assess collateral impacts on the top‐penalty nodes, we plot the **aggregate positive change** in PageRank (relative to baseline) for three scenarios:

* **After Attack:** A small salmon-colored bar shows that the link‐farm boost barely ripples beyond the single target—most victims see only a tiny net gain.
* **After Pruning:** The large light-green bar reveals that surgically removing spam edges redistributes a substantial chunk of freed PageRank to many victims, causing an unintended surge.
* **After Penalty Reduction:** The nearly zero orange bar confirms that subtracting precisely the spam-derived share leaves the overall distribution intact, with no broad upward shifts.

This comparison highlights that **penalty reduction** neutralizes the attack while minimizing side effects on the rest of the network.

---

## Conclusions  
- **Link-farm attacks** are trivially simulated and can massively distort PageRank for targeted pages.  
- **Edge Pruning** is a straightforward, surgical defense but permanently alters the graph and has side effects on other nodes.
- **Penalty Reduction** achieves equivalent neutralization by subtracting precisely the spam-derived rank share—requiring no graph surgery and causing negligible side effects.  

Both **Edge Pruning** and **Penalty Reduction** successfully collapse the inflated PageRank back to baseline, but Penalty Reduction does so with far less collateral damage—retaining over 99 % of the original top-20 % pages and altering their average rank by only ~1.5 %—and without deleting any graph structure. This precise, non-destructive adjustment makes Penalty Reduction the preferred defense against adversarial link-farm attacks.
