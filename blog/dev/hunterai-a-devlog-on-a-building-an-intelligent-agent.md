---
description: Project Devlog
---

# HunterAI: A Devlog on a building an Intelligent Agent

## Introduction

This is my project devlog to create a _certain complex_ AI. In each part of this page, I will write about my journey progress in developing and upgrading the complexity of the AI program from the previous version.

## Latest State

{% embed url="https://www.youtube.com/watch?v=XRnqPfFGsps" %}

On 28 January 2020, I created a **reflex agent with internal state** in an imperfect information environment. Specifically, I created a top-down shooter AI with limited vision range to its surrounding. Using common pathfinding algorithm \(in this case, BFS\), the AI agent is able to determine an efficient way to discover monsters and position itself in the nearest shooting position.

{% embed url="https://github.com/adamyordan/hunterAI" caption="Source code: https://github.com/adamyordan/hunterAI" %}

## Part 1: Creating a simple Intelligent Agent

> Written on 2019-09-16

### Today's Goal

For the first part, I will try to make an intelligent agent that can hunt monster. I name this agent: **Hunter**. \(_In the future, this version will be referenced as **Alexander**_\)

### Defining "Agent"

Before we talk about building agent, let's understand what this "agent" is about. How can we build something if we do not what it is?

**What is Agent?**

Agent is anything that can be viewed as: **perceiving** environment through **sensors**; and **acting** upon environment through **actuators**. It will run in **cycles** of perceiving, thinking, and acting.

**What is an Intelligent Agent?**

An agent that acts in a manner that causes it to be as _successful_ as it can. This agent acts in a way that is expected to **maximise** to its **performance measure**, given the _evidence_ provided by what it perceived and whatever _built-in knowledge_ it has. The **performance measure** defines the criterion of success for an agent.

### Designing an Intelligent Agent

Now that we already understand what an intelligent agent is, how do we start designing the agent?

When designing an intelligent agent, we can describe the agent by defining _PEAS Descriptors_:

* **P**erformance -- how we measure the agent's achievements
* **E**nvironment -- what is the agent is interacting with
* **A**ctuator -- what produces the outputs of the agent
* **S**ensor -- what provides the inputs to the agent

Alternatively, we can describe the agent by defining _PAGE Descriptor_:

* **P**ercepts -- the inputs to our system
* **A**ctions -- the outputs of our system
* **G**oals -- what the agent is expected to achieve
* **E**nvironment -- what the agent is interacting with

Formally, we can define structure of an Agent as:

$$
Agent = Architecture + Program
$$

where:

**Architecture**: a device with sensors and actuators.  
**Agent Program**: a program implementing _Agent Function_ $$f$$ on top an architecture, where **Agent Function** is a function that maps a **sequence of perceptions** into **action**.

$$
f(P) = A
$$

### Agent Interaction Concept

I propose a concept idea of agent interaction as follow:

* Agent is represented as $$<Program, Architecture>$$ .
* Architecture is responsible for **perceiving** environment through **sensors** and **acting** upon environment through **actuators**.
* _Percept_ perceived by architecture is then passed to Agent Program. According to agent function, Agent Program then map the _percept_ to an _action_. _Action_ is then passed to Architecture.
* Environment is a representation that will be perceived and acted upon by architecture.
* Action is represented as $$<id>$$, where _**id**_ is used to identify the type of the action.
* Percept is represented as $$<state>$$, where _**state**_ is a key-value map containing the information perceived by sensor.

Let's start by making a simple abstract code implementing the above concepts.

{% code title="concepts.py" %}
```python
class Environment:
    pass


class Action:
    def __init__(self, id):
        self.id = id


class Percept:
    def __init__(self, state):
        self.state = state


class Architecture:
    def perceive(self, environment):
        raise NotImplementedError()

    def act(self, environment, action):
        raise NotImplementedError()


class Program:
    def process(self, percept):
        raise NotImplementedError()


class Agent:
    def __init__(self, program, architecture):
        self.program = program
        self.architecture = architecture

    def step(self, environment):
        percept = self.architecture.perceive(environment)
        action = self.program.process(percept)
        if action:
            self.architecture.act(environment, action)
```
{% endcode %}

### Let's start simple

Let's try implementing our **Hunter** as a simple reflex agent. Because this is our first attempt, let's simplify things:

* There is one monster in battlefield.
* We only care about the monster visibility. We don't care about its position.
* When a monster is visible, hunter will react by shooting it.
* When a monster got hit, it will disappear.

I know that this looks oversimplified and seems very easy. But this is important as our building foundation.

First, let's define our environment. According to our simplification, there is only one state attribute: `monster_visible`.

{% code title="environment.py" %}
```python
from .concepts import Environment


class HunterEnvironment(Environment):
    def __init__(self):
        self.monster_visible = False
```
{% endcode %}

Next, let's define our Hunter's Agent program. According to our simplification, when the agent perceives that a monster is visible, it will do shooting action.

{% code title="program.py" %}
```python
from .concepts import Action, Program


class HunterProgram(Program):
    def process(self, percept):
        if percept.state['monster_visible'] is True:
            return Action('shoot')
        else:
            return None
```
{% endcode %}

