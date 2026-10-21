# DEEPSPEED AUTONOMOUS RACING SIMULATION HACKATHON 

## TEAM CalcuLOL 
**Position on Final Leaderboard: 1** 

* **Members:** Myra Dwivedi (@myradvd), Anish Gangavaram (AMGhalcyon) and Davis Lee (davislee23).
* **Algorithm:** MPC (Model Predictive Control) Profile with minimum curvature based on an offline pre-computed racing line (albeit regeneratable at runtime,it takes ~3-4 minutes for generation, hence, is shipped as a csv). Optimised speed-profile based on the simulator's real latency, steering and dynamics, and added an auto-breaking functionality within the MPC instead of following a fixed speed-profile to drop lap-time significantly. Implemented backward crawl and slow warmup safety procedures for collision prevent and recovery. 
* **Implementation:** RViz Visualiser. Control through ROS2. Done through SLSQP (Sequential Least Squares Programming) and Python. Control loop: 25ms.
* **Possible Failure Mode:** CPU overloading causing Real Time Factor(RTF) <= 1.5, causing information latency communication issues between MPC controller and simulation. Recommended to test on Intel Ultra Core 7 or better.

  
Results: <img width="2560" height="1115" alt="image" src="https://github.com/user-attachments/assets/85d17fa0-08ac-4ad3-a0b7-35615e472ca1" />

Running and set-up instructions for windows, macOS and linux are in the original project brief (see below).
SUBMISSION.md and HOW_IT_WORKS.md contain higher level and in depth (respectively) overviews of our approach, design considerations, failed attempts, alternate algorithm baseline results (Pure-Pursuit, Follow-the-gap, Pure Pursuit with pre-computed racing line) and implementation details.


---

<details>
<summary>💡 <b>See the original brief</b></summary>
<br>

## RoboRacer Autonomous Racing — Track 1

**NTU DeepSpeed Recruitment Hackathon 26/27**

Write the software that drives a 1/10-scale racing car around a circuit it has
never seen, as fast as it can, without hitting anything. Everything you need —
the simulator, the judging environment, reference algorithms and every official
track — is in this repository. You will not need to clone anything else.

> **Submission deadline: 18 October 2026, 23:59 (SGT)**

---

### Quick start

```sh
git clone --recurse-submodules https://github.com/NTUDeepSpeed/Recruitment_Hackathon_2627.git
cd Recruitment_Hackathon_2627
git checkout track1

./install/linux/setup.sh          # or install/macos, or install/windows from WSL
./install/linux/run.sh            # builds if needed, then drops you into the container
```

Then, inside the container:

```sh
cd /hackathon/race_ws && colcon build --symlink-install
source install/local_setup.bash

# Terminal 1 — the simulator
ros2 launch roboracer_referee simulator.launch.py

# Terminal 2 — your car
ros2 run team_driver driver
```

The car will set off and hit the first wall: the template is wiring, not a
driver, and writing one is the hackathon. Once yours does something, score
it:

```sh
./scripts/evaluate.sh --team your_team_name
```

Pushing also races your entry on GitHub Actions and writes the result to the
workflow summary. That workflow is for reference only — it runs on GitHub's
hardware and nothing it prints is scored. **Your result is the organisers' run
on the judging machine.**

Full walkthrough: **[docs/01-setup.md](docs/01-setup.md)**.

> Cloned without `--recurse-submodules`, or downloaded a ZIP? Run
> `git submodule update --init --recursive`. The setup scripts will also do it
> for you. A ZIP download will not work — submodules need a real clone.

---

### The challenge

You get a LiDAR scan and the car's exact position. You publish a steering angle
and a speed. That is the whole interface.

| | |
| --- | --- |
| **Car** | Ackermann steering, 0.33 m wheelbase, 0.31 × 0.58 m, steering limited to ±0.4189 rad |
| **Sensor** | 819-beam LiDAR, 270° field of view, 25 m range |
| **Localisation** | Ground-truth pose on `/ego_racecar/odom` — **allowed and recommended** |
| **Track** | `icra26`, in this repository. 17 x 18 m, about 78 m a lap, cone slaloms and a hairpin complex. |
| **Scored on** | Your single fastest lap, and your time for 10 consecutive laps |
| **Penalties** | +10 s on the lap for each collision; more than 10 collisions is a disqualification |
| **Judged on** | One machine: i9-14900HX, 32 GB, RTX 5060 Laptop. Times are in simulated seconds, so your own hardware does not affect your score. |

**No driver ships with this repository.** `team_driver` is wiring — it talks to
the simulator and drives in a straight line into the first wall. Writing
something that laps is the hackathon. Reactive algorithms like follow-the-gap
will get you round; planning against the map and your own position is where the
lap time is; and **replacing the approach outright is encouraged**:
reinforcement learning, MPC, imitation learning, a learned end-to-end policy,
anything you can defend. The environment is ROS 2 Jazzy on Python 3.12 with a
GPU available, so a learned policy is a realistic option.
[docs/04-algorithms.md](docs/04-algorithms.md) is the menu, with what each
approach needs and where each breaks.

---

### Where things are

