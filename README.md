# Nightscout-on-raspberry-pi
Installation einer Nightscout Instanz auf einem lokalen Raspberry PI3 / Building the Nightscout Website on own server with raspberry pi3

Basierend zu großen Teilen auf den Anleitungen von Kitekater und Johnmales und einigen anderen. 
Ich habe dies hier vor allem als Gedankenstütze für mich zusammengefasst, damit ich mir nicht wieder alles zusammensuchen muss, 
falls ich den Server neu aufsetzen muss/will.
 
Debian Buster aarch64 Betriebssystem wird vorausgesetzt.

Zugriff auf den Pi mit ssh, den Pi mit einer festen IP vom Router versehen. Optional: Den Pi über einen DynDNS Anbieter über das Internet erreichbar machen. Portfreigabe für tcp/1337 im Internetrouter einrichten, um Daten zu empfangen und die Webseite aufrufen zu können.
Falls der Server über das Internet erreichbar sein soll, muss auf jeden Fall noch dafür gesorgt werden, dass eine Firewall (z.B. iptables) eingerichtet wird. Darauf gehe ich hier aber nicht weiter ein. 

Schritt 1 - Debian aktualisieren

	sudo apt-get update
	sudo apt-get upgrade


Schritt 2 - Debian Stretch-Sourcen einstellen für MongoDB installation (geht bestimmt auch eleganter)

    sudo nano /etc/apt/sources.list
    
    

    #deb http://deb.debian.org/debian buster main contrib non-free
    deb http://deb.debian.org/debian stretch main contrib non-free
    #deb-src http://deb.debian.org/debian buster main contrib non-free
    deb-src http://deb.debian.org/debian stretch main contrib non-free

    #deb http://deb.debian.org/debian buster-updates main contrib non-free
    deb http://deb.debian.org/debian stretch-updates main contrib non-free
    #deb-src http://deb.debian.org/debian buster-updates main contrib non-free
    deb-src http://deb.debian.org/debian stretch-updates main contrib non-free

    #deb http://security.debian.org/ buster/updates main contrib non-free
    deb http://security.debian.org/ stretch/updates main contrib non-free
    #deb-src http://security.debian.org/ buster/updates main contrib non-free
    deb-src http://security.debian.org/ stretch/updates main contrib non-free


Schritt 3 - MongoDB (3.2) installieren

	sudo apt update 
    sudo apt install mongodb

Danach die sourcen wieder auf Buster zurücksetzen!!!


Schritt 4 - MongoDB einrichten

Daten in <> durch eigene Daten ersetzen!!!	

User anlegen über die Mongo Shell:
	
erst den Admin:

	sudo mongo -shell
	
		use admin
		db.createUser( { user: <"USER">, pwd: <"PASSWORD">, roles: [ "userAdminAnyDatabase" ] } )

dann die eigentliche Nightscout Datenbank und Nightscout User anlegen: 

		use mongodb   
		db.createUser( { user: <"USER_NS">, pwd: <"PASSWORD_NS">, roles: [ "readWrite", "dbAdmin" ] } )


Schritt 5 - NodeJS 

    sudo apt update
    sudo apt-get install npm

Versionen testen:

    npm -v	(sollte 6.14.1 sein)
    nodejs -v	(sollte 10.23.1 sein)
    node -v	(sollte 10.23.1 sein)
    
( eventuell hilft auch diese Anleitung: https://github.com/audstanley/NodeJs-Raspberry-Pi/
	sudo wget -O - https://raw.githubusercontent.com/audstanley/NodeJs-Raspberry-Pi/master/Install-Node.sh | sudo bash node -v )


Schritt 6 - Nightscout Version herunterladen (13.0.1. läuft auf pi3, bei 14 hatte ich Probleme)

Kopiere die Sourcen z.B. ins Home Verzechnis des Benutzers:

Falls noch nicht vorhanden, wget und unzip installieren:
    
    sudo apt-get wget unzip

    wget https://github.com/nightscout/cgm-remote-monitor/archive/13.0.1.zip

Entpacke die Sourcen:

	sudo unzip /home/user/13.0.1.zip
	
Damit später einfacher ein Update installiert, oder eine andere Version zum testen gesetzt werden kann, wird ein symbolischer Link auf das aktuelle Sourcen Verzeichnis gesetzt. Außerdem schreibt sich "nightscout" leichter als "cgm-remote-monitor-13.0.1". ;-)

	sudo ln -s cgm-remote-monitor-13.0.1 nightscout
	sudo chmod 777 cgm-remote-monitor-13.0.1


Schritt 7 - Swap aktivieren

Zumindest für die Installation über npm ist die Swapdatei nötig, sonst kommt es zum Abbruch. Der Speicher des pi3 reicht sonst nicht aus.
Ob die Seite danach auch ohne Swap läuft, könnt ihr ja mal testen.

     sudo dd if=/dev/zero of=/swapfile bs=1M count=4048
     sudo chmod 0600 /swapfile
     sudo mkswap /swapfile 
     sudo swapon /swapfile 
     