Next, let's define our Architecture. Perceiving is simple, it will check the `monster_visible` attribute of the environment. If shooting action is acted upon this environment, the monster will disappear, thus setting `monster_visible` into `False`.

{% code title="architecture.py" %}
```python
from .concepts import Architecture, Percept
import logging


class HunterArchitecture(Architecture):
    def perceive(self, environment):
        return Percept({'monster_visible': environment.monster_visible})

    def act(self, environment, action):
        if action.id == 'shoot':
            logging.debug('Pew! Shooting monster')
            environment.monster_visible = False
```
{% endcode %}

So far, we have already finished implementing our agent program and architecture. And this is sufficient to say that we already finished implementing our agent. Our agent can be instantiated with the following code:

```python
program = HunterProgram()
architecture = HunterArchitecture()
agent = Agent(program, architecture)
```

However, currently we don't have any way to run and test our agent. Let's make a simulator to run this agent.

{% code title="simulator.py" %}
```python
from .concepts import Agent
from .architecture import HunterArchitecture
from .program import HunterProgram
from .environment import HunterEnvironment
import logging
import os
import time


class Simulator:
    def __init__(self, environment, agents):
        self.environment = environment
        self.agents = agents
        self.time = 0

    def step(self):
        for agent in self.agents:
            agent.step(self.environment)
        self.time += 1

    def debug(self):
        logging.debug('monster_visible is %s', self.environment.monster_visible)

    @staticmethod
    def instantiate():
        environment = HunterEnvironment()

        program = HunterProgram()
        architecture = HunterArchitecture()
        agent = Agent(program, architecture)

        return Simulator(environment, [agent])
```
{% endcode %}

Our simulator class represents a _world_ where our agent is running. As you may realize, We have not yet defined what a _world_ is. Let's define **world** as a triplet $$<time, environment, agents>$$ , where _**time**_ is an increasing number that represents a moment in the world. A _world_ contains an _**environment**_ and a list of agent \(_**agents**_\) that will interact with the environment.

A moment in the world can be moved forward by calling `step()` function. When stepping, the agents will start processing the environment, perceiving and acting upon it; and the time will increase.

### Running our agent with simulator

Now that we finished programming our intelligent agent and simulator, let's run and play around with them.

Spawn a python interpreter, and import our `simulator.py` programs.

```python
$ python3

>>> from simulator import *

>>> logging.basicConfig(level='DEBUG')

>>> simulator = Simulator.instantiate()

>>> simulator.debug()
DEBUG:root:[time:0] monster_visible is False
```

We step one time unit period in our world simulator by calling `simulator.step()`.

```python
>>> simulator.step()

>>> debug(world)
DEBUG:root:[time:1] monster_visible is False
```

Let's try updating the environment. We want to see if our agent react accordingly when a monster appears.

```python
>>> debug(world)
DEBUG:root:[time:1] monster_visible is False

>>> simulator.environment.monster_visible = True

>>> debug(world)
DEBUG:root:[time:1] monster_visible is True
```

After running step, we will see that our **Hunter** should shoot the monster.

```python
>>> debug(world)
DEBUG:root:[time:1] monster_visible is True

>>> simulator.step()
DEBUG:root:Pew! Shooting monster

>>> debug(world)
DEBUG:root:[time:2] monster_visible is False
```

### Conclusion

Today, we learned about Intelligent agent and we have built a very simple intelligent agent. In the future, we will try to extend this agent to have more advanced features. I will also try to create a visualization for this agent. While this seems very simple, this is my first step towards creating _a certain complex_ AI program.

### Source Code

