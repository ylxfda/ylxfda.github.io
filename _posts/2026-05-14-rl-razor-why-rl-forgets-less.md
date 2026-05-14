---
layout: post
title: "Why Does a Student Learn Faster by Solving Problems First, Then Checking the Answer?"
date: 2026-05-14
categories: research
---


---

## 1. An itch I had during the post-training boom

When DeepSeek and the broader RL-for-post-training wave hit last year, one observation kept showing up: a base model fine-tuned with **reinforcement learning** (RL) tended to be better — better generalization, less catastrophic forgetting, more robust on out-of-distribution inputs — than the same model fine-tuned by **supervised fine-tuning** (SFT) on comparable data.

I had a naive but stubborn intuition about why, drawn straight from how human students learn:

> If you want to actually learn, you should *try the problem yourself first*, then look at the solution or get the teacher's feedback. Just being shown the worked solution and copying it down — that's much weaker.

That mapping felt obvious:

- **SFT** is "read the worked solution and imitate." The model is fed (input, golden answer) pairs produced by someone else.
- **RL** is "try yourself, then get graded." The model rolls out its own attempt, and the reward signal nudges it toward better behavior.

My intuition is that solutions produced by RL tend to stay closer to the model’s native distribution. However, I have not yet worked through the mathematical or mechanistic justification for this.

