# RANsliceOpt AI v22: PPO & DDQN for mIoT Overload

This repository contains the browser-executable artifact for the comparative study of reinforcement learning (RL) algorithms handling massive-IoT (mIoT) PRACH overload. The simulator directly evaluates Proximal Policy Optimization (PPO), Double Deep Q-Networks (DDQN), fixed Access Class Barring (ACB), an adaptive deterministic rule, and a "no control" baseline under strictly matched conditions. 

The primary research question investigates whether a gNB-side PPO controller can manage synchronized mIoT PRACH congestion more effectively than DDQN and deterministic overload-control policies under identical held-out access storms.

##  Core Features

*   **Browser-Native Execution:** The simulation, training loops, neural networks, and rendering engine run entirely within a single, self-contained HTML file without any backend dependencies or Python requirements.
*   **Decoupled Hyperparameters:** Supports completely independent parameter tuning for different algorithms, including separate inputs for PPO Learning Rate, DDQN Learning Rate, PPO Gamma, and DDQN Gamma.
*   **Independent Hyperparameter Sweeps:** Includes dedicated diagnostic tools to run automated 10-iteration random sweeps for both PPO and DDQN, optimizing algorithm-specific parameters (like clipping and target network updates) to prove structural capabilities over tuning artifacts.
*   **Methodology Validation (Ablation):** Supports toggling the neural network capacity between a standard 24-unit baseline and a 64-unit wide ablation layer to test representation limits.
*   **Multi-Seed Averaging:** Allows users to switch between deterministic single-seed training and a statistically robust 3-seed averaged training strategy to mitigate network initialization variance.

##  Simulation Design & Mechanics

The environment is a 3GPP TR 37.868-inspired 5G/6G research simulation focused specifically on gNB-side overload control. It is an aggregate research simulator rather than a standards-conformance implementation.

*   **Traffic Modeling:** Generates synchronized mIoT storms with fully configurable activated device counts (e.g., 3,000 devices), activation durations, evaluation horizons, and background arrival rates. Users can select Moderate, Severe, or Extreme profiles.
*   **Scope & Limitations:** The simulator intentionally isolates random access contention. Scheduled eMBB, URLLC, and mIoT physical resource blocks (PRBs) are neither modeled nor reallocated. 
*   **Contention Mechanics:** At each Random Access Opportunity (RAO), eligible devices pass through the controller's ACB gate. Admitted devices select a contention preamble uniformly. Singleton choices succeed, while collisions force a randomized retry backoff up to a maximum attempt limit. ACB-denied devices wait for a randomized barring interval before becoming eligible again.

##  Matched Controllers & AI Architecture

All controllers receive the same exogenous arrival schedule, preamble resources, device population, retry limits, and held-out seed. PPO and DDQN are trained on the exact same generated sequence of non-evaluation storms and receive the same training-episode count and environment-interaction count.

1.  **No overload control:** All eligible devices are admitted.
2.  **Fixed ACB:** Admission probability, barring interval, and retry backoff remain fixed.
3.  **Adaptive deterministic rule:** Admission is estimated from backlog and preamble capacity, while barring and backoff respond to disclosed collision thresholds.
4.  **DDQN:** A gNB-side Q-network using epsilon-greedy exploration, replay memory, and a target network to select a permitted action.
5.  **PPO:** A gNB-side actor observing aggregate PRACH state to select bounded control profiles.

### Observations, Actions, and Rewards
*   **State Vector:** The RL state contains normalized backlog, new arrivals, recent collision and success ratios, idle-preamble ratio, backlog and arrival trends, oldest-device wait, retry-exhaustion pressure, accumulated failures, mean delay, and current control settings.
*   **Action Set:** Actions include bounded gNB control profiles plus independent admission, barring, and retry-backoff adjustments.
*   **Reward Formulation:** Training penalizes collisions, failures, backlog, waiting pressure, and idle preambles while a backlog remains. It applies a terminal penalty for unfinished devices and a one-time reward for clearing the post-storm backlog, discouraging superficially low-collision policies that never reopen access.
*   **Deterministic Safety Guard:** Enforces a common action guard on all learned controllers, establishing an admission ceiling based on the backlog-to-preamble envelope and a recovery floor to prevent indefinite throttling when preambles are idle.

##  Acceptance Rule

To ensure rigorous evaluation, PPO is supported for a held-out test only when, relative to the best deterministic result, it meets all of the following criteria:
*   Causes no more access failures.
*   Does not reduce access success materially.
*   Keeps P95 access delay within 5%.
*   Improves failures, collision rate, or P95 delay by at least 5%.

##  Telemetry and Output

The user interface renders real-time matched results, exposing the identical held-out arrival schedules to all five controllers simultaneously.

*   **Key Performance Indicators (KPIs):** The artifact calculates and displays overall Success %, total device Failures, Collision %, Retries, Mean delay (ms), 95th-percentile (P95) delay (ms), peak backlog, and backlog-clearance RAO.
*   **Visualizations:** A dynamic canvas chart plots the active backlog of devices frame-by-frame across the Random Access Opportunity (RAO) horizon.
*   **Trace Logging:** A dedicated trace panel outputs hyperparameter sweep configurations, results, and ablation steps for transparent auditing.
*   **Validation Matrix:** Executes automated multi-scenario cross-validation over varying traffic profiles and independent evaluation seeds, reporting direct win/tie/loss counts and a 95% Wilson interval.
*   **Data Export:** Full trial telemetry and the synthetic validation matrix can be exported directly to CSV for external plotting and analysis.

##  How to Use

### 1. Standard Evaluation
1.  Open the `RANsliceOpt_AI_v22_PRACH.html` file in any modern web browser.
2.  Adjust the Methodology Validation settings (e.g., 3 Seeds Averaged) and input specific hyperparameters under Advanced PRACH Settings.
3.  Click **Train and Run PPO-DDQN Evaluation** to train the networks and simulate the held-out traffic storm.

### 2. Hyperparameter Sweeps
1.  To test an algorithm's structural limits, click **Run PPO Hyperparameter Sweep** or **Run DDQN Hyperparameter Sweep**.
2.  The engine will sample 10 random configurations and output the lowest-failure configuration to the Trace Log at the bottom of the screen.
3.  *Note: You must manually enter the winning parameters into the Advanced Settings menu to use them in the main evaluation.*

### 3. Multi-Scenario Validation
1.  Click **Run Multi-Scenario Validation Matrix**.
2.  The simulator will evaluate the current algorithms against Moderate, Severe, and Extreme profiles across multiple random seeds, generating a robust win/tie/loss comparative matrix.