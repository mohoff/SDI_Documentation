

****
LDAP
****

LDAP Einführung
###############

LDAP (Lighweight Directory Access Protocol) ist ein Protokoll mit dem über das Netzwerk auf spezielle Verzeichnisse zugegriffen werden kann. LDAP-Verzeichnisse zeichnen sich dadurch aus, dass sie sich automatisch replizieren- und über mehrere Systeme verteilen lassen. In LDAP-Verzeichnissen können Informationen verschiedenster Art gespeichert werden. Häufig sind die Informationen auf Benutzer innerhalb eines Unternehmens bezogen. Üblich sind beispielsweise Namen, Login-Daten und E-Mail-Adressen.

Daten dieser Art könnten ebenfalls in Datenbanken abgespeichert werden. Ein Vergleich zwischen Verzeichnissen und (relationalen) Datenbanken liegt also nahe. Der wohl größte Unterschied ist die Strukturierung der Daten. Bei Datenbanken sind einzelne Datensätze in Form von Tupeln in einer Vielzahl von Tabellen gespeichert, die sich untereinander referenzieren. Verzeichnisse sind baumartig, also hierarchisch, innerhalb eines **Directory Information Tree** (DIT) strukturiert. Dieser enthält eine Menge von **Objekten**, die wiederum in **Container** und **Blätter** unterteilt sind. Container können wiederum weitere Objekte, also Container und Blätter, enthalten. Die Container an den "Enden" sind üblicherweise die gespeicherten Personen. Diese enthalten eine Menge an Blättern, die die einzelnen Attribute der Person darstellen. Ein solches Attribut ist beispielsweise ein Name, eine E-Mail-Adresse oder Telefonnummer, etc. Jedes dieser Attribute hat einen Typ, der in einer **objectClass** definiert ist. 
Die generelle Struktur des Baums ist in einem **Schema** definiert.

Die Objekte eines Baums werden über den sog. **Distinguished Name** (DN) referenziert. Dieser setzt sich aus einer Reihe **Relative Distinguished Names** (RDN) zusammen, welche folgendermaßen unterteilt werden können:

- **Domain Component** (dc): Die Identifikation des Unternehmens. Üblicherweise ist dieser an dessen Internetdomain angelehnt, wobei die Punkte entfernt werden und die einzelnen Domainbestandteile in eigene RDNs verpackt werden. Aus *beispiel.de* wird also *dc=beispiel,dc=de*
- **Organizational Unit** (ou): Definiert die einzelnen Organisationseinheiten (Abteilungen) innerhalb des Unternehmens
- **Common Name** (cn): Der Name über den eine Person referenziert wird.

RDNs werden bei der Zusammensetzung des DN mit einem Komma als Abstandhalter aneinandergeheftet. Ein Angestellter *Max Mustermann* des Unternehmens *Beispiel.de*, dem die Rolle *Project-Manager* in der Abteilung *Development* zugeteilt ist, könnte beispielsweise den DN *cn=mustermann,ou=pm,ou=dev,dc=beispiel,dc=de* haben.

.. topic:: Abkürzungen

  .. glossary::
    DIT
      Directory Information Tree
    DC
      Domain Component
    DN
      Distinguished Name
    RDN
      Relative Distinguished Name
    CN
      Common Name
    OU
      Organizational Unit

Exercises
#########

Apache Directory Studio
***********************

Das Apache Directory Studio (ADS) ist ein auf Eclipse basierendes Tool, mit dem CRUD-Operationen auf LDAP-Datenbanken
ausgeführt werden können. Es ist für Windows, Linux und Mac OS X verfügbar und kann `hier <https://directory.apache.org/studio/downloads.html>`_ heruntergeladen werden.

Das Apache Directory Studio vereinfacht den Installationsprozesses und eignet sich hervorragend, um LDAP-Verzeichnisse zu durchsuchen. Für Administrierungsaufgaben sind jedoch Web-basierte Tools, wie der LDAP Account Manager (s. u.) geeigneter.

Um sich im Directory Studio mit einem LDAP-Server zu verbinden, muss der im untenstehenden Screenshot mit **1.** markierte Button im Interface betätigt werden, woraufhin das rechtsstehende Dialogfenster erscheint. Hier können die Verbindungsdaten eingegeben werden. Der **Verbindungsname** dient zum Zweck der Anzeige innerhalb des Programms und kann frei gewählt werden. **Hostname** und **Port** entprechen der Adresse des LDAP Servers. In diesm Fall handelt es sich um den der HdM. Als **Verschlüsselungs-Methode** kann optional TLS eingestellt werden.

