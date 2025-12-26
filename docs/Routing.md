# Routing

> Ricordando il concetto di Autonomous System (AS), cioè un insieme di reti IP e router sotto il controllo di una > > singola entità amministrativa (es. un ISP, un'Università, una grande Azienda) che presenta al mondo Internet una > politica di routing comune e coerente, ed ognuna con associato un identificativo a 16 o 32 bit (ASN),
> distinguiamo il routing in due livelli gerarchici:
>
> - Routing IGP - Interno agli AS  (Include protocolli come OSPF, RIP)
> - Routing EGP - Tra diversi AS (Protocollo BGP, gestisce l’internet)
>
> In questa sezione, tratteremo esclusivamente Routing IGP.

Il Routing è il processo di selezione da parte di un router, del percorso migliore per instradare i pacchetti di dati attraverso una rete.

Per capire meglio in cosa consiste il routing, immaginiamo di modellare la rete come un grafo pesato $G=(V,E)$, dove $V$ è l’insieme dei router, $E$ l’insieme dei link, con i relativi costi associati.

Sotto questa rappresentazione il compito del routing è quello di determinare il cammino minimo tra due nodi sorgente e destinazione.

<aside>
<img src="https://www.notion.so/icons/bookmark_lightgray.svg" alt="https://www.notion.so/icons/bookmark_lightgray.svg" width="20px" />

**Problema**: per poter calcolare un cammino minimo tra due nodi, ogni router deve avere una conoscenza (completa o limitata) della rete. Dove teniamo traccia di tale conoscenza?

</aside>

Un Router, è un sistema diviso concettualmente in due piani funzionali distinti:

- **Control Plane** - La CPU e la RAM eseguono un sistema operativo per router, che si occupa di gestire le configurazioni statiche e i protocolli di routing dinamico.
    
    L’output di questo piano è la **RIB (Routing Information Base), un database che racchiude la conoscenza che un router ha della rete.**
    
    E’ spesso ridondante (Potrebbe contenere più rotte diverse per raggiungere la stessa destinazione, per esempio perchè apprese secondo modalità/protocolli differenti), non ottimizzata per la velocità.
    
    Per questo, il control plane **“distilla” la RIB** in una seconda tabella detta **FIB (Forwarding Information Base), che contiene le sole rotte “migliori” per ogni destinazione.**
    
    Esempio di tabella di routing:
    
    | **Codice** | **Network Destination** | **Prefisso (Mask)** | **Next Hop / Gateway** | **Interface** | **AD** | **Metric** |
    | --- | --- | --- | --- | --- | --- | --- |
    | **C** | `192.168.0.0` | `/24` | *Directly Connected* | `Gig0/1` | 0 | 0 |
    | **L** | `192.168.0.100` | `/32` | *Local (My IP)* | `Gig0/1` | 0 | 0 |
    | **S** | `0.0.0.0` | `/0` | `10.10.10.1` | `Gig0/0` | 1 | 0 |
    | **O** | `172.16.0.0` | `/16` | `10.10.10.2` | `Gig0/0` | 110 | 20 |
    
    *(Nota: C=Connessa, L=Locale, S=Statica, O=OSPF)*
    
    Le rotte “migliori” da installare nella FIB, sono selezionate considerando in ordine:
    
    - Validità del next hop - Il router verifica se l’indirizzo del next hop è risolvibile: in caso negativo, la rotta viene scartata.
    - Longest Prefix Match - Supponiamo un router abbia due rotte per la stessa rete di destinazione:
        - Rotta A: `192.168.10.0/24` (comprende gli hosts da .0 a .255)
        - Rotta B: `192.168.10.0/28` (comprende gli hosts da .0 a .15)
        
        Arriva un pacchetto per `192.168.10.5`: entrambe le rotte sono valide, ma la Rotta B è più “specifica” (maschera di sottorete più lunga).
        
        Il router sceglierà **sempre** la rotta più specifica.
        
    - Distanza Amministrativa - A Parità di “precisione” di una rotta, si considera il valore del campo AD nella routing table.
        
        Questo numero indica al router “quanto è affidabile” la fonte da cui ha appreso tale rotta.
        
        - **0:** Connessione diretta
        - **1:** Rotta statica (Impostata dall’amministratore di rete)
        - **90:** EIGRP
        - **110:** OSPF
        - **200:** BGP
        
        Valori più bassi = affidabilità più elevata.
        
    - Metrica - Costo associato ad una rotta, in termini di numero di hops, larghezza di banda. ritardo o affidabilità del collegamento.
- **Data Plane** - Quando un pacchetto entra in un router, una porzione di hardware dedicata al forwarding sceglie dove inoltrarlo, riscrivendo l’header di livello 2 cambiando il MAC sorgente con il proprio (del router), ed il MAC destinazione col MAC corrispondente al next hop IP preso dalla FIB.
    
    Decrementa il TTL, ricalcola checksum, inoltra il pacchetto sull’interfaccia corrispondente al Next Hop IP.
    
    <aside>
    💡
    
    Questa divisione in livelli, permette una robusta resistenza agli errori: se il Control Plane crasha (il software di routing si blocca), il router può continuare a inoltrare pacchetti (compito del Data Plane) usando l'ultima FIB conosciuta.
    
    </aside>
    

Capito dove memorizziamo la conoscenza della rete, esistono due modalità con cui un router può apprenderla:

- Routing statico - L’amministratore di rete predispone manualmente i percorsi che i pacchetti dovranno seguire per arrivare ad una determinata rete, e li configura nel router sotto forma di voci nella RIB.
    
    Questo approccio prevede controllo totale sul traffico, oltre a prestazioni maggiori (I singoli router non hanno bisogno di calcolare da se le rotte del traffico, sono statiche ed impostate a priori).
    
    Tuttavia, scala molto male, sia in termini di complessità di configurazione iniziale (configurare rotte statiche su di una rete con 1000 router, è sicuramente dispensioso), sia in evoluzione (ogni modifica della rete, richiede riconfigurazione manuale delle rotte).
    
    <aside>
    <img src="https://www.notion.so/icons/bookmark_lightgray.svg" alt="https://www.notion.so/icons/bookmark_lightgray.svg" width="20px" />
    
    Il Routing statico può essere comunque utile in reti grandi, come “piano di backup” in caso di crash dei protocolli di routing dinamico.
    Un amministratore di rete può impostare delle rotte statiche con AD “artificialmente” molto alto (es: 200), che verranno utilizzate solo in caso il routing dinamico fallisse.
    
    </aside>
    
- Routing dinamico - I Router automaticamente prendono conoscenza della topologia della rete, scambiandosi informazioni e calcolando i percorsi migliori.

---

## Routing Statico

![images/image.png](images/image%2041.png)

Nella topologia sopra illustrata, ogni coppia di router, appartiene ad una sottorete a se, con numero massimo di hosts pari a 2.

La sottomaschera di queste reti sarà /30 (Vogliamo configurare 2 hosts, riservando due indirizzi per base e broadcast, necessitiamo di un Host ID pari a 2 bit).

Vogliamo permettere la comunicazione tra i due PC nelle LAN verde e lilla.

Il Router 6 è a conoscenza delle reti verde e gialla (poichè direttamente connesso ad esse), ma non sa come raggiungere le reti azzurre e lilla.

Impostiamo due rotte statiche per esse:

**Sintassi del comando di configurazione di una rotta statica**: **`router(config)# ip route <Dst Net Base addr> <Dst Net mask> <Next hop IP addr>`**

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#ip route 192.168.0.4 255.255.255.252 192.168.0.2
Router(config)#ip route 192.168.10.0 255.255.255.0 192.168.0.2**
```

Router 5 conosce le reti gialla e azzurra, ma non conosce verde e lilla.

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#ip route 192.168.10.0 255.255.255.0 192.168.0.6
Router(config)#ip route 192.168.1.0 255.255.255.0 192.168.0.1**
```

Router 7 conosce le reti azzurra e lilla, ma non conosce verde e gialla.

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#ip route 192.168.0.0 255.255.255.252 192.168.0.5
Router(config)#ip route 192.168.1.0 255.255.255.0 192.168.0.5**
```

![images/image.png](images/image%2042.png)

Ogni router conosce ora l’intera topologia della rete, la comunicazione tra i due PC è consentita.

---

## Routing Dinamico

### RIP (Routing Information Protocol)

RIP è un protocollo **Distance Vector** basato sull'algoritmo di **Bellman-Ford**.
A differenza dei protocolli Link State, i router RIP **NON hanno una mappa completa della rete: c**onoscono solo il costo totale per raggiungere una destinazione e il prossimo salto (Next Hop).

Utilizza il protocollo di trasporto **UDP.**

3 Versioni:

- RIPv1
    
    **Routing Classful:** Non invia la Subnet Mask negli aggiornamenti di routing, per questo non supporta Subnetting **VLSM** (Variable Length Subnet Mask). E’ in grado di annunciare sole reti /8, /16 o /24 (Classi A,B,C pure).
    
    **Trasmissione:** Invia gli aggiornamenti in **Broadcast** (255.255.255.255) ogni 30 secondi.
    
    Questo disturba ogni dispositivo nel dominio di broadcast (PC, stampanti ricevono il pacchetto, la NIC invia il pacchetto al S.O, rileva che non c’è nessun SW in ascolto per pacchetti RIP, lo scartano).
    
    **Sicurezza:** Nessuna autenticazione. Chiunque colleghi un router pirata può iniettare rotte false e dirottare il traffico.
    
- RIPv2
    
    **Routing Classless:** Invia la **Subnet Mask** insieme all'indirizzo IP. Supporta pienamente **VLSM** e **CIDR**.
    
    **Trasmissione:** Trasmette gli aggiornamenti in multicast ai soli router RIP (Gruppo multicast)
    
    **Sicurezza:** Supporta autenticazione (MD5 o Clear Text). Il router accetta aggiornamenti solo se la password corrisponde.
    
- RIPng
    
    Aggiunge supporto ad IPv6
    

### **Funzionamento di RIP:**

![images/image.png](images/image%2043.png)

Analizziamo la topologia in figura: all’avvio, ogni router riempie la propria tabella di routing inserendo le informazioni relative alle reti direttamente connesse. (Type = C (Connected), Metric=0)

![images/image.png](images/image%2044.png)

Ogni 30 secondi, i router inviano e ricevono un’informazione ridotta della tabella di routing dei vicini, che contiene:

- Reti conosciute
- Costo (Metrica, espressa in “Numero di router da attraversare” per poter arrivare alla rete destinazione)

Per ogni destinazione nella tabella del vicino, il router calcola:
$HopCount_{new} = HopCount_{vicino} + 1$
(Dove "+1" è il costo per raggiungere il vicino stesso).

Fatto questo calcolo, applica due regole logiche:

- **Discovery**
    
    Se la destinazione che sto analizzando, è sconosciuta per la mia tabella di routing, procedo ad “impararla”.
    
    Aggiungo alla mia tabella una entry contenente la rete che sto imparando, l’interfaccia che mi collega al vicino, la metrica calcolata.
    
- **Update**
    
    Se la destinazione che sto analizzando è già conosciuta, ma la metrica calcolata è minore, vuol dire che ho appreso una nuova strada più veloce.
    
    Aggiorno la rotta con il nuovo valore per HopCount, ed impostando come next hop il vicino dal quale ho ricevuto la tabella.
    

![images/image.png](images/image%2045.png)

Ora ogni router ha le informazioni necessarie per comunicare con l’intera rete.

Provando a trasmettere un messaggio tra PC2 e PC0, esso seguirà il percorso: PC2 → Router2 → Router0→ PC0

![images/image.png](images/image%2046.png)

Simuliamo un guasto tra Router2 e Router0

![images/image.png](images/image%2047.png)

Viene immediatamente segnalato il guasto del link (Triggered Update, senza dover aspettare il timer standard di 30 secondi), attraverso lo scambio di informazioni standard. 

Router2 Aggiorna la propria rotta verso la rete del PC0, impostando Router1 come Next Hop.

### **Configurazione RIPv1 in Packet Tracer:**

Situazione iniziale: Ogni router conosce le sole reti direttamente connesse.

![images/image.png](images/image%2048.png)

Partiamo dal Router0. Esso dovrà annunciare la conoscenza della rete 192.168.1.0 (Quella in cui si trova PC0), e della rete 192.168.0.0 (Quella che collega i 3 router assieme).

<aside>
<img src="https://www.notion.so/icons/bookmark_lightgray.svg" alt="https://www.notion.so/icons/bookmark_lightgray.svg" width="20px" />

RIPv1 è classful, pertanto dobbiamo annunciare l’intera rete 192.168.0.0/24, e non le singole sottoreti che collegano le coppie di router.

</aside>

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router rip
Router(config-router)#network 192.168.1.0
Router(config-router)#network 192.168.0.0
Router(config-router)#passive-interface GigabitEthernet 2/0**
```

Il comando **`router rip`** ordina al sistema operativo di avviare il demone software di **RIP**. Di default, la versione eseguita è RIPv1.

Col comando **`network A.B.C.D`**, il demone RIP esegue due azioni:

- Scansiona tutte le porte fisiche del router, rileva le interfacce associate alla rete indicata, e le attiva per la ricezione di pacchetti RIP.
- Comincia ad annunciare tale rete ai vicini.

Eseguendo il comando **`network 192.168.1.0`**, il router comincerà quindi contemporaneamente ad annunciare la conoscenza di tale rete ai vicini (corretto), ma anche ad inoltrare i pacchetti RIP all’interno della LAN (comportamento indesiderato).

Con **`passive-interface GigabitEthernet 2/0`**, dove **`GigabitEthernet 2/0`** è l’interfaccia associata alla LAN 192.168.1.0, indichiamo al router di smettere di inviare pacchetti RIP ad essa, pur continuando ad annunciare la rete agli altri router.

Ripetiamo per Router1:

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router rip
Router(config-router)#network 192.168.10.0
Router(config-router)#network 192.168.0.0
Router(config-router)#passive-interface GigabitEthernet 1/0**
```

Router2:

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router rip
Router(config-router)#network 192.168.20.0
Router(config-router)#network 192.168.0.0
Router(config-router)#passive-interface Ethernet 2/0**
```

![images/image.png](images/image%2045.png)

I Router hanno correttamente cominciato a scambiarsi messaggi RIP.

### **Configurazione RIPv2 in Packet Tracer:**

Operiamo sulla stessa topologia indicata sopra.

RIPv2 è classless, dobbiamo quindi annunciare singolarmente tutte le sottoreti (anche le /30 tra router).

Router0:

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router rip
Router(config-router)#version 2
Router(config-router)#network 192.168.1.0
Router(config-router)#network 192.168.0.4
Router(config-router)#network 192.168.0.0**
```

Router1:

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router rip
Router(config-router)#version 2
Router(config-router)#network 192.168.10.0
Router(config-router)#network 192.168.0.0
Router(config-router)#network 192.168.0.8**
```

Router2:

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router rip
Router(config-router)#version 2
Router(config-router)#network 192.168.0.4
Router(config-router)#network 192.168.0.8
Router(config-router)#network 192.168.20.0**
```

<aside>
<img src="https://www.notion.so/icons/bookmark_lightgray.svg" alt="https://www.notion.so/icons/bookmark_lightgray.svg" width="20px" />

Nota! RIPv2 trasmette gli aggiornamenti in modalità multicast.
La trasmissione in modalità multicast, implica comunque che i pacchetti vengono inoltrati in modalità flooding su tutte le interfacce, ma vengono scartati dalla NIC del destinatario (dopo un check sul MAC destinatario).

I PC nella LAN quindi ricevono comunque i pacchetti RIP, e seppur scartati dalla NIC, un utente smaliziato potrebbe usare un analizzatore di rete (es: Wireshark) per catturare i pacchetti RIP e acquisire conoscenza della rete.

**E’ pertanto comunque consigliato eseguire il comando passive-interface anche in RIPv2!**

</aside>

![images/image.png](images/image%2049.png)

---

### OSPF (Open Shortest Path First)

OSPF è un protocollo di routing Link State basato direttamente su IP.

A differenza di RIP, ogni router OSPF acquisisce conoscenza dell’intera topologia della porzione di rete in cui si trova, che utilizzerà per calcolare localmente i cammini minimi verso ogni destinazione.

![images/image.png](images/image%2050.png)

Per far funzionare tutto questo, il router deve gestire tre archivi distinti nella sua memoria:

- Neighbor Table - Lista dei router adiacenti (direttamente connessi), con i loro indirizzi IP, Interfacce, Router ID, Stato.
- Topology Table (LSDB - Link State Database) - Lista di tutti gli LSA (Link State Advertisements) raccolti. Rappresenta la mappa completa della rete.
- Routing Table - Popolata coi soli percorsi migliori, calcolati eseguendo l’algoritmo di Dijkstra sulle informazioni contenute nelle due tabelle precedenti.

### Funzionamento di OSPF

- **Inizializzazione del processo OSPF**
    
    ```
    Router# config term # accedere alla config mode
    Router(config)# router ospf 1 # avvio il processo OSPF, con Process ID pari ad 1 (questo serve ad identificare eventuali multiple istanze di OSPF in esecuzione sullo stesso router)
    ```
    
    Automaticamente, viene assegnato al router un ID univoco a 32bit (RID), con il quale esso potrà identificarsi.
    
    ![images/image.png](images/image%2051.png)
    
    Selezione delle interfacce da annunciare durante lo scambio di messaggi:
    
    ```
     Router(config)# network <ip-address> <wildcard-mask> area <area-id>
    ```
    
    Tutte le interfacce il cui indirizzo rientra (match) nella combinazione ip-address e wildcard-mask indicata:
    
    - Verranno annunciate ai router adiacenti.
    - Saranno destinazione di traffico OSPF
    
    <aside>
    <img src="https://www.notion.so/icons/bookmark_lightgray.svg" alt="https://www.notion.so/icons/bookmark_lightgray.svg" width="20px" />
    
    Vogliamo evitare di inviare pacchetto OSPF in reti composte da soli hosts (LAN).
    
    Per fare ciò, possiamo impostare le interfacce in modalità passiva
    
    </aside>
    
    ```
    Router(config-router)# passive-interface GigabitEthernet0/0
    ```
    
- **Inizio del processo di adiacenza**
    
    ![images/image.png](images/image%2052.png)
    
    - Stato iniziale (DOWN): Entrambi i router hanno Neighbor Table vuota.
    - R1 invia un pacchetto HELLO in Multicast a `224.0.0.5` (Gruppo a cui appartengono tutti i router OSPF)
        
        **Contenuto Pacchetto R1:**
        
        - **Source OSPF Router ID:** `1.1.1.1`
        - **Active Neighbor Field:** `[VUOTO]` (R1 non conosce nessuno).
        
        R2 riceve il pacchetto, controlla i parametri (Area, Subnet, Password, Hello Timer), vede che il campo “Active Neighbor” è vuoto, capisce che R1 non è ancora a sua conoscenza.
        
        R2 registra R1 nella sua Neighbor Table, impostandone lo stato a INIT.
        
        **INIT = Conosco R1, ma lui ancora non conosce me. La comunicazione è unidirezionale.**
        
    - R2 risponde con un nuovo pacchetto HELLO
        
        **Contenuto del Pacchetto R2:**
        
        - **Source OSPF Router ID:** `2.2.2.2`
        - **Active Neighbor Field:** `1.1.1.1`
        
        R1 lo riceve, legge il campo Source OSPF Router ID, riconosce la corrispondenza tra il suo RID e il campo Active Neighbor: Vuol dire che il suo precedente pacchetto HELLO è stato raggiunto da 2.2.2.2, che ne ha registrato la presenza.
        
        R1 registra R2 nella sua neighbor table con lo stato 2WAY: ora entrambi sono a conoscenza l’uno dell’altro.
        
    - R1 risponde con un ultimo HELLO con 2.2.2.2 nel campo Active Neighbor, così che anche R2 imposti lo stato 2WAY.
- **Scambio di Link State Advertisement**
    
    Prima di inviare dati, ogni coppia di router si sincronizza per eleggere un Master e uno Slave.
    
    In figura, 1.1.1.1 è il master (Nella maggior parte dei casi in realtà, il master è quello con RID più alto, in questo caso sarebbe 2.2.2.2)
    
    ![images/image.png](images/image%2053.png)
    
    1.1.1.1 invia un pacchetto DBD, in cui è contenuto un sommario del suo LSDB (I soli LSA Headers). Inviamo i soli header e non l’intero LSDB, poiché per ogni coppia di router, i due LSDB differiranno ad uno stesso momento per poche informazioni. Sarebbe uno spreco trasmettere l’intero LSDB.
    
    Anche 2.2.2.2 invia i propri headers.
    
    1.1.1.1 confronta gli header ricevuti con il proprio LSDB.
    
    Per ognuno, se il LSA manca o se quello ricevuto è più recente, viene aggiunto alla **LSA Request List**.
    
    Ipotizziamo 1.1.1.1 rilevi che gli manchino 50 LSA per essere sincronizzato:
    
    - 1.1.1.1 genera un (o più, si inseriscono quante più richieste possibili compatibilmente con MTU) pacchetto LSR. Al suo interno c'è la lista degli ID di tutti i 50 LSA mancanti.
    - 2.2.2.2 riceve la lista. Supponiamo che ogni LSA completo pesi 100 Byte.
    Totale dati: 50 * 100 = 5000$ Byte.
    L'MTU è 1500 Byte.
    R2 non può mandare tutto in un pacchetto.
    R2 invierà **4 pacchetti LSU (Link State Update)**:
    • LSU #1 (Primi 14 LSA)
    • LSU #2 (Secondi 14 LSA)
    • LSU #3 (Terzi 14 LSA)
    • LSU #4 (Ultimi 8 LSA)
    - 1.1.1.1 conferma la ricezione dei LSA con un ACK cumulativo.
    
    La procedura si ripete con 2.2.2.2 che richiede i LSA mancanti.
    
    Al termine, gli LSDB dei due router sono sincronizzati (Stato FULL), eseguono l’algoritmo di Dijkstra per popolare la tabella di routing.
    
- **Segnalazione di cambiamenti nella topologia di rete**
    
    Dopo la sincronizzazione (stato **FULL**), la rete è silenziosa. I router inviano solo piccoli Hello ogni 10s (Keepalive).
    Se però cade un link o cambia qualcosa, il router che rileva il guasto invia immediatamente un **LSU in flooding**.
    
    Tutti i router ricevono l'aggiornamento, aggiornano l'LSDB ed eseguono di nuovo Dijkstra.
    

### Metrica

In ogni pacchetto LSU, è associata ai LSA una metrica che OSPF calcola come:

$Metric=\frac{Reference Bandwidth(Mbps)}{LinkBandwidth(Mbps)}$

Questo valore è ciò che l’algoritmo di Dijkstra minimizza durante il calcolo dei percorsi migliori.

![images/image.png](images/image%2054.png)

### Aree

Per garantire la scalabilità e impedire che l'LSDB raggiunga dimensioni troppo elevate su reti vaste, OSPF divide logicamente gli AS in **Aree**.

![images/image.png](images/image%2055.png)

Questa segmentazione crea una gerarchia a due livelli:

1. **Dominio di Flooding Ridotto:** Ogni Area mantiene il proprio LSDB. 
2. **Isolamento dei Guasti:** Se un link cade nell'Area 1, l'algoritmo SPF viene ricalcolato **solo** dai router dell'Area 1.

![images/image.png](images/image%2056.png)

Tra le varie tipologie, utilizzeremo la **Stub Area**. È una configurazione ottimizzata di OSPF che **filtra le rotte esterne al Sistema Autonomo** (LSA Type 5), impedendo che entrino nell’LSDB (riducendone le dimensioni)
Per raggiungere destinazioni esterne, la Stub Area utilizza una **Default Route (`0.0.0.0/0`)** iniettata automaticamente dal router di confine (ABR), delegando a lui ogni decisione di instradamento verso l'esterno. (In figura, il router di confine per l’area 1 è R1) 

### Configurazione OSPF in Packet Tracer

![images/image.png](images/image%2057.png)

**Router 0:**

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router ospf 1
Router(config-router)# area 1 stub #configurazione area stub
Router(config-router)#network 192.168.1.0 0.0.0.255 area 1 #annuncio reti
Router(config-router)#network 192.168.0.0 0.0.0.3 area 1
Router(config-router)#network 192.168.0.4 0.0.0.3 area 1
Router(config-router)#passive-interface GigabitEthernet2/0 #non inoltro pacchetti OSPF nella LAN**
```

**Router 1:**

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router ospf 1
Router(config-router)# area 1 stub
Router(config-router)#network 192.168.10.0 0.0.0.255 area 1
Router(config-router)#network 192.168.0.0 0.0.0.3 area 1
Router(config-router)#network 192.168.0.8 0.0.0.3 area 1
Router(config-router)#passive-interface GigabitEthernet1/0**
```

**Router 2:**

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router ospf 1
Router(config-router)# area 1 stub
Router(config-router)#network 192.168.20.0 0.0.0.255 area 1
Router(config-router)#network 192.168.0.4 0.0.0.3 area 1
Router(config-router)#network 192.168.0.8 0.0.0.3 area 1
Router(config-router)#passive-interface Ethernet2/0**
```

![images/image.png](images/image%2058.png)

Con il comando `show ip ospf neighbor`, verifichiamo che le connessioni con i vicini siano in stato FULL, ad indicare che gli LSDB sono stati correttamente sincronizzati.