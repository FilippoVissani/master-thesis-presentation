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
Partiamo presentando le differenze tra il modello proattivo e quello reattivo nel contesto della programmazione aggregata:

Nel modello proattivo i dispositivi eseguono ciclicamente i round, dove per ogni round valutano l'espressione aggregata considerando il contesto relativo a quel round. Ogni round è suddiviso in tre fasi:

- Inizialmente viene definito il contesto locale sulla base dei messaggi dei vicini e sullo stato dei sensori.
- L'espressione aggregata viene valutata considerando il contesto locale e viene generato un export come risultato.
- L'export viene mandato in broadcast a tutti i vicini, che lo useranno per i loro round futuri.

In questo modello:
- La computazione avviene indipendente dal fatto che ci siano cambiamneti nell'ambiente.
- Non è possibile reagire ai cambiamenti dell'ambiente direttamente.
- L'intera espressione aggregata viene rivalutata interamente ad ogni round.
- Gli export vengono mandati anche nel caso in cui non ci siano cambiamenti.
-->

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

<!--
Il modello reattivo fornisce lo stato dei sensori e le informazioni dei vicini in maniera reattiva. Inoltre, questo modello consente di esprimere l'espressione aggregata come un grafo di sotto-espressioni, dove ogni sotto-espressione ritorna un valore reattivo.

Qui ad esempio vediamo lo schema di dipendenze dell'espressione aggregata relativa al gradiente. Nel momento in cui lo stato di un sensore cambia o viene ricevuto un messaggio da un vicino, viene rivalutata solo la parte di sotto-espressione interessata.
-->

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

<!--
L'obiettivo della tesi è dimostrare la fattiblità della programmazione aggregata reattiva in Kotlin:
- Utilizzando la libreria reattiva Flow.
- Prendendo ispirazione da FRASP, implementato in Scala usando Sodium come libreria reattiva.
- Implementando la soluzione nel framework Collektive, che al momento fa uso del modello proattivo.

Dopo una prima fase di analisi, sono state individuate due possibili soluzioni per integrare il paradigma reattivo in Collektive: un modello puramente reattivo e un modello in cui sono reattivi solo i sensori e i messaggi dei dispositivi.
-->

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

<!--
Il modello puramente reattivo rispetta la base teorica fornita da FRASP:

- I sensori e i messaggi vengono modellati come reattivi.
- Vengono gestite le dipendenze delle sotto-espressioni in maniera reattiva, questo implica che i costrutti aggregati sono vincolati al tipo Flow.
- Non viene mantenuta la compatibilità con il DSL attuale, dato che la signature dei costrutti viene modificata.
- L'introduzione dei flow direttamente nei costrutti aggregati comporta un calo dell'ergonomia rispetto al DSL originale, nella parte delle validazioni verrà fatta un'analisi più approfondita su questo aspetto.
-->

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

<!--
Il secondo modello proposto si limita ad introdurre il paradigma reattivo nei messaggi e nei sensori. In questo modello non vengono gestite le dipendenze delle sotto-espressioni. Quindi, al variare dello stato dei sensori e dei messaggi l'intera espressione aggregata viene rivalutata. Questo modello mantiene la compatibilità con il DSL attuale, dato che non vincola i costrutti aggregati al tipo Flow.
-->

---
layout: center
---

# Validation: Gradient with Obstacles

<img src="/gradient-environment.png" class="m-10 h-100" />

<!--
In fase di validazione viene fatta un'analisi sull'ergonomia dei DSL relativi ai modelli proposti. Il programma aggregato scelto per effettuare questa valutazione è il gradiente con ostacoli.
Il gradiente è un comportamento distribuito che si auto-stabilizza, in ciascun dispositivo del sistema distribuito, ad un valore che denota la sua distanza minima dal nodo sorgente più vicino, calcolato sommando le distanze da vicino a vicino lungo il percorso più breve verso la sorgente, adattandosi ai cambiamenti dell’insieme della sorgente e delle distanze.
In questa slide viene presentato l'ambiente in cui il gradiente viene eseguito:
- I dispositivi sono posizionati in una griglia di 5 righe e 5 colonne.
- Ogni dispositivo ha come vicini quelli che trova per primi sull'asse orizzontale e verticale.
- Il dispositivo con ID 0 è la sorgente.
- I dispositivi con ID 2, 7 e 12 vengono considerati come ostacoli.
-->

---

# Validation: RMSM

```kt {all|9-13|1-7|all}
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

<!--
Come detto precedentemente, il modello con messaggi e sensori reattivi non altera il DSL originale, quindi il programma aggregato che vediamo i questa slide è valido anche per il caso proattivo.

Il programma è suddiviso in due funzioni, quella evidenziata esegue una partizione di dominio per separare gli ostacoli, che ritorneranno il valore -1.0, dai nodi che eseguiranno effettivamente il gradiente.

Una caratteristica interessante di Collektive è utilizza i costrutti condizionali di kotlin per eseguire il branching e questo modello reattivo non altera questa caratteristica.

L'esecuzione del gradiente utilizza il costrutto share per la generazione dei field. All'interno di share viene utilizzato ancora una volta il costrutto when per distinguere la sorgente, che ritornerà 0.0, dal resto dei dispositivi.
-->

---

# Validation: PRM

```kt
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

<!--
Qui vediamo l'implementazione del gradiente con ostacoli nel modello puramente reattivo. Nonostante il programma aggregato risulti simile, viene introdotta una certa complessità a causa dell'utilizzo dei flow direttamente all'interno dei costrutti aggregati.
In questo modello, al posto dei costrutti condizionali di Kotlin, vengono introdotte versioni di branch e mux adatte a reagire ai cambiamenti delle condizioni che prendono in input.
I tipo di ritorno delle espressioni è vincolato a StateFlow, quindi i valori di ritorno, come quello a riga 13, devono essere wrappati in flow.
Un altro aspetto che introduce complessità è il fatto che, per utilizzare i valori all'interno delle espressioni, bisognia per forza impiegare le funzioni per gestire i flow, come mapStates.
-->

---

# Conclusion & Future Work

- Improve the ergonomics of the DSL of the purely reactive model
- Introduce throttling to regulate inbound and outbound reactivity

<!--
In conclusione, tramite l'introduzione di questi due modelli, possiamo dire che è possibile creare programmi aggregati reattivi in Kotlin.

Per quanto riguarda gli sviluppi futuri, alcuni miglioramenti che si possono introdurre sono:
- Migliorare l'ergonomia del DSL proposto nel modello puramente reattivo
- Introdurre il throttling per regolare il livello di reattività, ad esempio rivalutando l'espressione aggregata solo dopo un certo numero di messaggi ricevuti.
-->

---
layout: end
---
