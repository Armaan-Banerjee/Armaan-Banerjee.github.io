---
title: "Reinforcement Learning with LLMs"
date: 2026-11-03
---

![Diagram showing the LLM reinforcement-learning loop from prompt and response tokens to a reward]({{ '/assets/courses/reinforcement-learning-with-llms/figure-01.png' | relative_url }})

## Rewards

### Subjective

- Reinforcement learning from human feedback (RLHF) is subjective, based on human preferences, and modelled using Bradley–Terry
- Bradley–Terry is a probability model for comparisons between two items. Given items \(i\) and \(j\) with scores \(s(i)\) and \(s(j)\), it estimates the probability that \(i\) beats \(j\) as \(e^{s(i)} / (e^{s(i)} + e^{s(j)})\), which is the sigmoid of the difference between their scores
- Human preferences between responses are collected to train a reward model, which scores a response to any prompt; the policy is then optimised against that reward model

### Verifiable

- The solution to a problem is deterministic and can be checked
- Reinforcement learning with verifiable rewards (RLVR) uses these objective checks as rewards

## Goal

- Change the model weights so that the model produces more generations with higher rewards; the objective function is the expected reward under the policy

![Annotated reinforcement-learning objective showing the policy, trajectory, and cumulative reward]({{ '/assets/courses/reinforcement-learning-with-llms/figure-02.png' | relative_url }})

- To optimise the objective, find the derivative of the objective function

![Derivation of the policy-gradient objective on a chalkboard]({{ '/assets/courses/reinforcement-learning-with-llms/figure-03.png' | relative_url }})

