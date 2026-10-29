# Changelog

## 2026-10-28

Model and documentation corrections:

- Added standard upper bounds for GT and ST start-up variables (`u <= x`, `u <= 1 - x_{t-1}`).
- Reformulated CC1 to count GTs in combined-cycle mode explicitly.
- Reformulated CC4 with warm-up auxiliary variables `w`, requiring at least `m_k` GTs to be on during the full warm-up window and remain on at `t`.
- Relaxed CC5 to require at least `m_k` online GTs after ST shutdown (instead of all GTs).
- Enforced the demand balance as an equality (no-curtailment assumption).
- Replaced prefix-sum symmetry breaking with a superincreasing-weight lexicographic ordering.
- Corrected the horizon description: 21 load-level periods (7 days × 3 load levels), not hourly periods.
- Corrected demand statistics (average 660.0 MW; 5 periods below 500 MW).
- Regenerated `results/ccmucp_results.jld2`: 108 commitment patterns, 8 representatives, optimal cost 489,683.0.
