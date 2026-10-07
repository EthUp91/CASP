# Demand-Dependent Operating Ranges of Traffic-Signal Controllers at Gongguan Roundabout

**Authors:** Anonymous draft; replace with the submission author list.

**Abstract:** We stress-test Fixed-Time, DQN, PPO and recurrent PPO at Gongguan Roundabout across eight demand levels and five matched traffic seeds. Published checkpoints were unavailable, so source-based replacement policies were trained with documented implementation adaptations. Mean completion first falls below 90% at 130% demand for DQN/PPO, 150% for RPPO and 160% for Fixed-Time. At 170%, RPPO has the highest mean completion, whereas PPO has lower conditional delay. Rankings depend on demand and metric; five-seed uncertainty prevents a universal superiority claim. Completion accounting exposes degradation obscured by completed-trip delay alone.

**Keywords:** traffic signal control; reinforcement learning; demand robustness; SUMO; trip completion

## 1. Introduction

Traffic-signal controllers are often compared at selected demand conditions. A favorable result at one high-demand point does not establish how performance degrades as demand continues to rise. It also does not establish whether a controller serves the requested demand: travel-time averages can omit vehicles that remain unfinished or never enter the network.

The Gongguan Roundabout study, *A Deep Reinforcement Learning Approach to Roundabout Traffic Signal Control* [1], compared Fixed-Time and reinforcement-learning controllers using aggregate traffic metrics, including a 130% high-demand scenario and an accident scenario. We reuse its supplied SUMO network and controller code to ask: **How do controller operating ranges and degradation patterns change as demand increases beyond the source training condition?** The contribution is a systematic demand-response characterization, not a new learning algorithm.

We evaluate eight demand levels with matched traffic seeds and report completion alongside delay and queues. Practical warnings identify where mean completion drops below 90% or duration doubles relative to a controller's own baseline. These warnings describe a tested operating region; they are not estimates of physical capacity. The study also makes model provenance explicit. Because published checkpoints were absent, the results concern newly trained source-based policies rather than exact numerical reproduction of the publication.

## 2. Experimental setup and stress-test methodology

### 2.1 Reused system and policy provenance

The supplied Gongguan network is simulated in SUMO 1.22.0 [2]. The original 26-component observation, three green-selection actions, signal definitions and reward are preserved. Fixed-Time retains the source's 30-second cycling; RL retains its 15-second minimum hold and 3-second yellow handling. Thus source-specific timing mechanisms are retained rather than replaced by a new baseline. The repository label “DDQN” constructs plain Stable-Baselines3 DQN [3], not Double DQN; we therefore use **DQN** throughout.

Each replacement policy receives 512,000 transitions with training seed 42 at 130% demand, without an accident. DQN uses a 256-unit hidden layer; PPO has separate 128-unit actor/critic heads; RPPO has separate 512-unit LSTMs and 256-unit heads. Constructor settings and source callback code are retained. All policies normalize observations. PPO/RPPO also normalize training rewards; RPPO freezes normalization at 150,000 transitions. Full settings and artifacts accompany this paper.

Necessary adaptations are CPU execution instead of unavailable CUDA, portable paths, and four independent SUMO processes for PPO/RPPO. The original four-environment code shared one unlabelled TraCI connection; a diagnostic vector step advanced that single simulation four seconds rather than one second per independent simulation. Independent workers correct this orchestration defect without changing the architecture or reward. In-process libsumo accelerates execution; representative seeded Fixed-Time and PPO trajectories match socket TraCI in trip records and aggregate metrics. These adaptations and uncertain historical random streams preclude an exact published-weight reproduction claim.

### 2.2 Matched demand sweep and accounting

Demand scaling is 100, 110, 120, 130, 140, 150, 160 and 170% of the supplied generator baseline. The same trained policy is frozen across all levels. Seeds 42–46 generate matched demand files and simulator randomness across controllers; scaling retains flow composition and the same per-flow noise draws. The main experiment comprises 160 episodes, including the 40 existing Fixed-Time runs. Evaluation uses deterministic inference, frozen observation normalization and no reward normalization. No additional training occurs during evaluation.

