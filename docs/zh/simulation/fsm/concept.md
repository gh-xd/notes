# 有限状态机

## 目标

- 了解有限状态机的基本概念
- FSM用Python的简单实现

## 基本理解

- 有限状态机，也叫*状态机*或者*有限自动机*，是包含了一个状态集合（包括初始状态、一个或多个最终状态）、一个输入事件集合、一个输出事件集合以及状态转换函数的的抽象机器。

- 一个转换函数$q^+ = F(q, u)$接受当前状态$q$和输入事件$u$，返回一个输出事件的新集合和下一个状态

> A "Finite State Machine" (abbreviated FSM), also called "State Machine" or "Finite State Automaton" is an abstract machine which consists of a set of states (including the initial state and one or more end states), a set of input events, a set of output events, and a state transition function. A transition function takes the current state and an input event as an input and returns the new set of output events and the next (new) state. Some of the states are used as "terminal states". The operation of an FSM begins with a special state, called the start state, proceeds through transitions depending on input to different states and normally ends in terminal or end states. A state which marks a successful flow of operation is known as an accept state.



## 理论和记号

一个具有确定性的有限状态机基本的五元组为 $(\sum, S, s_0, \delta, F)$：

- $\sum$ 动作集合
- $S$ 有限的、非空的状态集合
- $s_0$ 是初始状态，$S$的一个状态
- $\delta$ 是状态转移方程 $\delta:\ S \times \sum \rightarrow\ S$，可以简单理解为事件触发有限集合的转变
- $F$ 是集合中最终状态，是$S$的一个子集



## Python案例1

函数式

1. 定义StateMachine，添加state、state转换函数、设置初始化state等
2. 手动编写state对应的转换函数
3. 定义系统输入、测试

```python
class T:
    """
    Generics
    type: action function
    """
    pass

class InitializationError(Exception):
    pass


class StateMachine:
    
    def __init__(self):
        self.handlers = {}
        self.startState = None
        self.endStates = []

    def add_state(self, name, handler: T, end_state=0):
        name = name.upper()
        self.handlers[name] = handler
        if end_state:
            self.endStates.append(name)

    def set_start(self, name):
        self.startState = name.upper()

    def run(self, cargo):
        try:
            handler = self.handlers[self.startState]
        except:
            raise InitializationError("must call the function .set-start() before run()")
        
        if not self.endStates:
            raise InitializationError("at least one state must be an end_state")
        
        while True:
            (newState, cargo) = handler(cargo)
            if newState.upper() in self.endStates:
                print("reached", newState)
                break
            else:
                handler = self.handlers[newState.upper()]

positive_adjectives = ["great", "super", "fun", "entertaining", "easy"]
negative_adjectives = ["boring", "difficult", "ugly", "bad"]      
        
def start_transitions(txt: str):
    splitted_txt = txt.split(None, 1)
    word, txt = splitted_txt if len(splitted_txt) > 1 else (txt, "")
    if word == "Python":
        newState = "Python_state"
    else:
        newState = "error_state"
    return (newState, txt)

def python_state_transitions(txt: str):
    splitted_txt = txt.split(None, 1)
    word, txt = splitted_txt if len(splitted_txt) > 1 else (txt, "")
    if word == "is":
        newState = "is_state"
    else:
        newState = "error_state"
    return (newState, txt)
    
def is_state_transitions(txt: str):
    splitted_txt = txt.split(None,1)
    word, txt = splitted_txt if len(splitted_txt) > 1 else (txt,"")
    if word == "not":
        newState = "not_state"
    elif word in positive_adjectives:
        newState = "pos_state"
    elif word in negative_adjectives:
        newState = "neg_state"
    else:
        newState = "error_state"
    return (newState, txt)

def not_state_transitions(txt: str):
    splitted_txt = txt.split(None,1)
    word, txt = splitted_txt if len(splitted_txt) > 1 else (txt,"")
    if word in positive_adjectives:
        newState = "neg_state"
    elif word in negative_adjectives:
        newState = "pos_state"
    else:
        newState = "error_state"
    return (newState, txt)

def neg_state(txt: str):
    print("Hallo")
    return ("neg_state", "")


m = StateMachine()
m.add_state("Start", start_transitions)
m.add_state("Python_state", python_state_transitions)
m.add_state("is_state", is_state_transitions)
m.add_state("not_state", not_state_transitions)
m.add_state("neg_state", None, end_state=1)
m.add_state("pos_state", None, end_state=1)
m.add_state("error_state", None, end_state=1)
m.set_start("Start")

# results
m.run("Python is great")
# >> reached  pos_state

m.run("Python is difficult")
# >> reached  neg_state

m.run("Perl is ugly")
# >> reached  error_state
```





## Python案例2

使用transition库

1. 定义state集合{$state_1,\ state_2,\ ...$}
2. 定义三元组，包含当前state，下一个state和转换函数 {$state,\  state^+,\ transition function$}

```python
from transitions import Machine

class Matter(object):
    pass

model = Matter()

# The states argument defines the name of states
states = ['solid', 'liquid', 'gas', 'plasma']

# The trigger argument defines the name of the new triggering methods
actions = [
    {'trigger': 'melt', 'source': 'solid', 'dest': 'liquid'},
    {'trigger': 'evaporate', 'source': 'liquid', 'dest': 'gas'},
    {'trigger': 'sublimate', 'source': 'solid', 'dest': 'gas'},
    {'trigger': 'ionize', 'source': 'gas', 'dest': 'plasma'},
]

machine = Machine(model=model, states=states, transitions=actions, initial='solid')

# Test
print(f'initial state: {model.state}')
model.melt()
print(f'action melt: -> {model.state}')
model.evaporate()
print(f'action evaporate:-> {model.state}')
model.sublimate()
print(f'action sublimate:-> {model.state}')
```



## 问题

- FSM和协程[Coroutine & FSM in Python](https://www.codementor.io/@arpitbhayani/building-finite-state-machines-with-python-coroutines-15nk03eh9l)

## 学习参考

- [8. Finite State Machine in Python](https://python-course.eu/applications-python/finite-state-machine.php)