# Motion Component Visualizer

This repository contains the source code for the [Motion](https://www.github.com/dm4ml/motion) component visualization tool, which is useful to understand the dataflows and state in a component.

## Usage

To use the tool, visit the [Github Pages site](https://dm4ml.github.io/motion-vis). You'll be prompted to upload a Motion component file. Once uploaded, the tool will display the component's state space and allow you to interact with it.

To get a Motion component file, you should run the CLI tool in the repository with your Motion component:

```bash
$ motion vis <filename>::<component_object>
```

For example, if I had a file called `main.py` like this:

```python
from motion import Component

ZScoreComponent = Component("ZScore")


@ZScoreComponent.init_state
def setUp():
    return {"mean": None, "std": None, "values": []}


@ZScoreComponent.serve("number")
def serve(state, props):  # (1)!
    if state["mean"] is None:
        return None
    return abs(props["value"] - state["mean"]) / state["std"]


@ZScoreComponent.update("number")
def update(state, props):  # (2)!
    # Result of the serve op can be accessed via
    # props.serve_result
    # We don't do anything with the results, but we could!
    value_list = state["values"]
    value_list.append(props["value"])

    mean = sum(value_list) / len(value_list)
    std = sum((n - mean) ** 2 for n in value_list) / len(value_list)
    return {"mean": mean, "std": std, "values": value_list}


if __name__ == "__main__":
    import time

    c = ZScoreComponent()  # Create instance of component

    # Observe 10 values of the dataflow's key
    for i in range(9):
        print(c.run("number", props={"value": i}))

    c.run("number", props={"value": 9}, flush_update=True)
    for i in range(10, 19):
        print(c.run("number", props={"value": i}))

    print(c.run("number", props={"value": 10}))
    time.sleep(5)  # Give time for the second update to finish
    print(c.run("number", props={"value": 10}, force_refresh=True))
```

I would run the CLI tool like this:

```bash
$ motion vis main.py::ZScoreComponent
```

This will generate and save a JSON file to the current directory. You can then upload this file to the [vis tool](https://dm4ml.github.io/motion-vis) visualize the component.
