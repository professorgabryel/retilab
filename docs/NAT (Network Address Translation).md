# NAT (Network Address Translation)

Il protocollo IPv4 prevede l’assegnazione ad ogni dispositivo connesso in rete di un indirizzo a 32 bit, lunghezza sufficiente a definire circa 4,3 miliardi di indirizzi unici.

La rapida espansione di internet ha generato la necessità di più indirizzi univoci, motivo per il quale è iniziata la migrazione ad IPv6.

Per prolungare la vita di IPv4, sono stati introdotti gli indirizzi IP privati.

L'idea alla base è semplice: non è necessario che ogni dispositivo (stampante, PC aziendale, smartphone domestico) abbia un indirizzo IP pubblico univoco raggiungibile da tutto il mondo, è sufficiente che sia identificabile univocamente solo all'interno della sua rete locale (LAN).

Lo standard **RFC 1918** ha riservato tre blocchi di indirizzi specifici per l'uso privato, riutilizzabili all'infinito in reti diverse, purché isolate tra loro.

> **IPv4 Private IP Ranges (RFC 1918)**
> 
> - **Class A Block:** `10.0.0.0` – `10.255.255.255` (CIDR: `10.0.0.0/8`)
> - **Class B Block:** `172.16.0.0` – `172.31.255.255` (CIDR: `172.16.0.0/12`)
> - **Class C Block:** `192.168.0.0` – `192.168.255.255` (CIDR: `192.168.0.0/16`)

!!! warning "Non Instradabilità"
    Gli indirizzi privati **non sono instradabili su Internet**. I router dei provider (ISP) sono configurati per scartare (drop) qualsiasi pacchetto che abbia come destinazione o sorgente un indirizzo IP appartenente a queste classi. Ciò significa che un dispositivo a cui è assegnato un indirizzo IP privato, non potrà comunicare con la rete pubblica utilizzando quello stesso indirizzo.

La soluzione è il NAT, un meccanismo di traduzione di indirizzi pubblici in privati e viceversa, adottato dai router di confine d’area al fine di permettere la navigazione in rete di dispositivi a cui è assegnato un indirizzo privato.

## NAT Statico:

