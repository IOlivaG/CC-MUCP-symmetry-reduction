# Symmetry Reduction in Combined Cycle Unit Commitment

This repository looks at symmetry reduction for the Combined Cycle Min-Up/Min-Down Unit Commitment Problem (CC-MUCP). The CC-MUCP extends the classical unit commitment problem by modelling the internal structure of combined cycle power plants, where multiple gas turbines (GTs) and one steam turbine (ST) are organised into packages.

When GTs within a package are identical, the problem exhibits permutation symmetry that creates multiple equivalent optimal solutions differing only in unit labelling. The notebook explores a demand-aware lexicographic ordering to reduce this redundancy.

## Repository Structure

```
├── ccmucp_formulation.md              — Mathematical formulation and instance data
├── ccmucp_symmetry_reduction.ipynb     — Jupyter notebook with the MILP model
└── results/
    └── ccmucp_results.jld2            — Precomputed enumeration results
```

## Notebook Overview

**`ccmucp_symmetry_reduction.ipynb`** covers:

- **Mathematical Formulation:** MILP formulation under a component-based approach, with GT and ST constraints, coupling constraints (CC1–CC5), and the global demand constraint. The full formulation is documented in `ccmucp_formulation.md`.
- **Symmetry Structure:** intra-package permutation symmetry and a demand-aware lexicographic ordering to break it.
- **Computational Study:** implementation in JuMP, enumeration of optimal schedules, and analysis of the resulting orbits.
- **Results:** comparison of baseline enumeration (108 solutions) against the symmetry-reduced enumeration.

## Results File

The file `results/ccmucp_results.jld2` contains precomputed results for the two-package instance ($K = 2$, $n_{gt} = 3$, $T = 21$):

| Variable | Description |
|---|---|
| `sols_base` | 108 optimal schedules (no symmetry breaking) |
| `sols_sym` | ~8 optimal schedules (with symmetry breaking) |
| `best_cost` | Optimal cost: \$489,683 |
| `demand_rank` | Permutation ranking periods by decreasing demand |

Results can be loaded without re-running the notebook:
```julia
using JLD2
results = load("results/ccmucp_results.jld2")
```

## Requirements

- Julia 1.9 or higher
- Packages: JuMP, HiGHS, JLD2

Install with:
```julia
using Pkg
Pkg.add(["JuMP", "HiGHS", "JLD2"])
```

## Running the Notebook

```bash
jupyter notebook ccmucp_symmetry_reduction.ipynb
```

The notebook can be run from start to finish, or individual sections can be explored independently using the precomputed results file.

## References

1. R. Kehlhofer, J. Warner, H. Nielsen, T. Bachmann. *Combined-Cycle Gas and Steam Turbine Power Plants*, 3rd ed., PennWell, 2009.
2. D. Rajan, S. Takriti. Minimum up/down polytopes of the unit commitment problem with start-up costs. IBM Research Report RC23628, 2005.
3. P. Bendotti, P. Fouilhoux, C. Rottner. The min-up/min-down unit commitment polytope. *J. Comb. Optim.*, 36(3), 1024--1058, 2018.
4. C. Liu, M. Shahidehpour, Z. Li, L. Wu. Component and mode models for the short-term scheduling of combined-cycle units. *IEEE Trans. Power Syst.*, 24(2), 976--990, 2009.
5. G. Morales-Espa\~na, C. M. Correa-Posada, A. Ramos. Tight and compact MIP formulation of configuration-based combined-cycle units. *IEEE Trans. Power Syst.*, 31(2), 1350--1359, 2016.
6. L. Fan, K. Pan, Y. Guan. A strengthened mixed-integer linear programming formulation for combined-cycle units. *EJOR*, 275(3), 865--881, 2019.
