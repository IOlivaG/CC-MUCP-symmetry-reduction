# CC-MUCP Mathematical Formulation

> Mathematical formulation for the CC-MUCP symmetry reduction study.
> See the companion notebook `ccmucp_symmetry_reduction.ipynb` for the computational implementation.
> See §5 for the results summary and conclusions.

---

## 1. Combined Cycle Power Plants

A combined cycle (CC) power plant recovers the exhaust heat of one or more gas turbines (GTs) through a heat recovery steam generator (HRSG) to drive a steam turbine (ST), achieving substantially higher thermal efficiency than open-cycle (OC) operation<sup>[[1]](#ref1)</sup>. This efficiency gain introduces operational coupling between components: the ST can only operate if a sufficient number of GTs have been running long enough to heat the HRSG<sup>[[4]](#ref4)</sup>.

Each CC plant is modelled as a **package** $k$, consisting of $n_{gt}$ gas turbines and one steam turbine in a $n_{gt} \times 1$ configuration<sup>[[5]](#ref5)</sup>. The set of all packages is $\mathcal{K} = \{1, \ldots, K\}$.

## 2. Min-Up/Min-Down Unit Commitment Problem (MUCP)

The MUCP is the core combinatorial optimisation problem underlying thermal unit commitment. Given a discrete time horizon and a set of production units, determine which units should be on or off (and how much to produce) at each period to meet demand at minimum cost, subject to<sup>[[2]](#ref2)</sup>:

- **Minimum up-time:** once started, a unit must remain on for $L$ periods.
- **Minimum down-time:** once shut down, a unit must remain off for $\ell$ periods.
- **Production bounds:** each unit operates within $[P_{\min}, P_{\max}]$ when on.

The MUCP polytope has been characterised polyhedrally in the $(x,u)$ variable space, with facet-defining up-set, min-up, and min-down inequalities established in<sup>[[3]](#ref3)</sup>.

## 3. Combined Cycle Unit Commitment Problem (CC-MUCP)

The CC-MUCP extends the MUCP by modelling the internal GT--ST structure of CC plants<sup>[[4]](#ref4),[[6]](#ref6)</sup>. In addition to the classical MUCP constraints, it includes:

- **Mode-dependent GT operation:** each GT operates in open-cycle (OC) or combined-cycle (CC) mode.
- **Thermal coupling:** the ST can only operate when enough GTs are active (CC1), and the ST start-up requires GTs to have been running for a warm-up period (CC4)<sup>[[4]](#ref4)</sup>.
- **Interdependent production:** ST output depends on the number of GTs operating in CC mode.

The formulation adopted here follows a **component-based** approach<sup>[[4]](#ref4)</sup>, where each GT and ST is treated as an individual decision unit. This is in contrast to **configuration-based** formulations<sup>[[5]](#ref5)</sup>, which represent each CC plant as a single unit selecting among predefined operating configurations. The component-based approach provides a finer-grained representation of the internal GT-ST coupling essential for symmetry analysis.

### 3.1 Decision Variables

For each package $k \in \mathcal{K}$, GT $g \in \{1, \ldots, n_{gt}\}$, and period $t \in \mathcal{T} = \{1, \ldots, T\}$:

<div align="center">

**Table A.** Decision variables for the CC-MUCP MILP formulation.

$$
\begin{array}{lll}
\hline
\textbf{Variable} & \textbf{Domain} & \textbf{Description} \\
\hline
\hline
x_{k,g}^t & \{0,1\} & \text{GT } g \text{ of package } k \text{ is on at } t \\
u_{k,g}^t & \{0,1\} & \text{GT } g \text{ starts up at } t \\
y_{k,g,\mathrm{OC}}^t & \{0,1\} & \text{GT operates in open cycle at } t \\
y_{k,g,\mathrm{CC}}^t & \{0,1\} & \text{GT operates in combined cycle at } t \\
p_{k,g,\mathrm{OC}}^t & \mathbb{R}_{\geq 0} & \text{GT production in open cycle (MW)} \\
p_{k,g,\mathrm{CC}}^t & \mathbb{R}_{\geq 0} & \text{GT production in combined cycle (MW)} \\
x_{k,v}^t & \{0,1\} & \text{ST of package } k \text{ is on at } t \\
u_{k,v}^t & \{0,1\} & \text{ST of package } k \text{ starts up at } t \\
p_{k,v}^t & \mathbb{R}_{\geq 0} & \text{ST production (MW)} \\
\hline
\end{array}
$$

</div>

### 3.2 Parameters

<div align="center">
<table style="border: none; border-collapse: collapse;">
<tr valign="top">
<td align="center" style="border: none;">

**Table B.** Gas turbine parameters (identical for all GTs in all packages).

$$
\begin{array}{lc}
\hline
\textbf{Symbol} & \textbf{Description} \\
\hline
\hline
P_{\min}^{\mathrm{OC}}, P_{\max}^{\mathrm{OC}} & \text{Production limits in open cycle (MW)} \\
P_{\min}^{\mathrm{CC}}, P_{\max}^{\mathrm{CC}} & \text{Production limits in combined cycle (MW)} \\
c_f^{\mathrm{OC}}, c_p^{\mathrm{OC}} & \text{Fixed and variable costs in open cycle} \\
c_f^{\mathrm{CC}}, c_p^{\mathrm{CC}} & \text{Fixed and variable costs in combined cycle} \\
c_0^g & \text{GT start-up cost} \\
L_g, \ell_g & \text{Minimum up/down times} \\
\hline
\end{array}
$$

</td>
<td width="60" style="border: none;"></td>
<td align="center" style="border: none;">

**Table C.** Steam turbine parameters (identical for all packages).

$$
\begin{array}{lc}
\hline
\textbf{Symbol} & \textbf{Description} \\
\hline
\hline
P_{\min}^v & \text{ST minimum production (MW)} \\
\alpha & \text{ST capacity per GT (CC mode) (MW)} \\
c_f^v, c_p^v, c_0^v & \text{ST fixed, variable, start-up costs} \\
L_v, \ell_v & \text{ST minimum up/down times} \\
\hline
\end{array}
$$

</td>
</tr>
</table>
</div>

<div align="center">

**Table D.** Coupling parameters (per package).

$$
\begin{array}{lc}
\hline
\textbf{Symbol} & \textbf{Description} \\
\hline
\hline
m_k & \text{Minimum GTs on to operate the ST} \\
\delta^{\mathrm{up}} & \text{Periods GTs must be on before ST can start} \\
\delta^{\mathrm{dn}} & \text{Periods GTs remain on after ST shuts down} \\
\hline
\end{array}
$$

</div>

### 3.3 MILP Formulation

**Objective.** Minimise total operating cost:

$$
\min \sum_{t} \sum_{k} \left[ \sum_{g} \left( c_f^{\mathrm{OC}} y_{k,g,\mathrm{OC}}^t + c_f^{\mathrm{CC}} y_{k,g,\mathrm{CC}}^t + c_0^g u_{k,g}^t + c_p^{\mathrm{OC}} p_{k,g,\mathrm{OC}}^t + c_p^{\mathrm{CC}} p_{k,g,\mathrm{CC}}^t \right) + c_f^v x_{k,v}^t + c_0^v u_{k,v}^t + c_p^v p_{k,v}^t \right]
$$

**GT constraints** (for each $k$, $g$, $t$, with $x_{k,g}^0 = 0$):

$$
\begin{align}
&\textbf{(SU-GT)} & u_{k,g}^t &\geq x_{k,g}^t - x_{k,g}^{t-1} \\
&\textbf{(MU-GT)} & x_{k,g}^{t+s} &\geq u_{k,g}^t, \quad s = 1,\ldots,\min(L_g-1,\,T-t) \\
&\textbf{(MD-GT)} & x_{k,g}^{t+s} &\leq 1-(x_{k,g}^{t-1}-x_{k,g}^t), \quad t\geq 2,\; s=1,\ldots,\min(\ell_g-1,\,T-t) \\
&\textbf{(MOD)}   & x_{k,g}^t &= y_{k,g,\mathrm{OC}}^t + y_{k,g,\mathrm{CC}}^t \\
&\textbf{(PB-OC)} & P_{\min}^{\mathrm{OC}} y_{k,g,\mathrm{OC}}^t &\leq p_{k,g,\mathrm{OC}}^t \leq P_{\max}^{\mathrm{OC}} y_{k,g,\mathrm{OC}}^t \\
&\textbf{(PB-CC)} & P_{\min}^{\mathrm{CC}} y_{k,g,\mathrm{CC}}^t &\leq p_{k,g,\mathrm{CC}}^t \leq P_{\max}^{\mathrm{CC}} y_{k,g,\mathrm{CC}}^t
\end{align}
$$

**ST constraints** (for each $k$, $t$, with $x_{k,v}^0 = 0$):

$$
\begin{align}
&\textbf{(SU-ST)} & u_{k,v}^t &\geq x_{k,v}^t - x_{k,v}^{t-1} \\
&\textbf{(MU-ST)} & x_{k,v}^{t+s} &\geq u_{k,v}^t, \quad s=1,\ldots,\min(L_v-1,\,T-t) \\
&\textbf{(MD-ST)} & x_{k,v}^{t+s} &\leq 1-(x_{k,v}^{t-1}-x_{k,v}^t), \quad t\geq 2,\; s=1,\ldots,\min(\ell_v-1,\,T-t) \\
&\textbf{(PB-ST)} & P_{\min}^v x_{k,v}^t &\leq p_{k,v}^t \leq \alpha \sum_{g=1}^{n_{gt}} y_{k,g,\mathrm{CC}}^t
\end{align}
$$

**Coupling constraints** (for each $k$, $t$):

$$
\begin{align}
&\textbf{(CC1)} && \textstyle\sum_{g=1}^{n_{gt}} x_{k,g}^t \geq m_k \cdot x_{k,v}^t && \text{min GTs for ST} \\
&\textbf{(CC2)} && y_{k,g,\mathrm{CC}}^t \leq x_{k,v}^t && \text{CC mode only if ST on} \\
&\textbf{(CC3)} && y_{k,g,\mathrm{OC}}^t \leq 1 - x_{k,v}^t && \text{OC mode only if ST off} \\
&\textbf{(CC4)} && m_k \cdot u_{k,v}^t \leq \textstyle\sum_{g=1}^{n_{gt}} x_{k,g}^{t-\delta^{\mathrm{up}}},\; t > \delta^{\mathrm{up}} && \text{ST start requires warm GTs} \\
&\textbf{(CC5)} && x_{k,g}^{t+s} \geq x_{k,v}^{t-1} - x_{k,v}^t,\; s=0,\ldots,\delta^{\mathrm{dn}}-1 && \text{GTs stay on after ST shutdown}
\end{align}
$$

**Global demand constraint:**

$$
\textbf{(D)} \quad \sum_{k \in \mathcal{K}} \left[ \sum_{g=1}^{n_{gt}} \left( p_{k,g,\mathrm{OC}}^t + p_{k,g,\mathrm{CC}}^t \right) + p_{k,v}^t \right] \geq D_t, \quad \forall t \in \mathcal{T}
$$

<div style="font-size: 0.8em;">

> **Note on formulation sources.** The GT and ST constraints follow the $(x,u,p)$ formulation of Rajan & Takriti<sup>[[2]](#ref2)</sup> as analysed in Bendotti et al.<sup>[[3]](#ref3)</sup>, and the coupling constraints (CC1--CC5) are based on the component-based model of Liu et al.<sup>[[4]](#ref4)</sup>. Two adaptations are introduced: **(CC4)** is reformulated as $m_k \cdot u_{k,v}^t \leq \sum_g x_{k,g}^{t-\delta^{\mathrm{up}}}$ to avoid requiring a specific designated subset of GTs for ST start-up; and the ST production upper bound in **(PB-ST)** uses $\alpha \sum_g y_{k,g,\mathrm{CC}}^t$, which is exact given that (CC2) and (CC3) force $y_{k,g,\mathrm{CC}}^t = x_{k,g}^t$ when the ST is on and $y_{k,g,\mathrm{CC}}^t = 0$ otherwise.

</div>

---

## 4. Instance Parameters (Two-Package Example)

The computational study considers a CC plant with $K = 2$ packages, each consisting of $n_{gt} = 3$ gas turbines and one steam turbine ($3 \times 1$ configuration), over a weekly horizon of $T = 21$ periods structured as three load levels per day (Low, Mid, High). The packages are assumed identical in their physical parameters, which is the typical setting for studying symmetry reduction.

In combined-cycle mode, GTs operate at lower fixed and variable costs than in open-cycle mode, reflecting the efficiency gain from heat recovery. The ST incurs no variable cost ($c_p^v = 0$) since it runs on residual exhaust heat, and its higher start-up cost ($c_0^v = 15{,}000$) and longer minimum up/down times ($L_v = 3$, $\ell_v = 2$) reflect the thermal inertia of the steam cycle. Coupling parameters $m_k = 2$, $\delta^{\mathrm{up}} = 1$, and $\delta^{\mathrm{dn}} = 1$ enforce that at least two GTs must be online before the ST can start, and must remain so after it shuts down.

<div align="center">
<table style="border: none; border-collapse: collapse;">
<tr valign="top">
<td align="center" style="border: none;">

**Table 1a.** Gas turbine parameters for the CC-MUCP instance.

$$
\begin{array}{lcc}
\hline
\textbf{Parameter} & \textbf{OC} & \textbf{CC} \\
\hline
\hline
P_{\max} \, (\text{MW}) & 150 & 145 \\
P_{\min} \, (\text{MW}) & 50 & 60 \\
c_f \, (\$/\text{h}) & 844 & 549 \\
c_p \, (\$/\text{MWh}) & 37.1 & 24.1 \\
c_0 \, (\$/\text{start}) & 25{,}000 & \text{---} \\
\text{Min-up } L_g \, (\text{h}) & 2 & \text{---} \\
\text{Min-down } \ell_g \, (\text{h}) & 1 & \text{---} \\
\hline
\end{array}
$$

</td>
<td width="60" style="border: none;"></td>
<td align="center" style="border: none;">

**Table 1b.** Steam turbine parameters for the CC-MUCP instance.

$$
\begin{array}{lc}
\hline
\textbf{Parameter} & \textbf{Value} \\
\hline
\hline
P_{\min}^v \, (\text{MW}) & 20 \\
\alpha \, (\text{MW/GT}) & 40 \\
c_f^v \, (\$/\text{h}) & 42 \\
c_p^v \, (\$/\text{MWh}) & 0 \\
c_0^v \, (\$/\text{start}) & 15{,}000 \\
\text{Min-up } L_v \, (\text{h}) & 3 \\
\text{Min-down } \ell_v \, (\text{h}) & 2 \\
\hline
\end{array}
$$

</td>
</tr>
</table>
</div>

<div align="center">

**Table 2.** Coupling parameters for the CC-MUCP instance.

$$
\begin{array}{lcc}
\hline
\textbf{Parameter} & \textbf{Value} & \textbf{Description} \\
\hline
\hline
m_k & 2 & \text{Minimum GTs required to operate the ST} \\
\delta^{\mathrm{up}} \, (\text{h}) & 1 & \text{Periods GTs on before ST can start} \\
\delta^{\mathrm{dn}} \, (\text{h}) & 1 & \text{Periods GTs remain on after ST shutdown} \\
\hline
\end{array}
$$

</div>

### 4.1 Demand Profile

The demand profile is structured as three load levels per day for a full week, totalling 21 periods. With a total system capacity of

$$K \cdot (n_{gt} \cdot P_{\max}^{\mathrm{CC}} + n_{gt} \cdot \alpha) = 2 \cdot (3 \cdot 145 + 3 \cdot 40) = 1110 \text{ MW},$$

peak periods require near-full utilisation of both packages, while low-demand periods can be covered by a reduced subset of units.

<div align="center">

**Table 3.** Weekly demand profile.

$$
\begin{array}{lcr}
\hline
\text{Period} & \text{Day-Level} & \text{Demand (MW)} \\
\hline
\hline
1 & \text{Mon-Low} & 420 \\
2 & \text{Mon-Mid} & 620 \\
3 & \text{Mon-High} & 900 \\
4 & \text{Tue-Low} & 400 \\
5 & \text{Tue-Mid} & 600 \\
6 & \text{Tue-High} & 880 \\
7 & \text{Wed-Low} & 430 \\
8 & \text{Wed-Mid} & 640 \\
9 & \text{Wed-High} & 920 \\
10 & \text{Thu-Low} & 410 \\
11 & \text{Thu-Mid} & 630 \\
12 & \text{Thu-High} & 910 \\
13 & \text{Fri-Low} & 450 \\
14 & \text{Fri-Mid} & 700 \\
15 & \text{Fri-High} & \mathbf{1000} \\
16 & \text{Sat-Low} & 520 \\
17 & \text{Sat-Mid} & 680 \\
18 & \text{Sat-High} & 820 \\
19 & \text{Sun-Low} & 500 \\
20 & \text{Sun-Mid} & 650 \\
21 & \text{Sun-High} & 780 \\
\hline
\end{array}
$$

</div>

**Key demand statistics:**
- Peak demand: **1000 MW** at $t=15$ (Fri-High)
- Minimum demand: **400 MW** at $t=4$ (Tue-Low)
- Average demand: **668.6 MW**
- Number of high-demand periods (demand $>$ 850 MW): 5
- Number of low-demand periods (demand $<$ 500 MW): 6

The demand ranking permutation $\pi$ orders periods from highest to lowest demand and is used to assign lower indices to units most active during high-demand periods:

$$\pi = [15, 9, 12, 3, 6, 18, 21, 14, 17, 20, 8, 11, 2, 5, 16, 19, 13, 7, 1, 10, 4]$$

### 4.2 Symmetry-Breaking Framework

The symmetry-breaking ordering constraints follow a **demand-aware lexicographic ordering** approach. The idea is to exploit the fact that not all time periods are equal: periods with higher demand stress the system more, so units active during those periods carry more significance. This provides a natural criterion for selecting a canonical representative from each equivalence class.

The approach works in two steps. First, time periods are ranked from highest to lowest demand using the permutation $\pi$. Then, within each package, GTs are ordered so that those covering more high-demand periods receive lower indices, enforced by prefix-sum inequalities:

$$
\sum_{r=1}^{R} x_{k,g}^{\pi(r)} \geq \sum_{r=1}^{R} x_{k,g+1}^{\pi(r)},
\quad \forall k,\; g = 1,\ldots,n_{gt}-1,\; R = 1,\ldots,T
$$

Only intra-package GT ordering is applied; packages are treated as distinguishable, reflecting the fact that each package is a distinct physical asset with its own steam turbine and HRSG. This follows the general structure of lexicographic symmetry-breaking methods in integer programming, adapted here to the temporal structure of unit commitment problems.

---

## 5. Results Summary and Conclusions

The computational study is conducted in the companion notebook `ccmucp_symmetry_reduction.ipynb` using the instance parameters described in §4. The baseline formulation (no symmetry breaking) yields **108 optimal solutions** at cost \$489,683.

**Symmetry reduction.** Applying the demand-aware GT-level ordering reduces the 108 baseline solutions to **approximately 8 canonical representatives** under the symmetry group $H = S_{n_{\mathrm{gt}}} \times \cdots \times S_{n_{\mathrm{gt}}}$ (order $(n_{\mathrm{gt}}!)^K = 36$ for $n_{\mathrm{gt}}=3$, $K=2$). The exact orbit decomposition depends on the specific schedules: orbits where packages have identical GT commitment patterns remain intact, while those with differences produce distinct representatives. This reflects the correct symmetry of the problem: GTs within the same package are interchangeable, but each package is a distinct physical asset.

<div align="center">

**Table 4.** Results summary.

$$
\begin{array}{lcc}
\hline
\text{Mode} & \text{Solutions} & \text{Reduction factor} \\
\hline
\hline
\texttt{:none} & 108 & 1\times \\
\texttt{:symmetry} & \sim8 & \sim13.5\times \\
\hline
\end{array}
$$

</div>

### 5.3 Conclusions

The demand-aware lexicographic ordering exploits the intra-package symmetry that arises when identical GTs share the same steam turbine within a CC package. The reduction from 108 to ~8 solutions eliminates all redundancy from GT relabelling while preserving the identity of each package. This matches the physical structure of the problem: GTs within a package are genuinely interchangeable (they share the same HRSG and operating conditions), but packages themselves are distinct assets.

The grouping of variables into symmetry-equivalent classes is also relevant beyond solution reduction: it relates to the polyhedral structure of the problem, where characterising the facet-defining inequalities under subgroup actions is a natural extension.

---

## 6. References

<a id="ref1"></a>
<sup>[[1]](#ref1)</sup> R. Kehlhofer, J. Warner, H. Nielsen, and T. Bachmann, *Combined-Cycle Gas and Steam Turbine Power Plants*, 3rd ed., PennWell, 2009.

<a id="ref2"></a>
<sup>[[2]](#ref2)</sup> D. Rajan and S. Takriti, Minimum up/down polytopes of the unit commitment problem with start-up costs. IBM Research Report RC23628, 2005.

<a id="ref3"></a>
<sup>[[3]](#ref3)</sup> P. Bendotti, P. Fouilhoux, and C. Rottner, The min-up/min-down unit commitment polytope. *J. Comb. Optim.*, 36(3), 1024--1058, 2018.

<a id="ref4"></a>
<sup>[[4]](#ref4)</sup> C. Liu, M. Shahidehpour, Z. Li, and L. Wu, Component and mode models for the short-term scheduling of combined-cycle units. *IEEE Trans. Power Syst.*, 24(2), 976--990, 2009.

<a id="ref5"></a>
<sup>[[5]](#ref5)</sup> G. Morales-Espa\~na, C. M. Correa-Posada, and A. Ramos, Tight and compact MIP formulation of configuration-based combined-cycle units. *IEEE Trans. Power Syst.*, 31(2), 1350--1359, 2016.

<a id="ref6"></a>
<sup>[[6]](#ref6)</sup> L. Fan, K. Pan, and Y. Guan, A strengthened mixed-integer linear programming formulation for combined-cycle units. *EJOR*, 275(3), 865--881, 2019.
