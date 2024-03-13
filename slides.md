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

<!--
Buongiorno, sono Filippo Vissani e la mia tesi tratta la fattibilità della programmazione aggregata reattiva in Kotlin.
-->

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
            <li>Aggregate computing & self-organizing systems</li>
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

<!--
L'aggregate computing è un metodo per coordinare sistemi distribuiti complessi. Questo approccio permette di concentrarsi sul macro-comportamento dell'intero sistema anziché sulle relazioni tra i dispositivi, i loro pari e l'ambiente in cui si trovano. L'aggregate computing è particolarmente adatto per scenari in cui il sistema distribuito:
-  È aperto, ovvero può cambiare o subire guasti imprevisti.
- Coinvolge un vasto numero di dispositivi che necessitano di astrazioni per il coordinamento.
- Deve avere la capacità di rispondere a eventi significativi per garantire la resilienza.

La maggior parte linguaggi per la programmazione aggregata utilizza un modello di esecuzione basato su round, in cui i dispositivi valutano ripetutamente il loro programma in modo ciclico o periodico. Questo approccio è semplice, ma manca di flessibilità per quanto riguarda la risposta ai cambiamenti dell'ambiente. Recentemente è stato proposto un nuovo modello, chiamato FRASP, implementato in Scala, che risolve questo problema facendo uso del paradigma reattivo.

L'obiettivo della tesi è quello di dimostrare la fattibilità della programmazione aggregata reattiva in Kotlin prendendo ispirazione da FRASP.
-->

---
layout: center
---

# Background: Proactive Model in Aggregate Computing

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

<!--
Partiamo presentando le differenze tra il modello proattivo e quello reattivo:

Nel modello proattivo i dispositivi eseguono ciclicamente i round, dove per ogni round valutano l'espressione aggregata considerando il contesto relativo a quel round. Ogni round è suddiviso in tre fasi:

- Inizialmente viene definito il contesto locale sulla base dei messaggi dei vicini e sullo stato dei sensori
- L'espressione aggregata viene valutata considerando il contesto locale e viene generato un export come risultato
- L'export viene mandato in broadcast a tutti i vicini, che lo useranno per i loro round futuri
-->

---
---

# Background: Reactive Model in Aggregate Computing

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

# Validation: Gradient with Obstacles

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

# Conclusion & Future Work

- Improve the ergonomics of the DSL of the purely reactive model
- Introduce throttling to regulate inbound and outbound reactivity
- Investigate ways to support deployment and execution on real-world distributed platforms

---
layout: end
---
