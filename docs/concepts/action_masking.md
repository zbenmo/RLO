# Action Masking

When we define an **action space** for the environment we develop, we set the rules by which an agent can influence its own destiny in the environment.
This action space should match the capabilities of the agent and should be relevant for the environment.

For example, if the observation space is the cell’s cartesian coordinates (x, y) in a maze where the agent finds itself, then the action space can be, as an example, {up, right, down, left}. It is convenient for the next steps in implementing the agent, that the action space is simple and fixed across all the different observation, which lends itself to a simpler deep neural network architecture. For example one with a fixed-sized output layer of action probabilities.

An issue arise when in different situations there are illegal moves. Now we are left to solve the situation in which the agent wants us to communicate to the environment an illegal action. Imagine an illegal Chess move such as moving the agent's Rock to the square where its second Rock is placed. How to address this scenario?

A possible solution in such a situation would be to terminate an episode, giving the agent a big negative reward. We hope that the agent learns to avoid illegal moves. A Gym wrapper can be developed that simply ignores an invalid action, bypasses the original environment, and just returns the current observation, the negative reward, and the terminated flag.
This may work eventually yet it is somewhat wasteful in the sense that we are in a position to communicate to the agent the rules of the game (environment). This may be somewhat an abuse of AI. An analogy would be to train an agent to multiply integers using supervised learning techniques. We’ll wind up with less than perfect capable agent, after having invested resources on things that should be addressed much simpler. On the other hand the agent may learn to avoid being stuck in a corner, as an example. So keep the option of returning a negative reward and terminating, also in your mind when designing an RL environment / agents.

Another option that comes to mind is to ask the agent for an alternative action, or to completely ignore the agent’s wish and take a random (legal) action if the agent fails to provide a valid action. For example use Gym ActionWrapper to achieved above. Care should be taken to communicate the action that was actually taken to the trainer so that the right conclusion shall be drawn from the relevant  interaction of the agent in the environment. The risk with this approach is that we protect our “baby” agent from facing the real world, which may have consequences of its ability to learn sophisticated behaviors.

Eventually we are getting to **action masking**. The idea is that the environment, or a wrapper of the environment shall communicate to the agent what are the possible moves when the agent is about to take an action. So while the agent is born with the capabilities to act for example {up, right, down, left}, it shall be told, "but now you should only choose from {right, left}". In order to achieve this functionality we need both the environment or a wrapper of it to add the legal moves to the observation, and also we need our implementation to be able to take this additional information. For example as an expected optional field in the observation dictionary (as is done in Tianshou), or as a extra parameter for the act / predict / forward call.

What happens behind the scenes? One way to address it in the RL library is to combine the mask with the outputs and to make the illegal moves get (almost) zero probability. The property of differential loss is kept and the learning by stochastic decent or by its variant optimization techniques is used. There are additional ways to address that. See for example the following research / survey paper [[1]](#1). 

## References
<a id="1">[1]</a> 
[A Closer Look at Invalid Action Masking in Policy Gradient Algorithms](https://arxiv.org/abs/2006.14171).
