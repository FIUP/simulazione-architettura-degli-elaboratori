# simulazione-architettura-degli-elaboratori
Svolgimento simulazione di Architettura degli Elaboratori @unipd

# Si spieghi in dettaglio la rappresentazione dei numeri reali secondo lo standard IEEE 754.

Lo IEEE 754 è lo standard internazionale per rappresentare i numeri a virgola mobile, ed è impiegato in tutti i moderni processori e coprocessori aritmetici.
Tale standard definisce un formato a 32 bit (detto *float*) e un formato a 64 bit (*double*), la cui dimensione del campo esponente (in formato polarizzato)
è rispettivamente di 8 e 11 bit. La base è 2 ed è implicita, il primo bit è relativo al segno del numero, e i restanti bit sono dedicati alla mantissa. 
Alcune combinazioni di bit servono per rappresentare dei particolari valori. I valori estremi dell'esponente, composti da tutti 0 e tutti 1
(2^8-1 in formato singolo, 2^11-1 in formato doppio), definiscono difatti dei valori speciali. 
Per esponenti nell'intervallo 1..2^8-1 in float e 1..2^11-2 in double vengono rappresentati i numeri normalizzati non nulli in virgola mobile.
L'esponente è polarizzato e varia da -2^7+2 a 2^7-1 in float e -2^10+2 e 2^10-1 in double. Un numero normalizzato richiede un bit a 1 a sinistra 
della virgola binaria. Tale bit è implicito, costituendo di fatto un significando effettivo a 24 o 53 bit, detto mantissa o frazione.
- Un esponente 0 con mantissa a 0 rappresente +0 o -0, a seconda del bit di segno.
- Un esponente di tutti 1 con mantissa 0 rappresenta l'infinito positivo o negativo, a seconda del bit di segno. Questo fatto lascia all'utente la
decisione finale di valutare come overflow come condizione di errore o come rappresentazione di infinito (ad esempio in un'applicazione dedicata al calcolo di limiti).
- Un esponente a 0 con una frazione non nulla rappresenta un numero denormalizzato. In questo caso, il bit a sinistra della virgola binaria
rappresenta un numero denormalizzato, e il vero esponente è -126 o -1022.
- Un esponente di tutti 1 con mantissa non nulla ha il valore simbolico di NaN, utile per segnalare condizioni di errore.

- - -

# Si descriva nel dettaglio la modalità di indirizzamento per spiazzamento.

L'indirizzamento per spiazzamento combina le capacità di quello diretto con quelle del registro indiretto. 
Il suo meccanismo di base è EA = A + (R), con:

  * A: contenuto di un campo indirizzo di un'istruzione
  * R: contenuto di un campo indirizzo di un'istruzione che riferisce a un registro
  * EA: indirizzo effetivo della locazione che contiene l'operando referenziato.
Tale tecnica prevede che l'istruzione abbia due campi indirizzo, di cui uno esplicito. Il valore A di un campo indirizzo è usato direttamente. L'altro campo indirizzo (oppure un riferimento implicito basato sull'OP) si riferisce ad un altro registro il cui valore, sommato ad A, dia l'indirizzo effettivo.
Esistono tre sottomodalità:
* indirizzamento relativo
* indirizzamento base-registro
* indicizzazione

1. Nell'indirizzamento relativo il registro implicitamente referenziato è il PC, quindi l'indirizzo dell'istruzione corrente viene sommato al campo indirizzo per produrre l'EA. Il campo indirizzo viene trattato come un numero in complemento a 2. Questa modalità di spiazzamento sfrutta la località dei riferimenti: se la maggior parte dei riferimenti alla memoria sono relativamenti vicini all'istruzione in esecuzione, allora l'impiego dell'indirizzamento relativo risparmia bit di indirizzo nell'istruzione.

2. Nell'indirizzamento registro-base il registro referenziato contiene un indirizzo di memoria, mentre il campo indirizzo contiene uno spiazzamento (solitamente rappresentato con un intero senza segno) rispetto a tale indirizzo. Il riferimento al registro può essere esplicito o implicito. Anche questa tecnica sfrutta la località degli accessi alla memoria. Si tratta di un metodo adatto all'implementazione della segmentazione.

3. Nell’indicizzazione il campo indirizzo rappresenta un indirizzo di memoria centrale e il registro referenziato contiene uno spiazzamento positivo da tale indirizzo. L'indicizzazione si rivela molto utile nelle operazioni operative. Supponendo di avere un elenco di numeri che parte dall'indirizzo A, e di dover scorrere tale elenco, sarà necessario incrementare A di (ad esempio) 1 ogni volta. Il valore A viene memorizzato nel campo dell'indirizzo dell'istruzione e al registro scelto, detto registro indice, viene inizialmente assegnato il valore 0. Dopo ciascuna operazione, il registro indice è incrementato. Alcune sistemi impiegano l'incremento all'interno dello stesso ciclo di clock dell'operazione; l'indicizzazione di tali sistemi è detta *autoindicizzazione*
di 1.

- - -

# Si metta a confronto criticamente il modo in cui un’architettura RISC utilizza l’ampio banco di registri a sua disposizione rispetto alla gestione di una cache

L’architettura RISC utilizza l’ampio banco di registri per conservare le variabili (prevalentemente scalari locali) che hanno un’alta probabilità di essere utilizzate con maggior frequenza. Sotto questo punto di vista il banco dei registri assomiglia molto alla memoria cache, sbbene sia molto più veloce. Ci sono però alcune sostanziali differenze:
Il BR contiene tutti gli scalari locali delle ultime N-1 procedure attivate e la cache contiene gli scalari locali usati di recente.
Le variabili globali sono assegnate dal compilatore per quanto riguarda il BR, mentre la cache contiene solo quelle usate di recente.
Mentre il BR contiene solo le variabili in uso, la cache importa un blocco di dati che potrebbe non essere utilizzato.
Il trasferimento dati tra registri e memoria è determinato in base alla profondità di annidamento delle procedure, mentre per la cache il salvataggio e il rimpiazzo è determinato in base all’algoritmo di sostituzione adottato.
Per accedere ad uno scalare locale in un BR viene utilizzato l’indirizzamento a registro, che è molto semplice e veloce.
Per quanto riguarda la cache invece, l’indirizzamento è molto più lento. La maggior parte delle cache infatti sono set-associative, il che comporta delle operazioni di confronto con il set e il tag per determinare la presenza o no di un dato in cache.
Anche nel caso della cache a indirizzamento diretto o fully-associative, questi modi sono comunque indirizzamenti a memoria che sono sempre più lenti di un indirizzamento a registro.

- - -

# Nel contesto di una pipeline, descrivere nel dettaglio la tecnica del data-forwarding: a cosa serve? Come funziona? Di che supporto hardware ha bisogno? 

Il data forwarding è una delle soluzioni utilizzate per superare le dipendenze dai dati dai dati che si possono verificare tra le istruzioni in una pipeline. 
Se viene individuato una dipendenza dai dati, il data forwarding, se il tipo di dipendenza lo permette, permette di trasferire i dati dall'output della ALU in ingresso alla ALU. Questo è possibile grazie ad appositi circuiti di ByPass e MUX, regolati da Unità di Controllo e potenzialmente anche da altre unità (dipende dall'architettura), che permettono alla ALU di caricare dati derivanti dalla memoria o dati che la ALU stessa ha mandato in output nel ciclo precedente. 
Nella realtà il dataforward può essere implementato in diversi modi che dipendono dall'architettura su cui lo si implementa.
Un esempio pratico di data forward lo troviamo nell'architettura MIPS in cui vi è un apposito circuito di identificazione delle dipendenze e gestione del dataforwarding. Tale circuito ha tre componenti fondamentali: 
 Forwarding Unit: unità che decide se attivare il forward attivando nell'opportuno modo i multiplexer della ALU 
 Hazard detection Unit: unità in grado di riconoscere le dipendenze e di generare stalli in caso di dipendenze non risolvibili 
 Control Unit: manda segnali di controllo che regolano il forward ed i dati che devono essere mandati (inoltre manda segnali di controllo che regolano l'esecuzione e l'utilizzo dell'hardware per la memorizzazione ed il write back) 
Nel caso in cui la dipendenza venga rilevata come risolvibile, allora nelle MIPS sarà possibile fare il forward di dati da fase EX a EX e da fase EX a MEM.

- - -

# Si elenchino, e si discutano, i fattori che condizionano e che sono condizionati dalla lunghezza del formato delle istruzioni.

- - -

# Nel contesto di una pipeline, si motivi e si spieghi in dettaglio la tecnica del buffer circolare per la gestione delle dipendenze da controllo.

- - -

# Si motivi la presenza nei processori RISC di un ampio banco di registri a uso generale. Spiegare in dettaglio il meccanismo di funzionamento di tale banco di registri.

- - -
