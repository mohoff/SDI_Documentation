
***
DNS
***

DNS Introduction
################

Was ist DNS und was macht ``nslookup``?
***************************************

DNS (Domain Name Service) ist ein Netzwerkprotokoll, mit dem logische Netzadressen in physische und umgekehrt umgewandelt werden. Man spricht je nach Aufloesungsrichtung von "lookup" oder "reverse lookup".

* Domainnamen <-> IP-Adressen
* Rechnernamen (in der gleichen Domaene) <-> IP-Adressen
* MX-Records
* Nameservice

Der Windows- und Linux-CLI-Command fuer DNS-lookups ist ``nslookup``. Optional koennen abhaengig von der Aufloesungsrichtung Domainnamen bzw. IP-Adressen angegeben werden, die aufgeloest werden sollen.

Falls die aufzuloesende Maschine in der gleichen Domaene wie der Requestor liegt, kann mit ``nslookup <hostname>`` dieser direkt ohne Kenntnis ueber den FQDN (Fully Qualified Domain Name) aufgeloest werden. Wenn die
Zielmaschine nicht in der gleichen Domaene liegt, erfolt der Lookup mit ``nslookup <hostname>.<domain>``.

Eigenheiten von ``nslookup``
++++++++++++++++++++++++++++
Folgende Befehle wurden ausgefuehrt:
::
		nslookup hdm-stuttgart.de		# geht nicht
		nslookup www.hdm-stuttgart.de		# geht
		nslookup mi.hdm-stuttgart.de		# geht

**ERKLAERUNG...mit Goik nochmal absprechen**

Warum braucht man DNS?
**********************
* IP Adressen koennen sich aendern. Ein referenzierender Domainname kann dagegen immer gleich bleiben.
* IP Adressen kann man sich schlecht merken, Domainnamen sind einpraegsamer.
* Fuer einen Domainname koennen mehrere IP-Adressen existieren (Ausfallsicherheit, Lastverteilung)

``nslookup`` im Detail
**********************
Mit ``nslookup`` allein, startet man das Tool, ueber das sich DNS-Server genauer abfragen lassen.

*Eingabe*
::
		> nslookup
		>> set type=mx
		>> mi.hdm-stuttgart.de

*Ausgabe*
::
		mx1.mi.hdm-stuttgart.de		
		mx2.mi.hdm-stuttgart.de
		mx3.mi.hdm-stuttgart.de
		mx4.mi.hdm-stuttgart.de
		mx5.mi.hdm-stuttgart.de

Wie in der Ausgabe zu sehen ist, hat der DNS-Server auf der Maschine ``mi.hdm-stuttgart.de`` sogar 5 Mailserver eingetragen. Inn diesem Fall dient das zur Lastverteilung (und auch Ausfallsicherheit durch Redundanz). Der DNS-Server kann so konfiguriert werden, dass er die 5 hinterlegten Adressen unterschiedlich auslastet. So kann z.B. angegeben werden, dass ``mx1.mi.hdm-stuttgart.de`` 50% aller Anfragen (=~ Traffic) erhalten soll, da er der leistungsstaerkste unter den 5 Eintraegen ist.

*Eingabe*:
::
		> nslookup
		> server sdi1a.mi.hdm-stuttgart.de

Erklaerung: Jetzt spricht man mit Eingaben direkt den DNS-Server auf ``sdi1a.mi.hdm-stuttgart.de`` an.



Beispiel google.de
++++++++++++++++++

Bei der Eingabe von
:: 
		nslookup www.google.de

kommt ein Eintrag zurueck. Bei der Eingabe von
::
		nslookup google.de

kommen jedoch mehrere Eintraege zurueck. Mit der ``rrset-order``-Einstellung im DNS-Server kann die Reihenfolge
der zurueckgegebenen A- (IPv4) oder AAAAA-Eintraege (IPv6) festgelegt werden, z.B. eine zufaellige Reihenfolge. Diese haette zur Folge,
dass die gelisteten IP-Adressen gleich stark ausgelastet werden (Ziel: Lastverteilung und Ausfallsicherheit).

