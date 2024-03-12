---
theme: seriph
title: 'Feasibility of Reactive Aggregate Programming via Kotlin Flows'
class: 'text-center'
highlighter: shiki
background: /background.jpg
drawings:
  persist: false
transition: 'slide-left'
mdc: true
lineNumbers: true
author: Filippo Vissani
---

# Feasibility of Reactive Aggregate Programming via Kotlin Flows

<p></p>

Presented by Filippo Vissani

---
layout: center
---

# Context & Motivations (1/2)

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

# Context & Motivations  (2/2)

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

# Goals

<center>
Scala -> Kotlin

Sodium -> Flow

FRASP -> Collektive
</center>

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
            <li>Computation occurs reactively in response to environmental changes.</li>
            <li>Devices only broadcast messages when necessary.</li>
            <li>Sub-expressions are reevaluated only when their dependencies change.</li>
            <li>Do not retain compatibility with the original Collektive DSL.</li>
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
            <li>Computation occurs reactively in response to environmental changes.</li>
            <li>Devices only broadcast messages when necessary.</li>
            <li>The entire aggregate expression undergoes reevaluation upon receiving a message.</li>
            <li>Retains compatibility with the original Collektive DSL.</li>
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

# Validation (1/3)

<img src="/gradient-environment.png" class="m-10 h-100" />

---
---

# Validation (2/3)

```kt {all|1-7|9-14}
fun Aggregate<Int>.gradient(source: Boolean): Double =
    share(Double.POSITIVE_INFINITY) { field ->
        when {
            source -> 0.0
            else -> (field + 1.0).min(Double.POSITIVE_INFINITY)
        }
    }

fun Aggregate<Int>.gradientWithObstacles(nodeType: NodeType): Double =
    if (nodeType == NodeType.OBSTACLE) {
        -1.0
    } else {
        gradient(nodeType == NodeType.SOURCE)
    }
    
```

---
---

# Validation (3/3)

```kt {all|1-8|10-15}
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
layout: end
---
