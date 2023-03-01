# Gym / Gymnasium
When we dive into RL we often discover that the environment is at least important as the exact algorithm. Not to say that all algorithms are suitable in all situations, but most of us will probably use an existing RL library and hope for the best.

The environment on the other hand is where we’ll spend our time to match it to the task we want to achieve.
Gym / Gymnasium (Open AI / Farama) is currently the de-facto standard for Python environments. A lot of RL Python packages for agents / policies are dependent on Gym. For example ‘stable-baselines3’ (sb3 for short) is dependent on ‘gym’, among other packages such as ‘torch’.

According to Gym, an environment has the following functionality:

-	Initial_obs = reset()

-	obs, reward, done, info = step(action)

-	render()

In addition the environment defines the ‘observation_space’ and the ‘action_space’.
An agent intended to be used with a specific Gym environment, should be adjusted to the relevant ‘observation_space’ and the relevant ‘action_space’.
Such and agent should provide the functionality to select and action given an observation. The learning setting involves loops where the observations, actions, rewards, and new observations are taken into account to update the policy of the agent based on the relevant RL learning algorithm.

Note that the time according to Gym is discrete, in each time step the agent decides on an action based on the current observation and this action is fed into the environment’s ‘step’ function.

The Gym environment is intended for training a single agent. A lot of times this is just what we need. Even if, for example the environment represents Chess game, and the agent represents the white, then the agent gets a chance to make the first move, then the environment also takes the action for the black. The new observation shall include both moves. Assume for now that we already have some good Chess software that can be used by the environment to play the opponent.

If we do want to have multiple agents for a turn-play, or for a simultaneous play with multiple agents, there is an extension of Gym called ‘PettingZoo’. In PettingZoo, one have an iterator over all the agents and is expected to call step for each.

The current state-of-the-art is a bit of a mess. Gym environment’s API was modified so that ‘step’ now returns instead of ‘done’, two new outputs: ‘terminated’, ‘truncated’. To indicate what exactly happened, whether we reached the “goal” or we’ve passed the allowed number of steps.
Gym ‘0.21.0’ is still with the ‘done’.

After that version we have:

-	obs, reward, terminated, truncated, info = step(action)

On top of that Open AI donated the code to Farama foundation, and the name was changed to Gymnasium. So you’ll sometimes in Python codes ‘import gymnasium as gym’.
When you start playing with RL, for example with ‘stable-baselines3’, you most likely want to make sure to use Gym with version ‘0.21.0’ or what sb3 depends on at the time.

If you want to create a new environment to represent your challenge, you can consider working with Gymnasium, yet make sure you have a matching RL package / code. You can use Tianshou or you can copy a Python (one file) implementation from CleanRL and do the relevant adjustments.

I want to add a side note that I find Gym environment suggested API and in fact the standard, to leave the taste of something missing. I find it horrifying that the existing RL packages are forced to make Gym to be their dependency. A better setting, for me, would be a loosely coupling between the package for the environment and the package for the RL agents / algorithms / training.

At the moment Gym / Gymnasium is here to stay, and you must wrap your head around it, what this is the way it works. There are benefits. All RL packages adjust their API to Gym environments so you can quickly try them out on your favorite environment.