Beispiel hdm-stuttgart.de
+++++++++++++++++++++++++
*Eingabe:*
::
		> set type=ns
		> mi.hdm-stuttgart.de

Die Ausgabe davon hat 3 Eintraege zur Ausfallsicherheit.

*Eingabe:*
::
		> hdm-stuttgart.de

Die Ausgabe davon hat 5 Eintrage. 2 davon intern, 3 davon sind von BelWue, dem Forschungsnetzwerk, an das die HdM angeschlossen ist. Das haengt damit zusammen, dass BelWue verlangt, dass 2 DNS ausserhalb der Einrichtung liegen muessen.

DNS Secure
**********
Die Domain Name System Security Extensions (DNSSEC) sind eine Reihe von Internetstandards, die DNS um Sicherheitsmechanismen zur Gewährleistung der Authentizität und Integrität der Daten erweitern. Ein DNS-Teilnehmer kann damit verifizieren, dass die erhaltenen DNS-Zonendaten auch tatsächlich identisch sind mit denen, die der Ersteller der Zone autorisiert hat. DNSSEC wurde als Mittel gegen Cache Poisoning entwickelt. Es sichert die Übertragung von Resource Records durch digitale Signaturen ab. Eine Authentifizierung von Servern oder Clients findet nicht statt.

*Quelle: http://de.wikipedia.org/wiki/Domain_Name_System_Security_Extensions*

DNS Zones
*********
Man braucht 2 Zonen, um einen einfachen DNS-Service einzurichten.

1. Forward-Zone: Rechnername -> IP-Adresse
2. Reverse-Zone: IP-Adresse -> Rechername

Bei der Administrierung von DNS-Services kann das umstaendlich sein, da fuer jeden Eintrag im semantischen Sinn jeweils 2 Zone-Eintraege getaetigt werden muessen. Durch Managing-Tools oder Hooks stehen haber Massnahmen zur Verfuegung, diesen Prozess zu vereinfachen.

DNS Forwarding
**************
DNS-Server sind hierarchisch in einer Baumstruktur geordnet. Wenn ein "Leaf"-DNS, z.B. der DNS-Service den wir im Rahmen der Veranstaltung aufsetzen, eine Eingabe nicht aufloesen kann, geht die Anfrage weiter an einen uebergeordneten DNS. Je hoeher der DNS-Server in der Struktur liegt, desto wahrscheinlicher ist id.R., dass er die Domain bzw. die IP-Adresse aufloesen kann. etwas nicht aufloesen kann, geht die Anfrage weiter an uebergeordnetes DNS, das evtl. mehr weiss.


DNS Logs
********
Logs sind default-maessig in ``/var/log``. Das ist der allgemeine Log-Ordner unter Linux, worunter viele Dienste ihre Logs ablegen. Im File ``syslog`` in diesem Verzeichnis werden u.a. DNS-Logs gespeichert, auch LDAP-Logs existieren vom Prozess ``slapd``.

Wenn Log-Files zu gross werden, koennen sie von einem eigenen Service umbenannt und seperat als Datei abgespeichert werden.

Mit ``tail`` laesst sich das Ende einer Datei ausgeben. Mit dem Parameter ``f``, also
::

		tail -f <dateiname>

kann eine Datei "live" getracked werden. Sobald in die Datei geschrieben wird, in unserem Fall also ``/var/log/syslog``, werden die letzten Aenderungen im CLI ausgegeben.

Ein DNS-Log-Eintrag kann z.B. mit einem Neustart des DNS-Services erreicht werden. Ein Neustart kann mit
::

		service bind9 restart

initiiert werden.

Verbunden mit dem Tool ``grep`` kann die Ausgabe weiter eingeschraenkt werden, z.B. mit:
::

		tail -f syslog | grep named | grep loaded

