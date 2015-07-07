
******
Nagios
******

Nagios Introduction
###################

Was ist Nagios
**************

Nagios ist eine quelloffene Software zur Überwachung von IT-Infrastrukturen. Es können einzelne **"Hosts"** überwacht werden, sowie die verschiedenen Dienste (**"Services"**), die auf diesen laufen. Unter dem Begriff "Service" sind sowohl Programme wie Webserver, DNS-Server, LDAP-Server, etc... zusammengefasst als auch interne Eigenschaften wie CPU-Auslastung, freier Festplattenspeicher, angemeldete Benutzer etc. Die Überwachung der Services erfolgt mithilfe eigenständiger Programme (**"Plug-ins"**). Für die wichtigsten Services stellt Nagios die entsprechenden Plug-ins im Paket ``nagios-plugins`` zur Verfügung. Sollte für einen Service kein vorgefertigtes Plug-in vorhanden sein (oder für den Fall, dass das mitgelieferte Plug-in nicht den eigenen Anforderungen entspricht), bietet sich die Möglichkeit, dieses selbst zu programmieren. Dabei ist nur zu beachten, dass die Ausgaben den Nagios-Richtlinien entsprechen.

Die Informationen über die überwachten Systeme werden über ein Webinterface zur Verfügung gestellt. Damit dieses nicht laufend abgefragt werden muss, bietet Nagios die Option, bei bestimmten Ereignissen Benachrichtigungen an ausgewählte Adressen zu senden. Der Benachrichtigungsservice kann dabei beliebig konfiguriert werden. Nachrichtenempfänger werden als Kontakte (**"Contact"**) hinterlegt, welche wiederum bestimmten Kontaktgruppen (**"Contact Groups"**) zugeordnet werden. Für jeden Kontakt kann individuell eingestellt werden, für welche Services auf welchen Hosts in welchem Zeitraum Benachrichtigungen verschickt werden sollen. Benachrichtigungen können wahlweise in Form von E-Mail, SMS, IM-Messages, etc. empfangen werden.


NRPE
****

Oftmals ist es nicht ohne weiteres möglich, Zustände von bestimmten Services auf dem Remote Host zu überprüfen; etwa wenn es sich um interne Services handelt, die von außen nicht sichtbar sind. Für diesen Zweck gibt es NRPE (Nagios Remote Plugin Executor). NRPE erlaubt das Ausführen von internen Checks auf dem Remote Host und besteht aus zwei Teilen: Einem NRPE-Dienst auf der Remote-Seite und dem NRPE-Plug-in auf der überwachenden Seite. Die Aufgabe des ersteren ist es, interne Überprüfungskommandos für die Außenwelt verfügbar zu machen. Bei letzterem handelt es sich um ein gewöhnliches Nagios-Plug-in, das den Status des NRPE-Dienstes abfragt. Die Kommunikation erfolgt dabei optional über eine verschlüsselte SSL-Verbindung. 

.. image:: images/Nagios/15-nrpe.png

Die Funktionsweise ist dabei die folgende:

Soll zum Beispiel der verfügbare Speicherplatz auf dem Remote Host überprüft werden, wird das Plugin **check_nrpe** mit dem Parameter **check_disk** auf dem überwachenden System ausgeführt. check_nrpe sendet hierfür den String **check_disk** an den überwachten Host. In der Konfigurationsdatei des NRPE-Dienstes sind mehrere solcher Strings definiert, die jeweils einem Kommando zugeordnet sind. Der auf Port 5666 hörende NRPE-Dienst vergleicht den ankommenden String mit denen in seiner Konfigurationsdatei hinterlegten. Findet er nun den String **check_disk** in seiner Konfiguration, führt er das zugehörige Kommando aus und schickt das Ergebnis (Exitcode und Ausgabe) an die Nagiosmaschine zurück. 
check_nrpe wiederum reicht das Ergebnis an Nagios weiter, wo es, wie die Ergebnisse anderer Plug-ins auch, dargestellt wird.

Quelle: http://wiki.monitoring-portal.org/nagios/howtos/nrpe

