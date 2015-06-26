
****************
Apache Webserver
****************

Webserver Einführung
####################
Die Aufgabe eines Webservers ist es, statische Dateien zur Darstellung von Webseiten auf Anfrage auszuliefern. Bei den Dateien handelt es sich vorrangig um HTML-Dokumente, Stylesheets und Bild-Dateien. Jede dieser Dateien wird über eine separate Anfrage angefordert. Üblicherweise verwendet man für diesen Vorgang das HTTP-, bzw. HTTPS-Protokoll. Der Ablauf von der Anfrage bis zur Darstellung einer Webseite kann grob in folgenden Schritten zusammengefasst werden:

* Der Benutzer gibt in die Adresszeile des Browsers die URL der gewünschten Seite ein
* Falls die URL einen Domainname enthaelt, wird dieser zunaechst per DNS auf eine IP-Adresse aufgeloest. An diese IP-Adresse wird eine HTTP-Anfrage auf Port 80 geschickt.
* Der Server empfängt die Anfrage und:

  * ent- und verschlüsselt Anfrage und Antwort, falls TLS eingesetzt wird 
  * überprüft ggf., ob der der Nutzer zum Empfang der Ressource berechtigt ist (Authentifizierung/Authorisierung)
  * generiert ggf. dynamische Teile der Website
  * protokolliert ggf. die Anfrage
  * cached ggf. die Anfrage für zukünftige Anfragen der gleichen Ressource
* Er antwortet mit der angeforderten Seite, bzw. einer Fehlerseite, falls die Ressource nicht gefunden wurde bzw. die Authentifizierung fehlgeschlagen ist.
* Der Browser des Clients interpretiert die vom Webserver erhaltenen Daten, fuehrt die Skripte aus, rendert die Seite und zeigt sie dem Benutzer an.

Virtual Hosts
#############
Falls auf einem Webserver mehrere Websites gehostet werden sollen, benötigt man Virtual Hosting. Es gibt zwei Techniken um dies zu erreichen: IP- und namensbasiertes virtuelles Hosting.

IP-basiertes Virtual Hosting
****************************

Beim IP-basierten Virtual Hosting wird jedem virtuellen Host eine eigene IP-Addresse zugewiesen. Der Webserver hat dafür mehrere Netzwerkschnittstellen (physikalisch oder virtuell) mit verschiedenen IP-Addressen.

Im Endeffekt kann der Server anhand der IP- Adresse, auf der eine Anfrage angekommen ist, entscheiden, welcher virtuelle Host angesprochen wurde.

::

    <VirtualHost 192.168.0.1:80>
      ...
    </VirtualHost>
    <VirtualHost 192.168.0.2:80>
      ...
    </VirtualHost>

Namensbasiertes Virtual Hosting
*******************************

Namensbasierte virtuelle Hosts verwenden mehrere Hostnames mit derselben IP. Dies erlaubt dem Server, mehrere Webseiten auf derselben IP zu betreiben. Der Server liest diese Information aus und reagiert entsprechend. Mit dieser Technik koennen mehrere Hosts ueber eine IP-Adresse betrieben werden. Der Vorteil ist, dass trotz IP-Adressenknappheit mehrere Hosts betrieben werden koennen.

Voraussetzungen:
* Browser gibt den gewuenschten Hostname im HTTP-Header seiner Anfrage mit an.
* DNS-Eintraege muessen so gewaehlt sein, dass sie alle in Frage kommenden Domainnamen auf die gleiche korrekte IP-Adresse uebersetzen.

::

    NameVirtualHost *:80    // fuer alle IP-Adressen wird auf Port 80 namensbasiertes virtuelles Hosting verwendet. Wenn nur "*" steht, gilt das sowohl fuer HTTP und HTTPS.

    <VirtualHost *:80>
      ServerName www.domain.tld
      ServerAlias domain.tld *.domain.tld
      DocumentRoot /www/domain
    </VirtualHost>

    <VirtualHost *:80>
      ServerName www.otherdomain.tld
      DocumentRoot /www/otherdomain
    </VirtualHost>


TLS
###

Bei TLS (Transport Layer Security, auch unter der Vorgaengerbezeichnung SSL bekannt) handelt es sich um ein Verschlüsselungsprotokoll in der OSI-Schicht 5 (Sitzungsschicht). Durch seinen erweiternden Charakter kann es verwendet werden um Protokolle hoeherer Schichten transparent zu verschluesseln. Am Beispiel von HTTP und HTTPS wird in beiden Faellen das HTTP-Protokoll verwendet, nur einmal mit der zusaetzlichen Sicherungsschicht realisiert durch TLS.

Funktionsweise
**************

Der Client startet einen Verbindungsversuch zum Server. Letzterer reagiert, indem er mit seinem eignenen Zertifikat antwortet. Der Client ueberprueft das Zertifikat und stellt sicher, dass der Servername mit dem im Zertifikat uebereinstimmt. Per assymetrischer Verschluesselung wird ein symmetrischer Schluessel ausgetauscht, der in der Sitzung zur Verschluesselung der Nutzdaten in Zukunft verwendet wird.

Im Fall von namensbasiertem virtuellen Hosting mit HTTPS gibt es eine Besonderheit zu beachten:

Bei HTTPS muss der Webserver fuer jeden Hostnamen ein eignenes Zertifikat ausliefern. Der Hostname ist dem Apache-Server aber erst nach dem TLS-Handshake bekannt. Eine Loesung besteht in der Erweiterung des TLS-Protocols um den Mechanismus Server Name Indication (SNI), welches seit TLS Version 1.2 verfuegbar ist. Hierbei wird die Hostname-Information bereits waehrend des TLS-Handshakes an den Apache-Server uebermittelt, sodass dieser das entsprechende Zertifikat zurueckgeben kann.

Exercises
#########

Einrichtung des Apache Webservers
*********************************
Zunächst wird der Apache Webserver über die Paketverwaltung mit dem Befehl ``sudo apt-get install apache2`` installiert.


Eigenen Inhalt hosten
*********************


Einrichtung von virtuellen Hosts
********************************

SSL-Einrichtung
***************

LDAP Authentifizierung
**********************

