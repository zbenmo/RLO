# Creating a new Gymnasium environment

Developing a new Gymnasium / Gym environment involves subclassing *gym,Env* (potentially you declared ```import gymnasium as gym```). You need to implement the following:

- reset
- step
- render

In addition, your environment should have the following two attributes / member variables: *observation_space* and *observation_space*. For example in the initialization of your environment you can have (copied from Gymnasium documentation):

``` py
    ...
    self.observation_space = spaces.Dict(
        {
            "agent": spaces.Box(0, size - 1, shape=(2,), dtype=int),
            "target": spaces.Box(0, size - 1, shape=(2,), dtype=int),
        }
    )

    # We have 4 actions, corresponding to "right", "up", "left", "down"
    self.action_space = spaces.Discrete(4)
    ...
```

'reset' should bring the environment to some initial state. It might involve randomness if relevant, or just to the starting position of, for example Chess game. 'reset' returns the initial observation.

For Gymnasium, the signature of 'reset' looks as follows:

``` py
def reset(self, seed=None, options=None):
  ...
```

It is a good idea to pass the 'seed' to 'super' even if no randomness is expected, to avoid environment checks failure.

``` py
def reset(self, seed=None, options=None):
  # We need the following line to seed self.np_random
  super().reset(seed=seed)
  ...
```

As mentioned above 'reset' should return the first observation and I'm adding here, that also a "free form" info.
For example:

``` py
def reset(self, seed=None, options=None):
  ...
  return self._get_obs(), self._get_info()
```

It is a good idea to "wrap" the logic of constructing the observation from the state in a function as you'll need it also in 'step'. The information output can be just an empty dict, ```{}```, if you wish.

'step' receives and action and should return a tuple with the following: the next observation, the reward, was the environment terminated, was the environment truncated, and again some info.

So for example:

``` py
def step(self, action):
  ...
  reward = 1
  done = False
  info = {'mask': [True, False, False, True]} # an interested agent can use this mask to avoid illegal moves
  return self._get_obs(), reward, done, False, info
```

'render' should for example nicely print the environment's state to the console.

My suggestion would be to develop your environment without thinking about RL nor about Gym. Let's call this environment the "native" environment.
Try to make your "native" environment passive in the sense, that in order to use it, one needs to call the methods of the environment from a loop. I've implemented an environment I'm calling "Collect Coins". See in [qwertyenv](https://github.com/zbenmo/qwertyenv).

My environment is a turns game, two players, similar to Chess. The aim of the game is to collect more coins from the board than the other player. I've started by creating the "game" in a separate Python file. This is not a Gym environment, yet it enabled me to "play" in turn the white and then the black and so on till the game is over. I've added a check that you are playing the right player when you call the "play" function. Added also a verification that the play is legal. I have no reward mechanism other than allowing the user of the "game" to see the board, the current counts of coins, and if the game is done, who won.

Now, to make it into an environment, I had to do the following:

- Keep an instance of the "game" in my environment. In 'reset' I could opt to create a new instance, or to call the equivalent on the *self.game* instance. I created a new instance of the "game" on each 'reset'.
- Find the Gym spaces that can be mapped to my "game" board and status, and the possible moves. For example, in my game, the board is a boolean 8x8 matrix where you still have coins, and in addition you have the location of the player and the location of the other player. Those three pieces of observation went into *gym.spaces.Dict*. Coming to think of it, an agent may wish to see the count of coins. I did not made this freely available for the agent in a straight manner in this version.
- The action space, I've decided to have as the target square. The reason is that in my game you can have various Chess pieces (this is determined in the initialization of the game). I have at the moment knights and rocks. The rocks are actually a limited version, where they can be moved only to a neighbour square. Maybe I'll add in the future also a "real" rock. By having an action space of the target square, I've avoided having two different action spaces for each. I'm just telling what I've done, don't take from that that it was a smart decision.  
- Develop a reward mechanism. In my first iteration I went for +1 for a win, 0 for draw, and -1 for a loss. 
- Now I had an engineering challenge. Let's say I let the external agent act as the first player (white). But where will the other player come from? This had to be implemented in the environment. So towards the end of the 'step' method, I've took a move for the other player. In the 'reset' this logic was also used if the external agent is the black and needs to move second.
- The 'render' functionality was implemented in the "native" game environment.

If you build a Python package, it is a good practice to 'register' your Gym environment, so that one can later find it for example with ```env =  gym.make('qwertyenv/CollectCoins-v0', pieces=['rock', 'rock'])```

A registration is done usually in the *\_\_init\_\_.py* of the package:

``` py
from gym.envs.registration import register

...
register(
    id='qwertyenv/CollectCoins-v0',
    entry_point='qwertyenv.collect_coins:CollectCoinsEnv',
    max_episode_steps=300
)
...
```

Here are some lessons I learned.

Make sure to debug the "native" environment (*assert*s, *pytest*). :smile:

For your Gym environment, write as a minimum, a *pytest* to call:

``` py
from gymnasium.utils.env_checker import check_env
..

...

def test_my_env():
  ..
  check(env)
```

The *observation_space* and the *action_space*, have in addition to "sample", also a very useful functionality for debugging. One can check if the space "contains" a specific value that he/she wants to return as an observation, or pass as an action. When the relevant space does not contain the value, this must be first solved by either changing the space, or realizing that the action or observation indicate a potential bug.

Above enabled me to test with SB3 (I wrote it for Gym '0.21.0'). I will tell more about my experiments when discussing Gym wrappers and PettingZoo (I will write about it soon......)
