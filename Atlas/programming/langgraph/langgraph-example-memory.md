---
parent: "[[langgraph-essential]]"
related:
  - "[[Programming]]"
  - "[[AI]]"
  - "[[code-example]]"
created: 2026-01-01
tags:
---
## Python

```python
import operator
from typing import TypedDict, Annotated, Literal
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.types import Command

# Define state
class State(TypedDict):
    nlist: Annotated[list[str], operator.add]

# Define nodes
def node_a(state: State) -> Command[Literal["b", "c", END]]:
    select = state["nlist"][-1]
    if select == "b":
        next_node = "b"
    elif select == "c":
        next_node = "c"
    elif select == "q":
        next_node = END
    else:
        next_node = END

    return Command(
        update = State(nlist = [select]),
        goto = [next_node]
    )

def node_b(state: State) -> State:
    return(State(nlist = ["B"]))

def node_c(state: State) -> State:
    return(State(nlist = ["C"]))

# Define memory
memory = InMemorySaver()
config = {"configurable": {"thread_id": "1"}}

# Build graph
builder = StateGraph(State)
builder.add_node("a", node_a)
builder.add_node("b", node_b)
builder.add_node("c", node_c)
builder.add_edge(START, "a")
builder.add_edge("b", END)
builder.add_edge("c", END)
graph = builder.compile(checkpointer=memory)

# Execute
while True:
    user = input('b, c, or q to quit: ')
    input_state = State(nlist = [user])
    result = graph.invoke(input_state, config )
    print( result )
    if result['nlist'][-1] == "q":
        print("quit")
        break
```
