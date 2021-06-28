# LAS2021-NGINX-LoadBalancer
![Logo UniVe](https://www.unive.it/pag/fileadmin/user_upload/extra/pid/img/loghi/logo_CF_1.png)

[![CodeFactor](https://www.codefactor.io/repository/github/ale-ben/las2021-nginx-loadbalancer/badge?s=c02b474e7b078e308bf81cdb75cc9137e02774cf)](https://www.codefactor.io/repository/github/ale-ben/las2021-nginx-loadbalancer)

Progetto per l'esame di Laboratorio Amministrazione Sistemi a.a. 2020/2021 Ca'Foscari Informatica.

# Documentazione
---
author:
- |
  Alessandro Benetton  
  \[874886\]
date: Anno Accademico 2020/2021
title: |
  Creazione di una serie di webserver con Load Balancer e log
  centralizzato tramite Docker  
  Progetto per il corso Laboratorio Amministrazione Sistemi —
  \[CT0157\]  
  Docente *Fabrizio Romano*
---

# Introduzione al progetto

L’obiettivo del progetto è creare una serie di webserver con accesso
regolato tramite load balancer e un server di log centralizzato, il
tutto sfruttando il sistema di containerizzazione **Docker**.

## Repo Github

L’intero progetto si può trovare su Github all’indirizzo
<https://github.com/ale-ben/LAS2021-NGINX-LoadBalancer>

## Struttura del sistema

Il sistema è composto da 3 componenti:

-   Webserver

    -   Elabora e risponde alle richieste dei browser

-   Load Balancer

    -   Reindirizza le richieste dei browser ai webserver

-   Log Server

    -   Raccoglie i log di tutti i servizi in un unico posto

## Struttura documentazione

Nella verrà presentato il progetto completo e verranno analizzati i file
nell’insieme.  
Dalla alla verranno analizzati i vari servizi nel dettaglio, con le
relative configurazioni e immagini docker.  
Nella sezione verrà fatto un approfondimento su un sistema alternativo
di load balancing: Docker Swarm

# Il progetto completo

## Docker compose

Verranno analizzate due varianti del docker compose, la prima relativa
al progetto, la seconda relativa ad un ambiente reale.

### Versione simulata

Questo è il docker compose usato nel progetto, in un ambiente reale una
configurazione simile avrebbe poco senso.

``` xml
version: "3"

services:
	WebServer1:
		build: Dockerfiles/webserver/.
		image: webserver
		container_name: WS1
		networks:
			- Internal
		depends_on:
			- Syslogserver
		logging:
			driver: syslog
			options:
				syslog-address: "tcp://localhost:514"
				tag: "WS1"
		restart: unless-stopped
	WebServer2:
		build: Dockerfiles/webserver/.
		image: webserver
		container_name: WS2
		networks:
			- Internal
		depends_on:
			- Syslogserver
		logging:
			driver: syslog
			options:
				syslog-address: "tcp://localhost:514"
				tag: "WS2"
		restart: unless-stopped
	WebServer3:
		build: Dockerfiles/webserver/.
		image: webserver
		container_name: WS3
		networks:
			- Internal
		depends_on:
			- Syslogserver
		logging:
			driver: syslog
			options:
				syslog-address: "tcp://localhost:514"
				tag: "WS3"
		restart: unless-stopped
	WebServer4:
		build: Dockerfiles/webserver/.
		image: webserver
		container_name: WS4
		networks:
			- Internal
		depends_on:
			- Syslogserver
		logging:
			driver: syslog
			options:
				syslog-address: "tcp://localhost:514"
				tag: "WS4"
		restart: unless-stopped
	LoadBalancer:
		build: Dockerfiles/load-balancer/.
		image: loadbalancer
		container_name: LB
		networks:
			- Internal
			- External
		ports:
			- "80:80"
		depends_on:
			- WebServer1
			- WebServer2
			- WebServer3
			- WebServer4
			- Syslogserver
		logging:
			driver: syslog
			options:
				syslog-address: "tcp://localhost:514"
				tag: "LB"
		restart: unless-stopped
	Syslogserver:
		build: Dockerfiles/rsyslog/.
		image: syslogserver
		container_name: Syslog
		volumes:
			- "[CARTELLA LOCALE LOG]:/var/log"
		ports:
			- 514:514
			- 514:514/udp
		cap_add:
			- SYSLOG
		restart: unless-stopped

networks:
	Internal:
		internal: true
	External:
```

### Analisi Docker Compose

Nel file sono presenti 6 servizi di cui: 4 web server, 1 load balancer e
1 server log.  
In questa configurazione l’unica porta da esporre verso l’esterno è la
80 indirizzata all’ip della macchina su cui girano tutti i servizi.  
Verrà esposta anche la 514 ma NON è necessario esporla fuori dalla lan
locale.  
I dettagli di configurazione dei vari servizi verranno analizzati in
seguito nelle relative sezioni, si noti però che tutti i servizi hanno
delle caratteristiche comuni:

-   *build*: Eventuale percorso al Dockerfile per construire l’immagine

-   *image*: Nome dell’immagine da usare O nome dell’immagine costruita
    tramite build

-   *container_name*: Nome univoco nel sistema del container

-   *networks*: Lista di network a cui collegare il sistema

-   *depends_on*: Lista di servizi da avviare prima del servizio in
    questione

-   *logging*

    -   *syslog-address*: Indirizzo del server di log, in questo caso
        *localhost*

    -   *tag*: Nome del servizio con cui identificarsi nel log server

-   *restart*: azione da intraprendere in caso di errore o arresto
    anomalo

### Versione effettiva

Versione utile in un ambiente reale.  
In questo caso il load balancer e i webserver vengono eseguiti su
macchine diverse.

``` xml
version: "3"

services:
	LoadBalancer:
		build: Dockerfiles/load-balancer/.
		image: loadbalancer
		container_name: LB
		ports:
			- "80:80"
		depends_on:
			- Syslogserver
		logging:
			driver: syslog
			options:
				syslog-address: "tcp://localhost:514"
				tag: "LB"
		restart: unless-stopped
	Syslogserver:
		build: Dockerfiles/rsyslog/.
		image: syslogserver
		container_name: Syslog
		volumes:
			- "[CARTELLA LOCALE LOG]:/var/log"
		ports:
			- 514:514
			- 514:514/udp
		cap_add:
			- SYSLOG
		restart: unless-stopped
```

Nel primo file troviamo il load balancer e il server log.  
La porta 80 del load balancer deve essere esposta al di fuori della lan
locale, la 514 del server log deve essere esposta SOLO SE le macchine su
cui gireranno i webserver saranno al di fuori della rete locale.

``` xml
version: "3"

services:
	WebServer:
		build: Dockerfiles/webserver/.
		image: webserver
		container_name: WS
		depends_on:
			- Syslogserver
		logging:
			driver: syslog
			options:
				syslog-address: "tcp://[IP Syslogserver]:514"
				tag: "WS-N"
		restart: unless-stopped
```

Nel secondo file troviamo la configurazione di un webserver.  
Questo dockerfile verrà distribuito a tutte le macchine che dovranno
ospitare un webserver.  
È necessario aggiornare l’indirizzo del server di log in *syslog-server*
con l’indirizzo della macchina su cui gira il servizio Syslogserver, è
inoltre necessario inserire un id univoco nella sezione *tag* del
logging al fine di consentire al server log di distinguere tra le varie
istanze del webserver.  
**Dato che non abbiamo più un network docker, è necessario esporre le
porte 80 di tutti gli host alla rete interna, al fine di consentire al
load balancer reindirizzare le richieste.**

## Come avviare un docker compose

Per avviare un docker compose è sufficiente, una volta avviato il deamon
docker, posizionarsi nella cartella contenente il file docker compose ed
eseguire il comando *docker compose up*, questo comando creerà e avvierà
tutti i servizi descritti nel file.

È inoltre possibile avviare solo una parte dei servizi specificando dopo
up il nome del servizio come definito nel docker compose.

Per terminare tutti i servizi si usa il comando *docker compose down*.

Per comandi più avanzati visitare la [documentazione ufficiale docker
compose cli](https://docs.docker.com/compose/reference/).

## Screenshot funzionamento

<figure>
<img src="images/dcLog.png" id="fig:dcLog" style="width:15cm" alt="Log di avvio servizi e richieste distribuite" /><figcaption aria-hidden="true">Log di avvio servizi e richieste distribuite</figcaption>
</figure>

Come si vede dalla , il primo servizio avviato è il server log, seguito
dai 4 webserver in parallelo e, per ultimo, dal load balancer.  
Una volta avviato, il load balancer risponde alle richieste dei client
distribuendole tra i server definiti nel config, come visibile nelle
ultime righe della .

<figure>
<img src="images/logDir.png" id="fig:logDir" style="width:10cm" alt="Cartella di salvataggio server di log ()" /><figcaption aria-hidden="true">Cartella di salvataggio server di log ()</figcaption>
</figure>

Nella si può notare il template del server di log in azione, i file
vengono suddivisi secondo il template e, come previsto, ogni ora viene
generato un nuovo file di log.

<figure>
<img src="images/logFile.png" id="fig:logFile" style="width:15cm" alt="Contenuto del file di log 2021/06/28/14-LB.log" /><figcaption aria-hidden="true">Contenuto del file di log <em>2021/06/28/14-LB.log</em></figcaption>
</figure>

La mostra il contenuto di uno dei file di log del server di log ().  
Ogni riga contiene:

1.  Un timestamp di ricezione del log

2.  Indirizzo ip del mittente

3.  Tag del servizio

4.  Pid del servizio

5.  Messaggio

Come previsto i messaggi mostrati in corrispondono con i log del
servizio LB visti in .

# I webserver NGINX

Pe l’implementazione del webserver vero e proprio useremo NGINX.  
La scelta di usare NGINX è arbitraria, si sarebbe tranquillamente potuto
usare un qualunque server web  
Partiremo da un immagine docker con NGINX pre installato e la
integreremo nel nostro insieme di servizi docker.

Nella sezione si è detto che vi saranno molteplici webserver tra cui
dividere le richieste, nella realtà questi webserver si troverebbero su
macchine differenti al fine di dividere il carico ma, dato che questo
progetto è puramente dimostrativo, qui verranno implementate nello
stesso host come istanze della stessa immagine docker (verranno comunque
evidenziati i cambiamenti necessari per implementare la soluzione multi
host).

## Dockerfile

Di seguito il dockerfile per la creazione di un singolo webhost NGINX.

``` xml
FROM nginx:1.20.0
RUN echo -e "\t\tCopying Config"
COPY Contents/nginx.conf /etc/nginx/nginx.conf
RUN echo -e "\t\tCopying WebServer"
COPY Contents/website /www/data
RUN echo -e "\t\Setting Permissions"
RUN chmod -R 555 /www/data/.
RUN chmod 555 /etc/nginx/nginx.conf
```

### Analisi Dockerfile

Scegliamo di utilizzare l’immagine *nginx* alla versione *1.21.0*
presente nella [repository di immagini ufficiale
docker](https://hub.docker.com/_/nginx).  
Si può facilmente notare che questa è una versione ufficiale poichè non
è preceduta dal nome dell’utente che la gestisce (altrimenti sarebbe
*utente/nginx*).  
La prima modifica che facciamo all’immagine è caricare il nostro file
*nginx.conf* nella cartella */etc/nginx/*. Andremo ad analizzare il file
di config nella .  
Successivamente ci assicuriamo che il config e i file del webserver
abbiano i giusti permessi, assegnando *rx* ad utente, gruppo e altri.

## Servizio docker compose

Aggiungiamo molteplici istanze del webserver web a docker compose

```
WebServer1:
	build: Dockerfiles/webserver/.
	image: webserver
	container_name: WS1
	networks:
		- Internal
```

La prima riga indica il nome univoco del servizio, ogni istanza del
webserver deve avere il proprio nome univoco.  
Riga 2 è opzionale e indica il percorso in cui effettuare la build
dell’immagine se questa non è presente.  
Riga 3 indica il nome dell’immagine. Se non è presente in locale verrà o
presa dalla repo remota o buildata (se è presente l’istruzione build).  
Riga 4 indica un nickname per il servizio.  
Riga 6 impone alla macchina di collegarsi al network *Internal*, il
quale non ha accesso alla rete esterna.  
È possibile mappare la cartella */www/data* dell’immagine ad una
cartella fisica, al fine di poter aggiornare i file del webserver senza
dover ricostruire l’immagine completa. Per fare ciò basta aggiungere la
voce

    		volumes:
                - "[CARTELLA LOCALE WEBSERVER]:/www/data"

al servizio webserver su dockercompose MA bisogna prestare attenzione
che l’immagine abbia permessi di lettura sulla cartella condivisa.

#### Per implementare un sistema multi host

le righe 5 e 6 vanno sostituite con la porta su cui esporre il servizio
nel formato *Porta fisica:Porta Virtuale*, dove:

-   *Porta Fisica* è la porta dell’host su cui esporre il servizio

-   *Porta Virtuale* è la porta del container a cui collegare la porta
    fisica

```
WebServer:
	build: Dockerfiles/webserver/.
	image: webserver
	container_name: WS
	ports:
		- "80:80"
```

Da notare che, dato che in questo caso i webserver vengono eseguiti su
macchine diverse, non è più necessario distinguere i nomi dei servizi.

## Configurazione

```
events {}
http {
	server {
		listen      80;
		location /images/ {
			root /www/static/images;
		}
		location / {
			root /www/data;
		}
	}
}
```

Nella configurazione del webserver viene definita la porta su cui
ascoltare e le posizioni in cui trovare i file, in questo caso
*"/www/static/images"* per le pagine all’indirizzo *"/images"*,
*"/www/data"* per tutte le altre.  
Nuovi path per il webserver verrano aggiunti qui, rispettando le linee
guida della [documentazione
NGINX](https://docs.nginx.com/nginx/admin-guide/web-server/web-server/).

# Il Load Balancer NGINX

A questo servizio si collegheranno tutti i client che necessitano di
accedere alla risorsa.

## Dockerfile

``` xml
FROM nginx:1.20.0
	RUN echo -e "\t\tCopying Config"
	COPY Contents/nginx.conf /etc/nginx/nginx.conf
	RUN echo -e "\t\Setting Permissions"
	RUN chmod 555 /etc/nginx/nginx.conf
```

### Analisi Dockerfile

Il file è uguale a quello visto nella tranne che per il fatto che non
carichiamo i file del webserver.

## Servizio docker compose

```
LoadBalancer:
	build: Dockerfiles/load-balancer/.
	image: loadbalancer
	container_name: LB
	networks:
		- Internal
		- External
	ports:
		- "80:80"
```

La prima riga indica il nome univoco del servizio.  
Riga 2 è opzionale e indica il percorso in cui effettuare la build
dell’immagine se questa non è presente.  
Riga 3 indica il nome dell’immagine. Se non è presente in locale verrà o
presa dalla repo remota o buildata (se è presente l’istruzione build).  
Riga 4 indica un nickname per il servizio.  
Riga 6 impone al container di collegarsi al network *Internal*, il quale
non ha accesso alla rete esterna.  
Riga 7 Consente al container di collegari al network *External*, il
quale ha accesso alla rete esterna.  
Riga 9 Specifica il mapping delle porte in ingresso, la porta del
container deve coincidere con quella specificata nel config del Load
Balancer.

#### Per implementare un sistema multi host

La riga 6 non è necessaria, dato che le macchine saranno collegate
tramite rete esterna.

## Configurazione

```
events {}
http {
	upstream balanceGroup1 {
		server WebServer1:80;
		server WebServer2:80;
		server WebServer3:80;
		server WebServer4:80;
	}

	server {
		listen 80;
		
		location / {
			proxy_pass http://balanceGroup1;
		}
	}
}
```

Alla riga 3 specifichiamo un gruppo di server di nome *balanceGroup1*.  
Dalla riga 4 alla riga 7 specifichiamo gli indirizzi dei server
appartenenti a *balanceGroup1*, in questo caso vengono utilizzati degli
indirizzi appartenenti alla rete *Internal* ma possono essere sostituiti
con normalissimi indirizzi HTTP(S).  
Alla riga 11 specifichiamo su che porta ascoltare.

#### Per implementare un sistema multi host

Sostituire gli indirizzi alle righe 4-7 con gli indirizzi effettivi dei
propri host su cui gira webserver (), prestare particolare attenzione
alle porte su cui rispondono i webserver ().

### Modalità di load balancing

NGINX supporta 3 modalità di load balancing:

1.  Round Robin

    -   Round Robin standard (Default)

    -   Round Robin pesato

2.  Least Connected

3.  Session Persistence (ip-hash)

#### Round Robin

La tecnica Round Robin consiste nel dividere il carico proporzionalmente
tra i vari host.

##### Round Robin standard (Default)

In questo caso tutti gli host hanno lo stesso peso → il carico viene
distribuito equamente tra tutti.

##### Round Robin pesato

In questo caso vengono specificati dei pesi per alcuni o tutti i server,
il round robin distribuisce le richieste tenendo conto dei pesi
specificati.  
In caso di omissione del peso questo viene considerato pari a 1.

```
upstream balanceGroup1 {
	server WebServer1:80 weight=3;
	server WebServer2:80;
	server WebServer3:80 weight=2;
	server WebServer4:80;
}
```

In questo caso specifico i server 1 e 3 riceveranno rispettivamente il
triplo e il doppio delle richieste rispetto ai server 2 e 4.

#### Least Connected

La tecnica Least Connected tiene traccia del carico di ogni server e
reindirizza al server con meno carico al momento della richiesta.  
Questa tecnica è molto utile nel caso in cui il tempo di risposta ad una
richiesta vari significativamente in base alla richiesta, in questo caso
con Least Connected evitiamo di caricare un server con molte richieste
in elaborazione.

```
upstream balanceGroup1 {
	least_conn;
	server WebServer1:80;
	server WebServer2:80;
	server WebServer3:80;
	server WebServer4:80;
}
```

#### Session Persistence (ip-hash)

Questa tecnica riassegna ad ogni ip sempre lo stesso server usando una
funzione di hash per mappare ad ogni ip un server.

```
upstream balanceGroup1 {
	ip_hash;
	server WebServer1:80;
	server WebServer2:80;
	server WebServer3:80;
	server WebServer4:80;
}
```

Informazioni più dettagliate sulle tipologie di load balancing di NGINX
si possono trovare nella [documentazione di
NGINX](https://nginx.org/en/docs/http/load_balancing.html).

# Il server log

Lo scopo di un server di log è quello di raccogliere log da altre
macchine e raggrupparli in un unico posto.

## Dockerfile

Il server log rsyslog verrà implementato senza usare un immagine docker
pre-compilata, installando le componenti manualmente.

``` xml
FROM ubuntu:21.10

RUN echo -e "\t\Updating system and installing rsyslog" \
&& apt-get update \
&& apt-get install --no-install-recommends -y rsyslog \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/*

RUN echo -e "\t\tCopying Config"
COPY Contents/rsyslog.conf /etc/rsyslog.conf

ENTRYPOINT ["rsyslogd", "-n"]
```

### Analisi Dockerfile

Partiamo caricando un immagine di *ubuntu:21.10* da *Docker Hub*, su
questa immagine, dopo aver aggiornato le sorgenti, installiamo il server
*rsyslog*.  
Una volta installato il server facciamo pulizia del garbage creato
dall’installazione, carichiamo il config di *rsyslog* e impostiamo come
punto di partenza il comando *rsyslogd -n*.

## Servizio docker compose

```
Syslogserver:
	build: Dockerfiles/rsyslog/.
	image: syslogserver
	container_name: Syslog
	volumes:
		- "[PERCORSO COMPLETO CARTELLA LOG LOCALE]:/var/log"
	ports:
		- 514:514
		- 514:514/udp
	cap_add:
		- SYSLOG
```

La prima riga indica il nome univoco del servizio.  
Riga 2 è opzionale e indica il percorso in cui effettuare la build
dell’immagine se questa non è presente.  
Riga 3 indica il nome dell’immagine. Se non è presente in locale verrà o
presa dalla repo remota o buildata (se è presente l’istruzione build).  
Riga 4 indica un nickname per il servizio.  
Riga 6 mappa una directory locale in cui salvare i log alla directory
remota */var/log*. Su questa cartella locale saranno salvati i log
ricevuti dalle macchine  
Riga 8 e 9 Aprono la port 514 in TCP e UDP per consentire al server di
ricevere i log.  
Se si intende usare solo uno dei protocolli (TCP o UDP), la porta
relativa all’altro protocollo va eliminata.  
Riga 11 specifica che il server ha bisogno di permessi aggiuntivi di
tipo *SYSLOG*, per info su questi permessi consultare *man 7
capabilities*.

## Configurazione

``` Bash
# Commentare per disabilitare UDP logging
#module(load="imudp")
#input(type="imudp" port="514")

# Commentare per disabilitare TCP logging
module(load="imtcp")
input(type="imtcp" port="514")

# Template nome file log remoto
template(name="RemoteDirTemplate" type="string" string="/var/log/remote/%$year%/%$Month%/%$Day%/%$Hour%-%APP-NAME%.log")
# Regole di log
if ($source != "localhost") then {
	action(type="omfile" dynaFile="RemoteDirTemplate")
}
```

In questa configurazione abilitiamo solo la versione TCP del servizio di
log, per abilitare anche UDP è necessario rimuovere il commento dalle
righe 2 e 3.  
Alla riga 3 e 7 definiamo le porte per il servizio di log
rispettivamente UDP e TCP, queste porte possono esssere modificate ma
DEVONO corrispondere a quelle definite alle righe 8 e 9 nella .  
La riga 10 definisce il template per il nome dei file su cui salvare i
log remoti, verrà analizzata a parte nella .  
Le righe 12 e 13 applicano il template definito alla riga 10 solo ai log
provenienti da sorgenti esterne, ovvero con l’attributo *source* diverso
da *localhost*.

### Template nome file

Il template per il nome di file è il seguente:  
*/var/log/remote/%$year%/%$Month%/%$Day%/%$Hour%-%APP-NAME%.log*

Possiamo suddividere il template in 3 parti:

1.  */var/log/remote/*

    -   Percorso FISSO della cartella root su cui salvare i log.

2.  *%$year%/%$Month%/%$Day%/*

    -   Percorso VARIABILE della cartella finale su cui salvare i log.

    -   Dipende da:

        -   *$year*

        -   *$Month*

        -   *$Day*

3.  *%$Hour%-%APP-NAME%.log*

    -   Nome del file in cui salvare i log

    -   Dipende da:

    -   -   *$Hour*

        -   *$APP-NAME*

            -   Identificativo del programma remoto da cui sono
                originati i log

            -   Può essere sostituito con *$fromhost*, l’hostname (o
                indirizzo ip de DNS non disponibile) della sorgente.

Se ad esempio la macchina con il programma *pippo* generasse un log il
01/01/1970 alle ore 00:05, il percorso finale verrebbe ad essere:  
**/var/log/remote/1970/01/01-pippo.log**  
È stato scelto questo ordine delle variabili arbitrariamente,
raccogliere i log per data e ora e, in seguito per macchina, consente di
avere una migliore visione di insieme.  
Altre alternative valide sarebbero potute essere:

-   */var/log/remote/%APP-NAME%-%$year%/%$Month%/%$Day%/%$Hour%.log*

    -   Suddivide prima per macchine e, successivamente, per data.

    -   Fornisce una migliore visione temporale per le singole macchine
        ma peggiore visione di insieme sul sistema completo.

-   */var/log/remote/%$year%/%$Month%/%$Day%/%$Hour%.log*

    -   Ignora l’attributo *APP-NAME*, raccoglie i log di tutte le
        macchine nello stesso file, suddivisi per data.

    -   Visione d’insieme sul sistema completo MA rischio di generare
        file molto pesanti e di difficile lettura.

-   Qualunque altra configurazione con le variabili presenti sopra e
    altre dalla [documentazione ufficiale
    rsynclog](https://www.rsyslog.com/doc/master/configuration/properties.html)

## Ricerca di un file di log

Usando il template definito sopra, per cercare un file di log si può
usare il seguente script bash:

``` Bash
#!/bin/sh

HOST="WS1"
YEAR=""
MONTH=""
DAY=""

LIMIT="5" # Numero massimo di elementi da visualizzare
SEPARATOR="/" # / su sistemi base Unix o Darwin, \ su sistemi base MS-DOS
BASE_DIR="./remote" # Directory di partenza

if [ -z "$HOST" ]; then
	HOST=".*"
fi

if [ -z "$YEAR" ]; then
	YEAR="[0-9][0-9][0-9][0-9]"
fi

if [ -z "$MONTH" ]; then
	MONTH="[0-9][0-9]"
fi

if [ -z "$DAY" ]; then
	DAY="[0-9][0-9]"
fi

if [ -z "$BASE_DIR" ]; then
	BASE_DIR="."
fi

REGEX=".*$SEPARATOR$YEAR$SEPARATOR$MONTH$SEPARATOR$DAY$SEPARATOR[0-9][0-9]-$HOST.log"

if [ -z "$LIMIT" ]; then
	find $BASE_DIR -regex $REGEX
else
	find $BASE_DIR -regex $REGEX | head -$LIMIT
fi
```

# Docker Swarm

## –scale

Avviando il docker compose con il flag *–scale nome_servizio=n* docker
crea automaticamente *n* istanze del servizio *nome_servizio* e gestisce
automaticamente il load balancing con tecnica Round Robin.  
Da notare che per effettuare uno scaling automatico, docker richiede che
nel compose del servizio non sia presente l’attributo
*container_name*.  
Questa era un opzione disponibile anche come flag dentro a docker
compose v2 ma è stata deprecata in favore di docker swarm.

Nel caso di questo progetto, per avviare molteplici istanze del
webserver il comando sarebbe dovuto essere *docker compose up WebServer1
–scale WebServer1=4*

## Docker Swarm

Docker Swarm è una feature che consente l’unione di più istanze docker
su macchine differenti sotto un controllo centralizzato.  
Ogni istanza di docker, chiamata *nodo*, può avere uno dei due ruoli:

-   Manager

-   Worker

Il nodo manager conosce in ogni momento lo stato dei nodi worker e si
occupa di dividere le task (e, di conseguenza, il carico) fra questi.  
Il nodo manager è anche responsabile di gestire i vari errori dei
singoli nodi, reindirizzando le task ad altri nodi worker.

Nel momento in cui si ha un docker swarm funzionante, per replicare il
sistema realizzato in questo progetto (ignorando il server log) è
sufficiente aggiungere al docker compose del webserver l’attributo
*scale: 4*.  
Questo attributo, disponibile solo se si usa docker in modalità swarm,
crea in automatico 4 istanze del webserver e gestisce il load balancing
tra queste.

Per maggiori informazioni su docker swarm visitare la [documentazione
ufficiale](https://docs.docker.com/engine/swarm/).
g