Alternativ können Kommandos auf dem Remote Host auch direkt über SSL mit dem **check_by_ssh**-Plug-in  aufgerufen werden. Diese Vorgehensweise ist einerseits sicherer als NRPE - auf der anderen Seite verursacht diese Methode aber ein deutlicher CPU-Overhead auf beiden Rechnern und ist somit langsamer.

Quelle: http://nagios.sourceforge.net/docs/nrpe/NRPE.pdf

Exercises
#########

Voraussetzungen
***************
In diesem Beispiel wird ein Nagios Server aufgesetzt, der einen Remote Host überwacht. Die Überwachung des Remote Hosts erfolgt teilweise mittels NRPE, teilweise ohne. 
Das *überwachende* System, auf dem Nagios läuft, hat den Hostnamen **sdi2a**. Die Adresse lautet ``141.62.75.2``.
Das *überwachte* System, also der Remote Host, hat den Hostnamen **sdi2b** und hat die Adresse ``141.62.75.2``.

Die Voraussetzungen für das überwachende System sind

1. ein funktionsfähiger Webserver
2. ein funktionsfähiger Mailserver

Falls diese Kriterien nicht gegeben sind, können die jeweiligen Programme mit den Befehlen ``apt-get install apache2`` (für den Webserver) und ``apt-get install postfix`` (für den Mailserver) installiert werden.


.. topic:: Relevante Dateien und Verzeichnisse

  .. glossary::
    allgemeine Nagios-Konfiguration
      ``/etc/nagios3/conf.d/nagios.cfg/``
    Konfiguration einzelner Services
      ``/etc/nagios3/conf.d/[service XYZ]``
    Standardkonfiguration für Services
      ``/etc/nagios3/conf.d/generic-service_nagios2.cfg``
    Standardkonfiguration für Hosts
      ``/etc/nagios3/conf.d/generic-host_nagios2.cfg``
    Konfiguration von Kontakten und Kontaktgruppen
      ``/etc/nagios3/conf.d/contacts_nagios2.cfg``
    Konfiguration von Zeitperioden
      ``/etc/nagios3/conf.d/timeperiods_nagios2.cfg``
    Verzeichnis der Überwachungsprogramme
      ``/usr/lib/nagios/plugins/``
    Kommandodefinitionen
      ``/etc/nagios-plugins/config/``
    allgemeine NRPE-Konfiguration
      ``/etc/nagios/nrpe.cfg`` (Auf dem Remote Host)
  

Einrichtung des Nagios Servers
*******************************
Der Nagios Server wird mit dem Befehl ``apt-get install nagios3 nagios-nrpe-plugin`` auf dem überwachenden System installiert. ``nagios-nrpe-plugin`` ist das Plugin mit dem später der NRPE Daemon auf dem Remote Host angesprochen wird.
Das Installationsskript fordert während der Installation zur Einstellung einiger Konfigurationswerte auf. Zunächst müssen die Mail-Einstellungen getroffen werden:

:: 

  Please select the mail server configuration type that best meets your needs.
    No configuration:
      Should be chosen to leave the current configuration unchanged.
    Internet site:
      Mail is sent and received directly using SMTP.
    Internet with smarthost:
      Mail is received directly using SMTP or by running a utility such as fetchmail. 
      Outgoing mail is sent using a smarthost.
    Satellite system:
      All mail is sent to another machine, called a 'smarthost', for delivery.
    Local only:
      The only delivered mail is the mail for local users. There is no network.
      
  1. No configuration  3. Internet with smarthost  5. Local only
  2. Internet Site     4. Satellite system

  General type of mail configuration: 2

In diesem Fall war Option **2. Internet Site** zutreffend.
Anschließend muss der FQDN der Mail-Adressen angegeben werden, an die Mails gesendet werden.

::

  The "mail name" is the domain name used to "qualify" _ALL_ mail addresses without a
  domain name. This includes mail to and from <root>: please do not make your machine
  send out mail from root@example.org unless root@example.org has told you to.
  
  This name will also be used by other programs. It should be the single, fully qualified
  domain name (FQDN).
  
  Thus, if a mail address on the local host is foo@example.org, the correct value for
  this option would be example.org.
  
  System mail name: hdm-stuttgart.de
  
