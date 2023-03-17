# Reinforcement Learning

We want to build software that automates tasks or acts on our behalf in various situations.
This is nothing new. We have build software for some time now. We have came up with algorithms and engineering solutions.

The future lies apparently with **machine learning**. We can build parts of our software systems in new ways. By **training** software from examples.
So instead of figuring how to do stuff, we communicate to the software what we want, and the software is built automatically from this description.

For example for teaching software to distinguish between cats and dogs in images we provide examples of cats and of dogs and we use **supervised learning**. This is a machine learning technique with which a **model** slowly picks up the statistical hints from the labeled examples till it can fulfil the task of predicting correctly the labels on unseen examples, and hence we have achieved a software solution that can carry on from now in telling dogs from cats (and humans can enjoy and supervise while the machines do the hard work).

Reinforcement learning is a similar concept. We want software that can play Chess against us, or control a self-driving car, and more. Note that here there are multiple steps. Not just a single decision cat or dog, but rather a series of **actions**. The software “wins” only if all its moves led to a victory. The car needs to get safely to the destination, following the traffic rules and in a reasonable time. Accelerating, slowing, changing the wheel orientation, all need to contribute for this complex task.

To some degree we can employ again supervised learning. In each situation, we can label the desired actions, for example by recording the actions of a human driver. We can then train a model to interpolate desired actions in similar situations. There are many Chess games to learn from, we can try to train a model based on those games. 

Reinforcement learning is a complementary technique to train models or here we’ll call those **agents**. Reinforcement learning may be more suitable for those control settings. For example with Chess a move is actually a part of a plan. Learning to repeat a move without gasping what needs to be done later may not work without extensive training set that may not be available, or is too hard to achieve with supervised training. The approach with reinforcement learning is to let the agent learn from interacting with the **environment**.

The environment produces **observations** and the actions of the agent are communicated to the environment. The state of the environment changes and the agent needs to learn to make smart decisions about its actions. The agent learns this time not from labels but rather by a **reward mechanism** that we provide to map a transition from one **state**[^1] in the environment to another state and its consequences.

The agent should learn a **policy** to map an observation to the best action as to be rewarded as much as possible during the **episode**. In other words, the agent shall attempt to maximize the cumulative reward to some time horizon. For example the agent shall try to learn to win the Chess game, or to bring the passengers safely to their destination, while avoiding fines and in a reasonable time. There is much to talk about reward and how to design the reward mechanism. If in supervised learning one needs to make sure the labels are correct here with reinforcement learning, the reward mechanism is the equivalent.

Note that the environment replaces the training set. By interacting with the environment, the agent can receive infinite feedback. There are some challenges however. We don’t want a car to self-drive in the real world before it is ready. We should have some simulator. The simulator should be very close to the real world and should reflect what happens in the real world. The reward mechanism that is often also associated with the environment should be relevant to learning the desired **behavior** for the agent.

Another very important observation here is that the experience that the agent collects depends on the agent's actions. If a robot keeps going to the left, it will never find out what is there on the right. While in supervised learning we expect the samples in the training set to be of the same distribution (identically independently distributed), the interaction of an agent with an environment are very much dependent on the agent's actions and hence cannot be assumed to be of the same distribution.

[^1]: A state may mean the same thing as an observation. In many settings the observation is a (partial) derivative of the state, in the sense that the environment's state is more complete and more accurate than the observation that is available to the agent.