Recently I read two papers from the same group at MIT (Pulkit Agrawal's lab) that, taken together, answer this very cleanly:

1. **RL's Razor: Why Online Reinforcement Learning Forgets Less** (Shenfeld, Pari, Agrawal, 2025) — the *diagnosis*.
2. **Self-Distillation Enables Continual Learning** (Shenfeld, Damani, Hübotter, Agrawal, 2026) — the *prescription*, for the case where you don't have a reward.

---

## 2. Background

**Continual learning** is the goal of letting a model pick up new skills or new knowledge without trashing what it already knew. Every time we fine-tune a strong base model on a narrow task, we risk damaging its general capabilities — the dreaded **catastrophic forgetting**.

**Self-distillation** is a special case of knowledge distillation: instead of a bigger teacher teaching a smaller student, the model serves as its own teacher (often a slightly different version of itself — an EMA copy (a wighted moving average), a context-augmented copy, etc.). It's typically used as regularization or to bootstrap better behavior from the model's existing knowledge.

---

## 3. Paper 1 — RL's Razor: a clean diagnosis

The headline claim of *RL's Razor*, in one sentence:

> Among all the policies that solve a new task, on-policy RL is implicitly biased toward the one with the **smallest KL divergence to the base model**. SFT has no such bias.

A few key points:

**(a) Forgetting is governed by distributional shift, not by the algorithm.** The authors show empirically that how much a fine-tuned model forgets is essentially a function of the KL divergence between the fine-tuned and base policy, evaluated on the new task. Plot KL on the x-axis and "drop on prior tasks" on the y-axis, and you get roughly a single curve, regardless of whether the points came from RL or SFT. The algorithm is not magic — the endpoint distribution is what matters.

![KL divergence on the new task predicts forgetting on prior tasks. Both RL and SFT points fall on essentially the same curve.](https://jyopari.github.io/posts/RL%27s%20Razor%20Why%20On-Policy%20Reinforcement%20Learning%20Fo%202573b66e4cf580cdb414d25c1f346f0c/Screenshot_2025-08-22_at_6.24.12_PM.png)
*Figure from Shenfeld, Pari & Agrawal (2025), [RL's Razor project page](https://jyopari.github.io/posts/rl_razor). KL divergence between the fine-tuned and base policy on the new task is an excellent single predictor of how much prior-task ability is lost — RL and SFT points lie on the same curve.*

**(b) Why RL stays close in KL.** On-policy policy gradient updates are weighted by the model's *own* current sampling distribution. Trajectories the model already considers improbable get sampled rarely and contribute almost nothing to the gradient. So the policy can't easily jerk itself onto a distribution very far from where it started. SFT, in contrast, is pulled by external labels and can converge to a policy arbitrarily far in KL from the base.

**(c) On-policyness is the active ingredient.** The authors compare several objectives — GRPO, an SFT variant with negative examples, and others — and find that whether the data is on-policy is what matters, not whether you have a reward or use negative examples. They also give a theoretical bound: one step of on-policy policy gradient produces a KL change bounded in a way SFT updates are not.

So the slogan **RL's Razor** is: *among the many ways to solve the new task, RL prefers the one closest in KL to the original model.* That's why RL forgets less.

This is, I think, a very satisfying explanation of the hand-wavy student-learning intuition. "Try yourself, then be graded" corresponds exactly to *on-policy* training: the trajectories you update on are sampled from your current policy. That's what keeps you anchored to who you already are while absorbing the new task.

---

## 4. Paper 2 — Self-Distillation Enables Continual Learning (SDFT)

*RL's Razor* tells us why on-policy training is the active ingredient. But on-policy RL has a hard requirement: you need a **reward function**. In many real-world cases — teaching a new skill, injecting new knowledge — you don't have a verifiable reward. You only have **expert demonstrations**. And once you fall back to demonstrations, you're back to SFT, which is off-policy and triggers the forgetting problem we wanted to avoid.

SDFT is the elegant workaround: get the on-policyness without needing a reward, by having the model teach itself.

### The core trick

The same model plays two roles, distinguished only by what's in its context window:

- **Teacher**: input is `[demonstration] + [task]`. With the demonstration in context, in-context learning makes the model behave as if it already knew how to do the task. It's effectively a "smarter version of itself," conditioned on the demo.
- **Student**: input is just `[task]`. This is the model we actually want to deploy.

The one-sentence summary of SDFT is:

> Take the ability to do the task **with a demonstration in context** and, via distillation, internalize it into the ability to do the task **without** the demonstration.

### The training loop

1. The **student** rolls out an answer `y` to the task — *no* demonstration in its context. So `y` is sampled from the student's current distribution: **on-policy**.
2. The **teacher**, which has the demonstration in its context, scores that same `y`. Because of the demo, the teacher's per-token predictions are more accurate.
3. The student is updated to **match the teacher's distribution on `y`**, token by token.

Two subtleties worth pausing on:

**What does "match the teacher's distribution" actually mean?** In ordinary supervised learning, the target is a **hard label** — one-hot, "the correct token is *X*, probability 1." In distillation, the target is the teacher's **soft distribution** over the whole vocabulary at that position. For example, on the token "France": hard label says `France: 1.0` and everything else `0`; the teacher's distribution might say `France: 0.85, France.: 0.10, Frankreich: 0.03, …`. The student matches the whole distribution — more informative than a hard label, since the teacher is also telling the student which alternatives would be almost as good and which are clearly wrong. In SDFT the teacher and student share weights; "more accurate" doesn't mean "from a stronger model," it means "from a version of itself temporarily boosted by in-context learning."

**The student's rollout `y` is a long sentence — what does the teacher actually output on it?** Not another sentence. At every token position of `y`, the teacher outputs a probability distribution over the entire vocabulary. Recall that an LLM at every position outputs a vocabulary-sized probability vector; there are two ways to use that. In **generation mode**, you sample, emit, feed back, repeat. In **scoring mode (teacher forcing)**, you take an already-existing sequence, run a single forward pass, and just read off the distribution at each position — no sampling. SDFT uses the teacher in scoring mode. Concretely, if the student rollout `y` is 20 tokens long, one forward pass through `[demonstration][task][y]` yields 20 distributions over the vocabulary at the answer positions. The student, run on the same `y` *without* the demonstration, produces 20 distributions of its own. The training objective aligns these two sets of 20.

### The loss

SDFT uses a forward-KL distillation loss, summed over the tokens of the student's rollout:

$$
\mathcal{L}_{\text{SDFT}}(\theta) = \mathbb{E}_{x \sim \mathcal{D},\ y \sim \pi_\theta(\cdot \mid x)} \left[ \sum_{t=1}^{|y|} \mathrm{KL}\Big(\, p_T(\cdot \mid d, x, y_{<t}) \;\Big\|\; \pi_\theta(\cdot \mid x, y_{<t}) \,\Big) \right]
$$

where $\theta$ is the (shared) model's parameters, $x$ is the task, $d$ is the demonstration (fed only to the teacher), $y \sim \pi_\theta(\cdot \mid x)$ is the student's own sample (the **on-policy** part), $p_T$ is the teacher's distribution (conditioned on $d$), and $\pi_\theta$ is the student's distribution (no $d$). In practice $p_T$ is computed under `stop_grad` — gradients only flow through the student's forward pass.

Compare against SFT, which fits an externally given demonstration $(x, y^*)$:

$$
\mathcal{L}_{\text{SFT}}(\theta) = -\sum_{t=1}^{|y^*|} \log \pi_\theta(y^*_t \mid x, y^*_{<t})
$$

Two structural differences jump out: **whose trajectory** — SFT fits an externally given $y^*$ (off-policy), SDFT fits the student's own $y$ (on-policy); and **what target** — SFT pushes probability toward a one-hot label, SDFT pushes the student's whole distribution toward the teacher's soft distribution, which is by construction close to the student (same model, just with extra context), so each update is gentler in KL. These two changes are exactly what turns off-policy imitation into something with the on-policy character that *RL's Razor* identifies as the source of RL's resistance to forgetting.

In experiments across skill learning, knowledge acquisition, and a continual-learning setup with three sequential tasks, SDFT outperforms SFT both on the new task *and* on retaining prior abilities.

---

## 5. How the two papers fit together

A clean **diagnosis → prescription** pairing, by mostly the same authors:

| | **RL's Razor** | **SDFT** |
|---|---|---|
| Role | *Why* RL forgets less | *How* to get the same property without RL |
| Key claim | Forgetting ≈ KL drift on the new task. On-policy RL is implicitly KL-minimal. | Use a context-conditioned self-teacher to get on-policy updates from demonstrations. |
| Required signal | Reward function | Expert demonstrations |
| When to use | Verifiable tasks (math, code, robotics) | Tasks with demos but no clean reward (knowledge injection, style, open-ended skills) |
| Replaces | Explains the gap with SFT | Replaces SFT |

The deeper shared thesis: **what matters for resistance to forgetting is *whose distribution the training samples come from*, not the loss function**. Make the samples come from the model itself, and you anchor the update — whether you grade those samples with a reward (RL) or with a context-augmented self-teacher (SDFT). This is the precise version of the student-learning intuition I started with. "Try the problem yourself first, then check the answer" is exactly: *sample from your own current distribution, then receive a signal about it*. And the formal reason it works is that updates restricted to your own samples can't drift far in KL from where you started — so you absorb the new skill without overwriting the old ones.

---

## 6. Other explanations in the literature

*RL's Razor* is the explanation I find cleanest, but it's not the only angle. A short tour:

- **SFT Memorizes, RL Generalizes** (Chu et al., ICML 2025). The earliest systematic empirical study: across rule-based reasoning and visual navigation, RL — especially with outcome-based rewards — generalizes to OOD variants, while SFT memorizes the training set. More "what" than "why," but it crisply established the phenomenon.

- **Why Does RL Generalize Better Than SFT? A Data-Centric Perspective** (Lu et al., 2026). A different and complementary angle: RL implicitly performs **sample selection**, focusing optimization on *medium-difficulty* examples. Hard samples in SFT degrade OOD generalization because the model has to bend itself out of shape to fit them. The authors propose Difficulty-Curated SFT, which filters by difficulty and recovers — even surpasses — RL's generalization, at much lower cost. So RL's edge may not be entirely about on-policyness; some of it is about *which* samples drive the gradients.

- **Why Reinforcement Fine-Tuning Enables MLLMs Preserve Prior Knowledge Better** (Liang et al., 2025). A learning-dynamics view: RL mostly reinforces samples already aligned with the base model's probability landscape, so its updates have small magnitude *and* point in a direction that doesn't conflict with prior knowledge. SFT, lacking this filter, is dragged toward labels regardless of how foreign they are to the base.

---

**References**

- Shenfeld, Pari, Agrawal. *RL's Razor: Why Online Reinforcement Learning Forgets Less*. arXiv:2509.04259, 2025.
- Shenfeld, Damani, Hübotter, Agrawal. *Self-Distillation Enables Continual Learning*. arXiv:2601.19897, 2026.
- Chu et al. *SFT Memorizes, RL Generalizes: A Comparative Study of Foundation Model Post-training*. ICML 2025 / arXiv:2501.17161.
- Lu et al. *Why Does RL Generalize Better Than SFT? A Data-Centric Perspective on VLM Post-Training*. arXiv:2602.10815, 2026.
- Liang et al. *Why Reinforcement Fine-Tuning Enables MLLMs Preserve Prior Knowledge Better: A Data Perspective*. arXiv:2506.23508, 2025.