Hier wurde **hdm-stuttgart.de** gewählt, da die Mails später an ``dh055@hdm-stuttgart.de`` gesendet werden sollen.


Anschließend muss noch ein Passwort für den Nagios-Admin eingegeben werden:

::

  Please provide the password to be created with the "nagiosadmin" user.
  
  This is the username and password you will use to log in to your nagios installation
  after configuration is complete.  If you do not provide a password, you will have to
  configure access to nagios yourself.
  
  Nagios web administration password:

Nach der Eingabe des Passworts ist die initiale Konfiguration des Nagios Servers abgeschlossen.
Das Admin-Passwort kann auch nachträglich mit dem Befehl ``htpasswd /etc/nagios3/htpasswd.users nagiosadmin`` geändert werden.

Über die URL *[Domain des Webservers]*/nagios3 kann nun auf das Nagios-Webinterface zugegriffen werden. Beim ersten Aufruf wird man zur Eingabe der Logindaten aufgefordert. Der Benutzername lautet **nagiosadmin** (sofern dies nicht geändert wurde) und das Passwort ist das Passwort, das in der eben durgeführten Konfiguration eingegeben wurde.

.. image:: images/Nagios/01-webinterface.png

Überwachung eines Services auf einem Remote Host
************************************************
In Nagios müssen alle Services, die überwacht werden sollen, explizit in einer Konfigurationsdatei definiert werden. Hierfür wird auf dem überwachenden System die Datei ``/etc/nagios3/conf.d/sdi2b.conf`` angelegt. In dieser muss zunächst der überwachte Host definiert werden:

::

    define host{
      use                         generic-host
      host_name                   sdi2b
      alias                       sdi2b
      address                     141.62.75.107
      check_command               check-host-alive
    }

.. topic:: Optionen

  .. glossary:: 
  
    use
      optionale Vorlage für den Host - alle nicht spezifizierten Optionen werden aus der Vorlage entnommen.
    host_name
      der Name des Hosts, mit dem er in anderen Definitionen referenziert wird
    alias
      der Anzeigename des Hosts
    address
      die IP-Adresse des Hosts
    check_command
      der auszuführende Kommando zur Überprüfung des Hoststatuses. **check-host-alive** erreicht dies mit dem Senden von ICMP-Paketen. 

  Eine vollständige Auflistung der verfügbaren Parameter befindet sich in der `offiziellen Dokumentation <http://nagios.sourceforge.net/docs/nagioscore/3/en/objectdefinitions.html#host>`_.


.. topic:: Hinweis

  In diesem und in einigen der folgenden Beispiele wird eine Vorlage verwendet (siehe Option **use**). Dies ermöglicht es, zwingend vorausgesetzte Optionen wegzulassen. Die Hostdefinition wäre in diesem Fall ohne **use** ungültig, da verpflichtende Optionen wie **contact** weggelassen wurden. Ein Blick in die Templatedefinition **generic-host** in ``/etc/nagios3/conf.d/generic-host_nagios2`` kann sich lohnen.
  

Außerdem soll der Webserver auf sdi2b überwacht werden. Hierfür wird die ``sdi2b.conf`` um folgende Servicedefinition erweitert:

::

    define service{
      use                   generic-service
      host_name             sdi2b
      service_description   HTTP Server
      check_command         check_http
    }