Sonstiges
*********
``hostname``
++++++++++++
Der Hostname eines Rechners kann mit ``hostname`` bestimmt werden.

``/etc/resolv.conf``
++++++++++++++++++++
Die Datei ``/etc/resolv.conf`` wird für die Namensauflösung nach DNS verwendet. ``nameserver`` ist die IP-Adresse eines DNS-Servers, der abgefragt werden soll. Bis zu drei Server werden in der Reihenfolge abgefragt in der sie aufgezählt sind. In folgendem Beispiel wird auf Localhost und auf einen Google-DNS mit der IP-Adresse 8.8.8.8 verwiesen.
::
		nameserver 127.0.0.1
 		nameserver 8.8.8.8

``/etc/hosts``
++++++++++++++
In der Datei ``/etc/hosts`` koennen konkrete Hostname<->IP-Adressen -Assoziationen eingetragen werden. Obwohl
ueblicherweise die Aufloesung ueber DNS stattfindet, wird i.d.R. die Loopback-Adresse statisch in das File eingetragen:
::
		127.0.0.1 localhost


Exercises
#########

Setup des DNS-Servers
*********************

Mithilfe von apt-get wurden zunächst die benötigten Pakete auf
dem Server installiert:
::
    apt-get update
    apt-get install bind9 bind9utils

Anschließend wurde unter /etc/default/bind9 die Option "-4" 
hinzugefügt. Die OPTIONS-Variable sah nun folgendermaßen aus:
::
    OPTIONS="-4 -u bind"

Der zusätzliche Eintrag versetzt BIND in den IPv4-Modus.

Als nächstes muss die Options-Datei von BIND bearbeitet werden.
Dies befindet sich unter ``/etc/bind/named.conf.options``

Im Block *options* wurden die folgenden Einträge hinzugefügt:

.. code-block:: html
  :linenos:
  
  options {
        directory "/var/cache/bind";
        recursion yes;                 
        //allow-recursion { trusted; }; 
        listen-on { 141.62.75.101; };   
        allow-transfer { none; };   
	
	forwarders {
	};
  ...
  };


Anschließend müssen die Zonen unter  ``/etc/bind/named.conf.local``
definiert werden:

.. code-block:: html
  :linenos:

  # Lookup zone
  zone "mi.hdm-stuttgart.de" {
    type master;
    file "/etc/bind/zones/db.mi.hdm-stuttgart.de"; # zone file path
  };

  # For reverse lookup
  zone "75.62.141.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.141.62.75"; # zone file path
  };


Nun müssen die jeweiligen Zone-Files (Forward- und Reverse-File) erstellt werden, in denen die einzelnen Aufloesungen definiert sind:

``/etc/bind/zones/db.mi.hdm-stuttgart.de`` :

.. code-block:: html
  :linenos:

  ;
  ; BIND data file 
  ;
  $TTL    604800
  @       IN      SOA     ns1a.mi.hdm-stuttgart.de. root.mi.hdm-stuttgart.de. (
                                3         ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
  ;
  
  ; name servers - NS records
          IN      NS      ns4.mi.hdm-stuttgart.de.
  ; name servers - A records
  ns1a.mi.hdm-stuttgart.de.          IN      A       141.62.75.101
  www1a.mi.hdm-stuttgart.de.         IN      A       141.62.75.101

``/etc/bind/zones/db.141.62.75`` :


.. code-block:: html
  :linenos:

  ;
  ; BIND reverse data file 
  ;
  $TTL    604800
  @       IN      SOA     ns1a.mi.hdm-stuttgart.de. root.mi.hdm-stuttgart.de. (
                                1         ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
  ;

  ; name servers - NS records
        IN      NS      ns1a.mi.hdm-stuttgart.de.

  ; PTR Records
  104   IN      PTR     sdi1a.mi.hdm-stuttgart.de.    ; 141.62.75.101
