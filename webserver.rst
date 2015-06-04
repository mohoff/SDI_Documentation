
*********
Webserver
*********

Webserver Einführung
####################
Die Aufgabe eines Webservers ist es, statische Dateien zur Darstellung von Webseiten auf Anfrage auszuliefern. Bei den Dateien handelt es sich vorrangig um HTML-Dokumente, Stylesheets und Bild-Dateien. Jede dieser Dateien wird über eine separate Anfrage angefordert. Üblicherweise verwendet man für diesen Vorgang das HTTP-, bzw. HTTPS-Protokoll. Der Ablauf von der Anfrage bis zur Darstellung einer Webseite kann grob in folgenden Schritten zusammengefasst werden:

* Der Benutzer gibt in die Adresszeile des Browsers die URL der gewünschten Seite ein
* Nachdem der zuständige Server per DNS identifiziert -- also dessen Name aufgelöst -- wurde, wird die Anfrage an den Server gesendet
* Der Server empfängt die Anfrage und...

  * ent- und verschlüsselt Anfrage und Antwort, falls TLS eingesetzt wird 
  * überprüft ggf., ob der der Nutzer zum Empfang der Ressource berechtigt ist (Authentifizierung/Authorisierung)
  * generiert ggf. dynamische Teile der Website
  * protokolliert ggf. die Anfrage
  * cacht ggf. die Anfrage für zukünftige Anfragen der gleichen Ressource

* ... und antwortet mit der angeforderte Seite, bzw. einer Fehlerseite, falls die Ressource nicht gefunden wurde / die Authentifizierung fehlgeschlagen ist / etc.
* der Browser interpretiert die vom Webserver erhaltenen Daten, inklusive der Skripte, rendert die Seite und zeigt sie dem Benutzer an.

Virtual Hosts
#############
Falls auf einem Webserver mehrere Domains gehostet werden sollen, benötigt man Virtual Hosting. Es gibt zwei Techiken um dies zu erreichen: IP- und namensbasiertes Virtual Hosting.

IP-basiertes Virtual Hosting
****************************

Beim IP-basierten Virtual Hosting hat jeder Host ( oder Site ) eine eigene IP- Addresse.
Der Webserver hat dafür mehrere Netzwerkschnittstellen ( physikalisch oder virtuell ) mit verschiedenen IP-Addressen.

Der Server hat zwei Möglichkeiten Anfragen zu beantworten:

-Er öffnet für jede IP-Adresse einen wartenden Socket
-Oder er öffnet nur einen Socket für alle Schnittstellen.

Im Endeffekt kann der Server anhand der IP- Adresse, auf der eine Anfrage angekommen ist, entscheiden, welche Site zurückgeliefert werden muss.

Namensbasiertes Virtual Hosting
*******************************

Namensbasierte virtuelle Hosts verwenden mehrere Hostnamen mit derselben IP. Dies erlaubt dem Server, mehrere Webseiten auf derselben IP zu betreiben.
Der Browser schickt hierbei im Request den Ziel-Hostname mit, der Server kann dann die entsprechende Site zurückliefern.

TLS
###

Bei TLS ( Nachfolger von SSL ) handelt es sich um ein Verschlüsselungsprotokoll.
Es wird vor allem mit HTTPS eingesetzt.

Funktionsweise:

Nach dem Aufbau einer TCP- Verbindung wird der SSL- HAndshake vom Klienten gestartet: Der Klient sendet einige Spezifikationen über sich ( TLS- Version, bevorzugte Verschlüsselung etc. )

Der Server entscheidet dann welche Verschlüsselung verwendet werden soll

Der Server sendet dem Klienten anschließend sein Zertifikat, welches der Klient verifizieren muss.
Anhand des verifizierten Zertifikats weiß der Klient, dass es sich beim verbundenen Server tatsächlich um den Server handelt, der angesprochen werden sollte, und nicht etwa um einen anderen Server (MITM- Angriff).

Nun wird ein Schlüssel generiert und per assymmetrischer Verschlüsselung ausgetauscht. 
Mit diesem Schlüssel können nun Informationen verschlüsselt ausgetauscht werden.

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