.. topic:: Optionen

  .. glossary:: 
  
    use
      optionale Vorlage für den Service - alle nicht spezifizierten Optionen werden aus der Vorlage entnommen.
    host_name
      der Name des überwachten Hosts. Es ist der Name, der in der Hostdefinition (s. o.) angegeben wurde
    service_description
      die Beschreibung des Services, der auf dem Webinterface angezeigt wird.
    check_command
      das auszuführende Kommando gefolgt von den mit ``!`` getrennten Argumenten (in diesem Fall ohne Argumente). Kommandos sind unter Debian im Verzeichnis ``/etc/nagios-plugins/config/`` definiert. In den Kommandodefinitionen sind wiederum die konkreten Programmaufrufe der Überwachungsprogramme eingetragen. Die verfügbaren Programme befinden sich im Verzeichnis ``/usr/lib/nagios/plugins``. Hinweise zur Benutzung der Programme können abgerufen werden, indem das jeweilige Programm mit dem Argument ``-h`` aufgerufen wird. Außerdem lohnt sich bei Unklarheiten zur Benutzung der Kommandos ein Blick in die Kommandodefinition.

  Eine vollständige Auflistung der verfügbaren Parameter befindet sich in der `offiziellen Dokumentation <http://nagios.sourceforge.net/docs/nagioscore/3/en/objectdefinitions.html#service>`_.

Die Konfiguration kann anschließend mit dem Befehl ``nagios3 -v /etc/nagios3/nagios.cfg`` überprüft werden.
Sollten keine Fehler aufgetreten sein, muss der Server neu gestartet werden: ``service nagios3 restart``

Das Webinterface zeigt nach einer kurzen Wartezeit beide Hosts an. Der überwachende Rechner wird ebenfalls angezeigt, da Nagios standardmäßig eine Konfigurationsdatei für den eigenen Host mitliefert (``/etc/nagios3/conf.d/localhost_nagios2.cfg``).

.. image:: images/Nagios/02-hostuebersicht.png

Navigiert man auf die Serviceübersichtsseite vom sdi2b, wird auch der korrekte Status des Webservers angezeigt:

.. image:: images/Nagios/07-http-up.png

E-Mail-Benachrichtigungen einrichten
************************************
Um E-Mail-Benachrichtigungen zu aktivieren, muss zunächst sichergestellt sein, dass der installierte Mailserver Mails an die angegebenen E-Mail-Adressen senden kann. In unserem Fall war dieses Kriterium nicht gegeben, sodass folgende Einstellungen in der ``/etc/postfix/main.cf`` gemacht werden mussten:
Die Zeile 

::

  mydestination = hdm-stuttgart.de, sdi2a.mi.hdm-stuttgart.de, localhost.mi.hdm-stuttgart.de, localhost
  
wurde mit 

::

    mydestination =
    
ersetzt und die Zeile

::

    strict_rfc821_envelopes = yes
    
eingefügt.

Sobald der Mailserver Mails senden kann, können die eigentlichen Einstellungen zum Versenden von Mails in Nagios getroffen werden.
Dazu muss ein Kontakt, sowie eine Kontaktgruppe in der Datei ``/etc/nagios3/conf.d/contacts_nagios2.cfg`` angelegt werden:

::

    define contact{
        contact_name                    dh055
        contactgroups                   admins
        alias                           David Hettler
        service_notification_period     24x7
        host_notification_period        24x7
        service_notification_options    w,u,c,r
        host_notification_options       d,r
        service_notification_commands   notify-service-by-email
        host_notification_commands      notify-host-by-email
        email                           dh055@hdm-stuttgart.de
    }

.. topic:: Optionen
  
  .. glossary::
  
    contact_name
      der Name des Kontakts, mit welcher der Kontakt künftig referenziert wird
    contactgroups
      Liste der Gruppen, welchen der Kontakt angehört
    alias
      optionaler Anzeigename 
    service_notification_period
      Zeitperiode, in der Mails bzgl. Services empfangen werden sollen. Die Zeitperiode ist in ``/etc/nagios3/conf.d/timeperiods_nagios2.cfg`` definiert.
    service_notification_period
      Zeitperiode, in der Mails bzgl. Hosts empfangen werden sollen. Die Zeitperiode ist in ``/etc/nagios3/conf.d/timeperiods_nagios2.cfg`` definiert.
    service_notification_options 
      wann Mails bzgl. Services gesendet werden sollen... w = warning, u = unknown, c = critical, r = recovery (wenn der Service wieder läuft)
    host_notification_options 
      wann Mails bzgl. Hosts gesendet werden sollen... d = down (wenn der Host down ist), r = recovery (wenn der Host wieder erreichbar ist)
    service_notification_commands
      welches Kommando ausgeführt werden soll, wenn eine Benachrichtigung bzgl. Services versendet werden soll
    notify-host-by-email
      welches Kommando ausgeführt werden soll, wenn eine Benachrichtigung bzgl. Hosts versendet werden soll
    email
      Die E-Mail-Adresse des Kontakts, an welche Benachrichtigungen gesendet werden.

  Eine vollständige Auflistung der verfügbaren Parameter befindet sich in der `offiziellen Dokumentation <http://nagios.sourceforge.net/docs/nagioscore/3/en/objectdefinitions.html#contact>`_.