Damit die Auslagerungsdatei auch nach einem Neustart noch benutzt wird, die Datei /etc/fstab in einem Editor mit Rootrechten öffnen und dort diese Zeile anhängen:

     sudo nano /etc/fstab
     /swapfile    none    swap    sw      0 0


Schritt 8 - Nightscout installieren

Dabei auf eventuelle Fehlermeldungen achten. Bei mir musste npm noch einmal aktualisiert werden, damit es fehlerfrei durchlief 
(npm install -g npm@latest).

    cd ~/nightscout
    npm update
    sudo npm i npm@latest -g    (muss als root ausgeführt werden)
    npm install

Der Installationsvorgang sollte nun ohne Fehler durchlaufen. Kann bei einem pi3 eine Weile dauern.
Schauen ob evtl. Fehlermeldungen auftauchen und evtl. Abhängigkeiten nachinstalliert werden müssen.


Schritt 9 - Konfiguration via Startskript (die Variante mit "my.env" funktionierte bei mir nicht)

    cd ~/nightscout
    nano start.sh	

Dort werden alle Einstellungen (Konfig, Plugins, etc vorgenommen):

	#!/bin/sh
	cd /home/user/nightscout
	export AUTH_DEFAULT_ROLES=denied
	export CUSTOM_TITLE="Mein Nightscout"
	export API_SECRET=PASSWORT
        
	#export SSL_KEY=/etc/letsencrypt/live/.../privkey.pem   (das wird später nachgetragen)
	#export SSL_CERT=/etc/letsencrypt/live/.../fullchain.pem
        
	BASE_URL="https://meine.nightscout.URL:1337"
        
	export MONGO_CONNECTION=mongodb://<"USER_NS">:<"PASSWORD_NS">@localhost:27017/nightscout
	
	export DISPLAY_UNITS=mg/dl
	export ENABLE="delta direction timeago devicestatus ar2 profile careportal boluscalc food rawbg iob cob bwp cage sage iage treatmentnotify basal pump openaps upbat errorcodes simplealarms bridge mmconnect loop"
	export DISABLE=""
	
	export TIME_FORMAT=24
	export NIGHT_MODE=off
	export SHOW_RAWBG=always
	export THEME=colors
	
	export ALARM_TIMEAGO_WARN=on
	export ALARM_TIMEAGO_WARN_MINS=15
	export ALARM_TIMEAGO_URGENT=on
	export ALARM_TIMEAGO_URGENT_MINS=30
	
	export PROFILE_HISTORY=off
	export PROFILE_MULTIPLE=off
	
	export BWP_WARN=0.50
	export BWP_URGENT=1.00
	export BWP_SNOOZE_MINS=10
	export BWP_SNOOZE=0.10
	
	export CAGE_ENABLE_ALERTS=true
	export CAGE_INFO=44
	export CAGE_WARN=48
	export CAGE_URGENT=72
	export CAGE_DISPLAY=hours
	
	export SAGE_ENABLE_ALERTS=false
	export SAGE_INFO=144
	export SAGE_WARN=164
	export SAGE_URGENT=166
	
	export IAGE_ENABLE_ALERTS=false
	export IAGE_INFO=44
	export IAGE_WARN=48
	export IAGE_URGENT=72
	export BRIDGE_USER_NAME=
	export BRIDGE_PASSWORD=
	export BRIDGE_INTERVAL=150000
	export BRIDGE_MAX_COUNT=1
	export BRIDGE_FIRST_FETCH_COUNT=3
	export BRIDGE_MAX_FAILURES=3
	export BRIDGE_MINUTES=1400
	
	export MMCONNECT_USER_NAME=
	export MMCONNECT_PASSWORD=
	export MMCONNECT_INTERVAL=60000
	export MMCONNECT_MAX_RETRY_DURATION=32
	export MMCONNECT_SGV_LIMIT=24
	export MMCONNECT_VERBOSE=false
	export MMCONNECT_STORE_RAW_DATA=false
	
	export DEVICESTATUS_ADVANCED="true"
	
	export PUMP_ENABLE_ALERTS=true
	export PUMP_FIELDS="reservoir battery clock status"
	export PUMP_RETRO_FIELDS="reservoir battery clock"
	export PUMP_WARN_CLOCK=30
	export PUMP_URGENT_CLOCK=60
	export PUMP_WARN_RES=50
	export PUMP_URGENT_RES=10
	export PUMP_WARN_BATT_P=30
	export PUMP_URGENT_BATT_P=20
	export PUMP_WARN_BATT_V=1.35
	export PUMP_URGENT_BATT_V=1.30
	
	export OPENAPS_ENABLE_ALERTS=false
	export OPENAPS_WARN=30
	export OPENAPS_URGENT=60
	export OPENAPS_FIELDS="status-symbol status-label iob meal-assist rssi freq"
	export OPENAPS_RETRO_FIELDS="status-symbol status-label iob meal-assist rssi"
	
	export LOOP_ENABLE_ALERTS=false
	export LOOP_WARN=30
	export LOOP_URGENT=60
	
	export SHOW_PLUGINS=careportal
	export SHOW_FORECAST="ar2 openaps"
	
	export LANGUAGE=en
	export SCALE_Y=log
	export EDIT_MODE=on
	
	PORT=1337 node server.js &   