![](https://ipcisco.com/wp-content/uploads/2018/10/nat-types-static-nat-ipcisco.jpg)

Associa ad ogni dispositivo all’interno di una rete privata (indirizzato al suo interno da un indirizzo privato), un indirizzo IP pubblico fisso.

Mappatura 1 a 1, necessito di tanti indirizzi IP pubblici tanti quanti sono gli host che vogliono comunicare in rete (un indirizzo pubblico per ogni indirizzo privato).

Utilizzato principalmente in ambiente server, assegnando ad ognuno un indirizzo IP pubblico fisso.

### Funzionamento del NAT Statico:

Un client con indirizzo IP `8.8.8.8` (supponiamo pubblico e statico), richiede un servizio fornito dal server con indirizzo IP pubblico `200.200.200.1`:

1. Il client invia un pacchetto attraverso internet.
    
    **Source:** `8.8.8.8`
    
    **Destination:** `200.200.200.1` (IP Pubblico del Server)
    
2. Il router della rete in cui si trova il server, riceve il pacchetto sull'interfaccia *Outside*.
    
    Controlla la NAT Table e trovala regola statica di conversione 200.200.200.1 → 192.168.0.1: modifica l'header del pacchetto
    
    **Source:** `8.8.8.8` (Invariato)
    
    **New Destination:** `192.168.0.1` (IP Privato)
    
3. Il router cerca nella routing table `192.168.0.1` , inoltra correttamente il pacchetto al server.
4. Il server risponde con la pagina web inviando un pacchetto al suo gateway (il router).
    
    **Source:** `192.168.0.1` (IP Privato)
    
    **Destination:** `8.8.8.8`
    
5. Il router riceve il pacchetto sull'interfaccia *Inside*.
Ricontrolla la NAT Table, sostituisce l’indirizzo IP Privato del server al fine di renderlo instradabile pubblicamente:
    - **New Source:** `200.200.200.1` (IP Pubblico)
    - **Destination:** `8.8.8.8` (Invariato)
6. Il pacchetto viaggia su Internet. Il Client riceve la risposta credendo che sia arrivata da `200.200.200.1`

### Configurazione NAT statico in Packet Tracer:

Per abilitare il NAT, il router di confine d’area della LAN deve sapere quali interfacce appartengono alla rete interna (da tradurre) e quali alla rete esterna (internet).

Marcatura dell’interfaccia connessa alla LAN:

```
Router(config)# interface [GigabitEthernet0/0]
Router(config-if)# ip nat inside
Router(config-if)# exit
```

Marcatura dell’interfaccia connessa ad internet:

```
Router(config)# interface [Serial0/0/0]
Router(config-if)# ip nat outside
Router(config-if)# exit
```

In modalità di configurazione globale, impostiamo manualmente le regole di conversione tra IP Privati e pubblici.

```
Router(config)# ip nat inside source static PRIVATE_IP PUBLIC_IP
```

# NAT Dinamico:

![](https://ipcisco.com/wp-content/uploads/2018/10/nat-types-dynamic-nat-www.ipcisco.jpg)

All’interno del router di confine d’area della LAN, è caricato un pool (insieme) di indirizzi IP pubblici, ed ogni host interno alla rete, potrà comunicare all’esterno utilizzando uno degli IP presenti in esso.

Esempio: potrei aver acquistato e caricato sul router 3 indirizzi IP pubblici, ma aver 50 host configurati nella LAN, come possono navigare tutti contemporaneamente su internet?

Non è possibile, il mio router permetterà solo a 3 host contemporaneamente di comunicare in rete.

Per configurare NAT Dinamico pertanto servono:

- Pool di indirizzi pubblici
- Elenco di indirizzi privati che andranno tradotti

> Attenzione! Non specifico una mappatura 1 a 1 come nel caso del NAT Statico, perchè l’assegnazione di un indirizzo pubblico piuttosto che un’altro, è dinamica, “casuale”, non definita univocamente tramite mappatura pre-configurata.
> 

### Configurazione NAT Dinamico in Packet Tracer:

Marcatura dell’interfaccia connessa alla LAN:

```
Router(config)# interface [GigabitEthernet0/0]
Router(config-if)# ip nat inside
Router(config-if)# exit
```

Marcatura dell’interfaccia connessa ad internet:

```
Router(config)# interface [Serial0/0/0]
Router(config-if)# ip nat outside
Router(config-if)# exit
```

Configuro una ACL contenente l’elenco degli indirizzi d’origine che andranno tradotti (a cui vogliamo permettere la navigazione in internet).

**Sintassi del comando: `access-list numeroACL permit/deny SOTTOINSIEME_INDIRIZZI_DA_TRADURRE WILDCARD_MASK`**

```
# Definisco quali IP possono essere tradotti
Router(config)# access-list 110 permit ip 192.168.0.0 0.0.0.15 any
```

Per concedere a tutti gli host configurati nella LAN di accedere ad internet (traduzione di tutti gli host nella LAN), posso utilizzare la sintassi

```
Router(config)# access-list numeroACL permit ip any any
```

Configurazione Pool di indirizzi pubblici e abilitazione del NAT dinamico:

```
Router(config)# ip nat pool NAME FIRST_IP_ADDRESS LAST_IP_ADDRESS netmask SUBNET_MASK
Router(config)# ip nat inside source list ACL_NUMBER pool NAME (Stesso nome dato al POOL di indirizzi)
```

# NAT “Overload” (PAT, Port Address Translation):

![](https://ipcisco.com/wp-content/uploads/2018/10/nat-types-pat-ipcisco.jpg)

Singolo indirizzo IP Pubblico (Assegnato dall’ISP e configurato sul Router), con cui tutti i dispositivi nella LAN comunicano.

Per distinguere quale dispositivo richiede la connessione, e a quale di conseguenza inoltrare i pacchetti in entrata (essendo che ora tutti gli host nella LAN comunicano all’esterno utilizzando lo stesso IP Pubblico), viene applicato un meccanismo di conversione dei numeri di porta detto PAT, con l’utilizzo di una PAT table.

### Funzionamento del NAT “Overload”

Due client nella rete interna, PC A (`192.168.0.1`) e PC B (`192.168.0.2`), vogliono visitare contemporaneamente lo stesso sito web (`8.8.8.8`).

Il router ha un solo indirizzo IP Pubblico: `200.200.200.1`

1. **Invio richieste:**
Il PC A apre il browser, il S.O assegna al processo una porta casuale (es. 3001).
**Source:** `192.168.0.1 : 3001` (IP Privato : Porta A)
**Destination:** `8.8.8.8 : 80` (IP Web Server : Porta HTTP)
Il Router riceve il pacchetto del PC A, nota che è da inoltrare al di fuori della LAN, sostituisce il suo IP pubblico nell’header IP e assegna una nuova porta libera.
    
    New **Source:** `200.200.200.1 : 6001` (IP Pubblico : Porta Assegnata dal Router)
    **Destination:** `8.8.8.8 : 80` (Invariato)
    
    Memorizza tale associazione nella NAT Table (che ora memorizzerà le associazioni IP-Port).
    
    Anche il PC B apre il browser. Per coincidenza, usa la stessa porta sorgente interna (3001).
    **Source:** `192.168.0.2 : 3001` (IP Privato : Porta B)
    **Destination:** `8.8.8.8 : 80`
    Il router assegna una nuova porta libera (es. 6002), e sostituisce il proprio IP
    New **Source:** `200.200.200.1 : 6002` (IP Pubblico : Porta Diversa)
    **Destination:** `8.8.8.8 : 80`
    Memorizza la nuova associazione.
    
    I due pacchetti vengono inoltrati sulla rete
    
- **Reply:**
    
    Il Server risponde alla prima richiesta:
    **Source:** `8.8.8.8 : 80`
    **Destination:** `200.200.200.1 : 6001`
    Il router riceve il pacchetto, legge la Porta di Destinazione (6001).
    
    Consultando la NAT Table, rileva che a `200.200.200.1: 6001`, corrisponde l’host A: `192.168.0.1 : 3001`.
    **Source:** `8.8.8.8 : 80`
    
    New **Destination:** `192.168.0.1 : 3001`
    
    Il pacchetto viene consegnato a PC A.
    
    La stessa procedura si ripete per il PC B.
    

### Configurazione NAT “Overload” in Packet Tracer:

Marcatura dell’interfaccia connessa alla LAN:

```
Router(config)# interface [GigabitEthernet0/0]
Router(config-if)# ip nat inside
Router(config-if)# exit
```

Marcatura dell’interfaccia connessa ad internet:

```
Router(config)# interface [Serial0/0/0]
Router(config-if)# ip nat outside
Router(config-if)# exit
```

Configuro una ACL contenente l’elenco degli indirizzi d’origine che andranno tradotti (a cui vogliamo permettere la navigazione in internet).

**Sintassi del comando: `access-list numeroACL permit/deny SOTTOINSIEME_INDIRIZZI_DA_TRADURRE WILDCARD_MASK`**

```
Router(config)# access-list 110 permit ip 192.168.0.0 0.0.0.15 any
```

Per concedere a tutti gli host configurati nella LAN di accedere ad internet (traduzione di tutti gli host nella LAN), posso utilizzare la sintassi

```
Router(config)# access-list numeroACL permit ip any any
```

Configurazione del servizio NAT sull’interfaccia:

```
Router(config)# ip nat inside source list <ACL_ID> interface <Interface_ID> overload
```

Tutto il traffico definito dall’ACL, verrà inoltrato verso l’esterno traducendo gli indirizzi privati con l’indirizzo pubblico dell’interfaccia FastEthernet 0/0.

# Port Forwarding

Nel NAT Dinamico ed Overload, le traduzioni nella NAT Table vengono create solo quando il traffico parte dall’interno della LAN e raggiunge l’esterno (Mentre nel NAT Statico, sono configurate manualmente dall’amministratore di rete, e rimangono indefinitamente nella memoria del router).

Per esporre un host al di fuori della rete privata, in caso d’uso di NAT Dinamico o Overload, dobbiamo utilizzare la funzionalità di port forwarding, ed impostare manualmente una regola di traduzione.

Mapping tra indirizzo esterno e porta → indirizzo interno e porta servizio

```
Router(config)# ip nat inside source static tcp [IPInterno] [PortaInterna] [IPEsterno] [PortaEsterna]
```