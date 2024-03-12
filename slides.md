---
theme: seriph
title: 'Feasibility of Reactive Aggregate Programming via Kotlin Flows'
class: 'text-center'
highlighter: shiki
background: none
drawings:
  persist: false
transition: 'slide-left'
mdc: true
lineNumbers: true
author: Filippo Vissani
---

# Feasibility of Reactive Aggregate Programming via Kotlin Flows

Filippo Vissani

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
    <img src="/proactive-model.svg" class="m-10 h-90" />
    <img src="/neighbors.svg" class="m-10 h-90" />
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
    <img src="/gradient-dependencies.svg" class="m-5 h-80" />
    <div>
        <img src="/gradient-dependencies-distributed.svg" class="m-5 h-50" />
        <img src="/gradient-dependencies-devices.svg" class="m-5 h-20" />
    </div>
</div>

---
---

# Goal

---
---

# Purely Reactive Model

---
---

# Model with Reactive Messages and Sensors

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