Die Kontaktgruppe:

::

    define contactgroup{
            contactgroup_name       admins
            alias                   Nagios Administrators
            members                 dh055
    }
    
.. topic:: Optionen

  .. glossary:: 
    
    contactgroup_name
      Name der Kontaktgruppe, mit dem der die Gruppe künftig referenziert wird
    alias
      optionaler Anzeigename der Kontaktgruppe
    members
      optionale Liste aller Benutzer in der Kontaktgruppe. Die Gruppenzugehörigkeit kann, wie oben gezeigt, ebenfalls pro Kontakt in der Kontaktdefinition erfolgen.

  Eine vollständige Auflistung der verfügbaren Parameter befindet sich in der `offiziellen Dokumentation <http://nagios.sourceforge.net/docs/nagioscore/3/en/objectdefinitions.html#contactgroup>`_.

.. topic:: Tipp

    Zum Testen kann es hilfreich sein, die Zeit zwischen Serverausfall und der gesendeten Benachrichtigung zu verringern. Diese beträgt in den Standardeinstellungen nämlich einige Minuten. Die Einstellung kann pro Service in seiner Konfigurationsdatei getroffen werden oder global in der Definition des generic Service (``/etc/nagios3/conf.d/generic-service_nagios2.cfg``). Der Parameter lautet ``first_notification_delay 1``. Der darauffolgende Wert gibt die Zeit an, die gewartet wird, bevor die erste Nachricht gesendet wird. Die Zeiteinheit kann in ``/etc/nagios3/`` mit dem Parameter ``interval_length=5`` verändert werden, wobei der angegebene Wert den Sekunden entspricht. In diesem Fall ist ein Intervall also 5 Sekunden lang. Zusammen mit der Einstellung ``first_notification_delay 1`` bedeutet dies, dass 5 Sekunden gewartet wird, bevor die erste Statusnachricht gesendet wird.

Nun können Benachrichtigungen wahlweise pro Host oder pro Service in der entsprechenden Definition eingestellt werden. In diesem Fall ist ein Benachrichtigungsservice für alle Services von sdi2b wünschenswert, weswegen die Hostdefinition (``/etc/nagios3/conf.d/sdi2b.conf``) wie folgt um die Direktive **contact_groups** erweitert wird:

.. code-block:: none
  :emphasize-lines: 7

    define host{
      use                         generic-host
      host_name                   sdi2b
      alias                       sdi2b
      address                     141.62.75.107
      check_command               check-host-alive
      contact_groups              admins
    }

Anschließend muss der Server neu gestartet werden: ``service nagios3 restart``

Wird der laufende Webserver auf dem Remote Host gestoppt, spiegelt sich die Änderung sogleich auf der Weboberfläche wider:

.. image:: images/Nagios/08-http-down.png

und Nagios sendet die Mail:

.. image:: images/Nagios/05-mail.png


