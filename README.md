# Symmetry Reduction in Combined Cycle Unit Commitment

This repository presents a study of symmetry reduction applied to the Combined Cycle Min-Up/Min-Down Unit Commitment Problem (CC-MUCP). The CC-MUCP extends the classical unit commitment problem by explicitly modelling the internal structure of combined cycle power plants, where multiple gas turbines (GTs) and one steam turbine (ST) are organised into packages. When packages and GTs within each package are identical, the problem exhibits a two-level permutation symmetry that generates large equivalence classes of optimal solutions differing only in unit labeling.

The notebook develops the MILP formulation, characterizes the wreath product symmetry group $S_{n_{gt}} \wr S_K$, and applies a demand-aware lexicographic ordering to eliminate redundant solutions while retaining exactly one canonical representative per orbit. Solution enumeration with and without symmetry breaking is used to quantify the reduction in equivalent optimal schedules.

## Repository Structure
```
symmetry-reduction-ccmucp/
├── README.md
├── symmetry_reduction_ccmucp.ipynb
└── ccmucp_results.jld2
```

## Notebook Overview

**`symmetry_reduction_ccmucp.ipynb`** is organized into the following sections:

- **Theoretical Framework:** introduces combined cycle power plants, the Min-Up/Min-Down Unit Commitment Problem (MUCP), and the CC-MUCP as an extension that models the internal GT–ST coupling structure.
- **Mathematical Formulation:** presents the MILP formulation under the component-based approach, including GT and ST constraints, coupling constraints (CC1–CC5), and the global demand constraint.
- **Symmetry Structure:** characterizes the wreath product symmetry $S_{n_{gt}} \wr S_K$ and develops the demand-aware lexicographic ordering constraints that break symmetry at the GT and package levels.
- **Computational Study:** describes the implementation in JuMP, defines the two-package example instance, and enumerates all optimal schedules under three symmetry-breaking schemes (`:none`, `:gt_order`, `:plant_wide`).
- **Results Analysis:** presents the orbit decomposition of the 108 baseline solutions into 4 equivalence classes, interprets the stabilizer structure, and discusses the physical meaning of the four reference schedules.
- **Discussion:** connects the orbit structure to operational behavior and discusses the scope and limitations of the approach.

## Results File

The file `ccmucp_results.jld2` contains the precomputed enumeration results for the two-package instance ($K = 2$, $n_{gt} = 3$, $T = 21$). It stores the three solution sets obtained under each symmetry-breaking scheme, together with the instance parameters and demand ranking used to generate them:

| Variable | Description |
|---|---|
| `sols_base` | 108 optimal schedules (no symmetry breaking) |
| `sols_gt` | 8 optimal schedules (GT-level ordering) |
| `sols_full` | 4 optimal schedules (plant-wide ordering) |
| `best_cost` | Optimal cost: \$489,683 |
| `demand_rank` | Permutation $\pi$ ranking periods by decreasing demand |

Results can be loaded directly without re-running the enumeration:
```julia
using JLD2
results = load("ccmucp_results.jld2")
```

## Implementation

### Prerequisites

- [Julia 1.9](https://julialang.org/downloads/) or higher
- [Jupyter Notebook](https://jupyter.org/install) with [IJulia](https://github.com/JuliaLang/IJulia.jl)

### Required Julia Packages
```julia
using Pkg
Pkg.add(["JuMP", "HiGHS", "JLD2"])
```

### Modeling and Solver

Optimization models are formulated using [JuMP](https://jump.dev/), a domain-specific algebraic modeling language for mathematical optimization in Julia. Mixed-integer linear programs are solved using [HiGHS](https://highs.dev/), interfaced through [HiGHS.jl](https://github.com/jump-dev/HiGHS.jl). Enumeration results are saved and loaded using [JLD2.jl](https://github.com/JuliaIO/JLD2.jl).

### Running the Notebook

Launch Jupyter and open the notebook:
```bash
jupyter notebook symmetry_reduction_ccmucp.ipynb
```

The notebook can be run end-to-end to reproduce all results, or individual sections can be executed independently using the precomputed `ccmucp_results.jld2` file.

## Contributing

Contributions are welcome. Please open an issue or submit a pull request for suggestions, improvements, or bug fixes. You can also reach out via [Isaac Oliva-González's homepage](https://iolivag.github.io).

## License

This project is licensed under the MIT License.