.. image:: images/LDAP/01-neue-verbindung.png

Mit einem Klick auf "Weiter" gelangt man auf die nächste Seite des Dialogfensters.

.. image:: images/LDAP/02-neue-verbindung-2.png

In diesem kann man sich authentifizieren. Im Fall des HdM-LDAP-Servers kann der Zugriff auch anonym erfolgen. Mit einem Klick auf "Fertigstellen" ist die Einrichtung abgeschlossen.

Um einen Eintrag zu finden muss die Filterfunktion bemüht werden. In diesem Beispiel ist die UID des gesuchten Benutzers bekannt. Es soll nach dem Benutzer mit der UID **dh055** gesucht werden. Hierfür wird der Zweig, in dem sich der Benutzer befindet, rechts geklickt werden und im Kontextmenü der Eintrag **Kind-Einträge filtern...** ausgewählt werden.

.. image:: images/LDAP/03-filtern.png

In der erscheinenden Maske wird die Abfrage formuliert. In diesem Fall lautet diese **(uid=dh055)**. Abfragen werden in einer speziellen LDAP-Syntax erstellt. Mehr dazu `hier <http://www.ldapexplorer.com/en/manual/109010000-ldap-filter-syntax.htm>`_.

.. image:: images/LDAP/04-filtern-2.png

Nach der Bestätigung durch **OK** wird der gesuchte Eintrag auf der Oberfläche angezeigt.

.. image:: images/LDAP/05-filtern-3.png


.. topic:: Hinweis

Standardmäßig werden im Directory Studio nur 1000 Einträge angzeigt. Bei Verzeichnissen, die mehr Einträge enthalten, muss der Wert entsprechend angehoben werden. Dazu muss der betroffene Zweig im LDAP Browser rechts geklickt werden -> Eigenschaften -> Verbindung -> Reiter "Browser Optionen" -> "Max. Anzahl". Der gewünschte Wert kann dort eingegeben werden.


Selbiges Ergebnis kann auch über die Kommandozeile mit dem Tool **ldapsearch** erzielt werden. Dieses befindet sich im Paket **ldap-utilities**.
Der Befehl zur Suche des Benutzers **dh055** lautet ``ldapsearch -x -W -b "ou=userlist,dc=hdm-stuttgart,dc=de" -p 389 -h "ldap1.mi.hdm-stuttgart.de" uid=dh055``. Die Kommandozeile zeigt daraufhin dieselben Informationen wie das Directory Studio.


Einrichtung eines LDAP-Servers
******************************

Man unterscheidet zwischen dem OpenLDAP Server Daemon im Package ``slapd`` und LDAP
Management Utilities im Package ``ldap-utils``.

::

  sudo apt-get install slapd ldap-utils

Während der Installation muss man Admin-Credentials festlegen, die für den
``rootDN`` der LDAP-Datenbank gesetzt werden. Per default ist heißt der ``rootDN``

::

  cn=admin,dc=mi,dc=hdm-stuttgart,dc=de

Neben der eigentlichen LDAP-Datenbank, in der später Daten gespeichert werden, wird eine Config-Database erstellt.

Die Installation erstellt eine lauffähige Konfiguration, darunter eine Datenbank in der die LDAP-Daten gespeichert werden koennen.

Der ``baseDN`` dieser Instanz wird vom Domainnamen des localhosts abgeleitet. Alternativ kann man die Datei ``/etc/hosts`` editieren, um manuell einen
Domainnamen für localhost vergeben, sodass ein erwünschter baseDN erstellt
werden kann. Die Default-Konfiguration in unseren VMs ist daher

::

  dc=mi,dc=hdm-stuttgart,dc=de

Die ``config``-Datenbank
++++++++++++++++++++++++

Der Inhalt der Config-Datenbank sieht aus wie folgt:

.. code-block:: html
  :linenos:

  /etc/ldap/slapd.d/
  /etc/ldap/slapd.d/cn=config
  /etc/ldap/slapd.d/cn=config/cn=module{0}.ldif
  /etc/ldap/slapd.d/cn=config/cn=schema
  /etc/ldap/slapd.d/cn=config/cn=schema/cn={0}core.ldif
  /etc/ldap/slapd.d/cn=config/cn=schema/cn={1}cosine.ldif
  /etc/ldap/slapd.d/cn=config/cn=schema/cn={2}nis.ldif
  /etc/ldap/slapd.d/cn=config/cn=schema/cn={3}inetorgperson.ldif
  /etc/ldap/slapd.d/cn=config/cn=schema.ldif
  /etc/ldap/slapd.d/cn=config/olcBackend={0}hdb.ldif
  /etc/ldap/slapd.d/cn=config/olcDatabase={0}config.ldif
  /etc/ldap/slapd.d/cn=config/olcDatabase={-1}frontend.ldif
  /etc/ldap/slapd.d/cn=config/olcDatabase={1}hdb.ldif
  /etc/ldap/slapd.d/cn=config.ldif

