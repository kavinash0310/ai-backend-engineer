# Reinforcement learning foundations

## 1. The interaction setting

In reinforcement learning (RL), an agent observes a state, chooses an action, receives a reward and transitions to another state. Unlike supervised learning, the desired action label is often not supplied for every state; learning uses consequences across time.

A Markov decision process (MDP) is defined by:
- states S and actions A;
- transition probability P(s'|s,a);
- reward function R(s,a,s');
- discount factor gamma from 0 to 1;
- initial-state distribution.

The Markov assumption says the current state summarizes relevant history for predicting future transitions and rewards. If it does not, the state may need memory or a belief distribution.

## 2. Return, value and policy

A policy pi(a|s) describes action choices. The return is discounted sum of rewards:
G_t = r_(t+1) + gamma r_(t+2) + gamma² r_(t+3) + ...

The state-value function V_pi(s) is expected return from state s under policy pi. The action-value function Q_pi(s,a) is expected return after taking action a and then following pi.

The Bellman expectation relationship decomposes value into immediate reward plus discounted future value. This recursion enables dynamic programming and value-based learning. A reward is a designed proxy for desired outcomes; if it omits constraints, optimization can exploit loopholes.

## 3. Exploration and exploitation

**Exploration** gathers information about uncertain actions. **Exploitation** chooses actions that seem best under current knowledge. Too little exploration can miss better strategies; too much can waste reward or harm users. Safe systems use constrained environments, simulation, offline data and limited pilots before real-world actions.

## 4. Q-learning

Q-learning learns an action-value estimate using a temporal-difference target:
target = r + gamma max_a' Q(s',a')
Q(s,a) <- Q(s,a) + alpha [target - Q(s,a)].

Here alpha is learning rate. The bracketed term is the TD error.

Numeric example: current Q(s,a)=2, reward r=1, gamma=0.9, and best next-state value is 4. The target is 1+0.9×4=4.6. With alpha=0.1, updated Q=2+0.1×(4.6-2)=2.26.

Tabular Q-learning is useful for intuition but cannot store values for huge continuous state spaces. Deep Q networks approximate Q with a neural net and use techniques such as replay buffers and target networks for stability; this adds instability and evaluation challenges.

## 5. Policy gradients and actor-critic

Policy-gradient methods optimize parameters of a policy directly using reward-weighted gradient estimates. They can handle stochastic policies and continuous actions but have high variance. Actor-critic methods learn both an actor (policy) and critic (value estimate) to reduce variance and improve learning.

These algorithms introduce sensitivity to sampling, reward scale, estimator bias, policy updates and environment differences. Results can vary substantially across random seeds; evaluate multiple runs and uncertainty.

## 6. Offline RL and reward learning

Offline RL learns from a fixed dataset without interacting with the environment during training. It avoids dangerous exploration but faces distribution shift: the learned policy may choose actions poorly represented in logged data. Conservative methods attempt to reduce unsupported actions; deployment still needs careful evaluation.

Reward modeling learns preferences or scores from human comparisons. A reward model can be exploited if it is an imperfect proxy for true preference or safety. Preference learning is used in language-model post-training, but language systems add token sequences, reference policies and specialized objectives.

## 7. Evaluation and safety

Evaluate in simulation and on held-out scenarios; test rare and adversarial states; compare against a simple policy; measure reward and task outcomes; inspect constraints and worst-case behavior. Simulation may not match reality, and reward can hide harmful behavior.

For high-impact applications:
- constrain actions independently of learned policy;
- support human override and emergency stop;
- limit exploration;
- monitor state/action/reward distribution;
- log decisions with privacy controls;
- define rollback and incident response.

RL should not be used simply because the task involves a sequence. A deterministic planner or supervised policy may be more efficient and reliable.

## Questions and answers

**Q: How is RL different from supervised learning?**  
A: Supervised learning usually has target labels per example. RL learns from rewards and transitions, often with delayed effects and actions that influence future data.

**Q: Why discount future reward?**  
A: Discounting can encode preference for nearer outcomes and help make infinite-horizon return finite under conditions. It changes the objective and should be chosen deliberately.

**Q: What is reward hacking?**  
A: A policy optimizes a proxy reward in an unintended way, exploiting gaps between the measured reward and true goal.

**Q: Does a high simulated reward prove a policy is safe?**  
A: No. The simulation may omit real-world states, incentives or failure modes. Use layered constraints and staged evaluation.

## Practice

1. Define states/actions/rewards for inventory restocking and list ways to game the reward.
2. Compute a Q-learning update with new values.
3. Explain why a policy trained offline may act poorly outside logged data.
4. Design safe stages from simulation to a limited real-world pilot.
5. Compare a rules-based scheduler, supervised predictor and RL policy for one service.

## Completion check

You can define an MDP, calculate return and a Q-learning update, distinguish value/policy methods, and explain exploration, reward misspecification and deployment safety.