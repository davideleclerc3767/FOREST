# FOREST

**Give it a research question. Turn it into ideas, real experiments, evidence, and a complete paper.**

FOREST is an end-to-end research agent framework for carrying a scientific project from an early question to a finished manuscript.

It can explore and compare research directions, search the literature, turn promising ideas into testable hypotheses, design experiments, run real code on your compute, analyze the results, and decide what to try next. As the project develops, FOREST can revise its plan, abandon weak directions, branch into alternatives, and build the figures, tables, and manuscript around the evidence that survives.

Instead of keeping the research process inside a long chat history, FOREST represents it as an editable research graph. Ideas, hypotheses, experiments, results, decisions, and paper sections remain connected. You can inspect the graph, change a plan, replace a hypothesis, branch from an earlier point, or redirect the research without starting over.

**Idea discovery → literature review → hypothesis formation → experiment design → real execution → result analysis → research-path revision → baselines & ablations → figures & tables → full-paper generation**


## What FOREST does

**Finds research directions.**  
Start with a broad question or an unfinished idea. FOREST explores possible directions, compares them, checks the surrounding literature, and turns the strongest candidates into concrete research paths.

**Turns ideas into experiments.**  
Hypotheses are converted into executable plans with baselines, controls, metrics, ablations, and explicit conditions for success or failure.

**Runs real experiments.**  
FOREST writes and executes research code, launches training and evaluation jobs, collects logs and metrics, and keeps experimental artifacts attached to the research path that produced them.

**Analyzes the evidence and changes course.**  
Results are not treated as the end of a task. FOREST interprets them, compares competing explanations, identifies what failed, and decides whether to extend, revise, or drop a direction.

**Keeps the research path editable.**  
Researchers can intervene at any point: edit a node, change an experiment, revisit an earlier assumption, or branch the graph to investigate an alternative approach without losing the existing work.

**Continues across execution rounds.**
Agents save durable checkpoints at bounded work-round boundaries and automatically continue the same research run. Cumulative turns and planning cycles do not determine scientific lifetime. Recovery uses the retained goal, graph, evidence and action receipts; all rounds and branches share resource accounting. Actual spending, compute limits, permissions and unresolved blockers still govern execution. See [research execution and recovery](docs/RUNTIME.md).

**Builds scientific figures and tables.**  
Experimental results can be turned into publication-ready visualizations and tables while remaining tied to their underlying runs and observations.

**Writes the paper.**  
FOREST carries the surviving evidence into a full manuscript, including related work, methods, experiments, results, figures, tables, and discussion. Metrics, figures, and scientific claims remain traceable to the evidence used to produce them.

---

## Quick Start

Requires **Python 3.11+**. The Web workspace also requires **Node.js 22+**.

```bash
git clone https://github.com/mikamikasuki/FOREST.git
cd FOREST
```

### Web Workspace — macOS / Linux

Install dependencies and start FOREST:

```bash
python3.11 scripts/install.py
.venv/bin/forest serve
```