Direkte Änderungen in der config-Datenbank sind nicht empfohlen, man soll besser über das LDAP Protokoll (Tool aus dem Package ``ldap-utils``) Änderungen vornehmen.

Das LDAP-Protokoll
******************

Befehl ``ldapsearch``:
::

  [sudo] ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config dn

Variation davon:
::

  [sudo] ldapsearch -x -LLL -H ldap:/// -b dc=example,dc=com dn

Entweder ohne Authentifizierung (Parameter ``-x``) oder mit "Simple Authentication
and Security Layer" (SASL) (-Y <SASL mechanism>).

.. topic:: ``ldapsearch``

  .. glossary::
    ``-Q``
      Use SASL Quiet mode. Never prompt.

    ``-LLL``
      Displaying: restricts output to LDIFv1, hides comments, disables
      printing of the LDIF version (each "L" restricts output more)

    ``-Y <mechanism>``
      Authentication: specifies the authentication mechanism. Common ones are ``DIGIEST-MD5``, ``KERBEROS_V4`` and ``EXTERNAL``.
      Here: ``EXTERNAL`` which enables authentication over a lower level security mechanism like TLS.

    ``-h <URIs>``
      Specify URI(s) referring to the LDAP server(s). Default is ``ldap:///``
      which implies LDAP over TCP. Used ``ldapi:///`` also uses the protocol LDAP but uses IPC
      (UNIX-domain socket) instead of TCP.

    ``-b <searchbase>``
      Specify a searchbase as the starting point for the search. In our
      case ``cn=config``

    ``-x``
      Use simple authentication instead of SASL.

    ``<filter>``
      Specifies an output filter. If not specified, the default filter ``(objectClass=*)``
      is used. We used ``dn``, so all distinguished names inside the searchbase will be displayed


LDIF Files
**********

Mit LDIF Files lassen sich LDAP-spezifische Daten speichern, z.B. um Einträge im LDAP Verzeichnis zu speichern, zu ändern oder hinzuzufügen.
Über ``slapadd`` im Terminal (LDAP-Server zur Sicherheit dafür stoppen) oder die
Import-Funktion des Apache Directory Studios lassen sich LDIF Files importieren.

Ein LDIF-File kann z.B. folgendermassen aussehen:

.. code-block:: html
  :linenos:

  dn:dc=betrayer,dc=mi,dc=hdm-stuttgart,dc=de
  changetype: add
  objectclass: dcObject
  objectclass: organizationalUnit
  dc: betrayer
  ou: config
  ou: betrayer Dot com

  dn: ou=departments,dc=betrayer,dc=mi,dc=hdm-stuttgart,dc=de
  changetype: add
  objectClass: top
  objectClass: organizationalUnit
  ou: departments

  dn: ou=software,ou=departments,dc=betrayer,dc=mi,dc=hdm-stuttgart,dc=de
  changetype: add
  objectClass: top
  objectClass: organizationalUnit
  ou: software

  dn: ou=devel,ou=software,ou=departments,dc=betrayer,dc=mi,dc=hdm-stuttgart,dc=de
  changetype: add
  objectClass: top
  objectClass: organizationalUnit
  ou: devel

  dn: uid=beam,ou=devel,ou=software,ou=departments,dc=betrayer,dc=mi,dc=hdm-stuttgart,dc=de
  changetype: add
  objectClass: inetOrgPerson
  uid: beam
  cn: Jim Beam
  givenName: Jim
  sn: Beam
  mail: beam@betrayer.com

Mit diesem LDIF-File werden mehrere neue OUs dem DIT hinzugefügt. Außerdem wurde ein neuer User hinzugefügt

LDAP mit Thunderbird
*********************************
Von einem Mail-Klienten aus (in unserem Beispiel Thunderbird) kann auf die Einträge im LDAP-Verzeichnis zugegriffen werden


Via Tools->Address Book->New->LDAP Directory (in Thunderbird) kann ein neues LDAP-Verzeichnis hinzugefügt werden.

.. image:: images/addressbooksettings.png

Ausserdem muss das Verzeichnis heruntergeladen werden.

.. image:: images/offline.png

Mit dem korrekten Filter können nun User-Einträge angezeigt werden (ein zusätzlicher User wurde zuvor angelegt).

.. image:: images/addressbook.png


LDAP Filter Search
******************

