---
theme: seriph
title: 'Feasibility of Reactive Aggregate Programming via Kotlin Flows'
class: 'text-center'
highlighter: shiki
background: /background.svg
drawings:
  persist: false
transition: 'slide-left'
mdc: true
lineNumbers: true
author: Filippo Vissani
---

# Feasibility of Reactive Aggregate Programming via Kotlin Flows

<style>
.flex-container {
  display: flex;
  justify-content: space-between;
  font-style: italic;
}
.left-col {
    text-align: start;
}
.right-col {
    text-align: end;
    margin-left: auto;
}
</style>

<div class="flex-container">
    <div class="m-5 left-col">
        Supervisor: Prof. Danilo Pianini
        <br />
        Co-supervisor: Dr. Gianluca Aguzzi
    </div>
    <div class="m-5 right-col">
        Candidate: Filippo Vissani
    </div>
</div>


---
---

# Introduction

<style>
.flex-container {
  display: flex;
}
</style>

<div class="flex-container">
    <div class="m-5">
        <ul>
            <li>Self-organizing systems & macro-programming</li>
            <li>Proactive & reactive models in aggregate programming</li>
            <li>Reactive aggregate programming in Kotlin</li>
        </ul>
    </div>
    <div class="m-5">
        <center>
            <img src="/iot.png" />
        </center>
    </div>
</div>

---
layout: center
---

# Background: proactive model in aggregate computing

<style>
.flex-container {
  display: flex;
}
</style>

<div class="flex-container">
    <div class="m-5">
        <center>
            <img src="/proactive-model.svg" />
            The behavior of the proactive (round-based) model
        </center>
    </div>
    <div class="m-5">
        <center>
            <img src="/neighbors.svg" />
            Devices distributed over the network that run the aggregate program
        </center>
    </div>
</div>

---
---

# Background: reactive model in aggregate computing

<style>
.flex-container {
  display: flex;
}
</style>

<div class="flex-container">
    <div class="m-5">
        <center>
            <img src="/gradient-dependencies.svg" />
            The reactive dataflow graph corresponding to the self-healing gradient
        </center>
    </div>
    <div class="m-5">
        <center>
            <img src="/gradient-dependencies-distributed.svg" />
            <img src="/gradient-dependencies-devices.svg" />
            Distributed dependency graph
        </center>
    </div>
</div>

---
---

# Goal

<p></p>

Demonstrate the feasibility of reactive aggregate programming in Kotlin:

<style>
.flex-container {
  display: flex;
}
</style>

<div class="flex-container">
    <div class="m-5">
        <ul>
            <li>Using the Kotlin Flow reactive library</li>
            <li>Taking inspiration from the FRASP model developed in Scala</li>
            <li>By implementing the solution directly into the Collektive framework</li>
        </ul>
    </div>
    <div class="m-5">
        <center>
            <img src="/collektive.png" />
        </center>
    </div>
</div>

---
---

# Purely Reactive Model

<style>
.flex-container {
  display: flex;
}
</style>

<div class="flex-container">
    <div class="m-5">
        <ul>
            <li>Computation occurs reactively in response to environmental changes</li>
            <li>Devices only broadcast messages when necessary</li>
            <li>Sub-expressions are reevaluated only when their dependencies change</li>
            <li>Do not retain compatibility with the original Collektive DSL</li>
        </ul>
    </div>
    <div class="m-5">
        <center>
            <img src="/prm.svg" />
        </center>
    </div>
</div>

---
---

# Model with Reactive Messages and Sensors

<style>
.flex-container {
  display: flex;
}
</style>

<div class="flex-container">
    <div class="m-5">
        <ul>
            <li>Computation occurs reactively in response to environmental changes</li>
            <li>Devices only broadcast messages when necessary</li>
            <li>The entire aggregate expression undergoes reevaluation upon receiving a message</li>
            <li>Retains compatibility with the original Collektive DSL</li>
        </ul>
    </div>
    <div class="m-5">
        <center>
            <img src="/rmsm.svg" />
        </center>
    </div>
</div>

---
layout: center
---

# Validation: gradient with obstacles

<img src="/gradient-environment.png" class="m-10 h-100" />

---
---

# Validation: RMSM

```kt {all|1-7|9-13|all}
fun Aggregate<Int>.gradient(source: Boolean): Double =
    share(Double.POSITIVE_INFINITY) { field ->
        when {
            source -> 0.0
            else -> (field + 1.0).min(Double.POSITIVE_INFINITY)
        }
    }

fun Aggregate<Int>.gradientWithObstacles(nodeType: NodeType): Double =
    when {
        nodeType == NodeType.OBSTACLE -> -1.0
        else -> gradient(nodeType == NodeType.SOURCE)
    }
    
```

---
---

# Validation: PRM

```kt {all|1-8|10-15|all}
fun Aggregate<Int>.gradient(sourceFlow: StateFlow<Boolean>): StateFlow<Double> =
    rShare(Double.POSITIVE_INFINITY) { fieldFlow ->
        rMux(
            { sourceFlow },
            { MutableStateFlow(0.0) },
            { fieldFlow.mapStates { it.plus(1.0).min(Double.POSITIVE_INFINITY) } },
        )
    }

fun Aggregate<Int>.gradientWithObstacles(nodeTypeFlow: StateFlow<NodeType>): StateFlow<Double> =
    rBranch(
        { nodeTypeFlow.mapStates { it == NodeType.OBSTACLE } },
        { MutableStateFlow(-1.0) },
        { gradient(nodeTypeFlow.mapStates { it == NodeType.SOURCE }) },
    )
```

---
---

# Conclusion

---
layout: end
---