Das Skript jetz noch ausführbar machen:
        
    sudo chmod +x start.sh
        
Jetzt kann der Server im nightscout-Verzeichnis mit 

    ./start.sh    
gestartet werden. 


Schritt 10 - Start/Stop mit Systemd-Service

Im Verzeichnis /etc/systemd/system/ oder /lib/systemd/sytem/ (geht wohl beides) wird eine Service-Datei erstellt:

    sudo nano nightscout.service

mit folgender Konfiguration:

    [Unit]
    Description=Nightscout Service      
    Documentation=none
    After=network.target

    [Service]
    Type=simple
    WorkingDirectory=/your/nightscout/path/cgm-remote-monitor
    ExecStart=/your/nightscout/path/cgm-remote-monitor/start.sh
    RemainAfterExit=yes

    [Install]
    WantedBy=multi-user.target

Dann Neustart von systemd:

    sudo systemctl daemon-reload

Dann den Nightscout-Service starten und auch bei Neustart automatisch starten lassen:

    sudo systemctl enable nightscout.service (das ".service" kann man auch weglassen)
    sudo systemctl start nightscout

Überprüfung, ob der Service korrekt läuft, oder Fehler ausgibt:

    sudo systemctl status nightscout



Schritt 11 - Nginx Server

Wer jetzt noch weitermachen und seine Seite über das Internet aufrufen möchte, kann noch einen Reverse-Proxy mit Nginx aufsetzen:

    sudo apt-get install nginx

Im Verzeichnis /etc/nginx/sites-available eine Datei mit folgendem Inhalt erstellen:

    sudo nano nightscout


    server {
        listen 80;

        server_name example.com;

        location / {
        proxy_pass http://127.0.0.1:1337;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        }
    }

Dann im Verzeichnis /etc/nginx/sites-enabled einen Symlink auf die Website legen:

    sudo ln -s /etc/nginx/sites-available/nightscout /etc/nginx/sites-enabled/nightscout

Dann den Nginx-Service neu starten:

    sudo systemctl restart nginx



Schritt 12 - Jetzt noch mit Let's Encrypt verschlüsseln

Let's Encrypt installieren:
    
    sudo apt-get install letsencrypt python-certbot-nginx
    sudo certbot

Let's Encrypt möchte Änderungen an der Seite vornehmen, das kann man zulassen.

Danach die Seite nochmal aufrufen:

    cd /etc/nxinx/sites-available
    sudo nano nightscout

Am Ende sieht meine Seite so aus:

    server {
          if ($host = meine.seite.url) {
          return 301 https://$host$request_uri;
          }
          listen 80;
          server_name meine.seite.url;
          #  if ($host = meine.seite.url) {
          #  return 301 https://meine.seite; # /$request_uri;
    }

    server {
            listen 443 ssl http2; # managed by Certbot
            server_name meine.seite.url;
            include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot

            location / {
            proxy_pass https://meine.seite.url:1337;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
            }

      ssl_certificate /etc/letsencrypt/live/meine.seite.url/fullchain.pem; # managed by Certbot
      ssl_certificate_key /etc/letsencrypt/live/meine.seite.url/privkey.pem; # managed by Certbot
      ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
    }

Die letzen Zeilen sind die Verweise auf die Schlüssel. Diese tragen wir nun noch in die start.sh ein (siehe Schritt 9):

     export SSL_KEY=/etc/letsencrypt/live/meine.seite.url/privkey.pem   
     export SSL_CERT=/etc/letsencrypt/live/meine.seite.url/fullchain.pem

Dann noch einmal den nginx-Service und den nightscout-Sevice neu starten

    sudo systemctl restart nginx
    sudo systemctl restart nightscout

Und dann sollte Nightscout im Browser erreichbar sein. 
Nach einem Neustart des Pi kann es sein, dass der nginx und nightscout service nicht automatisch gestartet werden, da ein Fehler auftritt. 
Dann die beiden mit systemctl restart noch mal neu starten (erst nginx, dann nightscout), dann sollte es klappen. 
Scheinbar ist ein benötigter Service nicht schnell genug geladen, so dass die Fehlermeldung kommt. Muss ich bei Gelegenheit noch mal nachschauen, woran das liegt. 
