# Wrappers

A wrapper in Gym/Gymnasium is just an environment that delegates its calls to another environment, potentially doing first some adjustments.

Why is it a good idea to introduce wrappers and when will we want to use wrappers?

The agents developed against Gym environments in mind, expect a specific interface. That is in particular the 'step' function. One is expected to pass as an argument to the 'step' function an action from the **action space** and one expects to receive in return an observation (the next observation) from the **observation space**. The **reward** that is returned from the 'step' function determines also if the agent is able to learn the desired behaviour, and is not less important from the observation and the learning algorithm itself.

What do you do when you want your agent to think of the problem a little different. What do you do if you want the action space to be a bit different?

I'll give an example. The environment expects a target destination on an '8x8' board game. You train an agent with your preferred RL library, and a suitable method, yet the agent seems to learn very slow. Let's assume that the agent avoids illegal moves, for example, by taking into account [action masking](../../concepts/action_masking).
Still the "learning" is slow.  
You brainstorm with friends and they point that your (agent) piece can only move {up, left, down, right}. You hypnotise that if the action space is changed to those four possible actions, the agent shall learn faster. To try, you quickly wrap the original environment, with an environment that declares its ‘action_space’ with above four (discrete) directions, and translates this action into the target square. You can now train an agent with the "new" environment and see if the learning is faster.

## ActionWrapper

While you can create a new environment to wrap the old environment, or also to create your custom wrapper based on the class *gym.Wrapper*, in above case you can actually base your wrapper on the class *gym.ActionWrapper*. If you base your wrapper on *ActionWrapper*, you only need to have the new *action_space* available, and to implement a function called *action*, that takes an action from the new action space (the one of the wrapper), and returns the matching action of the wrapped environment. For example, something I did in [qwertyenv](https://github.com/zbenmo/qwertyenv).

``` py
import gym
from typing import Tuple, Callable
import numpy as np


class UpDownLeftRight(gym.ActionWrapper):
  """
  A gym environment wrapper to enable U/D/L/R action space when the wrapped environment
  actually expects a destination in cartesian coordinates.
  """

  def __init__(self, env: gym.Env,
               get_current_location: Callable[[], Tuple[int, int]]):

    super().__init__(env)
    self.get_current_location = get_current_location

    self.action_space = gym.spaces.Discrete(4)

    self.udlr_action_to_board_action = {
      0: (0, -1), # up
      1: (0, +1), # down
      2: (-1, 0), # left
      3: (+1, 0), # right
    }

  def action(self, action: int) -> Tuple[int, int]:
    current_location = self.get_current_location()
    return tuple(np.add(current_location, self.udlr_action_to_board_action[action]))
```

Note, that you are very welcome to wrap your wrapped environment yet another time, and as many times are you need. A wrapped environment is still just an environment.

## ObservationWrapper

There are (at least) two additional useful wrapper base classes. The one is *ObservationWrapper*.
For example, you may wish to convert an RGB image into a monocolour image, as you believe it is simpler to learn from monocolour image.

You can also build a *ObservationWrapper* to enrich the observation by introducing a short history to capture motion. Think of a single frame from an ATARI game, say a ball is heading towards you. Can you tell if it is going to the left or to the right? If you are given a few additional frames from the past you may see the direction (frame stacking).

<figure>
  <img src="../../images/breakout - Made with Clipchamp.gif" title="ATARI Breakout"/>
  <figcaption>ATARI Breakout</figcaption>
</figure>

Wait, aren't we breaking the **Markov property** assumption? And what is the Markov property assumption to begin with?

In order to simplify the world a little, we assume that at any given moment the state of the environment represents everything that we need to know in order to act. What happened before with the environment, our (the agent's) past mistakes, nothing matters.
Wanted to use here "carpe diem", yet the expression is not appropriate as the future is very relevant. You do now what is right to do to the best of your policy, yet of course you need to take into account the future, and try to foresee it, and what will be the consequences of your current action. Regrets are not helpful, but planning of some sort is the essence of RL.

The environment, if indeed adheres to the Markov property, behaves, subject to randomness, similarly when we return to the same state, and take the same action. This is exactly what an agent needs to learn.
As we associate the reward mechanism with the environment, and we assume Markov property for the next state as well as for the reward, we can refer to the environment as a Markov Decision Process (__MDP__), which is exactly that, an environment with some dynamics that depends only on the current state, and the action the agent shall take. Note that the environment is often stochastic, and this does not contradic it being an __MDP__.

So, back to the question, did we break the Markov property assumption by introducing history?

By changing the observation space, we've created a new __MDP__, for which there are more states. So still an __MDP__ but a more detailed one. That engineering intervention can potentially help or it might harm the learning process. Yet to experiment.

## RewardWrapper

A third useful base class wrapper is *RewardWrapper*. Why to touch the reward? Maybe a reward of *+1*/*-1* at the end of the game is correct, but it takes the agent a lot of time to get to a win, and on the way getting no signal for progress is not helpful. We can add indication of good progress by giving some small rewards signals on the way. This shall be under **reward shaping**, which is a very important concept. Another reason that we may want to change the reward is in order to scale / normalize it for the benefit of our neural network learning optimization.
