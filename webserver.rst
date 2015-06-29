
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


Apache Funktionsweise
#####################

Apache wird standardmaessig in das Verzeichnis ``/etc/apache2/`` installiert. Dieses Root-Verzeichnis enthaelt zu Beginn folgende Dateien und Unterordner:

::

    apache2.conf
    conf-available/
    conf-enabled/
    envvars
    magic
    mods-available/
    mods-enabled/
    ports.conf
    sites-available/
    sites-enabled/

Das Standardverzeichnis fuer Webinhalte ist unter ``/var/www/html/``.

Im Folgenden wird auf die vier Dateien ``apache2.conf``, ``envvars``, ``magic`` und ``ports.conf`` eingegangen, sowie auf die sechs Unterordner, die *verfuegbare* (``available``) und *aktivierte* (``enabled``) Konfigurationen, Mods und Seiten enthalten.

``apache2.conf``
****************
``apache2.conf`` ist eine der beiden ``.conf``-Dateien, mit denen sich der Apache-Webserver konfigurieren laesst (frueher: ``httpd.conf``). Als Hauptkonfigurationsdatei fuegt die Teile anderer Konfigurationsdateien (``mods``, ``confs``, ``sites`` und ``ports.conf``) zusammen, wenn der Server gestartet wird. Beispielhaft sind hier ein paar wichtige Einstellungen aufgefuehrt:

* ``ServerRoot``: Root-Verzeichnis, unter dem die Konfigurationen, Error- und Logdateien des Servers gehalten werden. Bsp.: ``/etc/apache2``.
* ``Timeout``: Anzahl Sekunden, die gewartet wird, bevor ein Timeout-Signal gesendet wird. Bsp.: ``300``.
* ``KeepAlive``: Ob permanente Verbindungen (mehrere Anfragen pro Verbindung) erlaubt sind oder nicht. Bsp.: ``On``.
* ``MaxKeepAliveRequests``: maximale Zahl an Anfragen, die pro persistenter Verbindung erlaubt sind. Bsp.: ``100``.
* ``HostnameLookups``: Ob nur die IP-Adresse oder auch der Hostname ueber einen versuchten DNS-Lookup in Logdateien gespeichert wird. Hat zur Folge, dass pro eingehender Verbindung mindestens 1 Lookup stattfindet. Bsp.: ``Off``.
* ``ErrorLog``: Pfad der Errorlog-Files. Dient als Fallback, wenn die virtuellen Hosts diesen Wert nicht setzen. Bsp.: ``${APACHE_LOG_DIR}/error.log``.
* ``Include`` bzw. ``IncludeOptional``: Andere Konfigurationsdateien werden eingebunden. Bsp.: ``ports.conf``.
* ``LogLevel``: Gibt die "Strenge" an, mit der Nachrichten gelogged werden sollen. Bsp.: ``warn``.
* globales, default Security-Model mittels ``Directory``-Direktiven:
  
  ::
  
      <Directory />
        Options FollowSymLinks
        AllowOverride None
        Require all denied
      </Directory>
      
      <Directory /usr/share>
        AllowOverride None
        Require all granted
      </Directory>
      
      <Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
      </Directory>

  Damit wird der Zugriff auf das Root-Filesystem explizit verboten (erste Direktive) und der Zugriff auf ``/usr/share/`` und ``/var/www/`` erlaubt (zweite und dritte Direktive). Host-spezifische (Directory-)Direktiven koennen in den entsprechenden VirtualHost-Direktiven in ``/etc/apache2/sites-available`` festgelegt werden.
* ``AccessFileName``: Der Name der Datei, die in jedem Ordner gesucht wird, um nach zusaetzlichen Konfigurations-Direktiven zu schauen.
* ``<FilesMatch "^\.ht">Require all denied</FilesMatch>``: Mit dieser Direktive koennen die Dateien ``.htaccess`` und ``.htpasswd`` nicht von Clients gelesen werden.
* ... und einige Umgebungsvariablen, z.B. ``${APACHE_PID_FILE}``, die aus der Datei ``envvars`` referenziert werden.


``ports.conf``
**************
``ports.conf`` wird immer von ``apache2.conf`` eingebunden. Es enthält Direktiven, die festlegen, auf welchen TCP-Ports Apache lauschen soll. Ueblicherweise sind das die Ports 80 fuer HTTP und Port 443 fuer HTTPS. Die Datei kann z.B. wie folgt aussehen:

::

    # If you just change the port or add more ports here, you will likely also
    # have to change the VirtualHost statement in
    # /etc/apache2/sites-enabled/000-default.conf
    
    Listen 80
    
    <IfModule ssl_module>
            Listen 443
    </IfModule>
    
    <IfModule mod_gnutls.c>
            Listen 443
    </IfModule>

Wenn der Webserver nur auf Anfragen vom localhost antworten soll, kann anstatt ``Listen 80`` folgende Ergaenzung gemacht werden:

::

    Listen localhost:80
    ...

``envvars`` und ``magic``
*************************
In ``envvars`` werden, wie der Name schon erahnen laesst, Apache-Umgebungsvariablen gesetzt. Z.B. die beiden folgenden fuer den User und die Gruppe ``www-data``:

::

    ...
    export APACHE_RUN_USER=www-data
    export APACHE_RUN_GROUP=www-data
    ...

``magic`` enthaelt Regeln, um anhand der führenden Bytes einer Datei einen MIME-Typ, also den Inhalt eines Dokuments, zu erkennen. Wenn man sich die Datei anschaut (s.u.), ist sie vierspalitig aufgebaut und enthaelt pro Zeile eine Matching-Regel:
* der Byteoffset, an dem das Pattern beginnt
* Typ der Daten, der gematched werden soll
* das Pattern selbst
* die Erkenntnis in Form eines MIME-Typs nach dem Schema ``<Hauptgruppe>/<Untergruppe>``.

Die Datei wird von dem Modul ``mime_magic`` verwendet.

::

    # xml based formats!
    
    # svg
    0       string          \<?xml
    #                       text/xml
    >38     string          \<\!DOCTYPE\040svg      image/svg+xml

    # xml
    0       string          \<?xml                  text/xml