Filter kann man über das CLI oder über das Apache Directory Studio festlegen.
Die ``ldapsearch``-Syntax ist oben aufgeführt.

Im Apache Directory Studio stellt man Filter ein, indem man auf den zu filternden
Knoten rechtsklickt und "Filter Children" auswählt. Im Popup-Fenster lässt ssich
dann ein Suchstring eingeben. Um die Syntax näher zu beleuchten, hier ein paar
Beispiele:

.. topic:: Beispiele zu LDAP Search Filtern

  .. glossary::
    ``(objectClass=*)``
      default Search Filter. Lässt alle objectClasses zu.

    ``(uid=*b*)``
      Jeder uid-Eintrag, der ein "b" enthaelt.

    ``(cn=b*)``
      Jeder uid-Eintrag, der mit einem "b" beginnt.

    ``(&(objectClass=user)(email=abc*))``
      Jeder Eintrag mito ``objectClass=user`` UND einer E-Mail-Adresse, die
      mit "abc" beginnt.


Allgemein: die Search-Syntax unterstützt Operatoren (!, &, |, =, ~=, <=, >=) und
Wildcards (*). Gruppierungen erfolgt durch Einklammern. Falls nach reservierten
Sonderzeichen gesucht werden muss (Klammern, !, ^, ...) lassen sich diese im
Suchstring escapen.


Search Filter Aufgaben
++++++++++++++++++++++

Der Filter ``(uid=b*)`` filtert Einträge mit einem Attribut uid welches mit "b" beginnt.

Der Filter ``(|(uid=*)(ou=d*))`` filtert Einträge die ein uid-Attribut haben oder deren OU mit dem Buchstaben “d” beginnt.

Einträge erweitern
******************
Der User Jim Beam bekommt eine ObjectClass ``posixAccount`` mit Hilfe des folgenden .ldif-files:

.. code-block:: html
  :linenos:

  dn: uid=beam,ou=devel,ou=software,ou=departments,dc=betrayer,dc=mi,dc=hdm-stuttgart,dc=de
  changetype: modify
  add: objectClass
  objectClass: posixAccount
  -
  add: uidNumber
  uidNumber: 600
  -
  add: gidNumber
  gidNumber: 600
  -
  add: homeDirectory
  homeDirectory: /home/beam/

LDAP Account Manager
********************
Der LDAP Account Manager (LAM) stellt Funktionen zur Administration von LDAP-Verzeichnissen über ein Webinterface zur Verfügung.
LAM kann über die Kommandozeile mit dem Befehl ``[sudo] apt-get install ldap-account-manager`` installiert werden.


Der LAM läuft ohne weiteres Zutun auf Apache-Webservern und ist nach der Installation unter der Adresse ``http://localhost/lam`` erreichbar. Auf dem Interface lassen sich sogleich die LAM-Einstellungen vornehmen. Das Default-Master-Passwort lautet **lam**.

.. image:: images/LAM/lamlogin.png

Der Reiter **General Settings** umfasst Einstellungen zur Sicherheit, Passwörtern und deren Policies, und Logging.

Damit auf den installierten LDAP-Server zugegriffen werden kann, müssen unter Server-Profiles die Daten des Servers eingestellt werden.

.. image:: images/LAM/ServerSetting.png

Zudem müssen die richtigen Security-Settings eingestellt werden:

.. image:: images/LAM/SecuritySettings.png

Im Anschluss kann man sich auf dem LDAP-Server anmelden.

Auch unter "Account Types" müssen für User, Hosts und Groups die entsprechenden LDAP-Suffixes angegeben werden:

.. image:: images/LAM/AccountSettings.png

Mit diesen Einstellungen werden eingetragenen Benutzer unter dem Reiter **Users** korrekt angezeigt:

.. image:: images/LAM/UserList.png


Unter **Modules** können die objectClasses der LDAP-Entitätstypen verwaltet werden.

Unter **Module Settings** lassen sich u.a. Einstellungen zu den UIDs für User, Groups und Hosts vornehmen. Also z.B. die Art des UID-Generators, sowie die Range, in der sich generierte UIDs befinden dürfen.


LDAP Replikation (Theorie)
*******************************
LDAP Replikation dient der Ausfallsicherheit, so dass die LDAP services immer noch verfügbar sind wenn ein Knoten in der LDAP-Struktur crasht.

Im HdM-Netzwerk gibt es einen LDAP-Master und mehrere LDAP-Slaves (ldap1.mi etc..).
Abhängig von der Konfiguration können Updates entweder vom Master auf jedem Slave eingespielt werden (Single-Source) oder bidirektional.