.. topic:: Tipp

    Zum Testen kann es hilfreich sein, die sog. **Flap-Detection** entweder global- oder für einzelne Services zu deaktivieren.  Mit Flap-Detection können häufige Statusschwankungen erkannt werden. Ändert sich der Status eines Statuses zu oft, werden die Benachrichtigungen für den Service temporär deaktiviert. Dies kann in der Praxis hilfreich sein, um unnötige Spamnachrichten bei einer Fehlkonfiguration zu vermeiden. Da beim Testen Fehler provoziert werden sollen, ist dieser Schutzmechanismus für unsere Zwecke eher nachteilig. Um Flap Detection zu deaktivieren, muss der Parameter ``flap_detection_enabled    0`` in die betreffende Servicekonfiguration eingefügt werden, bzw. der Wert von ``1`` auf ``0`` geändert werden, falls der Parameter schon vorhanden war. Soll Flap-Detection standardmäßig deaktiviert werden, muss diese Einstellung in der Servicevorlage (``/etc/nagios3/conf.d/generic-service_nagios2.cfg``) vorgenommen werden.


Einrichtung des NRPE Servers
*****************************
Auf dem überwachten System wird der NRPE Server mit dem Befehl ``apt-get install nagios-nrpe-server`` installiert.
Standardmäßig ist der Aufruf von Nagios-Plugins auf dem Remote System aus Sicherheitsgründen nur ohne Argumente erlaubt. Um Argumente zu aktivieren, muss in der Konfigurationsdatei ``/etc/nagios/nrpe.cfg`` die Option ``dont_blame_nrpe=1`` gesetzt werden. Zusätzlich muss der Zugriff des überwachenden Systems explizit gestattet werden. Dies wird durch die Option ``allowed_hosts=141.62.75.102`` erreicht.

Ebenfalls in dieser Datei sind die Kommandos definiert, wie sie vom überwachenden System aufgerufen werden. Standardmäßig sind nur Kommandos definiert, die von dem überwachenden System ohne Argumente aufgerufen werden. Bei diesen sind die Argumente hartcodiert:

::

  command[check_users]=/usr/lib/nagios/plugins/check_users -w 5 -c 10
  command[check_load]=/usr/lib/nagios/plugins/check_load -w 15,10,5 -c 30,25,20
  command[check_hda1]=/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -p /dev/hda1
  command[check_zombie_procs]=/usr/lib/nagios/plugins/check_procs -w 5 -c 10 -s Z
  command[check_total_procs]=/usr/lib/nagios/plugins/check_procs -w 150 -c 200

Da in diesem Beispiel Kommandos mit Argumenten aufgerufen werden sollen, werden diese Einträge nicht gebraucht und können auskommentiert werden. 
Eine Kommandodefinition für ein Kommando mit Argumenten sieht ähnlich aus. Der Unterschied ist, dass an die Stelle der hartkodierten Werte Argument-Platzhalter stehen. Die Kommandos zur Überwachung der Benutzer, Auslastung, Plattenspeicher und Prozesse sehen beispielsweise folgendermaßen aus.

::

  command[check_users]=/usr/lib/nagios/plugins/check_users -w $ARG1$ -c $ARG2$
  command[check_load]=/usr/lib/nagios/plugins/check_load -w $ARG1$ -c $ARG2$
  command[check_disk]=/usr/lib/nagios/plugins/check_disk -w $ARG1$ -c $ARG2$
  command[check_procs]=/usr/lib/nagios/plugins/check_procs -w $ARG1$ -c $ARG2$
  
Nachdem die Kommandos definiert wurden, muss der NRPE-Daemon neugestartet werden, damit die Änderungen übernommen werden: ``service nagios-nrpe-server restart``

Auf der Seite des überwachenden Systems müssen zur Überwachung dieser Dienste folgende Einträge in die Datei ``/etc/nagios3/conf.d/sdi2b.cfg`` eingefügt werden:

**Anzahl der Benutzer:**

::

  define service{
    use                             generic-service
    host_name                       sdi2b
    service_description             Disk Space
    check_command                   check_nrpe!check_users!20 50
  }

**Prozessorauslastung:**

::

  define service{
    use                             generic-service
    host_name                       sdi2b
    service_description             Current Load
    check_command                   check_nrpe!check_load!5.0,4.0,3.0 10.0,6.0,4.0
  }

**Festplattenspeicher:**

::

  define service{
    use                             generic-service
    host_name                       sdi2b
    service_description             Disk Space
    check_command                   check_nrpe!check_disk!20% 10%
  }
  