``mods-available`` und ``mods-enabled``
***************************************
Der Apache ist modular aufgebaut. Das bedeutet, dass nur Basisfunktionen im Kern enthalten sind. Erweiterte Funkionen werden durch Module bereitgestellt, die in Apache geladen werden können. Standardmäßig werden einige Basismodule bei der Installation hinzugefügt, im Nachhinein lassen sich aber weitere Module jederzeit integrieren.

Im Ordner ``mods-available`` werden alle lokal vorhandenen Konfigurationsdateien fuer Module gelistet. In meiner Umgebung sind das bereits 130 Module, die ueberwiegend bei der Installation von Apache zur Verfuegung gestellt werden. Allerdings besitzt nicht jedes Modul eine eigene Konfigurationsdatei.

Im Ordner ``mods-enabled`` werden die aktuell verwendeten Module aufgelistet. Der Ordner enthält symbolische Verweise zu Dateien in ``/etc/apache2/mods-available``. Wenn eine Modul-Konfigurationsdatei einen symbolischen Verweis besitzt, wird sie beim nächsten Neustart von apache2 mitgeladen. Aktuell sind in meiner Umgebung 37 Stueck von den verfuegbaren 130 in Verwendung.

Mit folgendem Befehl kann z.B. das MySQL Authentication-Modul installiert werden:

::

    sudo apt-get install libapache2-mod-auth-mysql

Aktiviert werden kann das Modul mit folgendem Command. Anschliessend wird der Apache neu gestartet, damit er das Modul laden kann.

::

    sudo a2enmod auth_mysql
    sudo service apache2 restart

Mit ``a2dismod auth_mysql`` laesst sich das Modul wieder deaktivieren.

``sites-available`` und ``sites-enabled``
*****************************************
Der systematische Aufbau von ``sites`` ist der selbe wie bei ``mods`` - es gibt *verfuegbare* und *aktivierte* Seiten. Damit gleich nach der Apache-Installation eine Standardseite ueber den Browser erreichbar ist, existiert eine Standardseite, die in der Datei ``000-default.conf`` definiert ist. In dieser Art von Dateien muss mindestens eine ``VirtualHost``-Direktive stehen, mehrere sind aber auch moeglich. Wichtig ist, dass pro aktivierter ``site`` dann mehrere Hosts gestartet werden koennen (vgl. Kapitel "namebased und IP-based virtual hosting").

Eine einfache Seite koennte wie folgt in einer ``VirtualHost``-Direktive definiert sein:

::

    <VirtualHost *:80>
        ServerAdmin admin@example.com
        ServerName example.com
        ServerAlias www.example.com
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>