The existing horizon and termination convention are retained: one-second steps stop at 3601 seconds, with no warm-up exclusion. Completion is arrivals divided by all loaded demand, including vehicles unable to depart. Incomplete demand includes both unfinished departed vehicles and undeparted vehicles. SUMO tripinfo duration, waiting time and time loss are averaged only over completed trips. Queue is the source's time-average sum of halted vehicles on designated approaches; it excludes undeparted demand. These definitions are identical across controllers. Raw records, hashes and arrival accounting are independently checked.

### 2.3 Warnings and uncertainty

For each controller, the duration warning uses the mean of five paired ratios between a demand level and the same seed at 100%; the criterion is greater than two. The completion criterion is a mean below 90%. The first crossing is a practical warning; the preceding tested level is the last grid point before that warning. Threshold-overlapping intervals indicate uncertainty. Adjacent queue changes describe congestion growth without fitting a critical point.

Means, standard deviations and Student-t 95% intervals use five seed-level results, not individual vehicles. Pairing is retained for differences and baseline changes. Three selected two-sided completion contrasts compare RPPO with Fixed-Time at 160/170% and with PPO at 170%; paired-t p-values receive Holm adjustment. Exact paired sign-flip sensitivity enumerates 32 assignments. With five pairs its minimum two-sided p-value is 0.0625. Inference is conditional on one trained policy per algorithm and assumptions on paired differences; the traffic seeds are not claimed to be held-out training streams.

## 3. Results

### 3.1 Operating ranges and degradation

Table 1 summarizes practical warnings and selected operating points. Fixed-Time retains mean completion above 90% through 150%, RPPO through 140%, and DQN/PPO through 120%. At their first crossings, DQN's and RPPO's completion intervals lie entirely below 90%. PPO's 130% interval, 88.62–91.23%, overlaps the criterion. Fixed-Time's 160% completion interval also overlaps 90%, becoming wholly lower only at 170%. Its duration-ratio interval overlaps two; DQN's mean duration ratio crosses two at 150%, becoming wholly above two at 160%. Neither PPO nor RPPO reaches the duration-ratio warning within the tested grid.

Figure 1 shows why duration alone is insufficient. PPO starts with a relatively high 100% duration, 97.45 seconds, versus 76.74 seconds for RPPO and 78.58 seconds for Fixed-Time. Its modest proportional duration growth nevertheless accompanies a declining completion ratio. Normalized duration therefore measures relative deterioration, not absolute quality; it is retained as supplementary evidence.

Figure 2 identifies distinct degradation patterns. DQN completion falls from 92.89% to 82.00% between 120/130%, then remains near 77–79%. PPO declines more smoothly. RPPO declines from 93.08% at 130% to 82.65% at 170%. Fixed-Time instead falls from 92.64% at 150% to 81.88% at 160% and 55.11% at 170%. Its queue rises from 53.91 to 83.21 to 140.20 vehicles over those levels (Figure 3). The 150–170% region is consistent with an abrupt congestion transition, but a sparse grid and large seed variability do not establish a unique tipping point.

### 3.2 Demand- and metric-dependent rankings

Fixed-Time is competitive at moderate demand. At 130%, durations are 108.67 seconds for Fixed-Time, 152.73 for DQN, 107.70 for PPO and 110.06 for RPPO. Completion is respectively 95.34%, 82.00%, 89.92% and 93.08%. RPPO's paired duration difference from Fixed-Time is 1.39 seconds (95% interval −0.99 to 3.77), but its completion difference is −2.26 percentage points (−2.96 to −1.56). A similar duration therefore does not imply equal service.

At 170%, RPPO has the highest mean completion, 82.65%, while PPO has lower mean completed-trip duration, 128.74 versus 137.25 seconds, and lower queue, 51.40 versus 63.62 vehicles. The paired duration difference has an interval spanning zero (−0.65 to 17.67 seconds); the paired queue difference is 12.23 vehicles (5.19–19.26). PPO's lower numerical delay coexists with lower completion, not demonstrated faster service for all demanded vehicles.

DQN is not uniformly inferior. At 100% its duration is lower than PPO's, and its queue is slightly lower than Fixed-Time's. At 120–150%, however, it deteriorates strongly. Its 170% completion exceeds Fixed-Time numerically despite higher completed-trip duration. No controller dominates every demand/metric combination.

### 3.3 Selected paired inference

RPPO-minus-Fixed-Time completion differences are 3.59 percentage points at 160% (95% interval −8.03 to 15.20; Holm-adjusted paired-t p=0.440) and 27.53 at 170% (−4.58 to 59.65; p=0.152). Thus RPPO's substantial extreme-demand mean advantage is not a clear five-seed completion advantage over Fixed-Time.

