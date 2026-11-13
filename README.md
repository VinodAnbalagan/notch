# Notch

**What should a robot remember?**

Notch compares the signals that could decide what a world-model planner writes into a small, fixed memory. One of them is new: whether an observation changed the plan.

Paper working title: *What Should a Robot Remember? Plan Change as a Write Signal for Lean Memory in World-Model Planning*

| | |
|---|---|
| Status | Step 01 in progress |
| Author | Vinod Anbalagan |
| Target | Preprint on arXiv, simulation demos, then a real-arm demo |
| Progress | Tracked step by step on the [Lab page](https://vinodanbalagan.vercel.app/lab) |
| Convention | Anything marked `[TBD: ...]` is unconfirmed. It is a placeholder, not a value. |

---

## 1. The idea in one paragraph

A robot that acts only on its current camera frame fails as soon as the frame stops showing what it needs: an object that is now hidden, a cue that has disappeared. Many ways of adding memory to robot policies now exist, from compressed video and language summaries to learned gates and memory slots (Section 3). Memory always has a price, and inside a sampling-based planner the price is multiplied: the world model reads its memory at every step of every imagined rollout, so the cost scales with the number of samples times the planning horizon. For a planner, keeping memory small matters more than for a policy.

A small memory forces a choice: **what gets written?** Notch studies that choice directly. On one trained world model, with one fixed memory budget, it compares the signals that could trigger a write:

1. **Time:** write at fixed intervals.
2. **Chance:** write at random.
3. **Surprise:** write when the world model's own prediction of the next state was wrong.
4. **Plan change (new):** write when the new observation made the planner change its plan.
5. **Surprise × plan change:** surprise weighted by whether it mattered.

**Why plan change.** A planner re-plans at every step, so it can check directly whether what it just saw changed what it intends to do. A policy has no plan to compare. A moving distractor is surprising but leaves the plan untouched; an object that is not where it should be changes the plan. Plan change is measured from the planner itself, with no labels, no extra network and no language model.

**What we do not assume:** that any of these signals equals importance. Each is a hypothesis. Section 2 states, for each kind of event, what each signal is expected to catch and miss, and the Signal Audit (Step 03) measures it before any memory system is built.

---

## 2. Research question and claims

**RQ.** Inside a sampling-based latent world-model planner, which signal should decide what is written into a small, fixed memory, and what does that memory cost in planning time?

The claims are separate and are reported separately. Success on one is never presented as success on another.

| ID | Claim | Tested by | What would falsify it |
|---|---|---|---|
| H1 | **The tasks need memory.** A memoryless planner fails on them and an oracle given the hidden state succeeds. | No-memory baseline vs oracle | The memoryless planner succeeds anyway (redesign the tasks) |
| H2 | **Efficiency.** A compact, fixed-size memory recovers most of the reliability of full history, and its planning-time advantage grows with episode length. | Compact memory vs full history vs recurrent state, across episode lengths | Compact memory is clearly less reliable at every budget, or there is no speed gap at any tested length |
| H3 | **Selection matters.** At the same write budget, a signal-based write rule beats writing by time or by chance. | Surprise and plan change vs periodic and random writes, same budget, same model weights | Periodic or random writes do as well |
| H4 | **Plan change.** Plan change, alone or combined with surprise, beats surprise alone, mainly by ignoring changes that do not matter. | Rules 3, 4 and 5 compared, including on distractor and appearance-change variants | Plan change is no better than surprise, including when distractors are present |
| H5 | **Limits.** Each signal catches and misses different kinds of events. | The Signal Audit, per event type | Every signal behaves the same across event types |

**Event types and what each signal is expected to do:**

| Event type | Example | Surprise | Plan change |
|---|---|---|---|
| Outside change that matters | The object is revealed, or moved by something else | Fires | Fires |
| Irrelevant novelty | A distractor moves or flickers | Fires (false alarm) | Silent |
| Appearance-only change | Recolouring, lighting | May fire: latent prediction error is known to respond to appearance (Hristov et al., 2026) | Silent |
| Self-caused progress | "I just finished item 3" | Silent, because the robot caused it | Probably silent: the plan advances as expected |

Self-caused progress is the expected blind spot of both signals. If the audit confirms it, the paper reports it as a limit and tests a separate progress trigger as an extension `[TBD: decided after Step 03]`.

---

## 3. Prior work and positioning

| Work | What it does | How Notch differs |
|---|---|---|
| **MEM** (Torne et al., 2026, Physical Intelligence) | Compressed video plus language summaries as memory for a VLA | A compact world-model planner; no video buffer, no language model |
| **Worth Remembering** (Gorlo, Wise, Speranzon, Carlone, 2026) | Surprise-gated episodic memory, used by a language-model agent to answer questions; no timing | Memory inside the control loop; surprise is one candidate among several; planning time is a primary result |
| **AURA-Mem** (Chen, 2026) | Surprise-informed learned write gate with constant memory, wrapped around a VLA; compares against budget-matched random and periodic writes | A planner, not a policy; relevance is measured from the plan, not learned; tasks require memory |
| **Gated Memory Policy** (Gao, Liu, Li, Song, 2026) | A learned gate that decides when a diffusion policy reads its history | Studies what is written, not when to read; planner setting |
| **MemWM** (Sun, Zhu, 2026) | Always-on learned memory slots added to LeWorldModel, on LeWM's original tasks | Tasks that need memory; an explicit write rule; planning time reported. Used as a baseline |
| **MemoWM** (Zeng et al., 2026) | Stores only what a world model cannot recover and what matters for future tasks; argues surprise alone is not enough; web-agent question answering, measured in storage | Closed-loop control; relevance comes from the planner online, not from offline estimates |
| **HiMem-WAM**, **Memory as Plans** (2026) | Memory gated on skill or segment boundaries in multi-billion-parameter world-action models | About 15M parameters; no segment labels |
| **Hristov, Slavov, Vasilev** (2026) | Shows latent prediction error in video world models is confounded by appearance, repetition and motion | Their controls are built into our Signal Audit |
| **Counterfactual Memory Audit** (Zhang, Xiang, Chen, 2026) | Shows that memory changing an action does not prove memory guided it | Their test is part of our evidence (Section 5.4) |
| **GateL0RD** (Gumbsch, Butz, Martius, 2021) | A recurrent latent state trained to update sparsely, for prediction and planning under partial observability | Used as a baseline style: learned sparse updates vs explicit write rules |
| **LeWorldModel** (Maes, Le Lidec, Scieur, LeCun, Balestriero, 2026) | Compact JEPA world model that plans from pixels | Our base model. Its benchmark tasks do not require memory |

Further background to verify before citing `[TBD: verify each citation]`: Schmidhuber's history compression (1992, store only what a predictor fails to predict), Titans (Behrouz et al., 2025, surprise-driven memory writes at test time), EM-LLM (Fountas et al., 2024, surprise-based event segmentation), and event segmentation theory in cognitive science (Zacks et al.).

**Positioning sentence (draft):** *Memory is being added to robot policies and world models in many ways, but the rule that decides what is written is usually fixed, learned end to end, or borrowed from surprise. We compare write signals on equal terms inside a sampling-based latent planner, introduce plan change as a signal a planner can measure about itself, and report reliability and planning time together.*

---

## 4. Method sketch

```
frame o_t ──► encoder ──► z_t ──┐
                                 ├──► predictor(z, a, memory) ──► ẑ_{t+1}
memory M (K slots) ─────────────┘
                                         │
planner (sampling-based MPC) ◄───────────┘

surprise     s_t = distance( ẑ_t , z_t )                   prediction made one step ago vs what happened
plan change  c_t = distance( plan before o_t , plan after o_t )
write rule:  time | chance | s_t | c_t | s_t × c_t         same budget for all
```

1. **Encoder and predictor:** LeWorldModel's, with the predictor extended to read K memory slots.
2. **Memory:** K fixed slots holding past latents `[TBD: K, start with 4 to 16]`. One eviction rule, shared by all write rules `[TBD]`.
3. **One model, many write rules.** The plan is to train the predictor with memory slots filled by random subsets of earlier latents from the same episode, so it learns to use whatever the memory holds. Every write rule is then evaluated on the same weights, which removes training as a confound `[TBD: confirm this training scheme works, Step 04]`.
4. **Plan change.** The planner already holds the plan it made one step ago. Plan change compares that plan with the one made after the new observation `[TBD: exact measure, action distance or cost difference; confirm the planner exposes both plans, Step 01]`. Sampling-based planners are noisy, so the measure must separate real plan changes from sampling noise (see Risks).
5. **Budget matching.** All write rules write the same average number of times per episode.

---

## 5. Experiments

### 5.1 Tasks

LeWorldModel plans toward a goal image, so the tasks are hidden-state tasks: the outcome of an action depends on something that is no longer visible.

| Group | Tasks | Purpose |
|---|---|---|
| Diagnostic | A Two-Room variant in which information needed to reach the goal is visible only early in the episode `[TBD: exact design]` | Cheap, fully controlled, ground-truth state for the audit |
| Benchmark | Two or three tasks from MIKASA-Robo (Cherepanov et al.), candidates: ShellGame, RememberColor, TakeItBack `[TBD: confirm they fit goal-image planning and that LeWorldModel trains on their data, Step 01]` | Standard, externally defined memory tasks |
| Variants | Each task with a distractor, and with appearance-only changes | False alarms (H4) |
| Control | Original LeWorldModel tasks | Memory must not hurt tasks that do not need it |

### 5.2 Baselines

- No memory (plain LeWorldModel)
- Frame stacking (last k latents)
- Recurrent state in the predictor
- Sparsely updating recurrent state (GateL0RD style)
- Always-on learned slots (MemWM style)
- Full history (all past latents, attention)
- Oracle memory (ground-truth hidden state), as an upper bound

### 5.3 Metrics

- Task success rate, with confidence intervals
- Planning time per decision (ms) and rate (Hz), on stated hardware, across episode lengths
- Memory footprint (slots, bytes) and writes per episode
- **Signal audit:** for each signal and each event type, how often it fires on labelled important moments (recall) and on unimportant ones (false-alarm rate)

### 5.4 Controls and evidence

- **Appearance controls in the audit:** recolouring, repeated frames and wrong actions, following Hristov et al., to see what actually moves each signal.
- **Counterfactual memory check:** hold the present state identical, swap the memory contents, and test whether the planner chooses the branch the memory warrants, following Zhang et al. A memory that changes behaviour without guiding it does not count.
- **Episode-length sweep:** the speed claim (H2) is tested where it could matter, at hundreds to thousands of steps `[TBD: range]`.
- **Statistics:** 5 seeds; enough evaluation episodes per condition to resolve a 10-point difference in success rate (about 400) `[TBD: final numbers]`; intervals reported for every headline number.
- **Reproducibility:** hypotheses in this file are tagged in git before any result exists; every table and figure is regenerated from logged runs by one command; code, configs and logs are released with the preprint.

### 5.5 Ablations

- Memory size K
- Thresholds for each signal
- How plan change is measured (actions vs cost)
- Where memory is read (in the predictor vs only in the cost)

---

## 6. Demos

1. **Diagnostic, in simulation:** the memoryless planner beside Notch on the same episode, with the live signal traces and memory slots overlaid from the real model.
2. **Manipulation, in simulation:** a MIKASA-Robo shell game with a simulated arm, same overlay.
3. **Real arm:** the shell game on a physical arm with a camera `[TBD: arm]`. Planning with a world model on a real arm needs teleoperated data; the fallback is to run the Signal Audit on real footage.

---

## 7. Code, compute and hardware

- **Base code:** LeWorldModel (`lucas-maes/le-wm`), MIT licence, PyTorch, built on `stable-worldmodel`. Environments: two-room, pusht, cube, reacher. Datasets and pretrained checkpoints are on Hugging Face (`quentinll/lewm-*`).
- **Model size:** about 15M parameters per the paper's abstract `[TBD: confirm; another paper's table lists 18M]`.
- **GPU:** rented cloud GPU. LeWorldModel reports training on a single GPU in a few hours `[TBD: confirm once Step 01 is timed]`.
- **Benchmark:** MIKASA-Robo, MIT licence, built on ManiSkill3, with released expert trajectories.

---

## 8. The Lab: six steps to the paper

Each step is one entry on the website's Lab page, and each is broken into parts that are done one at a time. Status values match the site: `building`, `next`, `planned`, `shipped`. The first three steps check assumptions before any memory system is built: the base model works, the tasks really need memory, and the write signals mark the moments that matter.

| Step | Title | Question | Done when | Status |
|---|---|---|---|---|
| 01 | The Baseline | Does LeWorldModel work for us, and can its planner tell us when its plan changes? | Success rate and planning time per decision measured on our hardware; the `[TBD]`s in Sections 4, 5.1 and 7 resolved | building |
| 02 | Tasks That Need Memory | Do our tasks really need memory? | Memoryless planner fails and oracle succeeds (H1), on the diagnostic task and at least one benchmark task | next |
| 03 | The Signal Audit | Before building memory: which signal marks the moments worth remembering? | **Go/no-go:** recall and false-alarm rate per signal and event type, with the plan-change noise floor known | planned |
| 04 | Remembering Everything | What do the expensive memories achieve, and at what price? | Memory-reading predictor trained; baseline table filled | planned |
| 05 | The Write Rule | With the same model and the same budget, which write rule fills a small memory best, and does the memory guide the plan? | H3 and H4 answered separately; counterfactual check passed or reported | planned |
| 06 | The Price of Memory | How do reliability and planning time trade off as episodes get longer? | Main results table, trade-off plot (H2), ablation table | planned |
| Paper | Notch | | Preprint live, with code, logs and both simulation demos | planned |

### Parts of each step

**01 The Baseline**
1. Read the LeWorldModel paper; write a reading note in `notes/`.
2. Set up the environment on a cloud GPU; download the Two-Room dataset and checkpoint.
3. Evaluate the pretrained checkpoint: success rate and planning time per decision.
4. Retrain Two-Room from scratch and compare with the released checkpoint and the paper.
5. Read the planner code: where the plan lives between steps, and whether the plans before and after a new observation can both be read out.
6. Check whether MIKASA-Robo tasks fit goal-image planning and LeWorldModel's data format.

**02 Tasks That Need Memory**
1. Build the diagnostic task in Two-Room: information needed for the goal is visible only early in the episode. Log the ground-truth hidden state at every step.
2. Generate training data for it and train LeWorldModel on it.
3. Run the memoryless planner and the oracle (H1 on the diagnostic task).
4. Port one MIKASA-Robo task (first choice: ShellGame) and repeat parts 2 and 3.
5. Add distractor and appearance-only variants of the diagnostic task, for Step 03.

**03 The Signal Audit**
1. Log surprise and plan change at every step of planner rollouts.
2. Measure the plan-change noise floor: replan many times from an identical state and see how much the plan moves by chance.
3. Label moments by event type: outside change that matters, irrelevant novelty, appearance-only change, self-caused progress.
4. Compute recall and false-alarm rate for each signal and event type.
5. Run the appearance controls: recolouring, repeated frames, wrong actions.
6. Decide: go or no-go, and whether a progress trigger is needed.

**04 Remembering Everything**
1. Extend the predictor to read K memory slots.
2. Train it with slots filled by random subsets of earlier latents, so it can use whatever the memory holds; check that it does.
3. Run the baselines: frame stacking, recurrent state, sparsely updating recurrent state, always-on learned slots, full history, oracle.

**05 The Write Rule**
1. Implement the five write rules: time, chance, surprise, plan change, surprise × plan change.
2. Match budgets: every rule writes the same average number of times per episode.
3. Evaluate all five on the same weights (H3, H4).
4. Run the counterfactual memory check: identical present state, swapped memory, does the planner choose the branch the memory warrants?

**06 The Price of Memory**
1. Measure planning time per decision against episode length for every memory variant.
2. Full runs: 5 seeds, enough episodes per condition for confidence intervals, on the diagnostic and benchmark tasks.
3. Ablations: memory size K, thresholds, how plan change is measured, where memory is read.

**Paper**
1. Generate every table and figure from logged runs with one command.
2. Record the two simulation demos from real runs: the diagnostic task and the shell game, memoryless planner beside Notch, with live signal traces and memory slots.
3. Write; outside review; release code, configs and logs; arXiv.

After the paper: the real-arm demo.

---

## 9. Risks and fallbacks

| Risk | Fallback |
|---|---|
| Plan change is noisy, because sampling-based planners give different plans from run to run | Compare plans under shared random numbers, or compare costs instead of actions; measure the noise floor in Step 03 before relying on the signal |
| Neither signal marks the moments that matter (Step 03) | The audit itself is the result: a controlled account of where each signal fails. Test a progress trigger |
| LeWorldModel does not train well on the benchmark tasks | Keep the diagnostic tasks in LeWorldModel's own environments as the core; report the benchmark attempt honestly |
| No speed gap at realistic episode lengths | Report it. The selection results (H3, H4) stand on their own |
| The field moves quickly | Re-check the literature at the start of every Lab step |
| Scope grows | Core is the diagnostic task, one benchmark task, H1 to H4. Drop ablations and extra tasks first |

---

## 10. Open decisions

- [ ] Exact measure of plan change (after Step 01)
- [ ] Diagnostic task design and final benchmark tasks (after Steps 01 and 02)
- [ ] Memory size K and eviction rule
- [ ] Whether a progress trigger is needed (after Step 03)
- [ ] Robot arm for the demo
- [ ] Venue after arXiv `[TBD]`

---

## 11. Later directions (not in this paper)

- **Another body:** the same memory on a simulated drone, unchanged.
- **Cross-episode memory:** distilled lessons kept in a compact graph and recalled only when needed.
- **Environmental memory:** the agent leaves itself traces in the scene (see *Artifacts as Memory Beyond the Agent Boundary*, Martin et al., 2026).
- **Leader-follower control** of several robots through a learned world model.

---

## 12. Repository layout (planned)

```
notch/
├── README.md        this file
├── notes/           one reading note per paper
├── envs/            memory-requiring task variants
├── src/             memory module, write rules, planner glue
├── experiments/     configs and run scripts
├── results/         logs, tables, figures
└── paper/           LaTeX draft
```

---

## 13. References

1. Maes, Le Lidec, Scieur, LeCun, Balestriero. *LeWorldModel: Stable End-to-End Joint-Embedding Predictive Architecture from Pixels.* 2026. https://arxiv.org/abs/2603.19312 (code: https://github.com/lucas-maes/le-wm)
2. Torne, Pertsch, et al. *MEM: Multi-Scale Embodied Memory for Vision Language Action Models.* 2026. https://arxiv.org/abs/2603.03596
3. Gorlo, Wise, Speranzon, Carlone. *Worth Remembering: Surprise-Gated Robot Episodic Memory.* 2026. https://arxiv.org/abs/2606.03787
4. Chen. *AURA: Action-Gated Memory for Robot Policies at Constant VRAM.* 2026. https://arxiv.org/abs/2606.02775
5. Gao, Liu, Li, Song. *Gated Memory Policy: In-Context Memorization and Adaptation.* 2026. https://arxiv.org/abs/2604.18933
6. Sun, Zhu. *Enhancing Latent World Models with Associative Memory (MemWM).* 2026. https://arxiv.org/abs/2610.19683
7. Zeng et al. *MemoWM: How World Models Change What Agents Need to Remember.* 2026. https://arxiv.org/abs/2610.10778
8. Sun et al. *HiMem-WAM: Hierarchical Memory-Gated World Action Models for Robotic Manipulation.* 2026. https://arxiv.org/abs/2606.10363
9. Zhao et al. *Memory as Plans: World-Action Modeling with Memory-Grounded Planning.* 2026. https://arxiv.org/abs/2609.11561
10. Hristov, Slavov, Vasilev. *When Does Latent Prediction Error Support Physical Surprise Claims in Video World Models?* 2026. https://arxiv.org/abs/2611.01815
11. Zhang, Xiang, Chen. *Memory That Changes Action Is Not Memory That Guides It: Counterfactual Auditing of History-Conditioned Robot Policies.* 2026. https://arxiv.org/abs/2609.27247
12. Gumbsch, Butz, Martius. *Sparsely Changing Latent States for Prediction and Planning in Partially Observable Domains.* 2021. https://arxiv.org/abs/2110.15949
13. Cherepanov, Kachaev, Kovalev, Panov. *Memory, Benchmark & Robots: A Benchmark for Solving Complex Tasks with Reinforcement Learning (MIKASA).* https://arxiv.org/abs/2502.10550
14. Martin et al. *Artifacts as Memory Beyond the Agent Boundary.* 2026. https://arxiv.org/abs/2604.08756