[https://github.com/adamyordan/hunterAI/tree/master/alexander](https://github.com/adamyordan/hunterAI/tree/master/alexander)

### References

* [https://hackernoon.com/rational-agents-for-artificial-intelligence-caf94af2cec5](https://hackernoon.com/rational-agents-for-artificial-intelligence-caf94af2cec5)
* [https://www.geeksforgeeks.org/agents-artificial-intelligence/](https://www.geeksforgeeks.org/agents-artificial-intelligence/)
* [http://www.cs.bham.ac.uk/~jxb/IAI/w4.pdf](http://www.cs.bham.ac.uk/~jxb/IAI/w4.pdf)



## Part 2: Integrating AI Agent with Unity

> Written on 2019-09-19

### Today's Goal

In this part, I will try to create a 3D visualization using Unity and connect it to our agent written in python. You may want to read the previous part about **Alexander** AI concept because we will reuse a lot of the concepts. \(_In the future, today's version will be referenced as **Barton**_\).

![Part 2&apos;s Milestone](../../.gitbook/assets/image%20%2820%29.png)

### Idea

I will still continue using the concepts introduced in previous part. However, there is an update that should be done to accommodate remote functionality.

Updated remote agent concepts in python side:

* **Remote environment**: remote environment is similar with common environment, a representation that will be perceived and acted upon by architecture. The differences are:
  * The architecture is not allowed to modify environment state directly. Action from architecture will be stored at `remote_action`.
  * Environment state is updated at every moment / step. This constant update is required because the Remote environment is actually a representation of the corresponding environment objects in the Unity side.
* **Remote architecture**: implementation-wise there is no difference with common architecture. However, there is an important concept that must be followed by remote architecture during actuating actions. Remote architecture should never modify environment state directly, instead it should use `set_remote_action(action)` function to pass the action \(`remote_action`\) to the Unity side. The action will be then realized by the corresponding agent object in the Unity side.

Next, We have to define a **remote communication module** that allows communication between the python and Unity side. One of the simplest approach is by serving a **python HTTP server**, and let Unity side send requests and receive responds from the python side.

![Communication Protocol for our AI python process and Unity](../../.gitbook/assets/image%20%2823%29.png)

The HTTP server behaviour is defined as follows:

* Endpoint `/init` will initiate the time, environment and agents. The initiation procedure is passed as `init_function` parameter when constructing `Remote` module.
* Endpoint `/step` will trigger the _world_ step, moving the time forward. This endpoint accepts environment state in JSON format. These actions will be executed:
  * Retrieve the environment state from Unity \(from the HTTP body\), then update the the python representation of remote environment will update its state accordingly to the passed state.
  * The agents will start processing the environment, perceiving the environment and producing `remote_action`.
  * The `remote_action` will be returned as HTTP Response. The Unity side will read this action and actuate it accordingly.
  * Increase time.

### The Unity

![Unity Editor of our AI visualization](../../.gitbook/assets/image%20%2821%29.png)

For illustration, I show the completed today's Unity project above. Quick explanation of what is happening in the Unity scene:

* The blue cube represents the `Hunter` \(player\) object.
* The red cube represents the `Monster` object.
* The blue cube can spawn `Projectile` object that will hit the red cube out, making it fall off the ground \(then destroyed\).

The main problem here is how to make `Hunter` object communicate with our Hunter agent in python? As discussed in the _Idea_ section, we are going to make the Unity communicate with our python with an HTTP client.

First, put an empty object \(`Simulator`\) into our Unity scene. Then, add `HunterSDK` and `SimulatorScript` as components to `Simulator` object.

The `HunterSDK` is essentially the code to communicate with our server protocol. It provides the `Initiate()` and `Step()` method. The `initiate()` method will send a request to `/init` endpoint, initiating agents and environment representation in python side. The `Step()` method accepts the current environment state in Unity, then send it in JSON format alongside a request to `/step` endpoint, and then pass the respond to the callback function. The respond should be containing the action to be actuated.

{% code title="HunterSDK.cs \(partial\)" %}
```csharp
public class HunterSDK : MonoBehaviour
{
    public string Host = "http://127.0.0.1:5000/";

    public void Initiate()
    {
        StartCoroutine(PutRequest(Host + "init"));
    }

    public void Step(State state, Action<StepResponse> callback)
    {
        string stateJson = JsonUtility.ToJson(state);
        StartCoroutine(PutRequest(Host + "step", stateJson, (res) =>
        {
            StepResponse stepResponse = JsonUtility.FromJson<StepResponse>(res);
            callback(stepResponse);
        }));
    }

    IEnumerator PutRequest(string uri, string data = "{}", Action<string> callback = null)
    {
        using (UnityWebRequest webRequest = UnityWebRequest.Put(uri, data))
        {
            webRequest.SetRequestHeader("Content-Type", "application/json");
            yield return webRequest.SendWebRequest();
            callback?.Invoke(webRequest.downloadHandler.text);
        }
    }
}
```
{% endcode %}

Next, the `SimulatorScript` is a behavioural code that should define the behaviour of our agents and environment in Unity side. Initially in `Start()`, it will run `HunterSDK.Initiate()`. Then, after a fixed interval, it will periodically call the `SimulatorScript.Step()` function. In this step function, you need to determine the current environment state with Unity functionality \(e.g. Calculate the visibility of monster\), then pass it to `HunterSDK.Step(state)`. Finally, you also need to define what should be done to actuate action accordingly \(e.g. Spawn a projectile\).

{% code title="SimulatorScript.cs" %}
```csharp
public class SimulatorScript : MonoBehaviour
{
    public float stepInterval;
    HunterSDK hunterSDK;

    void Start()
    {
		hunterSDK = gameObject.GetComponent<HunterSDK>();
        hunterSDK.Initiate();
        InvokeRepeating("Step", stepInterval, stepInterval);
    }

    void Step()
    {
        State state = new State();

        // todo: Sensor code block; fill in state values

        hunterSDK.Step(state, (stepResponse) =>
        {
            AgentAction agentAction = stepResponse.action;

            // todo: Actuator code block; actuate action accordingly

        });
    }
}
```
{% endcode %}

### Python Code

#### Remote Agent Concept

As discussed above, let's define the updated concepts in `barton/concept.py`. Note that we will still be reusing the old concepts from `alexander/concept.py`.

{% code title="barton/concepts.py" %}
```python
from alexander.concepts import Architecture, Environment


class RemoteArchitecture(Architecture):
    def perceive(self, environment):
        raise NotImplementedError()

    def act(self, environment, action):
        raise NotImplementedError()


class RemoteEnvironment(Environment):
    def __init__(self):
        self.remote_action = None

    def update(self, state):
        self.remote_action = None
        self.update_state(state)

    def set_remote_action(self, action):
        self.remote_action = action

    def get_remote_action(self):
        return self.remote_action

    def update_state(self, state):
        raise NotImplementedError()
```
{% endcode %}

#### Remote Communication Module

As discussed above, we will be serving a python HTTP server, and let Unity side send requests and receive responds from the python side. I use `flask` for its simplicity for running HTTP server. The HTTP server behavior is defined as follows:

* Endpoint `/init` will initiate the time, environment and agents. The initiation procedure is passed as `init_function` parameter when constructing `Remote` module.
* Endpoint `/step` will trigger the _world_ step, moving the time forward. This endpoint accepts environment state in JSON format. These actions will be executed:
  * Retrieve the environment state from Unity \(from the HTTP body\), then update the the python representation of remote environment will update its state accordingly to the passed state.
  * The agents will start processing the environment, perceiving the environment and producing `remote_action`.
  * `remote_action` will be returned as HTTP Response. The Unity side will read this action and actuate its accordingly.
  * Increase time.

The implementation is as follows:

{% code title="barton/remote.py" %}
```python
from flask import Flask, jsonify, request
from flask_cors import CORS


class Remote:
    class Controller:
        @staticmethod
        def ping():
            return 'pong'

        @staticmethod
        def init(remote):
            remote.init()
            return jsonify({'ok': True})

        @staticmethod
        def step(remote):
            state = request.get_json()
            response = remote.step(state)
            return jsonify(response)

    def __init__(self, init_function):
        self.init_function = init_function
        self.agents = None
        self.environment = None
        self.time = 0

    def init(self):
        self.environment, self.agents = self.init_function()
        self.time = 0

    def step(self, state):
        self.environment.update(state)
        for agent in self.agents:
            agent.step(self.environment)
        action = self.environment.get_remote_action()
        action_serializable = action.__dict__ if action is not None else None
        self.time += 1
        return {'action': action_serializable}

    def app(self):
        app = Flask(__name__)
        CORS(app)
        app.add_url_rule('/', 'ping', lambda: Remote.Controller.ping())
        app.add_url_rule('/init', 'init', lambda: Remote.Controller.init(self), methods=['PUT'])
        app.add_url_rule('/step', 'step', lambda: Remote.Controller.step(self), methods=['PUT'])
        return app
```
{% endcode %}

### Writing Hunter Agent with the Remote Function in Python

First, let's define our hunter architecture using the updated remote architecture concept. If you notice, the logic is still similar with the previous `alexander/architecture.py`. The difference is that instead of changing state `environment.monster_visible` directly, we will delegate the action to the Unity side through `remote_action`.

{% code title="barton/architecture.py" %}
```python
from alexander.concepts import Architecture, Percept
import logging


class HunterRemoteArchitecture(Architecture):
    def perceive(self, environment):
        return Percept({'monster_visible': environment.monster_visible})

    def act(self, environment, action):
        if action.id == 'shoot':
            logging.debug('Pew! Shooting monster')
        environment.set_remote_action(action)
```
{% endcode %}

Next, let's define our hunter environment using the updated remote environment concept.

{% code title="barton/environment.py" %}
```python
from .concepts import RemoteEnvironment


class HunterRemoteEnvironment(RemoteEnvironment):
    def __init__(self):
        super().__init__()
        self.monster_visible = False

    def update_state(self, state):
        if 'monster_visible' in state:
            self.monster_visible = state['monster_visible']
```
{% endcode %}

Finally, let's write our `main` file. In main, we will define our `init_function` that will instantiate the hunter environment and agent. Then we will pass the it to `Remote` module, and run the python remote server.

{% code title="barton/main.py" %}
```python
from .remote import Remote
from .architecture import HunterRemoteArchitecture
from .environment import HunterRemoteEnvironment
from alexander.program import HunterProgram
from alexander.concepts import Agent

if __name__ == '__main__':
    def init_function():
        environment = HunterRemoteEnvironment()
        program = HunterProgram()
        architecture = HunterRemoteArchitecture()
        agent = Agent(program, architecture)
        return environment, [agent]

    remote = Remote(init_function)
    remote.app().run()
```
{% endcode %}

We can start the python remove server by running the following shell command:

### Hunter Agent in Unity Side

Remember that we need to write the code to fill in environment state values and actuating agent action in Unity scene.

According to our current Hunter scenario, there is only one state information that we need to attain: monster visibility \(`monster_visible`\). Let's just implement that `monster_visible` is true if the monster object exists and the distance between player object and the monster object is less than 4 unit.

Next, in actuator code block, we check that if the action id is `"shoot"`, then we call the method `playerScript.Shoot()`. This method is essentially spawn a projectile and add a big force toward the monster object direction. By the physics of Unity engine, the monster object will be pushed backward by the projectile and it will fall off the ground.

I also add a method `SetAIEnabled()` to toggle the activation of the AI. This method is bound to the Toggle UI "Enable AI". There is also a `SpawnMonster()` method that will be executed when clicking the button "Spawn Monster".

The complete Unity Simulator script is as follows:

{% code title="SimulatorScript.cs" %}
```csharp
public class SimulatorScript : MonoBehaviour
{

    public GameObject monster;
    public GameObject player;
    public float stepInterval;

    bool aiEnabled = true;
    HunterSDK hunterSDK;
    PlayerScript playerScript;


    void Start()
    {
		hunterSDK = gameObject.GetComponent<HunterSDK>();
        playerScript = player.GetComponent<PlayerScript>();
        hunterSDK.Initiate();
        InvokeRepeating("Step", stepInterval, stepInterval);
    }

    void Step()
    {
        if (aiEnabled)
        {
            playerScript.ShowThinking(true);
            GameObject monsterGameObject = GameObject.FindGameObjectWithTag("Monster");

            State state = new State();
            state.monster_visible = monsterGameObject != null
                && Vector3.Distance(player.transform.position, monsterGameObject.transform.position) <= 4.0f;

            hunterSDK.Step(state, (stepResponse) =>
            {
                AgentAction agentAction = stepResponse.action;
                if (agentAction.id == "shoot")
                {
                    playerScript.Shoot();
                }
                Helper.RunLater(this, () => playerScript.ShowThinking(false), 0.1f);
            });
        }
    }

    public void SpawnMonster()
    {
        GameObject monsterGameObject = GameObject.FindGameObjectWithTag("Monster");
        if (monsterGameObject == null)
        {
            Instantiate(monster, new Vector3(2, 6, 0), transform.rotation);
        }
    }

    public void SetAIEnabled(bool isEnabled)
    {
        aiEnabled = isEnabled;
    }
}
```
{% endcode %}

### Demo & Conclusion

{% embed url="https://www.youtube.com/watch?v=P3zoo9JVg3w" %}

In this Part 2, we learned how to connect an Intelligent agent with a Unity scene. The Unity scene acts as a visualization or a frontend for our agent. In the future, I will keep using visualization in order to better showcase the progress of our agent. And the agent feels more like a real robot, right?

### Source Code

[https://github.com/adamyordan/hunterAI/tree/master/barton](https://github.com/adamyordan/hunterAI/tree/master/barton)



## Part 3: Top-down Shooter AI

> Written on 2020-01-28

### Today's Goal

In this part, I will try to create a **reflex agent with internal state** in an imperfect information environment. The agent is a top-down shooter with a limited vision range, therefore it needs to store information about the map it already visited. Furthermore, it needs to make a rational decision to find enemies and take necessary actions to shoot them.

_For future reference, today's version will be referenced as **Caleb**._

![Part 3&apos;s Milestone](../../.gitbook/assets/image%20%2824%29.png)

### The Scenario

We are going to expand from the previous part. In today's scenario, we are going to put our hunter in a grid of `M x N`.

* There will be some monsters placed at random coordinates.
* Hunter's vision range is limited. Thus making hunter to not have perfect information.
* Projectile range is limited. Thus, hunter needs to position itself accordingly to shoot monsters.
* In a moment, the actions that hunter can take are: 
  * $$MoveForward$$
  * $$RotateLeft$$
  * $$RotateRight$$
  * $$Shoot$$

### Demo

I put the finished demo in the early section of this part with hope of motivating the readers.

{% embed url="https://www.youtube.com/watch?v=XRnqPfFGsps" %}

### Finding Shortest Path with BFS

In this scenario, we are going to face a problem where we need to find the shortest path to multiple cells. For example, there are multiple monsters in the grid, and hunter needs to approach the nearest monster.

To solve this problem, we can use Breadth-first-Search \(BFS\); with Hunter's coordinate as starting point and monsters coordinates as goals. With BFS, since we are going to do level order traversal, the first goal that we reach is the nearest goal. We can then follow the path formed, which is the shortest path to the nearest goal.

![BFS to find nearest target](../../.gitbook/assets/image%20%2822%29.png)

{% code title="caleb/algorithm/bfs.py" %}
```python
def neighbors(current, grid):
    size_x = len(grid[0])
    size_y = len(grid)

    res = []
    if current[0] - 1 >= 0:
        res.append((current[0] - 1, current[1]))
    if current[0] + 1 < size_x:
        res.append((current[0] + 1, current[1]))
    if current[1] - 1 >= 0:
        res.append((current[0], current[1] - 1))
    if current[1] + 1 < size_y:
        res.append((current[0], current[1] + 1))
    return res


def bfs(start, goals, grid):
    """
    Using BFS to get path to nearest goal.
    """
    size_x = len(grid[0])
    size_y = len(grid)

    visited = [[False for _ in range(size_x)] for _ in range(size_y)]
    parent = [[None for _ in range(size_x)] for _ in range(size_y)]
    queue = [start]
    visited[start[1]][start[0]] = True

    while queue:
        current = queue.pop(0)
        if current in goals:
            path = []
            while parent[current[1]][current[0]]:
                path.append(current)
                current = parent[current[1]][current[0]]
            return path[::-1]
        for neighbor in neighbors(current, grid):
            if not visited[neighbor[1]][neighbor[0]]:
                queue.append(neighbor)
                parent[neighbor[1]][neighbor[0]] = current
                visited[neighbor[1]][neighbor[0]] = True

    raise ValueError('No Path Found')
```
{% endcode %}

Let's create a test script to check the correctness of our implementation.

{% code title="caleb/test.py" %}
```python
def test_bfs():
    grid_str = [
        '-M------',
        '------M-',
        '--M-----',
        '--------',
        '--------',
        '--------',
        '----H---',
        '--------',
    ]

    print('\nTesting: BFS to nearest monster\n')

    grid, hunter, monsters = parse_grid(grid_str)
    debug_grid(grid)

    while True:
        if hunter in monsters:
            break
        path = bfs.bfs(hunter, monsters, grid)
        next_hunter = path[0]
        set_content(grid, hunter, GridContent.EMPTY)
        set_content(grid, next_hunter, GridContent.HUNTER)
        hunter = next_hunter
        debug_grid(grid, refresh=False)


if __name__ == '__main__':
    test_bfs()
```
{% endcode %}

By running the test script, we can see that the hunter is going toward the monster on the left bottom, which is the nearest monster.

{% embed url="https://player.vimeo.com/video/387264527" %}

### The Agent Implementation in Python

I will reuse the concepts explained from previous parts.

#### Generalized Remote Architecture and Remote Environment Framework

Based on the idea in Part 2, let's make it more generalized for remote architecture and remote environment. So in the future we might be able to reuse the general piece of code.

For remote architecture, there are two functions that we must implement in an `Architecture` class.

* In `perceive` function, we just return a Percept instance containing the environment state.
* In `act` function, we just pass the action to remote environment through `set_remote_action`.

  ```text
  # caleb/architecture.py

  from alexander.concepts import Percept, Architecture


  class GeneralRemoteArchitecture(Architecture):
      def perceive(self, environment):
          return Percept(environment.state)

      def act(self, environment, action):
          environment.set_remote_action(action)
  ```

For remote environment:

* We store a state in a Key-Value data structure.
* We provide a function to update this state data: `update_state`.

  ```text
  # caleb/environment.py

  from barton.concepts import RemoteEnvironment


  class GeneralRemoteEnvironment(RemoteEnvironment):
      def __init__(self):
          super().__init__()
          self.state = {}

      def update_state(self, state):
          self.state = state
  ```

### Implementing the Hunter Agent Program

Next, we are going to write the Hunter agent program. First, let's define the actions that the hunter agent can possibly do. Also define the possible content of a cell in the grid.

```python
# caleb/program.py (partial)

class Actions(Enum):
    MOVE_FORWARD = Action('move_forward')
    SHOOT = Action('shoot')
    ROTATE_LEFT = Action('rotate_left')
    ROTATE_RIGHT = Action('rotate_right')

class GridContent(Enum):
    UNKNOWN = -1
    EMPTY = 0
    MONSTER = 1
    HUNTER = 2
```

Our agent is a **reflex agent with internal state**. The state information is initiated at `__init__()`. The `process()`, which implements the agent function, will perform three tasks when invoked:

* Update the agent's state to reflect new percept;
* Choose the best action;
* Update the agent's state according to action.

The code:

```python
# caleb/program.py (partial)

class HunterProgram(Program):
    def __init__(self):
        pass # todo: initiate default state here

    def process(self, percept):
        self.update_state_with_percept(percept)
        action = self.choose_action()
        self.update_state_with_action(action)
        return action

    def update_state_with_percept(self, percept):
        pass # todo

    def choose_action(self):
        return None # todo

    def update_state_with_action(self, action):
        pass # todo
```

Then, we are going to write the main file. In main, we define our `init_function` that will instantiate the hunter environment and agent. Then we pass the it to Remote module \(Read Part 2\), and run the python remote server.

```python
from alexander.concepts import Agent
from barton.remote import Remote
from .architecture import GeneralRemoteArchitecture
from .environment import GeneralRemoteEnvironment
from .program import HunterProgram


def init_function():
    environment = GeneralRemoteEnvironment()
    program = HunterProgram()
    architecture = GeneralRemoteArchitecture()
    agent = Agent(program, architecture)
    return environment, [agent]


if __name__ == '__main__':
    remote = Remote(init_function)
    remote.app().run(debug=True)
```

We can now run the hunter agent by running `main.py`.

```bash
$ python3 -m caleb.main
 * Serving Flask app "barton.remote" (lazy loading)
 * Environment: production
 * Debug mode: on
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

#### The CLI World Environment

Before building the world environment with Unity, we can first build an environment with CLI.

The `cli_world.py` contains an implementation of hunter world environment. When running, it will communicate with the Hunter agent via HTTP connection and update itself in an eternal loop. In each loop step, it will update itself, send the environment state to agent, and actuate the action received from agent to the environment.

```python
# caleb/test/cli_world.py

class CliWorld:

    def __init__(self, grid_str, hunter_face):
        self.grid, self.hunter, self.monsters = parse_grid(grid_str)
        self.hunter_face = hunter_face

    def run(self):
        self.debug(sleep=1)
        self.init()
        while True:
            self.update()
            state = self.get_state()
            step_response = self.step(state)
            self.actuate(step_response['action'])
            self.debug(refresh=False, sleep=1)

    def init(self):
        requests.put('http://localhost:5000/init')

    def step(self, state):
        r = requests.put('http://localhost:5000/step', data=json.dumps(state))
        return r.json()

    ...


if __name__ == '__main__':
    grid_str = [
        '-M-------',
        '------M--',
        '--M------',
        '---------',
        '---------',
        '---------',
        '----H----',
        '---------',
        '---------',
    ]

    print('\nTesting: Hunter Agent\n')
    world = CliWorld(grid_str, hunter_face=0)
    world.run()
```

The State passed from this CLI World is as follows:

```python
state = {
    'Vision': visionContent,
    'MonsterCount': len(self.monsters),
    'MapSize': {'x': map_size_x, 'y': map_size_y},
    'HunterPosition': self.hunter,
    'HunterRotation': self.hunter_face,
    'HunterProjectileDistance': 3
}
```

We can run the script and this CLI World will connect to our Hunter Agent program through HTTP connection. However, there is no functionality as of now.

```bash
$ python3 -m caleb.test.cli_world

Testing: Hunter Agent

  - M - - - - - - -
  - - - - - - M - -
  - - M - - - - - -
  - - - - - - - - -
  - - - - - - - - -
  - - - - - - - - -
  - - - - H - - - -
  - - - - - - - - -
  - - - - - - - - -
```

### Implementing the Agent Logic

#### Scanning Grid with Limited Vision

We are going to implement the algorithm of how our Hunter Agent can explore the grid. Because our agent has limited vision, it does not have perfect information about the contents of every cells. Thus, it need to store the content of the cells already seen. We store it in `self.grid` variable.

In `update_state_with_percept`, we update the internal state with the percept received from environment. We also update the `grid` variable with the `Vision` percept.

```python
def update_state_with_percept(self, percept):
    if not self.initiated:
        self.grid_size = percept.state['MapSize']
        self.grid = [[GridContent.UNKNOWN for _ in range(self.grid_size['x'])] for _ in range(self.grid_size['y'])]
        self.initiated = True

    self.hunter_position = percept.state['HunterPosition']
    self.hunter_rotation = percept.state['HunterRotation']
    self.monster_count = percept.state['MonsterCount']
    self.hunter_projectile_range = percept.state['HunterProjectileDistance']
    for vision in percept.state['Vision']:
        x, y = vision['Position']
        self.grid[y][x] = GridContent(vision['Content'])
```

Then, in `choose_action`, we will return an action to move towards the nearest unknown cell. Using the BFS algorithm we discussed earlier, we can know the shortest path to the nearest unknown cell. Then, the hunter is heading to the first step in the shortest path, while considering rotation action if necessary.

```python
def get_unknown_cells(self):
    unknown_cells = []
    for y, row in enumerate(self.grid):
        for x, col in enumerate(row):
            if col == GridContent.UNKNOWN:
                unknown_cells.append((x, y))
    return unknown_cells

def action_move_towards(self, cells):
    path_to_target = bfs.bfs(self.hunter_position, cells, self.grid)
    if path_to_target:
        next_cell = path_to_target[0]
        rotate_action = self.calc_rotate_action(next_cell) # check if rotation is necessary
        return rotate_action or Actions.MOVE_FORWARD.value

def choose_action(self):
    if self.monster_count > 0:
        unknown_cells = self.get_unknown_cells()
        return self.action_move_towards(unknown_cells)
```

As of now, our Hunter Agent can already do basic grid scanning function. With this function, the agent can gain more information inside the imperfect information environment.

{% embed url="https://player.vimeo.com/video/387417991" %}

#### Shooting Monsters

Now we are going to implement the functionality for our Hunter agent to shoot monsters.

The idea is as follows:

* List the coordinates of monsters already sighted.
* If no monsters are sighted yet, move towards the nearest unknown cell.
* Else, going to shoot the nearest monster:
  * Calculate the possible shooting position cells. The shooting position is a coordinate in which the Hunter can shoot a monster in a direction.
  * If hunter already in shooting position, shoot the target. Do rotation if necessary.
  * If hunter is not yet in shooting position, use BFS to move towards the nearest shooting position.

The related code:

```python
# caleb/program.py

def get_monster_cells(self):
    monsters_sighted = []
    for y, row in enumerate(self.grid):
        for x, col in enumerate(row):
            if col == GridContent.MONSTER:
                monsters_sighted.append((x, y))
    return monsters_sighted

def get_shooting_position_cells(self, monsters_sighted):
    target_cells = []
    target_cells_monster = {}
    for x, y in monsters_sighted:
        for i in range(1, self.hunter_projectile_range + 1):
            target_cells_curr = [(x + i, y), (x - i, y), (x, y + i), (x, y - i)]
            for target_cell in target_cells_curr:
                target_cells.append(target_cell)
                key = '%d,%d' % (target_cell[0], target_cell[1])
                target_cells_monster[key] = (x, y)
    return target_cells, target_cells_monster

def action_shoot_monster(self, target_cells_monster):
    key = '%d,%d' % (self.hunter_position[0], self.hunter_position[1])
    monster_cell = target_cells_monster[key]
    rotate_action = self.calc_rotate_action(monster_cell)
    return rotate_action or Actions.SHOOT.value

def choose_action(self):
    if self.monster_count > 0:
        unknown_cells = self.get_unknown_cells()
        monsters_sighted = self.get_monster_cells()

        if len(monsters_sighted) == 0:
            return self.action_move_towards(unknown_cells)
        else:
            target_cells, target_cells_monster = self.get_shooting_position_cells(monsters_sighted)
            if self.hunter_position in target_cells:
                return self.action_shoot_monster(target_cells_monster)
            else:
                return self.action_move_towards(target_cells)
```

As of now, our agent is already fully working. If we connect our agent with the previous `cli_world` environment, we can see that our agent is working as expected.

{% embed url="https://player.vimeo.com/video/387427033" %}

### The Unity World Environment

Now that we already have a fully working agent in CLI environment, let's make a 3D World environment visualized using Unity. Please take a loot at Part 2 to understand the mechanic to connect Unity visualization with our python agent program.

![The Unity Editor](../../.gitbook/assets/image%20%2819%29.png)

As shown in the image above, I already made a Unity scene containing a grid of `M x N` cells. The next thing interesting to discuss is the `simulation.cs` which contains the implementation of our simulation program logic.

* In `Initiate()`, we are going to initiate the connection with our agent program via `HunterSDK.Initiate()`. Then, we will invoke `Step()` to be run every `SdkStepInterval` seconds \(default: 0.1\).
* In `Step()`:
  * Get state from Unity environment,
  * Send the state via `HunterSDK.Step()`
  * Receive the action from the agent response, then actuate the action to Unity environment.

The code:

```csharp
// simulator.cs

public class Simulator : MonoBehaviour
{
    public void Initiate()
    {
        ...
        if (SdkAiEnabled)
        {
            hunterSDK = new HunterSDK<State, StepResponse>(this, SdkHost);
            hunterSDK.Initiate();
            InvokeRepeating("Step", 2.0f, SdkStepInterval);
        }
        ...
    }

    void Step()
    {
        if (SdkAiEnabled)
        {
            TheHunter.ShowThinking(true);
            State state = GetState();
            hunterSDK.Step(state, (stepResponse) =>
            {
                AgentAction action = stepResponse.action;
                ActuateAction(action);
                Helper.RunLater(this, () => TheHunter.ShowThinking(false), 0.1f);
            });
        }
    }

    public State GetState()
    {
        List<GridWithContent> vision = getVision();

        State state = new State();
        state.Vision = vision;
        state.MonsterCount = monsterObjs.Count;
        state.MapSize = TheGridManager.mapSize;
        state.HunterPosition = TheHunter.targetPosition;
        state.HunterRotation = Mathf.RoundToInt(TheHunter.targetRotation.eulerAngles.y);
        state.HunterProjectileDistance = HunterProjectileDistance;

        return state;
    }

    ...
}
```

I also created a simulation option panel that allows user to configure the simulation option. For example, user can configure the grid size, the simulation step speed, or the hunter vision range. This allows us to experiment further, such as knowing the optimal vision range \(in case having long vision range costs something else\).

![Option Panel](../../.gitbook/assets/image%20%2818%29.png)

#### The Challenges with Unity: Different Position Indexing

In Unity 3D, there are three axis in coordinates, i.e. `x`, `y`, `z`, which consecutively represents horizontal position, vertical position, and depth position. Because we are interested in 2D grid, we can ignore `y` coordinate, thus focusing in `x` for horizontal axis in grid, and `y` for vertical axis in grid.

But there is another problem related in axis positional value. In our python agent program, the grid is represented vertically in _X_ axis from index `0` to `M`, and horizontally in _Y_ axis from index `0` to `N`. Meanwhile, in Unity, the position grid is represented vertically in _X_ axis from index `-M/2` to `M/2`, and horizontally in _Y_ axis from index `N/2` to `-N/2`. This inconsistency proves to become a problematic hassle during implementation, making implementation prone to indexing errors.

![Different indexing in Unity and our python program](../../.gitbook/assets/image%20%2817%29.png)

One way to solve this problem is by adding a position translation in python side:

```python
def translate_pos(self, x, y):
    return self.grid_size['x'] // 2 + x, self.grid_size['y'] // 2 + y
```

### Demo & Conclusion

{% embed url="https://www.youtube.com/watch?v=XRnqPfFGsps" %}

In this Part 3, we learned how to create a **reflex agent with internal state** in an imperfect information environment. Specifically, we created a top-down shooter AI with limited vision range to its surrounding. Using common pathfinding algorithm \(in this case, BFS\), our AI agent is able to determine an efficient way to discover monsters and position itself in the nearest shooting position.

Combined with the configurability of our visualization, I believe this is a good learning material in learning how to implement and playing around basic AI.

### Source code

[https://github.com/adamyordan/hunterAI/tree/master/caleb](https://github.com/adamyordan/hunterAI/tree/master/caleb)

### Reference

* [https://people.eecs.berkeley.edu/~russell/aima1e/chapter02.pdf](https://people.eecs.berkeley.edu/~russell/aima1e/chapter02.pdf) \(Used as reference when implementing reflex agent function\)