RPPO-minus-PPO completion at 170% is 5.01 points (3.76–6.26; Holm-adjusted paired-t p=0.0011). This is exploratory parametric support for these fixed policies. The exact sign-flip p-value is 0.0625, illustrating the coarse resolution of five seed blocks. We therefore distinguish a consistent observed difference from a distribution-robust superiority claim and make no general algorithm-ranking claim.

## 4. Discussion

### 4.1 Practical implications

The longest pre-warning range and the highest extreme-demand completion need not identify the same controller. Fixed-Time has the latest mean warning, while RPPO retains more average service after Fixed-Time's collapse region. At 170%, PPO has lower conditional delay and queues than RPPO but completes less demand. Controller selection should therefore specify both the demand range and the operational objective. A 90% completion warning is a service criterion, not a universal safety or capacity threshold.

Censoring is central. Fixed-Time duration decreases between 160/170% even as completion deteriorates sharply. Slow unfinished trips are absent from completed-trip averages, and vehicles unable to enter the network are absent from approach queues. These observations explain why smaller delays or measured queues cannot alone establish robustness. They do not identify which individual vehicles would finish faster under another controller.

### 4.2 Relationship to the previous study and limitations

The new evidence is the matched demand-response sweep, accounting and warning uncertainty, rather than the previous paper's aggregate scenario results. The supplied original training logs show a strong RPPO advantage that the new 130% evaluation does not reproduce: RPPO's exit-lane travel proxy changes from 49.76 seconds in the last five logged training episodes to 91.48 seconds in deterministic evaluation; Fixed-Time changes from 119.18 to 89.92 seconds. These are like-named proxy comparisons, not comparisons with full-trip duration. Training episodes, counters and published-policy provenance differ. A two-seed DQN diagnostic produces identical results with its training and common evaluation rerouting configurations, so that checked difference does not explain those discrepancies. No verified cause or exact contradiction of the publication's evaluation table is asserted; the numerical table and original artifacts were not supplied.

The study concerns one newly trained policy per algorithm, one network, five traffic seeds and a finite horizon. Independent worker seeding, corrected vectorization and CPU execution differ from the unrecoverable historical setup. Marginal t intervals and selected tests depend on small-sample assumptions; they exclude training-seed variability. Fixed-Time and RL also retain distinct source timing mechanisms and switch-penalty accounting. Results support this source-based operational case study, not exact replication or general algorithm superiority. Bibliographic details of [1] must be completed before submission.

## 5. Conclusion

A systematic demand sweep exposes operating tradeoffs hidden by a single high-demand scenario. Fixed-Time remains competitive through moderate demand, whereas the newly trained RPPO has the highest average completion at the extreme tested level. PPO's lower conditional delay does not establish greater total-demand service. The observed warnings and uncertainty support demand- and metric-dependent controller selection, not a universal robustness winner.

## Figure and table captions

**Figure 1.** Completed-trip duration versus demand. Lines are means of five seed-level means; shading is marginal Student-t 95% confidence intervals. Unfinished and undeparted demand is excluded from duration, so interpret with Figure 2.

**Figure 2.** Completed trips as a percentage of all loaded demand, including undeparted vehicles. Shading denotes five-seed 95% intervals; the dashed line marks the descriptive 90% warning criterion.

**Figure 3.** Time-averaged halted vehicles on the source's designated approach lanes. Shading denotes five-seed 95% intervals. Queues do not include undeparted demand.

**Table 1.** First mean warning and selected operating points. Warning means completion below 90% or paired duration ratio above two. PPO's 130% and Fixed-Time's 160% first-warning intervals overlap their criteria. Duration is conditional on completion.

## References

[1] *A Deep Reinforcement Learning Approach to Roundabout Traffic Signal Control.* Bibliographic details pending verification: authors, year, venue and identifier were not supplied with the source archive.

[2] P. A. Lopez et al., “Microscopic Traffic Simulation using SUMO,” *2018 IEEE International Conference on Intelligent Transportation Systems*, pp. 2575–2582, 2018.

[3] A. Raffin et al., “Stable-Baselines3: Reliable Reinforcement Learning Implementations,” *Journal of Machine Learning Research*, vol. 22, no. 268, pp. 1–8, 2021.