**Anzahl der ausgeführten Prozesse:**

::

  define service{
    use                             generic-service
    host_name                       sdi2b
    service_description             Total Processes
    check_command                   check_nrpe!check_procs!250 400
  }
  
An die Stelle der eigentlichen Überwachungskommandos tritt das vorgestellte Kommando **check_nrpe**. Damit dieses zur Verfügung steht, muss das entsprechende Plugin, wie anfangs erwähnt, mit dem Befehl ``apt-get install nagios-nrpe-plugin`` installiert worden sein. 

.. topic:: Hinweis

  Zu beachten ist hier, dass die einzelnen Argumente *NICHT*, wie bei der normalen Überwachung ohne NRPE, mit einem "**!**" getrennt sind, sondern mit einem **Leerzeichen**. Der Grund dafür ist, dass alle Argumente des aufzurufenden Kommandos zu *einem* Argument des *NRPE*-Kommandos zusammengefasst werden. D. h. z. B. im Fall der ausgeführten Prozesse: "Rufe **check_nrpe** mit zwei Argumenten auf: **'check_procs'** und **'250 400'**"


Nach einem Neustart des Servers mit ``service nagios3 restart`` zeigt die Übersichtsseite nun die per NRPE überwachten Services an.

.. image:: images/Nagios/09-nrpe-services.png

Überwachung der HTTPS-Authentifizierung
***************************************
HTTPS-Authentifizierung lässt sich mit dem Programm ``check_http --ssl -I [IP] -a [username:password]`` überwachen. Da das Kommando die Kenntnis über die Credentials von mindestens einem authorisierten Benutzer auf dem Remote Host voraussetzt, bietet sich hier die Überwachung per NRPE an. Zusätzlich will man die Credentials evtl. nicht über das Netzwerk schicken. Die Idee ist, auf dem überwachten System ein Kommando ohne Argumente zur Verfügung zustellen, welcher von dem überwachenden System aufgerufen wird. Die Credentials sind in der Kommandodefinition auf der überwachten Seite angegeben. Somit muss die überwachende Seite keine Credentials wissen und übers Netzwerk schicken.

Auf der überwachten Seite wird das Kommando in der Datei ``/etc/nagios/nrpe.cfg`` folgenermaßen definiert:

::

  command[check_http_auth]=/usr/lib/nagios/plugins/check_http --ssl -I localhost -a beam:password

Diese Zeile definiert eine neues Kommando mit der Bezeichnung **check_http_auth**, welcher das **check_http**-Programm mit den hartkodierten Argumenten **--ssl**, **-I** und **-a** aufruft. In letzterem Argument werden die Credentials angegeben. Diese sind in diesem Fall die des Beispielbenutzers **beam**. Sein Passwort ist **password**.

Anschließend wird der Daemon neu gestartet: ``service nagios-nrpe-server restart``.

Auf dem Nagios-Server auf der überwachenden Seite wird das Kommando in ``/etc/nagios3/conf.d/sdi2b.cfg`` aufgerufen:

::

  define service{
    use                             generic-service
    host_name                       sdi2b
    service_description             HTTPS Auth
    check_command                   check_nrpe_1arg!check_http_auth
  }
  
``check_nrpe_1arg`` ruft ein Kommando auf dem Remote Host nur mit dem nachfolgenden Kommando auf, also ohne zusätzliche Argumente (das Kommando wurde schließlich mit hartkodierten Argumenten definiert).

Nach einem Neustart des Services (``service nagios3 restart``) erscheint der überwachte Service auf dem Webinterface:

.. image:: images/Nagios/10-https-ok.png

Um zu überprüfen, ob der Test funktioniert, ändern wir das Passwort zu einem falschen Passwort, sodass die Authentifizierung fehlschlägt:

::

  command[check_http_auth]=/usr/lib/nagios/plugins/check_http --ssl -I localhost -a beam:bad_credentials
  # man beachte das fehlerhafte Passwort
  
Nach einem Neustart zeigt die Weboberfläche die Änderung korrekt an:

.. image:: images/Nagios/11-https-warning.png

Überwachung des LDAP-Servers
****************************
Analog zum vorherigen Abschnitt kann der LDAP-Server auf dem Remote Host überwacht werden.
Zunächst wird das Kommando **check_ldap** auf der NRPE-Seite in ``/etc/nagios/nrpe.cfg`` definiert:

::

  command[check_ldap]=/usr/lib/nagios/plugins/check_ldap -H localhost -b dc=betrayer,dc=com -3
  
Mit dem Argument ``-b [base-dn]`` wird der Basis-DN des DIT angegeben. In diesem Fall lautet dieser **dc=betrayer,dc=com**. Mit dem Argument ``-3`` wird angegeben, dass es sich um einen LDAP-Server nach der LDAP-Protokollversion **3** handelt. Dieses Argument wird zwingend vorausgesetzt.

Der NRPE-Server muss nun neu gestartet werden: ``service nagios-nrpe-server restart``

Anschließend wird auf der überwachenden Seite die Servicedefinition zum Aufrufen des Befehls in die ``/etc/nagios3/conf.d/sdi2b.cfg`` aufgenommen:

::

  define service{
    use                     generic-service
    host_name               sdi2b
    service_description     LDAP
    check_command           check_nrpe_1arg!check_ldap
  }
  
Nach einem Neustart des Nagios-Daemons (``service nagios3 restart``) erscheint der Service auf dem Webinterface:

.. image:: images/Nagios/12-ldap-ok.png

Einrichten von Serviceabhängigkeiten
************************************
Oftmals bestehen logische Abhängigkeiten zwischen den überwachten Services. Der gerade eingerichtete **HTTPS Auth**-Service ist beispielsweise vom **LDAP**-Service abhängig, da die HTTPS-Authentifizierung über LDAP realisiert ist. Fällt der LDAP-Server aus, funktioniert folglich die Authentifizierung auf dem Webserver nicht mehr. Für den Fall, dass der LDAP-Server ausfällt, sendet der Nagios-Daemon standardmäßig eine Benachrichtigungsmail für den Ausfall des LDAP-Servers sowie für jeden Service, der aufgrund der Nichterreichbarkeit von LDAP ausfällt. In einem realen Szenario wären noch viel mehr Services von LDAP abhängig, als nur der Webserver. Die Folge ist eine Kaskade an Benachrichtigungsmails, die niemandem etwas bringen, da klar ist, dass die abhängigen Services nicht funktionieren *können*.

Das Webinterface zeigt den Effekt, den das Ausschalten des LDAP-Servers hat:

.. image:: images/Nagios/12-ldap-down.png

Erwartungsgemäß kommen zwei E-Mails:

.. image:: images/Nagios/13-redundant-mails.png

Dies soll unterbunden werden. Hierfür wird eine Abhängigkeit in unserer Konfigurationsdatei ``/etc/nagios3/conf.d/sdi2b.cfg`` mit folgendem Eintrag definiert:

::

  define servicedependency{
    host_name                       sdi2b
    service_description             LDAP
    dependent_host_name             sdi2b
    dependent_service_description   HTTPS Auth
    notification_failure_criteria   o,w,u,c
  }

Diese Definition sagt aus, dass der Service mit dem Bezeichner **HTTPS Auth**, der auf dem Host **sdi2b** läuft, vom Service **LDAP**, der ebenfalls auf **sdi2b** läuft, abhängig ist. **notification_failure_criteria** bestimmt, in welchen Fällen *KEINE* Benachrichtigungen gesendet werden sollen. Die Werte **o,w,u,c** geben an, dass keine Benachrichtigungen gesendet werden sollen, wenn sich der **Masterservice** in einem der Zustände **OK** (o), **Warning** (w), **Unknown** (u) oder **Critical** (c) befindet.

Wird der LDAP-Server nun gestoppt, wird nur *eine* Mail für den *LDAP*-Service versendet, auch wenn der Zustand des *HTTP-Auth*-Services ebenfalls kritisch ist:

.. image:: images/Nagios/14-one-mail.png