Open [localhost:8000](http://127.0.0.1:8000/), configure a model provider in **Settings**, and create a project with a research goal and an explicit spending budget.

### CLI / Headless

Run the API and worker without Node.js or the Web build:

```bash
python3.11 scripts/install.py --headless
.venv/bin/forest serve --headless --data-dir ./var
```

Keep the service running and use another terminal for client commands. Run these commands from the repository root:

```bash
.venv/bin/forest --help
.venv/bin/forest init --name "My research" --goal "Your research question"
.venv/bin/forest status
```

The CLI supports project and node management, experiment execution, logs, run control, evidence inspection, budgets, and manuscript tasks. It shares the same research state as the Web workspace.

See the [CLI guide](https://github.com/mikamikasuki/FOREST/blob/main/docs/CLI.md) or try the [numerical experiment example](https://github.com/mikamikasuki/FOREST/blob/main/examples/cli/README.md) without model calls.

### Repository Inputs and Diagnostics

Prepare a public GitHub repository in the selected project:

```bash
.venv/bin/forest repo clone https://github.com/OWNER/REPOSITORY.git --wait
```

Use the returned run ID to inspect its source commit and execution diagnostics:

```bash
.venv/bin/forest repo inspect RUN_ID
.venv/bin/forest run diagnostics RUN_ID
```

Git must be installed on the worker; SSH access also requires OpenSSH. Private repositories use worker-side HTTPS or SSH credential profiles.

Agent, command, and experiment tasks can prepare their own source checkout through a `repository` configuration. See [Repository Inputs and Authentication](https://github.com/mikamikasuki/FOREST/blob/main/docs/GITHUB_REPOSITORIES.md) for configuration and examples.

### Development

After installing the Web workspace:

```bash
# Start development services
./scripts/dev.sh

# Check the local environment
.venv/bin/python scripts/doctor.py

# Run backend tests, frontend compilation, and frontend tests
./scripts/test.sh
```

FOREST approaches scientific research as a **continuous, evidence-driven process rather than a sequence of isolated experiments**. Its persistent Research Graph connects hypotheses, experimental results, and research decisions, enabling agents to iteratively explore, evaluate, and refine scientific solutions. We evaluated this approach using **GPT-6 Luna (Medium)** as the research model, demonstrating autonomous experimentation and measurable scientific improvements across multiple research domains.

| Research Task | Dataset | Metric | Baseline | FOREST | Improvement |
|---|---|---|---:|---:|---:|
| PDE Gradient Optimization | Darcy64 | Relative Gradient L2 | 0.303645 | **0.256810** | **15.42% ** |
| Time-Series Forecasting (Study 1) | ETTh1 | MAE | 2.914060 | **2.553352** | **12.38% ** |
| PDE Solution Optimization | Darcy64 | Relative L2 | 0.108735 | **0.099608** | **8.39% ** |
| Time-Series Forecasting (Study 2) | ETTh1 | MAE | 2.913240 | **2.728309** | **6.35% ** |
| Medical Image Classification | BloodMNIST | Macro-F1 | 0.965410 | **0.969351** | **+0.394 pp** |

### Research Modes

- **Manual** — edit the research graph and choose tasks to execute.
- **Assisted** — use model-assisted planning and tools while directing the research.
- **Auto** — enable autonomous planning and execution within configured goals, budgets, and delivery requirements.

Saved runs retain decisions, logs, metrics, artifacts, and execution history for inspection. See [Workflow interventions](docs/WORKFLOW_INTERVENTIONS.md) for scoped instructions, durable action review, branch controls, and evidence acceptance.

### Paired Statistics Input Provenance

For paired statistics submitted to `POST /api/projects/{id}/statistics/paired`,
the worker retains the exact CSV bytes it parses as
`runs/{run_id}/paired-input.csv`. The run metrics and its Analysis record include
an `input_provenance` object with that project-relative artifact path, the
original project-relative `source_path`, SHA-256, and byte size. The Analysis
`path` points to the retained input, and `metrics_path` points to that run's
metrics artifact.

Read the run through `GET /api/runs/{run_id}` and fetch the retained bytes with
`GET /api/projects/{id}/download?path=runs/{run_id}/paired-input.csv`. Uploading
a correction at the original source path still marks analyses and authored
claims that depend on that path as needing an update; it does not rebind their
saved run, metrics, or retained input. Starting another paired run creates a
distinct run-local input artifact. This records bytes at worker consumption,
not at enqueue time, and is not a general file-version history.

PDF compilation requires `tectonic` or `pdflatex`. Configure spending and execution limits before unattended runs, and keep credentials outside the repository.

See [Deployment](https://github.com/mikamikasuki/FOREST/blob/main/docs/DEPLOYMENT.md) for Docker, PostgreSQL, background services, backups, and Windows via WSL2 or Docker Desktop. See [Security](https://github.com/mikamikasuki/FOREST/blob/main/SECURITY.md) for execution boundaries.

### Runtime Notes

- Prefer a remote model API when experiments need the local GPU. Local model inference and long attention caches compete with training and evaluation for memory.
- Local execution runs generated code with the worker account’s permissions. For untrusted tasks, select the Docker execution backend and restrict mounted data, network access, and resources. A separate working directory does not provide operating-system isolation.
- Configure model spending, task duration, worker capacity, and execution limits before unattended runs. Container resource limits require the container backend.
- Recompute important results from raw artifacts.
- Record dependency versions, dataset versions, model IDs, seeds, and execution configuration for experiments supporting scientific claims.
- Keep provider credentials and machine-specific configuration outside the repository.

See [Security](https://github.com/mikamikasuki/FOREST/blob/main/SECURITY.md) and [Deployment](https://github.com/mikamikasuki/FOREST/blob/main/docs/DEPLOYMENT.md) for execution boundaries and configuration.

### Local Executor Cleanup

The local worker owns the process handles for commands it starts. After ordinary
cancellation or project deletion, it polls those owned handles and releases the
handle and log-progress offset only after the child exits. A deleted run row
allows the worker to poll its existing handle; it does not authorize killing a
live process or scanning unrelated PIDs. Active and paused runs remain with the
normal monitor and result-reconciliation path. Pending-intervention runs remain
under their existing intervention and monitor-skip guards. This behavior is
limited to local worker-owned processes; it does not add remote/container cleanup
or worker-crash recovery guarantees.

The focused backend regression population is
`tests/test_worker_reaping.py`, `tests/test_worker.py`, and
`tests/test_budget_lifecycle.py`, run serially without `pytest-xdist`. Keep the
reaping regression and the existing cancellation, budget, and monitor controls
together when checking this lifecycle. This Python-only change is checked with
Python AST parsing and Git whitespace validation; those checks are not static
typing, and no frontend build or TypeScript check is part of this scope.

---

## Submission Workflow

Scientific projects default to a complete submission and `full_paper`, with real experiments, accepted-paper comparison, and a review-and-repair loop.

The workflow connects literature, experimental design, baseline reproduction, execution, independent analysis, visual selection, manuscript authoring, and submission review. Delivery checks retain missing evidence and outstanding work when a pilot finishes or a budget is exhausted.

Figures and manuscript values bind to completed runs and recorded observations. Visuals attach to argumentative paragraphs, and compilation checks their actual placement in the document.

See the [submission workflow](docs/PUBLICATION_DELIVERY.md) and [16-paper reference corpus](docs/PUBLICATION_REFERENCE_CORPUS.md). The reference corpus informs framework design; each research project selects topic-matched papers.

[Paper authoring](docs/PAPER_AUTHORING.md) explains independent visual selection, manuscript placement, and manual insertion from Figure Studio.

---

## Why FOREST

Modern language models can formulate hypotheses, write code, inspect literature, run tools, interpret results, and draft technical papers. The harder problem begins when these capabilities are composed into a research process that unfolds over hundreds of dependent decisions.

Long-horizon research creates a compounding state-management problem. As the trajectory grows, decision-relevant evidence can become harder to retrieve, unverified interpretations can propagate into later decisions, and locally reasonable actions can shift the process away from its original objective. Summarization may discard failure-relevant information, competing hypotheses can remain unresolved, and self-critique can reproduce correlated errors when it relies on the same underlying assumptions.

Existing work identifies related failure modes. *Lost in the Middle* and RULER examine how effective use of context differs from nominal context capacity [1,2]. Work on long-horizon search identifies context rot and premature termination as context grows [3](https://arxiv.org/abs/2606.29718). Goal-drift evaluations motivate explicit goal tracking across extended trajectories [4]. Research on intrinsic self-correction shows that reconsidering an answer without reliable external feedback can preserve or amplify errors [5].

FOREST frames autonomous research as a **state-control problem**: each decision should draw on an explicit goal, current dependencies, and inspectable evidence.

> **FOREST makes a research claim traceable to its evidence and a changed premise traceable to the work it affects.**

---

## Long-Horizon Research Drift

We use **Long-Horizon Research Drift** to describe the progressive divergence between the research state required by the original objective and the state that actually drives the agent.

Let:

- $g$ denote the research goal;
- $z_t$ denote the scientific state relevant to step $t$;
- $\hat{z}_t$ denote the scientific state represented to the agent;
- $a_t$ denote the next research action.

The agent selects:

```math
a_t \sim \pi_\theta(a \mid \hat{z}_t, g)
```

A transcript-centered workflow retains an accumulated history:

```math
H_t = H_{t-1} \oplus (a_t, o_{t+1})
```

As the trajectory grows, reconstructing the current goal, valid evidence, and unresolved dependencies from that history becomes an increasingly important part of every decision.

Four recurring sources of drift motivate explicit research state:

| Failure Mode | Effect |
|---|---|
| **Context dilution** | Decision-relevant evidence becomes harder to locate within accumulated context. |
| **Verification debt** | Unchecked interpretations become assumptions for later claims. |
| **Goal displacement** | Locally attractive tasks replace the original research objective. |
| **Premature conservative convergence** | Uncertainty becomes a stopping reason while useful discriminating experiments remain available. |

The final failure mode is particularly important in scientific work.

Uncertainty describes the current evidence state. A useful next decision identifies what remains unresolved and which observation could change the conclusion.

FOREST keeps uncertainty actionable: an unresolved claim can remain connected to the experiments capable of resolving it.

---

## Why Small Errors Become Large Failures

Let $\epsilon_t$ denote the conditional probability of a trajectory-breaking error at step $t$, given that no previous step has already broken the trajectory:

```math
\epsilon_t =
P(D_t = 1 \mid D_1 = 0, \ldots, D_{t-1} = 0)
```

By the chain rule, the probability that the trajectory remains valid through $T$ consequential steps is:

```math
P_{\mathrm{valid}}(T)
=
\prod_{t=1}^{T}(1-\epsilon_t)
```

If the conditional failure probabilities satisfy:

```math
\epsilon_t \ge \epsilon > 0
```

then:

```math
P_{\mathrm{valid}}(T)
\le (1-\epsilon)^T
\le e^{-\epsilon T}
```

Trajectory reliability therefore depends on both the local error rate and the number of consequential steps.

FOREST exposes intermediate state and evidence checks at the handoffs where an unsupported result could become a downstream premise.

---

## Research State as a Graph

FOREST externalizes research state into an editable graph:

```math
G_t = (V_t, E_t)
```

Nodes and associated records represent concepts such as:

```text
Goal
Question
Hypothesis
Method
Experiment
Result
Evidence
Claim
Failure
Decision
Artifact
```

Execution dependencies determine which tasks can run and which upstream outputs they consume. Evidence relationships connect results, sources, and claims.

Each scientific claim can retain the evidence supporting it, the experiments that tested it, its dependencies, and unresolved competing hypotheses.

The runtime tracks execution, research, and deliverable status separately. This distinction allows a task to finish executing while its scientific interpretation or manuscript contribution still requires review.

The complete graph can grow throughout a research program while each model request receives a selected working context.

Conceptually, FOREST constructs an **active frontier**:

```math
F_t = R(G_t, g)
```

Here, $R$ selects the goal, relevant branch state, dependencies, current artifacts, and immediate decision context. Retained originals remain available for further inspection.

Persistent project state and per-request reasoning context serve different purposes: the former preserves the research record; the latter supports the current decision.

---

## Goal Contract

FOREST retains the project goal, constraints, success criteria, and budgets as explicit planning inputs. These form a conceptual **Goal Contract**:

```math
C = (g, S, K, B)
```

where:

- $g$ is the research objective;
- $S$ contains success and falsification criteria;
- $K$ contains operational and scientific constraints;
- $B$ contains compute, time, model, and experiment budgets.

These inputs remain outside the rolling conversation history and participate in planning.

The planner’s trade-offs can be expressed through a conceptual action utility:

```math
U(a)
=
\lambda_g R_g(a)
+
\lambda_i I_G(a)
-
\lambda_c C(a)
-
\lambda_r R(a)
```

where:

- $R_g(a)$ represents relevance to the goal;
- $I_G(a)$ represents expected information value;
- $C(a)$ represents resource cost;
- $R(a)$ represents execution or scientific risk.

This expression describes the decision criteria. The implemented planner receives the stored project goal, constraints, budgets, graph state, and research evidence.

---

## Evidence-Grounded Research

Agent output is treated as a proposal until execution or supporting evidence establishes the relevant result.

FOREST assigns distinct responsibilities across the research workflow:

```text
Planning
   ↓
Literature / Design / Implementation
   ↓
Experiment Execution
   ↓
Independent Analysis
   ↓
Evidence and Claim Review
   ↓
Figures and Manuscript
   ↓
Submission Review
```

Code is executed. Metrics can be recomputed from original observations. Assertions are checked against referenced artifacts. Competing explanations can be examined through additional analysis or experiments. Failed operations remain visible in the research record.

A claim can therefore be rejected even when the prose describing it is persuasive.

> **LLMs propose. Evidence decides.**

---

## Independent Verification

Independent verification grounds judgments in executed code, recomputed metrics, and original artifacts.

Consider a harmful local error with conditional probability $\epsilon_t$. Let $d_t$ be the probability of detecting that error, conditional on its occurrence. Let $r_t$ be the probability of successful repair, conditional on detection.

Holding the local error opportunities fixed, and assuming verification and repair introduce no additional harmful errors, the propagated-error hazard becomes:

```math
\