- Using \(d\log f(x)/dx = f'(x)/f(x)\), rearrange the expression so that the derivative of \(\pi_\theta\) is \(\pi_\theta\) multiplied by the derivative of \(\log \pi_\theta\)

![The reinforcement-learning objective annotated with the question of how to obtain its gradient]({{ '/assets/courses/reinforcement-learning-with-llms/figure-04.png' | relative_url }})

- \(\tau \sim \pi_\theta\) means that a trajectory is sampled from the probability distribution parameterised by \(\theta\)

## REINFORCE

- Given a prompt \(x\), sample a response \(y\) from the model
- Evaluate the response by passing \(x\) and \(y\) to the reward model to obtain \(R\)
- Compute the log-probability gradient \(g = R \nabla_\theta \log \pi_\theta(y \mid x)\), then update the parameters

![The four REINFORCE steps: sample, evaluate, compute the gradient, and update parameters]({{ '/assets/courses/reinforcement-learning-with-llms/figure-05.png' | relative_url }})

![Rewriting the policy objective as an expectation under a fixed policy]({{ '/assets/courses/reinforcement-learning-with-llms/figure-06.png' | relative_url }})

- Another way to write the objective is to multiply and divide by a policy with fixed parameters, turning it into an expectation sampled from that fixed policy
- This is cheaper and can be parallelised: generation with a changing policy must wait for each token before producing the next, whereas evaluation receives the complete, already-generated response and can process it in parallel

![Chalkboard comparison between sequential generation and parallel evaluation]({{ '/assets/courses/reinforcement-learning-with-llms/figure-07.png' | relative_url }})

## Challenges

- High-variance gradients: without a reward baseline, the reward function can generate excessively large gradients
- A baseline makes the scale manageable; for example, use 100 as the baseline, give a good answer \(+1\), and give a bad answer \(-1\), resulting in rewards of 101 and 99
- No explicit token-level credit: the reward function grades the complete response rather than each token
- The magnitude of policy updates must be controlled so that an already-good starting policy does not lose performance on previous tasks or learn to game the reward system

## PPO

- Proximal Policy Optimisation (PPO) uses three policies: \(\pi_\theta\), the policy being updated; \(\pi_{old}\), the policy used to generate the batch; and \(\pi_{ref}\), the base or reference model
- It maximises advantage while preventing the policy from deviating too far from its previous behaviour
- It computes the expected final reward given the prefix generated so far, which helps estimate how much each token affected the reward

![PPO advantage for a token as the difference between two expected final rewards]({{ '/assets/courses/reinforcement-learning-with-llms/figure-08.png' | relative_url }})

- PPO introduces a value function: a separate model trained alongside the policy that takes \(x\) and the response generated so far as input, then estimates the expected future reward

![PPO advantage expressed as the value with and without the current token]({{ '/assets/courses/reinforcement-learning-with-llms/figure-09.png' | relative_url }})

- The value function estimates the effect of a token, but that effect is not independent: a better token now can lead to better future tokens that are unknown at the time the current token is generated
- Generalised Advantage Estimation (GAE) looks at future rewards and attributes them back to a given token

![PPO objective using the current-to-old policy ratio to reuse a batch for multiple updates]({{ '/assets/courses/reinforcement-learning-with-llms/figure-10.png' | relative_url }})

- The same batch can be reused by maximising an expectation weighted by the ratio between the current and old policies
- The ratio can grow very large for significant changes, so clipping can stop the reward from increasing beyond a chosen range
- The advantage was calculated under the old policy that generated the batch, so the policy ratio rescales that existing advantage
- KL divergence measures the difference between probability distributions \(P\) and \(Q\), treating \(P\) as the reference distribution
- A KL-divergence penalty can be used alongside or instead of clipping to compare the updated policy against the reference model

![PPO objective with a KL-divergence penalty that keeps the policy close to the reference model]({{ '/assets/courses/reinforcement-learning-with-llms/figure-11.png' | relative_url }})

## GRPO

![PPO pipeline with policy, reference, reward, and value models producing an advantage through GAE]({{ '/assets/courses/reinforcement-learning-with-llms/figure-12.png' | relative_url }})

- Group Relative Policy Optimisation (GRPO) is similar to PPO: it maximises advantage while limiting how far the policy deviates, but it calculates advantages without a value function
- Given a prompt \(x\), sample \(G\) outputs from the model and calculate the advantage of each output
- The advantage of an output is its reward minus the mean reward across the group, divided by the standard deviation of the group's rewards

![GRPO advantage calculation across a group of sampled outputs]({{ '/assets/courses/reinforcement-learning-with-llms/figure-13.png' | relative_url }})

- GRPO uses the same clipping and KL-divergence ideas to prevent the policy from deviating too far

![GRPO objective combining clipped updates with a KL-divergence penalty]({{ '/assets/courses/reinforcement-learning-with-llms/figure-14.png' | relative_url }})

![GRPO pipeline from a prompt through grouped outputs and rewards to group-relative advantages]({{ '/assets/courses/reinforcement-learning-with-llms/figure-15.png' | relative_url }})

- The calculated advantages are used to compute the policy gradient

![Side-by-side comparison of the GRPO and PPO objectives]({{ '/assets/courses/reinforcement-learning-with-llms/figure-16.png' | relative_url }})

- Both methods use the ratio between the probabilities assigned by the current and old policies to scale the advantage
- Both use clipping to keep the current policy close to the old policy
- GRPO normally calculates advantage at the completion level relative to the other outputs, but it can also use token-level advantages, represented by the \(t\) subscript; PPO calculates advantage at the token level
- Reinforcement learning has been observed to increase response lengths
- Because the GRPO formula scales each output by the inverse of its length, the model can prefer a long bad response over a short bad response because the longer response receives a smaller per-token disadvantage
- DAPO addresses this by equalising token-level contributions

![DAPO remedy for the GRPO length bias by equalising token-level contributions]({{ '/assets/courses/reinforcement-learning-with-llms/figure-17.png' | relative_url }})

## On-policy distillation

- KL divergence measures how poorly distribution \(Q\) approximates a world governed by \(P\)

![Definition and interpretation of KL divergence from P to Q]({{ '/assets/courses/reinforcement-learning-with-llms/figure-18.png' | relative_url }})

- In on-policy distillation, the student is \(P\) and the teacher is \(Q\), even though the teacher is the better model; this is called reverse KL divergence
- Reverse KL forgives probability mass that the student misses where the teacher assigns a high probability, but penalises the student for placing probability where the teacher assigns little

![Comparison of forward KL in supervised fine-tuning and reverse KL in reinforcement learning]({{ '/assets/courses/reinforcement-learning-with-llms/figure-19.png' | relative_url }})

- This makes the student mode-seeking: it preserves existing modes and avoids inventing unsupported new peaks
- See [On-policy distillation](https://thinkingmachines.ai/blog/on-policy-distillation/) for more detail
- On-policy distillation can keep the student model itself as the teacher while supplying the teacher with a hint

![Example of using the same student model as a teacher by providing it with a reasoning hint]({{ '/assets/courses/reinforcement-learning-with-llms/figure-20.png' | relative_url }})

- Multiple teachers can also be used, with each teacher acting as an expert in its associated domain

## Direct Preference Optimisation

- Direct Preference Optimisation (DPO) optimises a model's output to align with human preferences

![Annotated Direct Preference Optimisation loss for preferred and dispreferred responses]({{ '/assets/courses/reinforcement-learning-with-llms/figure-21.png' | relative_url }})
