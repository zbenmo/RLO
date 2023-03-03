# Spaces

Gym environment defines **observation space** and **action space**. Think of those as the “types” of the input and the output for the agent’s decision. The spaces also communicate the range or set of possible values. 

This makes a lot of sense as when one develops an agent to interact with that environment, the agent needs to be designed to expect observations of, for example images / pixels (ex. an ATARI game), or the output of some sensor, for example representing an angle. The actions of the agent may themselves be a discrete choice from a finite (small) set of possible values, as an example, yet it can also be a collection of values.

    self.action_space = spaces.Discrete(3)
    self.observation_space = spaces.Box(0, 1, shape=(2,))

In above example, the action space is 3 possible actions, {0, 1, 2}, standing for example for: {left, right, none}. It is up to the environment to translate the given integer number from the set to the matching action. The observation space in the example above is a vector of two entries, each taking a value from [0, 1]. You can imagine it as a location of a joystick in a 1 by 1 squared box.

The reward is a single 'float' number. ‘truncated’ and ‘terminated’ (or ‘done’) are of type 'bool'. There is also a general purpose output from the ‘step’ function (and also from 'reset'), that is referred to as ‘info’ and is a dictionary with whatever values the environment wants to communicate to the outside world in addition to the fixed contract given by the observation space and the action space.

In the example of a Chess game, we may want to communicate to the agent where are its pieces, where are the other player’s pieces, and in this way unify the cases when the player is the white and when it is actually the black. In Chess we may also want to communicate if specific moves, such as castling, are still valid.

What would be the action space for a Chess agent? A naïve approach can be to pick a piece (a square), and move it to another square. A lot of those “actions” are just moving air. A lot of them are trying to move the opponents pieces, or to move a piece of yours to an illegal place. Yet still by this approach of choosing two squares, we may make a simple interface to an RL agent implementation.

And so spaces are both the type of the inputs and the outputs, and also the possible values that are accepted. Is it a finite discrete choice, ex. {up, right, down, left}, or two numbers, each of which from [-3.1, 3.8], etc. One can define spaces that are a combination of values of complimentary meaning, values, and types. The combinations are defined, among other options, with *Dict* spaces.

The environment should define it observation space and its action space based on its needs, however, once defined, we need to find RL algorithm implementation that supports the required observation and action spaces. 

For example, if the environment wish to provide images as observations, we may need an RL implementation that have maybe a CNN. If the action space is discrete we may have the output to be a fixed size layer with softmax activation. *Dict* observations may require some software construct that shall combine those in some clever way to the inputs of a NN for example.

A nice feature of spaces is that you can sample from them. Which means that you can play with an environment, even before having any agent. For example (taken from Gymnasium documentation):

    import gymnasium as gym
    env = gym.make("LunarLander-v2", render_mode="human")
    observation, info = env.reset()

    for _ in range(1000):
        action = env.action_space.sample()  # agent policy that uses the observation and info
        observation, reward, terminated, truncated, info = env.step(action)

        if terminated or truncated:
            observation, info = env.reset()

    env.close()

Note that sampling from the observation space may return "images" that should never actually be returned from the environment (ex. complete random noise). That is because the observation space is wider than the actual distribution of values the agent will actually see. The action space may also be wider than the legal actions. If you stample on such an issue, you may want to read about [action masking](../../concepts/action_masking).

Another item to make our lives even a bit more interesting, is for example a game like Contract Bridge where you have multiple stages. In Contract Bridge we have the bidding, and the rest of the game. The observation space as well as the action space of an environment are fixed. It is up to you to come up with a setting that can handle all the stages. For example by having an agent for stage, each interacting with one environment dedicated for that stage, or by concatenating somehow the action apaces, and adding in the observation some hint in which stage we are. Or also as mentioned above by using [action masking](../../concepts/action_masking).

As far as I know, most environments do not allow for setting their state explicitly, but rather they either start from the same point (ex. a Chess game), or start from a random state (ex. a Contract Bridge cards shuffle). If relevant, you may hack your environment to start from a specific state, this would be just software engineering.