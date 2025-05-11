# Deba - HackMyVM (Hard)

![Deba.png](Deba.png)

## Übersicht

*   **VM:** Deba
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Deba)
*   **Schwierigkeit:** Hard
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 12. September 2022
*   **Original-Writeup:** https://alientec1908.github.io/Deba_HackMyVM_Hard/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser "Hard"-Challenge war es, Root-Zugriff auf der Maschine "Deba" zu erlangen. Der Angriff begann mit der Identifizierung einer Node.js/Express-Anwendung auf Port 3000. Eine Analyse der von der Anwendung gesetzten Cookies offenbarte eine unsichere Speicherung von Benutzerdaten (Base64-kodiertes JSON), was zu einer Prototype Pollution Schwachstelle führte. Diese wurde ausgenutzt, um Remote Code Execution (RCE) zu erlangen und initialen Zugriff als Benutzer `www-data` zu erhalten. Anschließend ermöglichte eine fehlerhaft konfigurierte `sudo`-Regel dem Benutzer `www-data`, ein Python-Skript (`/home/low/scripts/script.py`) als Benutzer `low` auszuführen. Da ein von diesem Skript importiertes Modul (`main.py`) für `www-data` schreibbar war, konnte der Code manipuliert und so eine Shell als `low` erlangt werden. Der finale Schritt der Privilegieneskalation von `low` zu `root` wurde im Writeup nicht detailliert dokumentiert, aber die Flags wurden schließlich erlangt.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `curl`
*   `base64`
*   Burp Suite / Browser (für Cookie-Manipulation)
*   `python2` (für `nodeshell.py`)
*   `nodeshell.py` (Payload-Generierung)
*   `nc` (netcat)
*   `python3`
*   `stty` (zur Shell-Stabilisierung)
*   `sudo`
*   Standard Linux-Befehle (`cat`, `ls`, `echo`, `cd`, `id`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Deba" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration:**
    *   IP-Findung mittels `arp-scan` (Ziel: `192.168.2.148`).
    *   `nmap`-Scan identifizierte SSH (22/tcp), Apache (80/tcp) und eine Node.js/Express-Anwendung (3000/tcp).
    *   `gobuster` auf Port 80 fand nur die Standard-Indexseite; Fokus verlagerte sich auf Port 3000.
    *   `curl -X OPTIONS` auf Port 3000 bestätigte das Express-Framework.

2.  **Cookie Analysis & Prototype Pollution (Initial Access als www-data):**
    *   Die Node.js-Anwendung setzte einen Base64-kodierten JSON-Cookie, der Benutzerdaten enthielt.
    *   Diese unsichere Handhabung ermöglichte die Manipulation des Cookies und eröffnete einen Vektor für Prototype Pollution.
    *   Ein RCE-Payload wurde mit `nodeshell.py` generiert und in einen JSON-String (`{"rce":"_$$ND_FUNC$$_[JS-Code]_"}`) eingebettet.
    *   Der JSON-Payload wurde Base64-kodiert und als Cookie-Wert an den Server gesendet.
    *   Dies löste die RCE aus und etablierte eine Reverse Shell als Benutzer `www-data`.

3.  **Privilege Escalation (von `www-data` zu `low`):**
    *   Als `www-data` wurde mit `sudo -l` eine Regel gefunden: `www-data` darf `/usr/bin/python3 /home/low/scripts/script.py` als Benutzer `low` ohne Passwort ausführen.
    *   `script.py` importierte das Skript `main.py` aus demselben Verzeichnis.
    *   `main.py` war für `www-data` schreibbar.
    *   `main.py` wurde mit einem Python-Payload (`import os;os.system("/bin/bash")`) überschrieben.
    *   Die Ausführung von `sudo -u low /usr/bin/python3 /home/low/scripts/script.py` startete dann eine Shell als Benutzer `low`.

4.  **Privilege Escalation (von `low` zu `root` - undokumentiert):**
    *   Der genaue Mechanismus für die Eskalation von `low` zu `root` wurde im analysierten Writeup nicht dokumentiert.
    *   Anschließend wurden aus einer Root-Shell die User- und Root-Flags gelesen.

## Wichtige Schwachstellen und Konzepte

*   **Node.js Prototype Pollution:** Eine unsichere Verarbeitung von clientseitigen JSON-Daten (in Cookies) führte zu einer Prototype Pollution Schwachstelle, die RCE ermöglichte.
*   **Unsichere Cookie-Handhabung:** Speicherung von Session-Daten als unverschlüsseltes, unsigniertes JSON im Cookie.
*   **Unsichere `sudo`-Regel / Python Script Hijacking:** Eine `sudo`-Regel erlaubte die Ausführung eines Python-Skripts, das ein vom aufrufenden Benutzer beschreibbares Modul importierte. Dies ermöglichte die Code-Injektion und Privilegieneskalation zum Zielbenutzer.
*   **Fehlende Dateiberechtigungen:** Das importierte Python-Modul (`main.py`) hatte falsche Besitz- und Schreibrechte.

## Flags

*   **User Flag (`/home/low/user.txt`):** `justdeserialize`
*   **Root Flag (`/root/root.txt`):** `BoFsavetheworld`

## Tags

`HackMyVM`, `Deba`, `Hard`, `Node.js`, `Express`, `Prototype Pollution`, `RCE`, `Cookie Manipulation`, `sudo`, `Python Script Hijacking`, `Privilege Escalation`, `Linux`
