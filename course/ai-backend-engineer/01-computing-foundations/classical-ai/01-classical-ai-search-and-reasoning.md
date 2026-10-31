# Classical AI: search, logic, planning and decision-making

## Where this fits

AI predates modern neural networks. Classical AI often represents states, actions, goals and rules explicitly, then searches or reasons over them. Machine learning estimates behavior from examples. Many real products combine both: a model proposes a label, while deterministic rules enforce permissions and workflow constraints.

## 1. State-space formulation

A search problem can be defined by:
- **state space**: possible configurations;
- **initial state**: starting configuration;
- **actions/operators**: legal transitions;
- **transition function**: result of applying an action;
- **goal test**: whether a state satisfies the objective;
- **path cost**: cost of an action sequence.

For route planning, a state can be a city, actions are roads, goal is destination, and path cost is travel time. The search algorithm explores candidate states. A correct formulation matters more than the choice of algorithm.

A search tree may contain the same state via multiple paths. Track visited/best-known cost to avoid repeated work. Branching factor b and solution depth d can make naïve search exponential.

## 2. Uninformed search

- **Breadth-first search (BFS)** explores shallow states first. It finds a shortest path in number of edges when all step costs are equal, but memory can grow exponentially.
- **Depth-first search (DFS)** explores one branch deeply. It can use less memory, but may go very deep or fail to find a shortest path; use cycle detection and depth limits where appropriate.
- **Uniform-cost search (UCS)** expands the lowest accumulated path cost g(n). With nonnegative costs, it finds a least-cost path but may explore many states.

“Shortest” depends on the objective: fewest steps, distance, time, money or risk. Pick the cost definition with the product owner.

## 3. Heuristic search and A*

A heuristic h(n) estimates remaining cost from state n to a goal. Greedy best-first search chooses the smallest h(n); it can be fast but is not generally optimal. A* chooses minimum f(n)=g(n)+h(n), combining known path cost and estimated remaining cost.

If h is **admissible** (never overestimates true remaining cost) and conditions hold, A* can find an optimal path. A **consistent** heuristic obeys a triangle-like condition across transitions and helps avoid re-opening nodes. A poor heuristic can make A* no better than uninformed search; memory still can be large.

For a grid with four-direction movement and unit cost, Manhattan distance is admissible when movement is only horizontal/vertical and obstacles do not change that lower bound.

## 4. Constraint satisfaction

A constraint satisfaction problem has variables, domains and constraints. Sudoku has cell variables, digit domains and row/column/box constraints. Backtracking assigns values and returns when constraints fail. Heuristics such as minimum remaining values choose the most constrained variable first; constraint propagation reduces domains after assignments.

This gives a practical contrast:
- search explores states/actions;
- CSP explores assignments;
- optimization searches for the best score;
- planning seeks a sequence of actions that transforms an initial state to a goal.

## 5. Propositional and first-order logic

Propositional logic uses statements that are true or false and connectives such as AND, OR, NOT and implication. A rule like “if user is active and owns document, allow read” can be written as a logical implication. Truth tables define connective behavior.

First-order logic adds objects, predicates, variables and quantifiers:
- universal: for every x;
- existential: there exists an x.

Logic is useful for explicit rules and formal properties, but real data may be incomplete, contradictory or uncertain. A knowledge base is only as current and correct as its facts and update process. Do not rely on an LLM to silently replace an authoritative access-control rule.

## 6. Planning

Classical planning describes actions with preconditions and effects. For example, action “ship order” may require payment accepted and inventory reserved, then change order state. A planner searches for a sequence reaching a goal.

Production workflows often need additional realities:
- actions may fail or be delayed;
- state can change concurrently;
- external effects may not roll back;
- approvals and compensation are needed;
- observations can be uncertain.

Therefore, application workflows, transactions, state machines, sagas and idempotency often provide clearer guarantees than asking an LLM to invent a plan.

## 7. Adversarial search and game trees

Minimax selects a move assuming an opponent chooses counter-moves to minimize your outcome. Alpha-beta pruning avoids exploring branches that cannot change the current choice. These concepts illustrate decision-making over possible future actions. They are less directly relevant to RAG but useful in game AI and planning.

## 8. Uncertainty and decision theory

A decision combines beliefs, actions and consequences. Expected utility is sum over outcomes of probability×utility. A rational decision under the model chooses the action with highest expected utility, but the probabilities, utilities and constraints can be wrong or incomplete.

A model score is not an action policy. For a high-impact choice, account for uncertainty, asymmetric harms, human review and the ability to defer. Decision thresholds should be justified by actual consequences and policy.

## 9. Reinforcement learning preview

Reinforcement learning models an agent interacting with an environment:
- state s;
- action a;
- reward r;
- next state s';
- policy mapping states to action distributions;
- return as accumulated future reward.

A Markov decision process assumes the current state summarizes information needed for future transitions/reward. The discount factor gamma controls how future reward is weighted. A value function estimates expected return from a state; a Q-function estimates return from state-action choices.

Exploration gathers information; exploitation uses current knowledge. Reward design is difficult: optimizing a proxy can produce unintended behavior. RL also faces delayed credit assignment, sample cost and safety challenges. Preference optimization for language models is related but has its own methods and assumptions; see [LLM training and adaptation](../../07-nlp-llms-and-generative-ai/04-llm-training-and-adaptation.md).

## 10. Symbolic AI and machine learning together

Symbolic methods are explicit and inspectable but can be brittle when rules/data are incomplete. ML handles statistical patterns but can be uncertain and hard to explain. Hybrid systems may use:
- deterministic authorization and invariants;
- model classification or extraction;
- retrieval of current facts;
- human review for uncertain/high-impact cases;
- explicit workflow state and audit.

For AI backends, keep trusted policy and irreversible actions in deterministic services. Use models for tasks where statistical inference adds value.

## Questions and answers

**Q: Is AI the same as machine learning?**  
A: No. AI is the broader field. Machine learning is one family of methods within it; search, planning, logic and constraint solving are other important approaches.

**Q: When does BFS give the shortest path?**  
A: When each edge/action has equal cost. If costs differ, uniform-cost search is the relevant basic algorithm.

**Q: Why does A* need a heuristic?**  
A: It ranks work using an estimate of remaining cost. Quality depends on whether the heuristic is informative and whether its assumptions preserve optimality.

**Q: Should an LLM plan and execute a business workflow?**  
A: Only with carefully bounded, authorized tools and explicit state/approval. Many workflows are safer as deterministic state machines with a model assisting at selected steps.

**Q: What is the key challenge in RL?**  
A: Learning action choices from delayed feedback while balancing exploration, reward design, sample efficiency and safety.

## Practice

1. Define states, actions, goal and cost for a help-desk routing workflow.
2. Compare BFS, DFS and UCS on a graph with unequal edge costs.
3. Design an admissible heuristic for a simple grid and explain why it cannot overestimate.
4. Express a document-read permission rule using predicates; identify what data authority supplies each predicate.
5. Formulate an RL problem for queue scheduling and list ways a reward could be gamed.

## Completion check

You can distinguish search, planning, logic, constraints, ML and RL; formulate a state-space task; explain BFS/UCS/A* conditions; and identify where deterministic policy should remain outside a model.