

****
LDAP
****

The LDAP concept
################

SOMETHING AMNOG THOSE LINES:

- schemas (also built-ins like cosine, nis, inetorgperson)
- tree structure

-  A LDAP directory is a tree of data entries that is hierarchical in nature and is called the Directory Information Tree (DIT).
- An entry consists of a set of attributes.
- An attribute has a type (a name/description) and one or more values.
- Every attribute must be defined in at least one objectClass.
- Attributes and objectclasses are defined in schemas (an objectclass is actually considered as a special kind of attribute).
- Each entry has a unique identifier: its Distinguished Name (DN or dn). This, in turn, consists of a Relative Distinguished Name (RDN) followed by the parent entry's DN.
- The entry's DN is not an attribute. It is not considered part of the entry itself.
- The terms object, container, and node have certain connotations but they all essentially mean the same thing as entry, the technically correct term.
- For example, below we have a single entry consisting of 11 attributes where the following is true:
- DN is "cn=John Doe,dc=example,dc=com"
- RDN is "cn=John Doe"
- parent DN is "dc=example,dc=com"

_ Abkuerzungen:

* dc
* dn (distinguished name)
* rdn (relative distinguished name)
* cn
* ou

Apache Directory Studio
#######################

Ein auch als Eclipse-Plugin erhaeltliches Tool, mit dem CRUD-Operations auf LDAP-Datenbaenken
ausgefuehrt werden koennen.

Dient zur Unterstuetzung des Instalationsprozesses sowie der Entwicklung und dem Debugging unter LDAP.
Fuer die alltaeglichen Administrierungsaufgaben sind jedoch webbasierte Tools geeigneter.



Setting up a LDAP-Server
########################

Man unterscheidet zwischen dem OpenLDAP server daemon im Package ``slapd`` und LDAP
management utilities im Package ``ldap-utils``.

::

  sudo apt-get install slapd ldap-utils

Waehrend der Installation muss man Admin credentials festlegen, die fuer den
``rootDN`` der LDAP-Datenbank gesetzt werden, by default ist das

::

  cn=admin,dc=mi,dc=hdm-stuttgart,dc=de

Neben der eigentlichen LDAP-Datenbank, in der spaeter Daten gespeichert werden, wird
eine config-database erstellt, auf die man sich auf eine andere Art und Weise (TODO)
authentifizieren muss.

Die Installation erstellt eine lauffaehige Konfiguration, darunter eine Datenbank
in der die LDAP-Daten gespeichert werden koennen.

Der "base DN" (DN = Distinguished Name) dieser Instanz wird vom Domainnamen des
localhosts abgeleitet. Alternativ kann man die Datei ``/etc/hosts`` editieren, um manuell
Domainnamen fuer localhost vergeben, sodass ein erwuenschter base DN erstellt
werden kann. Die default Konfiguration in unseren VM ist daher

::

  dc=mi,dc=hdm-stuttgart,dc=de

Die ``config``-Datenbank
************************

Der Inhalt der config-Datenbank sieht aus wie folgt:

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

Direkte Aenderungen in der config-Datenbank sind nicht empfohlen, man soll besser
ueber das LDAP Protocol (Tool aus dem Package ``ldap-utils``) Aenderungen vornehmen.

The LDAP-Protocol
#################
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
##########

Mit LDIF Files lassen sich LDAP-spezifische Daten speichern, z.B. als Export-Funktion.
Ueber ``slapadd`` im Terminal (LDAP-Server zur Sicherheit dafuer stoppen) oder die
Import-Funktion des Apache Directory Studios lassen sich LDIF Files importieren.

Ein LDIF-File kann z.B. folgendermassen aussehen:

.. code-block:: html
  :linenos:

  dn:dc=betrayer,dc=com
  changetype: add
  objectclass: dcObject
  objectclass: organizationalUnit
  dc: betrayer
  ou: config
  ou: betrayer Dot com

  dn: ou=departments,dc=betrayer;dc=com
  changetype: add
  objectClass: top
  objectClass: organizationalUnit
  ou: departments

  dn: ou=software,ou=departments,dc=betrayer;dc=com
  changetype: add
  objectClass: top
  objectClass: organizationalUnit
  ou: software

  dn: ou=devel,ou=software,ou=departments,dc=betrayer;dc=com
  changetype: add
  objectClass: top
  objectClass: organizationalUnit
  ou: devel

  dn: uid=beam,ou=devel,ou=software,ou=departments,dc=betrayer;dc=com
  changetype: add
  objectClass: inetOrgPerson
  uid: beam
  cn: Jim Beam
  givenName: Jim
  sn: Beam
  mail: beam@betrayer.com