Erklaerung der verwendeten Attribute:
* ``ServerAdmin``: Diese Direktive legt fest, welche E-Mail-Adresse als Adresse des Server-Administrators angegeben wird. Der vorgegebene Wert ist ``webmaster@localhost``. Dieser Wert sollte in eine E-Mail-Adresse geändert werden, über die man den Webmaster erreichen kann. Falls auf der Website ein Problem auftritt, wird ein Fehlerhinweis mit dieser E-Mail-Adresse angezeigt, um das Problem zu melden. Um global fuer alle Hosts die gleiche E-Mail-Adresse festzulegen, kann die Direktive auch in das bereits erklaerten ``apache.conf`-File geschrieben werden.
* ``ServerName``: Diese Direktive ist optional und gibt den FQDN an, auf den der VirtualHost reagieren soll. Sobald mehr als ein VirtualHost angegeben ist, sind fuer die zusaetzlichen Eintraege jedoch eindeutige ``ServerName``s Voraussetzung.  Bsp.: ``Servername www.example.com``.
* ``ServerAlias``: Mit dem ``ServerAlias`` lassen sich alternative Nutzungs-URLs einrichten. Normalerweise ist es wuenschenswert wenn ein VirtualHost, der auf ``example.com`` reagiert, auch auf ``www.example.com`` antwortet. Oder man will alle Subdomains auf die Hauptdomain leiten. Mit einer Wildcard koennen VirtualHost so konfiguriert werden, dass sie auf jede Anfrage, die auf ``.sdi1b.mi.hdm-stuttgart.de`` endet, reagiert. Der Eintrag fuer Letzteres ist dann z.B. ``ServerAlias *.sdi1b.mi.hdm-stuttgart.de``.
* ``DocumentRoot``: Diese Direktive gibt an, wo sich die Webinhalte fuer einen VirtualHost befinden. Der Standardwert ist ``/var/www/html``. Wenn andere Pfade angegeben werden, muss sichergestellt werden, dass eine Zugriffsberechtigung mittel der ``Directory``-Direktive vorliegt.
* ``ErrorLog``: Legt den Namen und Ort des ErrorLog-Files fest.
* ``CustomLog``: Legt eine neue Logdatei an, die mit einem individuellen ``LogFormat`` kommt. Das ``LogFormat`` wird als letzter Parameter (standardmaessig ``combined``) angegeben.

``conf-available`` und ``conf-enabled``
***************************************
Enthaelt wie ``mods-...`` auch Konfigurationsdateien. Nur erweitern diese das Funktionsspektrum des Apaches nicht direkt (wie es bei den Modulen der Fall ist), sondern kommen von externen, auf Apache beruhenden Anwendungen. Z.B. existieren Konfigurationsdateien in diesem Ordner fuer den LDAP-Account-Manager, Nagios, phpMyAdmin und die Apache-Dokumentation. Alle Tools kommen mit einer Weboberflaeche, die vom Apache verwaltet wird und fuer die die Speicherorte z.B. ueber ``Directory``-Direktiven freigegeben werden muessen.

Wie eine solche Konfigurationsdatei aussieht, steht im Aufgabenkapitel.

Apache Befehle
##############
Der Apache-Webserver laesst sich mit einigen Commands steuern. Einige sind Skripte, deren Ausfuehrung dem User Zeit spart im Vergleich nur manuellen Variante (die ``a2enXXX`` bzw. ``a2disXXX``-Commands).
* ``a2ensite`` und ``a2dissite``: Aktiviert bzw. deaktiviert eine Seite aus dem ``/etc/apache2/sites-available``-Verzeichnis und erstellt bei Aktivierung einen symbolischen Link in ``sites-enabled``. Bei Deaktivierung wird der symbolische Link wieder geloescht. Die ausfuehrliche, manuelle Variante zur Aktivierung einer Seite, die das gleiche bewirkt aber mehr Tipparbeit ist, waere ``ln -s /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-enabled/000-default.conf``.
* ``a2enmod`` und ``a2dismod``: Das gleiche wie Obiges nur fuer Module und deren zwei spezifischen Ordnern.
* ``a2enconf`` und ``a2disconf``: Das gleiche wie Obiges nur fuer Konfigurationsdateien und deren zwei spezifschen Ordnern.
* ``apache2 -v``: Gibt die Versionsnummer und den Built-Timestamp aus.
* ``apache2 -t``: Checked Syntax von den Konfigurationsdateien und gibt etwaige Fehler in der Konsole aus.
* ``service apache [restart|reload|start|stop|force-reload]``: Kontrolliert den Status des Webservers. Er laesst sich mit diesen Parametern starten, stoppen, neu laden (Konfigurationsdateien werden neu geladen und aktuelle Verbindungen werden aufrecht erhalten), neu starten (bricht aktuelle Verbindungen ab) und "zwanghaft neu laden" (wie ``reload``, nur bricht aktuelle Verbindungen ab wenn das die Konfigurationsdateien erfordern).

*Quelle: https://wiki.ubuntuusers.de/apache#Apache-steuern*

Apache Prozesse
###############
Wie in folgendem Auszug aus der Konsole zu sehen ist, existieren mehrere zu Apache zugeordnete Prozesse gleichzeitig wenn der Webserver gestartet ist. Grund hierfuer ist, dass bei Serverstart ein ``apache2``-Prozess vom User ``root`` gestartet wird, der die TCP-Ports oeffnet und ein paar Kindprozesse (standardmaessig 5 an der Zahl) unter dem User ``www-data`` forked, die als *Worker* die Client-Anfragen beantworten. Diese Kindprozesse werden je nach Auslastung vom Mutterprozess gespawned oder gekilled. Parameter, wie die initiale Anzahl an gestarteten Kindprozessen bei Serverstart, koennen ueber Direktiven in der bekannten ``apache2.conf`` festgelegt werden.

.. image:: images/Apache/00_apacheProzessePsAux.png

**Bemerkung**: Der User ``www-data`` wird bei der Apache-Installation erstellt und ist ein Systemuser, sprich ohne Homeverzeichnis. Der Vorteil von einen neuen User ist, dass die Rechte individuell pro Service/Daemon anpassbar sind und kein Service ausserhalb seiner Berichtigungsgrenzen arbeitet.

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

Einrichtung des Apache Webservers und erste Schritte
****************************************************
Zunächst wird der Apache Webserver über die Paketverwaltung mit dem Befehl ``sudo apt-get install apache2`` installiert.

Eine eigene ``index.html`` mit folgendem Content wurde im Default-Verzeichnis ``/var/www/html`` angelegt:

::

    <html>
        <head>
            <title>testpage</title>
        </head>
        <body>
            testcontent
        </body>
    </html>

Wenn man ``sdi1b.mi.hdm-stuttgart.de`` im Browser aufruft, erscheint wie erwartet unsere Testseite.

.. image:: images/Apache/01_customIndexHTML.png

Benannt man die ``index.html`` in ``doc.html`` um, erscheint die IndexOf-Seite, da der Einstiegspunkt einer ``index.html``-Datei nicht mehr vorhanden ist.

.. image:: images/Apache/02_renamedToDocHTML.png

Installation von ``apache2-doc`` sowie Suche der URL
****************************************************
Installiert werden kann die Apache Doku mit dem Command ``sudo apt-get install apache2-doc``.

Verstaendnis 1:
Die URL des Repositories finden, von dem das Package ``apache2-doc`` heruntergeladen wird. Das geht nicht mit dem in der Aufgabe erwaehnten Tipp "dpkg...", sondern geht ueber den Command ``apt-cache policy apache2-doc``, welcher die URLs wie folgt ausgibt:

::

    apache2-doc:
      Installed: 2.4.7-1ubuntu4.4
      Candidate: 2.4.7-1ubuntu4.4
      Version table:
     *** 2.4.7-1ubuntu4.4 0
            500 http://archive.ubuntu.com/ubuntu/ trusty-updates/main amd64 Packages
            500 http://security.ubuntu.com/ubuntu/ trusty-security/main amd64 Packag  es
            100 /var/lib/dpkg/status
         2.4.7-1ubuntu4 0
            500 http://archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
 
Verstaendnis 2:
Den Pfad finden, ueber den der Apache Webserver die installierte Doku zur Verfuegung stellt. Laut Tipp ist ein Hinweis in einer Datei im ``apache2-doc``-Package zu finden. Mit dem Command ``dpkg -L apache2-doc`` lassen sich nun alle zum Packe zugehoerigen Dateien samt absolutem Pfad ausgeben. Die Ausgabe ist jedoch zu komplex und kann mit dem grep-Filter entsprechend reduziert werden. Eine uebersichtlichere Ausgabe laesst sich mit dem Befehl ``dpkg -L apache2-doc | grep -vE '(manual|examples)'`` erzeugen:

::

    /.
    /usr
    /usr/share
    /usr/share/doc
    /usr/share/doc/apache2-doc
    /usr/share/doc/apache2-doc/copyright
    /usr/share/doc/apache2-doc/changelog.Debian.gz
    /usr/share/doc-base
    /etc
    /etc/apache2
    /etc/apache2/conf-available
    /etc/apache2/conf-available/apache2-doc.conf

Wie zu sehen ist, wurden die in Frage kommenden Files erheblich reduziert. Die einzigste Datei, die Sinn macht, ist die ``/etc/apache2/conf-available/apache2-doc.conf``. Ein Apache-Kenner haette sofot in dieser Datei nachschauen koennen, da in diesem Verzeichnis alle Konfigurationsdateien von auf Apache beruhenden Packages, also z.B. der Apache-Doku, dem MySql-Frontend und dem Nagios-Frontend, gehalten werden.

Die gefundene Datei enthaelt:

::

    Alias /manual /usr/share/doc/apache2-doc/manual/
    
    <Directory "/usr/share/doc/apache2-doc/manual/">
        Options Indexes FollowSymlinks
        AllowOverride None
        Require all granted
        AddDefaultCharset off
    </Directory>

In dieser Datei sind 2 Pfade zu sehen:
* ``/usr/share/doc/apache2-doc-manual``: Der absolute Pfad, auf dem die Apache-Doku auf dem Server liegt.
* ``/manual``: Ein relativer Pfad als Alias, ueber den die Doku im Browser aufgerufen kann. In unserem Fall waere das also ``sdi1b.mi.hdm-stuttgart.de/manual``.

Ruft man die Seite ``sdi1b.mi.hdm-stuttgart.de/manual`` im Browser auf, erscheint erwartungsgemaess die Apache-Doku:

.. image:: images/Apache/03_apacheDocSlashManual.png

Auffaellig ist, dass beim Browsen dieser URL eine automatische Weiterleitung nach ``sdi1b.mi.hdm-stuttgart.de/manual/en/index.html`` erfolgt. Diese Weiterleitung wird von einer ``index.html`` im ``/manual``-Verzeichnis angestossen.

SDI-Doku hochladen und zugaenglich machen
*****************************************
Die SDI-Doku besteht aus mehreren Files, daher macht es Sinn die Doku vor dem Upload in eine Datei zu packen. Somit muss man nur eine Datei manuell hochladen. Gepackt wurde die Doku in einen Tarball mittels ``tar -cvzf sphinxdoku.tgz html`` (**ERKLAERUNG DER PARAMETER**)). Die Uebertragung von lokalem PC auf den Server ist mit dem Tool ``scp`` realisierbar, konkret dem Befehl ``scp sphinxdoku.tgz root@141.62.75.106:.`` (**ERKLAERUNG DER PARAMETER**). Durch die Angabe des Punkts hinten, landet die Datei dann serverseitig im Homeverzeichnis des Users root. Anschliessend muss die Datei wieder entpackt werden, z.B. mit dem Befehl ``tar -xvf sphinxdoku.tgz``. Unsere SDI-Doku liegt nun also auf dem Server in dem Verzeichnis ``/home/sdidoc/``.

Nun muss der Apache entsprechend konfiguriert werden, damit die Doku auch ueber einen Browser erreichbar ist:

::

    <Directory /home/sdidoc/>
           Options Indexes FollowSymLinks
           AllowOverride None
           Require all granted
    </Directory>
 
Es gibt 2 Moeglichkeiten:  Eine Redirect-Directive oder einen Alias. Vorraussetzung fuer beide Varianten ist, dass im SDI-Doku-Verzeichnis eine ``index.html`` als Einstiegspunkt existiert, was bei uns von unserem Doku-Tool Shinx bereits so erstellt wurde.

``Alias``-Direktive:

Alias wurden im Prinzip schon in der letzten Aufgabe rund um ``apache2-doc`` behandelt. Die Alias-Direktive nimmt einen relativen Pfad (relativ zum ServerName), also ``/mh203``, entgegen und mappt diesen auf einen anderen Pfad, in unserem Fall also ``/home/sdidoc``.
::

    <VirtualHost *:80>
            ServerName sdi1b.mi.hdm-stuttgart.de
            DocumentRoot /var/www/html
            Alias /mh203 /home/sdidoc
            <Directory /home/sdidoc>
                    Options Indexes FollowSymLinks
                    AllowOverride None
                    Require all granted
            </Directory>
    </VirtualHost>

Wie folgender Screenshot zeigt, funktioniert dieser Ansatz:

.. image:: images/Apache/04_sdiDocSlashMH203.png

``Redirect``-Direktive:

Hierbei wird die Anfrage nach ``sdi1b.mi.hdm-stuttgart.de/mh203`` auf einen anderen Host, also wie in diesem Beispiel auf ``sdidoc.mi.hdm-stuttgart.de``, weitergeleitet. Der Client muss dabei eine neue HTTP-Anfrage an die neue URL schicken. Demnach gibt es in der Apache-Konfigurationsdatei auch zwei ``VirtualHost``-Eintraege, einen fuer die Weiterleitung, den anderen fuer den eigentlichen Aufenthalt der SDI-Doku auf ``sdidoc.mi.hdm-stuttgart.de``.

**Bemerkung**: Der virtuelle Host ``sdidoc.mi.hdm-stuttgart.de`` muss vom DNS-Server korrekt aufgeloest werden. Auf meinem Server habe ich daher dieses Domainnamen in meine Zonefile des DNS-Servers mit aufgenommen, sodass dieser auf die IP 141.62.75.106 aufgeloest wird. Vergleiche auch mit naechster Aufgabe.

::

    <VirtualHost *:80>
            ServerName sdi1b.mi.hdm-stuttgart.de
            DocumentRoot /var/www/html
            Redirect /mh203 http://sdidoc.mi.hdm-stuttgart.de
    </VirtualHost>
    <VirtualHost *:80>
            ServerName sdidoc.mi.hdm-stuttgart.de
            DocumentRoot /home/sdidoc/
            <Directory /home/sdidoc>
                    Options Indexes FollowSymLinks
                    AllowOverride None
                    Require all granted
            </Directory>
    </VirtualHost>

Auch dieser Ansatz funktioniert, wenn der DNS-Eintrag fuer ``sdidoc.mi.hdm-stuttgart.de`` eingetragen ist:

.. image:: images/Apache/05_sdiDocSubdomain.png

Einrichtung von virtuellen Hosts
********************************
Die Konfigurationsdatei, mit der das Verhalten erzielt werden kann sieht folgendermassen aus:

::

    <VirtualHost *:80>
           ServerAdmin webmaster@localhost
           DocumentRoot /var/www/html
           ErrorLog ${APACHE_LOG_DIR}/error.log
           CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    <VirtualHost *:80>
            ServerName mh203.mi.hdm-stuttgart.de
            DocumentRoot /home/sdidoc
            <Directory /home/sdidoc>
                    Options Indexes FollowSymLinks
                    AllowOverride None
                    Require all granted
            </Directory>
    </VirtualHost>
    <VirtualHost *:80>
            ServerName manual.mi.hdm-stuttgart.de
            DocumentRoot /usr/share/doc/apache2-doc/manual/
    </VirtualHost>

Die eigene ``index.html`` mit dem Inhalt ``testcontent`` ist weiterhin ueber ``sdi1b.mi.hdm-stuttgart.de`` erreichbar (erster VirtualHost-Eintrag). Ein ServerName muss nicht zwangsweise mit angegeben werden, denn so wird dieser VirtualHost fuer alle Anfragen verwendet, die nicht einen anderen ServerName anfragen (s. folgende VirtualHosts), eine Art Fallback also. Der zweite VirtualHost-Eintrag ermoeglicht den Zugriff auf die SDI-Doku ueber ``mh203.mi.hdm-stuttgart.de``, der dritte Eintrag auf die Apache-Doku ueber ``manual.mi.hdm-stuttgart.de``. Ersteren muss man wieder ueber die ``Directory``-Direktive erweitern, sodass das Verzeichnis ``/home/sdidoc`` zugaenglich ist.

**Bemerkung**: Auch hier wieder: die beiden Subdomains muessen in die Zonesfile des DNS-Servers aufgenommen werden, damit diese Namen auf die IP des Servers (141.62.75.106) verweisen. DNS-Serverneustart mit ``service bind9 restart``. 

Damit auch der eigene DNS-Server zur Aufloesung verwendet wird, muss unter Ubuntu dieser manuell eingetragen werden. Das Ziel ist, dass in der Datei ``/etc/resolv.conf`` unser eigener DNS-Server an erster Stelle steht. Dazu kann der Eintrag in ``/etc/resolvconf/resolv.conf.d/head`` hinzugefuegt werden. Hintergrund ist, dass die ``/etc/resolv.conf`` aus den beiden ``head``- und ``base``-Dateien generiert wird. Der Inhalt von ``head`` wird bei der Generierung immer vor dem von ``base`` in das resultierende File eingefuegt.

*Quelle: http://askubuntu.com/questions/157154/how-do-i-include-lines-in-resolv-conf-that-wont-get-lost-on-reboot*

Wir fuegen also den Eintrag in die ``head``-Datei ein:

::

    # Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
    #     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
    nameserver 141.62.75.106

Die Warnung steht am Anfang dort, weil diese den User davon bewahren soll, die generierte ``resolv.conf`` zu aendern. In unserem Fall koennen wir die Warnung ignorieren. Mit dem Befehl ``sudo resolvconf -u`` kann ``resolv.conf`` neu generiert werden. Das Resultat in ``resolv.conf``:

::

    # Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
    #     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
    nameserver 141.62.75.106
    nameserver 127.0.1.1

Wie zu sehen ist, steht unser DNS-Server an erster Stelle, gefolgt von Nameserver des Host-OS (Ubuntu laeuft hier in einer VM als Guest-OS).


SSL-Einrichtung
***************
Damit SSL genutzt werden kann, muss das entsprechende Modul zuerst aktiviert und der Apache neu gestartet werden:

::

    sudo a2enmod ssl
    sudo service apache2 force-reload

Der folgende prinzipielle Ablauf ist: Wir erstellen uns eine eigene Root-CA, die wir in den Browser importieren. Anschliessend erstellen wir das Server-Zertifikat, das wir mit dem Key der Root-CA signieren und auf unseren Server ``sdi1b.mi.hdm-stuttgart.de`` laden. Dort erstellen wir einen passenden ``VirtualHost``, der SSL-faehig ist und starten den Webserver neu. Anschliessend kann mit dem Browser, der das Root-CA geladen hat, problemlos die HTTPS-Version der Seite angesurfed werden.

Die eigentliche Erstellung der Keys und Zertifikate sowie die Apache-Konfiguration erfordern mehrere Schritte, auf die im Folgenden der Reihe nach eingegangen wird (*Vorgehensweise ist auf http://datacenteroverlords.com/2012/03/01/creating-your-own-ssl-certificate-authority/*).

Erstellen des Root-Keys und des Root-Zertifikats
++++++++++++++++++++++++++++++++++++++++++++++++

Das Tool, das zur Generierung von den Keys/Zertifikaten verwendet wird ist ``OpenSSL``. Diese freie Software wird zunaechst verwendet, um eine eigene private CA zu erstellen.

Erstellen des Root-Keys (ohne Passwortschutz) mit:

::

    openssl genrsa -out rootCA.key 2048 [-des3]
    
Der erstellte private Key wird dadurch in dem File ``rootCA.key`` im aktuellen Verzeichnis gespeichert. Als Schluessellaenge sind die Werte 1024, 2048 und 4096 moeglich, wobei 2048 Bits state-of-the-art ist.

In der Praxis ist dieser Key strengstens geheim zu halten, da die gesamte CA-Sicherheit darauf beruht. In unserem Fall ist der Key neu generiert und wird nicht verwendet, daher ist es unbedenklich ihn hier exemplarisch auszugeben.

Der Inhalt der ``rootCA.key``:

::

    -----BEGIN RSA PRIVATE KEY-----
    MIIEpAIBAAKCAQEAswErEDIhF2XuSMX6t2BS/d7pN7RnoZjdmkREDqUbhv9qdbED
    6614h5NIoE9Q7C//08tR3FRe1qNdAJwbd5Q/M8pWOu9bLjPI9EO9MD2VnEfaw/3a
    HOzNT+aaaKuGulrC1ysvrUmddxigzS/qXhO/7GJ4nQKLuIiyvQM5hSMHoM6UbUy2
    oOlFp8SJRDqb5RpGph4BdOqGwlE7FW47B6/8Ewr031uKkHtPF/i0O24m29xTPUcd
    DpVqhrm46983drBl/08gPAfXRyFeWmGsaSNykwUZSVe35uRjRv7K3JemKg6Nj+RK
    3Gsn7Xmc40ABW2ONuiy/kZ0Qe3trryjvsPBM8wIDAQABAoIBADAA5TJ/rv4eWKtt
    ZFHe5AKuz64flBby7qKbIgEnZ91pC3yqTtnZ+FOrPLO48YOVT67VfItwmMpInv01
    3+vOdrCF12AQpkwamQ5gSBDywvdZEaKzpdVR77E4rznYndXF1zBqpWt8LV0t3pmH
    +vrlovdaZOv+Lcf76KrlfXxlq3sE2FT151R80t+UJLGTt/rPGBKmZRziwjheY73H
    dcz4QuSw/w9hX9aCSZdQ31vP7d06v+egVxRluCJDNovvX8kYqFabun4RJofNai1/
    Zl9YL9ShM+XsB8wqvZIW+sdxPBRUGADxy4j0+9r5CxmWB+p8ym33DUkOQVPhvBUx
    9LKPJTkCgYEA2j9FHPtsYLVsKdLJHETH/0d4NwTSedFn3i6lajGQHVrLGZfGnujs
    4NeTF50pcoLQnwDaSgji9EjvBTykPY+E7yvJvUQn4eIgrKh2p2uFGRYnr/zwwkPD
    zZr+S7l0dSoM2m30gMgqeKoGitIy1e9bM5++P7t4+j4idlZh6+NUIt0CgYEA0fga
    NiOvzDjYWNWM6yGcA6uFWvz6xWPqvr1mKlbQw2Tyf9ep6DuxNpIqHSPSOCPXWfMY
    diNE8iQqNwK9PstE56bYf+KR3FskaRz4hjI1cQ72fI2WM50AbSp4xW1lPcQuU1hU
    YE2qNFrfkp5EsD49rgl0fzkf5ps5szA5hVjSKg8CgYEAj3LutnH7dkVI5uSJE2+S
    FRSgy2j7/t3I65y1VOtm0iSPQi8keeaXa+HF6MuAJqgc/6XL1MWqhu45TLPjMCNZ
    OOsLGr/lhu7ekx3Xf4uIXEjXpTEX/lgUJtwtMRCgnzIYaKnE/7CRWeZHjo9CoqNE
    ytXAcpuClNiqRiXZDvbIonECgYAxV5WJTX+dhWpKDf0ssFxCfIc9J6AizOU4Z7ff
    kUFD+bAHHP4/pe9yPI0LBgT7zmhoKfmAoa8tHBSTkuA4JnqN2aac/vh6CS0YSoeY
    uanXZMXF5Vv7yxRzkvyIoLme90BPs704WGf8H7LUeL75j9bc1Dn9P8ZgYfAATP/n
    d0UEPwKBgQC2kmrYF5wnrN3aIj9f84XbVHSWfH3SgDrOqEVQn+/Q4Sr20shxjJ9y
    DpNmGADk4IwGnTY/4fwks4EDtu2bQ5hSH1AmNre+df2gTwuwRMu3WIfxeSL2eNiN
    czOMahif2jBhoGTDPcauwRyjqHoqeNNy71T1JDI3X2yBh2Squ/u1Aw==
    -----END RSA PRIVATE KEY-----

Im naechsten Schritt wird das selbst-signierte Zertifikat erstellt. Auch hier wieder mit dem Tool ``OpenSSL``:

::

    openssl req -x509 -new -nodes -key rootCA.key -days 1024 -out rootCA.pem

Dadurch wird ein Skript gestartet, das ein paar Nutzereingaben erwartet:

.. image:: images/Apache/06_rootCApemErstellen.png

Nach den Eingaben wird ein Zertifikat namens ``rootCA.pem`` erstellt (selbst-signiert), das 1024 Tage gueltig ist. Dieses dient nun als Root-Zertifikat der eigenen Root-CA. Auch offizielle Root-Zertifikate sind selbst-signiert.

Der Inhalt des Zertifikats ``rootCA.pem``:

::

    -----BEGIN CERTIFICATE-----
    MIIEQzCCAyugAwIBAgIJAKvcnpp5Bln4MA0GCSqGSIb3DQEBCwUAMIG3MQswCQYD
    VQQGEwJERTEbMBkGA1UECAwSQmFkZW4tV3VlcnR0ZW1iZXJnMRIwEAYDVQQHDAlT
    dHV0dGdhcnQxHjAcBgNVBAoMFUhvY2hzY2h1bGUgZGVyIE1lZGllbjEMMAoGA1UE
    CwwDTU1CMSIwIAYDVQQDDBlzZGkxYi5taS5oZG0tc3R1dHRnYXJ0LmRlMSUwIwYJ
    KoZIhvcNAQkBFhZtaDIwM0BoZG0tc3R1dHRnYXJ0LmRlMB4XDTE1MDYyOTEyMzYy
    MFoXDTE4MDQxODEyMzYyMFowgbcxCzAJBgNVBAYTAkRFMRswGQYDVQQIDBJCYWRl
    bi1XdWVydHRlbWJlcmcxEjAQBgNVBAcMCVN0dXR0Z2FydDEeMBwGA1UECgwVSG9j
    aHNjaHVsZSBkZXIgTWVkaWVuMQwwCgYDVQQLDANNTUIxIjAgBgNVBAMMGXNkaTFi
    Lm1pLmhkbS1zdHV0dGdhcnQuZGUxJTAjBgkqhkiG9w0BCQEWFm1oMjAzQGhkbS1z
    dHV0dGdhcnQuZGUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCzASsQ
    MiEXZe5Ixfq3YFL93uk3tGehmN2aREQOpRuG/2p1sQPrrXiHk0igT1DsL//Ty1Hc
    VF7Wo10AnBt3lD8zylY671suM8j0Q70wPZWcR9rD/doc7M1P5ppoq4a6WsLXKy+t
    SZ13GKDNL+peE7/sYnidAou4iLK9AzmFIwegzpRtTLag6UWnxIlEOpvlGkamHgF0
    6obCUTsVbjsHr/wTCvTfW4qQe08X+LQ7bibb3FM9Rx0OlWqGubjr3zd2sGX/TyA8
    B9dHIV5aYaxpI3KTBRlJV7fm5GNG/srcl6YqDo2P5ErcayfteZzjQAFbY426LL+R
    nRB7e2uvKO+w8EzzAgMBAAGjUDBOMB0GA1UdDgQWBBRtaQgcy2MVQ9RBFe6kaWzc
    lEQGejAfBgNVHSMEGDAWgBRtaQgcy2MVQ9RBFe6kaWzclEQGejAMBgNVHRMEBTAD
    AQH/MA0GCSqGSIb3DQEBCwUAA4IBAQAP92HfSg8uhzF1XMDDoLzCxtbR2j3e4Zx7
    vXOO8Ocr0pRPw/xf9PEIeA5HGGkI3AlouyJR+4nTIaTeSrKTdhN75KxC5kpHXfq3
    AetQJXjrmf8WDYIdgrnhI7LsfXrL7lqMvoTu/l8JxaFZS1Pel43Rlq+YccqIhk/i
    eqD27WTRx8rSdk+wF2szVVSN9/A46Hu1AyAIV4VILKp+jptKyM+9SqsVVpxHrDw0
    u/MIgqXhYda/I6WJ1y4uSSqpDafqOQe9yWaegJUjug00r32o62d8EYJBhFbRrMFH
    4oM5/b2j0o/6npgN8BvcQZkRlJAcr7HRMkjLmA4eFL89d/ioFQxq
    -----END CERTIFICATE-----

Import des Root-Zertifikats in den Browser
++++++++++++++++++++++++++++++++++++++++++

Dieses Zertifikat muss nun in den Browser des Clients, der die HTTPS-Verbindung speater aufbauen soll, importiert werden. Dazu wurde ``rootCA.pem`` unter Windows ueber das GUI-Took ``WinSCP`` auf den Client geladen und unter Linux folgender ``scp``-Command ausgefuert: ``scp root@141.62.75.106:rootCA.key``. Voraussetzung fuer den Linux-Command ist, dass das Zertifikat im Home-Verzeichnis des Users ``root`` liegt.

.. image:: images/Apache/07_zertifikatManager.png
.. image:: images/Apache/08_zertifikatVertrauenPopup.png
.. image:: images/Apache/09_zertifikatDetails.png
.. image:: images/Apache/10_zertifikatListe.png

Erstellen des Server-Keys und des Server-Zertifikats
++++++++++++++++++++++++++++++++++++++++++++++++++++

Nun brauchen wir noch ein Zertifikat, mit dem sich unser Server beim Client identifizieren kann. Dieses neue Zertifikat wird mit dem zuvor erstellten Root-Key signiert, sodass der Client beim Aufruf der HTTPS-Seite den Server als vertrauenswuerdig einstuft, da sein Zertifikat von einem im Browser eingetragenen und damit glaubwuerdigem Root-CA signiert wurde.

Jeder neue Server (Device) braucht ein eigenes Zertifikat.

Um dieses zu erstellen wird zunaechst wieder ein privater Schluessel erstellt:

::

    openssl genrsa -out device.key 2048

Aus diesem Key wird ein "Certificate Signing Request" (CSR) erstellt:

::

    opensll req -new -key device.key -out device.csr

Wieder werden einige User-Eingaben verlangt. Wichtig ist hierbei nur, dass unter der Eingabe "common name" der Hostname oder die IP-Adresse des Servers eingetragen werden soll, der das Zertifikat speater verwenden soll.

.. image:: images/Apache/11_CSRRequest.png

Der erstellte CSR hat folgenden Inhalt:

::

    -----BEGIN CERTIFICATE REQUEST-----
    MIIDBDCCAewCAQAwgaUxCzAJBgNVBAYTAkRFMRswGQYDVQQIDBJCYWRlbi1XdWVy
    dHRlbWJlcmcxEjAQBgNVBAcMCVN0dXR0Z2FydDEMMAoGA1UECgwDSGRNMQwwCgYD
    VQQLDANNTUIxIjAgBgNVBAMMGXNkaTFiLm1pLmhkbS1zdHV0dGdhcnQuZGUxJTAj
    BgkqhkiG9w0BCQEWFm1oMjAzQGhkbS1zdHV0dGdhcnQuZGUwggEiMA0GCSqGSIb3
    DQEBAQUAA4IBDwAwggEKAoIBAQDkPj4mC7tyGfSIQZ7n4dI4LfQP/xwS4n73jUne
    TVYVeejFMSz5AIJyoAsyfyw98st5ceHGtQkyc4PhFxCWJvMwAsH1zmYJzu0jkPTb
    AWRVANVyHwGNVudqUToCXujTPZqmJHtWp2kLF5vO63ic7sra5xRGtLyUDZD1i4Gd
    LMWxe6wOE3TjNRAPW2xjzUisXsvs0ls3H4n4JZysZM6+JuD+JUaT5ZlPWP3l4u8q
    LicSXH0EeykhTVyJKdvD60hr9nsX/ULkXUltrz0oB+kRw4xtPlzmpzxofKNfKd/h
    yjNOVLI7kdf/HQHBOE1fYxOeVDJJB9GfjAdhqRZWCfKNkT3BAgMBAAGgGTAXBgkq
    hkiG9w0BCQcxCgwIcGFzc3dvcmQwDQYJKoZIhvcNAQELBQADggEBACD3IAC675wA
    PIL68HbsY4OPFnrPrlb0P1rYQOYR4fEHCQY+P3EHj+1XwtM/TvtMUtUn227UF5hs
    2zO/xVbeDfxDITrMBP+rHy997EEofxB8f7y2zYDFtrgw5a0j0PgpK2cx6Hffa29p
    JqGHlUTZ+Xx1m6rfQupm4ooz3uffCciJOCftJ0G181H6i3+62MYBrPmYRfimLaWd
    Im4kOJtHjXtF6n18wCM83/5DcScxx722pFHK8hAJcbyEuc2caH1ii2hD7zwNMPZK
    T4b2AMnWdAYtyqwSsTsqdAcIeaL1Zjs0LSySUPTKvI1s8+HuLJVyxQMA/lW8DfBP
    ncLhvKcJ+0E=
    -----END CERTIFICATE REQUEST-----

Nun wir der CSR mit dem ganz zu Beginn erstelltem privaten Schluessel der Root-CA signiert:

::

    openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 500

Mit dem Input des Root-Zertifikats und des Root-Keys, erstellt dieser Command ein Zertifikat (``device.crt``), dass nun unser Server verwenden kann. Es ist 500 Tage gueltig.

``device.crt`` sieht nun folgendermassen aus:

::

    -----BEGIN CERTIFICATE-----
    MIID2jCCAsICCQC0dnZBTZ061TANBgkqhkiG9w0BAQsFADCBtzELMAkGA1UEBhMC
    REUxGzAZBgNVBAgMEkJhZGVuLVd1ZXJ0dGVtYmVyZzESMBAGA1UEBwwJU3R1dHRn
    YXJ0MR4wHAYDVQQKDBVIb2Noc2NodWxlIGRlciBNZWRpZW4xDDAKBgNVBAsMA01N
    QjEiMCAGA1UEAwwZc2RpMWIubWkuaGRtLXN0dXR0Z2FydC5kZTElMCMGCSqGSIb3
    DQEJARYWbWgyMDNAaGRtLXN0dXR0Z2FydC5kZTAeFw0xNTA2MjkxMzQ3MjZaFw0x
    NjExMTAxMzQ3MjZaMIGlMQswCQYDVQQGEwJERTEbMBkGA1UECAwSQmFkZW4tV3Vl
    cnR0ZW1iZXJnMRIwEAYDVQQHDAlTdHV0dGdhcnQxDDAKBgNVBAoMA0hkTTEMMAoG
    A1UECwwDTU1CMSIwIAYDVQQDDBlzZGkxYi5taS5oZG0tc3R1dHRnYXJ0LmRlMSUw
    IwYJKoZIhvcNAQkBFhZtaDIwM0BoZG0tc3R1dHRnYXJ0LmRlMIIBIjANBgkqhkiG
    9w0BAQEFAAOCAQ8AMIIBCgKCAQEA5D4+Jgu7chn0iEGe5+HSOC30D/8cEuJ+941J
    3k1WFXnoxTEs+QCCcqALMn8sPfLLeXHhxrUJMnOD4RcQlibzMALB9c5mCc7tI5D0
    2wFkVQDVch8BjVbnalE6Al7o0z2apiR7VqdpCxebzut4nO7K2ucURrS8lA2Q9YuB
    nSzFsXusDhN04zUQD1tsY81IrF7L7NJbNx+J+CWcrGTOvibg/iVGk+WZT1j95eLv
    Ki4nElx9BHspIU1ciSnbw+tIa/Z7F/1C5F1Jba89KAfpEcOMbT5c5qc8aHyjXynf
    4cozTlSyO5HX/x0BwThNX2MTnlQySQfRn4wHYakWVgnyjZE9wQIDAQABMA0GCSqG
    SIb3DQEBCwUAA4IBAQBzvOBWGJII6p3SNGdlYBXXP04OG/J/p24PQtnNL7ZhTF6R
    WXVv1MPJArCqc3L1LS+45tvW38rGaDyZWeyRmt2reh/fMffLSHtmuH20mPi/JD7g
    25engjxcW0EYJe5lmWLW3bxgbcZ0iDGGoMZgnOqi8tPlKiLNestWPrIvX/Mj2By7
    MnhJYmjKZuCg5O1DVvJkH+wzSKt7H0wUvnJyaMZ7FN4MWH7zc2cyMgED436QDgfv
    x3LhYXKRVSGquhYUiVhy+S/gi+d0xOKA6W3+7HH76gfPAxL9CjyBoTQQSUOZMxaJ
    CTK3zfXlxAM11PYQhGJw35ACS3n0rohvHNN/kx/D
    -----END CERTIFICATE-----

Bei der Erstellung wird gleichzeitig eine ``rootCA.srl``-Datei erstellt. Diese Datei wird durch den Parameter ``-CAcreateserial`` erstellt und enthaelt initial eine zufaellige gerade Zahl in Hexadezimaldarstellung. Diese Seriennummer fliesst in das erstellte Zertifikat mit ein und wir bei jeder neuen Erstellung eines Zertifikats inkrementiert und wieder in das ``.srl``-File geschrieben.

Exemplarisch der Inhalt der aktuellen ``rootCA.srl``:

::

    B47676414D9D3AD5

*Bemerkung*: Das Root-Zertifikat ist 1024 Tage gueltig, es macht also keinen Sinn das Device-Zertifikat ueber einen laengeren Zeitraum auszustellen. Nach Ablauf des Root-Zertifikats wird auch dieses ungueltig werden.

zugehoeriger ``VirtualHost`` unter Apache
+++++++++++++++++++++++++++++++++++++++++

Der private Device-Key und das Device-Zertifikat muessen nun auf dem Server ``sdi1b.mi.hdm-stuttgart.de`` in das richtige Verzeichnis kopiert werden. Common sense ist, dass man die beiden Files unter ``/etc/ssl/certs/`` zu den anderen Zertifikaten packt.

Folgende Commands kopieren die beiden Files in das gewuenschte Verzeichnis.

::

    cp device.crt /etc/ssl/certs/device.crt
    cp device.key /etc/ssl/certs/device.key


Seitens Key und Zertifikat ist das Setup beendet. Nun muss der Apache im letzten Schritt noch angewiesen werden, diese beiden Dateien in seiner SSL-Konfiguration auch zu verwenden.

Ein passender ``VirtualHost`` sieht z.B. folgendermassen aussehen:

::

    <VirtualHost *:443>
            ServerAdmin mh203@hdm-stuttgart.de
            ServerName sdi1b.mi.hdm-stuttgart.de
            DocumentRoot /var/www/html
    
            SSLEngine on
            SSLCertificateFile /etc/ssl/certs/device.crt
            SSLCertificateKeyFile /etc/ssl/certs/device.key
    </VirtualHost>


Neu sind die 3 Zeilen in der Mitte: sie sagen aus, dass die ``SSLEngine`` fuer diesen Host aktiv sein soll und gibt die Pfade zum ``SSLCertificateFile`` und zum ``SSLCertificateKeyFile`` an, die im letzten Schritt jeweils in das Verzeichnis ``/etc/ssl/certs`` kopiert wurden.

Praxistest
++++++++++

Der Aufruf von ``https://sdi1b.mi.hdm-stuttgart.de`` funktioniert nun. Der Firefox gibt auch die Zusatzinfo aus, dass dieser Seite vertraut wird.

.. image:: images/Apache/12_pageTrusted.png

Mit einem Click auf "Weitere Informationen..." kann das Server-Zertifikat (Device-Zertifikat) begutachtet werden.

Wenn das Root-Zertifikat nicht im Browser integriert ist, kommt folgender bekannter Warnhinweis:

.. image:: images/Apache/13_pageNotTrusted.png


LDAP Authentifizierung
**********************

