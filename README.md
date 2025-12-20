# Appunti Laboratorio Reti di calcolatori - Emiddio Ingenito

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
[![MkDocs](https://img.shields.io/badge/Built%20with-MkDocs-indigo)](https://www.mkdocs.org/)

**Raccolta di appunti, approfondimenti, esercizi e configurazioni Cisco per il corso di Laboratorio di Reti di calcolatori, Università degli Studi di Milano**
> [!NOTE]  
> Potete scaricare, modificare, ridistrubuire il materiale contenuto in questo repository, citando il creatore. Non usare per scopi commerciali senza licenza.

## [🚀 READ THEM ONLINE](https://emikodes-unimi.github.io/retilab/)

---

## 📚 Indice:

### 1. Apparati di Rete
* **Hub:** Funzionamento a Livello 1, domini di collisione, Half Duplex.
* **Switch:**
    * Logica di smistamento (Livello 2).
    * Tabella CAM e processo di *Learning, Forwarding, Filtering*.
    * Gestione del Flooding e Broadcast.
* **Bridge:** Differenze con gli switch e segmentazione della rete.

### 2. Cablaggi Strutturati
* **Cavi Ethernet:**
    * Doppini intrecciati e cancellazione del rumore (diafonia).
    * Tipologie: UTP, FTP, STP, SFTP.
* **Pinout e Standard:** T568A vs T568B.
* **Cavi Straight vs Crossed:**
    * Differenze MDI vs MDI-X.
    * Funzionamento dell'Auto-MDI/MDI-X nelle NIC moderne.
* **Fibra Ottica:** Core, Cladding, Coating e principi di rifrazione.

### 3. Protocollo DHCP
* **Funzionamento:** Il processo DORA (Discover, Offer, Request, Ack).
* **Gestione:** Lease time e rinnovo.
* **DHCP Relay Agent:** Configurazione del router come proxy per reti remote (`ip helper-address`).
* **Laboratorio:** Configurazione Pool DHCP su Server e Router Cisco.

### 4. Routing
* **Concetti Base:** Control Plane (RIB) vs Data Plane (FIB).
* **Selezione della Rotta:**
    1.  Validità Next Hop.
    2.  Longest Prefix Match.
    3.  Distanza Amministrativa (AD).
    4.  Metrica.
* **Routing Statico:** Configurazione manuale e sintassi Cisco.
* **Routing Dinamico (RIP):**
    * Differenze tra RIPv1 (Classful, Broadcast) e RIPv2 (Classless, Multicast, Auth).
    * Algoritmo Distance Vector e limitazioni.
    * Comandi: `network`, `passive-interface`, `version 2`.

### 5. VLAN (Virtual LAN)
* **Concetti:** Segmentazione del dominio di broadcast a Livello 2.
* **Trunking (IEEE 802.1Q):** Frame tagging e configurazione porte Trunk/Access.
* **Routing Inter-VLAN:**
    * Soluzione legacy (link fisici multipli).
    * Soluzione **Router on a Stick** (Sotto-interfacce e incapsulamento dot1Q).

---
Emiddio Ingenito [emikodes-UniMI](https://github.com/emikodes-UniMI) [emikodes](https://github.com/emikodes)