**ERKLAERUNG DAZU**

LDAP with mail client Thunderbird
#################################
**Address Book .... was muss man da eingeben... und was kann man dann damit machen**

LDAP Filter Search
##################

Filter kann man ueber das CLI oder ueber das Apache Directory Studio festlegen.

Die ``ldapsearch``-Syntax ist oben aufgefuehrt.

Im Apache Directory Studio stellt man Fliter ein, indem man auf den zu filternden
Knoten rechtsklickt und "Filter Children" auswaehlt. ImPopup-Fenster laests sich
dann ein Suchstring eingeben. Um die Syntax naeher zu beleuchten, hier ein paar
Beispiele:

.. topic:: Beispiele zu LDAP Search Filtern

  .. glossary::
    ``(objectClass=*)``
      default Search Filter. Laesst alle objectClasses zu.

    ``(uid=*b*)``
      Jeder uid-Eintrag, der ein "b" enthaelt.

    ``(cn=b*)``
      Jeder uid-Eintrag, der mit einem "b" beginnt.

    ``(&(objectClass=user)(email=abc*))``
      Jeder Eintrag mito ``objectClass=user`` UND einer E-Mail-Adresse, die
      mit "abc" beginnt.

    ``(weitere bsp)``
      Erklaerungen

Aus Goik's Aufgaben:

All users with a uid attribute value starting with the letter “b”.

All entries either with either a defined uid attribute or a ou attribute starting with letter “d”.

Allgemein: die Search-Syntax uenterstuetzt Operatoren (!, &, |, =, ~=, <=, >=) und
Wildcards (*). Gruppierungen erfolgt durch Einklammern. Falls nach reservierten
Sonderzeichen gesucht werden muss (Klammern, !, ^, ...) lassen sich diese im
Suchstring escapen.

Extending an existing Entry
###########################
Aufgabe:

The entry uid=beam,ou=devel,ou=software,ou=departments,dc=betrayer;dc=com may be
extended by the objectclass posixAccount. Construct a LDIF file to add the
attributes uidNumber, gidNumber and homeDirectory by a modify/add operation.

LDAP Account Manager (LAM)
##########################
Installation unter Ubuntu mit
::

    [sudo] apt-get install ldap-account-manager

Der LAM laeuft auf Apache und ist nach der Installation sofort unter
``http://sdi1b.mi.hdm-stuttgart.de/lam`` erreichbar. Auf dieser Webseite
lassen sich gleich die LAM-Einstellungen vornehmen. Das default-Master-Passwort
ist ``lam``.

Die "General Settings" umfassen Einstellungen zur Sicherheit, Passwoertern und
deren Policies, und Logging.

Die "Server Profiles" bestehen aus den "General Settings", "Account Types", "Modules"
und "Module Settings". Unter "General Settings" sollte man den Tree-Suffix setzen,
in unserem Fall ist das ``dc=mi,dc=hdm-stuttgart,dc=de``. Weiter unten kann man die
User angeben, die sich ueber das Web-UI anmelden koennen. Wir haben
::

  cn=admin,dc=mi,dc=hdm-stuttgart,dc=de

und
::

  uid=boss,ou=software,ou=departments,dc=betrayer,dc=mi,dc=hdm-stuttgart,dc=de

zusaetzlich eingetragen.

Auch unter "Account Types" muessen fuer User, Hosts und Groups die entsprechenden
LDAP-Suffixes angegeben werden.

Unter "Modules" koennen die "objectClass"es der LDAP-Entitaetstypen verwaltet
werden.

Unter "Module Settings" lassen sich u.a. Einstellungen zu den UIDs fuer Users, Groups 
und Hosts vornehmen. Also z.B. die Art des UID-Generators, sowie die Range, in der sich
generierte UIDs befinden duerfen.


LDAP Replication (basic theory)
###############################
LDAP Replication serves failure safety, so the LDAP services are still available when
some nodes crash in the LDAP-infrastructure.


The HdM environment contains a LDAP-master and serveral LDAP-slaves like ``ldap1.mi``.
Depending on the configuration, updates can either be propagated from the master to all slaves
(single source) or bidirectional.


In our environment, user rights get included via a LDIF-file for each LDAP instance.
