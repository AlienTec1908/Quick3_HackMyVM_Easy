# Quick3 - HackMyVM (Easy)

![Quick3.png](Quick3.png)

## Übersicht

*   **VM:** Quick3
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=Quick3)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 2024-05-10
*   **Original-Writeup:** https://alientec1908.github.io/Quick3_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser Challenge war es, Root-Rechte auf der Maschine "Quick3" zu erlangen. Der Weg dorthin begann mit der Enumeration der Webseite, die eine Mitarbeiterliste enthielt. Die daraus abgeleiteten Benutzernamen wurden verwendet, um eine Insecure Direct Object Reference (IDOR)-Schwachstelle in `/customer/user.php` auszunutzen. Durch Iteration von User-IDs konnten Klartext-Passwörter aus dem HTML-Quellcode extrahiert werden. Mit den Credentials `mike:6G3UCx6aH6UYvJ6m` wurde SSH-Zugriff erlangt. Die finale Rechteausweitung zu Root gelang durch das Auslesen der MySQL-Root-Credentials (`root:fastandquicktobefaster`) aus `/var/www/html/customer/config.php`. Dieses Passwort war identisch mit dem System-Root-Passwort, was einen erfolgreichen `su root`-Befehl ermöglichte.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `vi`
*   `dirb`
*   `nikto`
*   `nmap`
*   `gobuster`
*   `curl`
*   `grep`
*   `awk`
*   `tr`
*   `hydra`
*   `ssh`
*   `find`
*   `sudo` (versucht)
*   `getcap`
*   Standard Linux-Befehle (`cat`, `su`, `ls`, `id`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Quick3" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration:**
    *   IP-Adresse des Ziels (192.168.2.124) mit `arp-scan` identifiziert. Hostname `que3.hmv` in `/etc/hosts` eingetragen.
    *   `nmap`-Scan offenbarte Port 22 (SSH, OpenSSH 8.9p1) und Port 80 (HTTP, Apache 2.4.52) mit dem Titel "Quick Automative - Home".
    *   `dirb` und `gobuster` fanden Standardverzeichnisse sowie `/customer/` und `/modules/`.
    *   Auf der Webseite wurden Mitarbeiterinformationen (Namen, E-Mails) gefunden, aus denen eine Benutzerliste (`coos`, `mike`, `juan`, `john`, `lara`, `nick`) abgeleitet wurde.
    *   Das Verzeichnis `/customer/` enthielt eine PHP-Anwendung (`index.php`, `login.php` (Status 500), `user.php` etc.).

2.  **Credential Gathering (Web Scraping & IDOR) & Initial Access (SSH als `mike`):**
    *   Eine Insecure Direct Object Reference (IDOR)-Schwachstelle wurde in `/customer/user.php` identifiziert. Durch Iteration des GET-Parameters `id` (z.B. `user.php?id=X`) konnten Benutzerprofile aufgerufen werden.
    *   Im HTML-Quellcode der jeweiligen Benutzerprofile waren die Passwörter im Klartext gespeichert (z.B. in einem versteckten Feld).
    *   Ein Skript (oder manuelle `curl`-Aufrufe mit `grep`/`awk`) wurde verwendet, um Benutzernamen und zugehörige Klartext-Passwörter für IDs 1-30 zu extrahieren.
    *   Mittels `hydra` wurde ein Brute-Force-Angriff auf SSH mit den extrahierten Benutzer/Passwort-Paaren durchgeführt. Die Credentials `mike`:`6G3UCx6aH6UYvJ6m` waren erfolgreich.
    *   SSH-Login als `mike`.
    *   Die User-Flag (`HMV{717f274ee66f8541a3031f175f615e72}`) wurde in `/home/mike/user.txt` gefunden.

3.  **Privilege Escalation (von `mike` zu `root` via MySQL Password Reuse):**
    *   Als `mike` (zuvor auch als `www-data` möglich) wurde die Datei `/var/www/html/customer/config.php` gelesen.
    *   Diese Datei enthielt die MySQL-Root-Zugangsdaten: Benutzer `root` (für MySQL), Passwort `fastandquicktobefaster`.
    *   Es wurde versucht, mit `su root` und dem Passwort `fastandquicktobefaster` zum System-Root-Benutzer zu wechseln.
    *   Der Wechsel war erfolgreich, da das MySQL-Root-Passwort identisch mit dem System-Root-Passwort war.
    *   Die Root-Flag (`HMV{f178761104e933f9341f13f64b38538a}`) wurde in `/root/root.txt` gefunden.

## Wichtige Schwachstellen und Konzepte

*   **Information Disclosure (Mitarbeiterliste):** Eine öffentlich zugängliche Webseite gab Namen und E-Mail-Adressen von Mitarbeitern preis, die als Benutzernamen verwendet werden konnten.
*   **Insecure Direct Object Reference (IDOR):** Die Datei `/customer/user.php` erlaubte den Zugriff auf Benutzerdaten nur anhand einer User-ID im GET-Parameter, ohne Autorisierungsprüfung.
*   **Klartextpasswörter im HTML/Datenbank:** Passwörter wurden unverschlüsselt im HTML-Quellcode der Benutzerprofile gespeichert und waren somit leicht auszulesen.
*   **Passwort-Wiederverwendung (MySQL Root = System Root):** Das Passwort für den MySQL-Root-Benutzer war identisch mit dem Passwort des System-Root-Benutzers.
*   **Klartext-Datenbank-Credentials in Konfigurationsdatei:** Das MySQL-Root-Passwort war im Klartext in `config.php` gespeichert und für den Benutzer `mike` (und zuvor `www-data`) lesbar.

## Flags

*   **User Flag (`/home/mike/user.txt`):** `HMV{717f274ee66f8541a3031f175f615e72}`
*   **Root Flag (`/root/root.txt`):** `HMV{f178761104e933f9341f13f64b38538a}`

## Tags

`HackMyVM`, `Quick3`, `Easy`, `IDOR`, `Information Disclosure`, `Cleartext Passwords`, `Password Reuse`, `MySQL`, `SSH`, `Linux`, `Web`, `Privilege Escalation`, `Apache`
