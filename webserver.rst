
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
* der Browser interpretiert das HTML-Dokument, inklusive der enthaltenen Skripte, und zeigt sie dem Benutzer an 

Virtual Hosts
#############

TLS
###

Übungen
#######

Einrichtung des Apache Webservers
*********************************

Eigene Seiten hosten
*********************

Einrichtung von virtuellen Hosts
********************************

SSL-Einrichtung
***************

LDAP Authentifizierung
**********************

