# Nightscout-on-raspberry-pi
Installation einer Nightscout Instanz auf einem lokalen Raspberry PI3 / Building the Nightscout Website on own server with raspberry pi3

Basierend zu großen Teilen auf den Anleitungen von Kitekater und Johnmales und einigen anderen. 
Ich habe dies hier vor allem als Gedankenstütze für mich zusammengefasst, damit ich mir nicht wieder alles zusammensuchen muss, 
falls ich den Server neu aufsetzen muss/will.
 
Debian Buster aarch64 Betriebssystem wird vorausgesetzt.

Zugriff auf den Pi mit ssh, den Pi mit einer festen IP vom Router versehen. Optional: Den Pi über einen DynDNS Anbieter über das Internet erreichbar machen. Portfreigabe für tcp/1337 im Internetrouter einrichten, um Daten zu empfangen und die Webseite aufrufen zu können.
Falls der Server über das Internet erreichbar sein soll, muss auf jeden Fall noch dafür gesorgt werden, dass eine Firewall (z.B. iptables) eingerichtet wird. Darauf gehe ich hier aber nicht weiter ein. 


Installation of a Nightscout instance on a local Raspberry PI3 / Building the Nightscout Website on own server with raspberry pi3

Based in large part on the instructions from Kitekater and Johnmales and a few others. I have summarized this here mainly as a reminder for myself so that I don't have to look up everything again if I have to / want to set up the server again.

Debian Buster aarch64 operating system is required for this.

Access to the Pi with ssh, the Pi with a fixed IP from the router. Optional: Make the Pi accessible via a DynDNS provider over the Internet. Set up port sharing for tcp / 1337 in the Internet router in order to receive data and to be able to call up the website. If the server is to be accessible via the Internet, it must be ensured that a firewall (e.g. iptables) is set up. I will not go into that further here. 

1 - Debian aktualisieren / Update Debian

	sudo apt-get update
	sudo apt-get upgrade


2 - Debian Stretch-Sourcen einstellen für MongoDB Installation (geht bestimmt auch eleganter) / Choose Stretch-sources for MongoDB installation

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


3 - MongoDB (3.2) installieren / Install MongoDB

       sudo apt update 
       sudo apt install mongodb

Danach die sourcen wieder auf Buster zurücksetzen! / Reset sources back to Buster after this step!


4 - MongoDB einrichten / Setup MongoDB

Daten in <> durch eigene Daten ersetzen! / Replace data in <> with own data!	
User anlegen über die Mongo Shell: / Create User:

erst den Admin: / first Admin:

	sudo mongo -shell
	
		use admin
		db.createUser( { user: <"USER">, pwd: <"PASSWORD">, roles: [ "userAdminAnyDatabase" ] } )

dann die eigentliche Nightscout Datenbank und Nightscout User anlegen: / then Nightscout database and user:

		use mongodb   
		db.createUser( { user: <"USER_NS">, pwd: <"PASSWORD_NS">, roles: [ "readWrite", "dbAdmin" ] } )


5 - NodeJS 

       sudo apt update
       sudo apt-get install npm


Versionen testen: / test version

    npm -v	(sollte 6.14.1 sein)
    nodejs -v	(sollte 10.23.1 sein)
    node -v	(sollte 10.23.1 sein)
    
( eventuell hilft auch diese Anleitung: https://github.com/audstanley/NodeJs-Raspberry-Pi/
	sudo wget -O - https://raw.githubusercontent.com/audstanley/NodeJs-Raspberry-Pi/master/Install-Node.sh | sudo bash node -v )


6 - Nightscout Version herunterladen (13.0.1. läuft auf pi3, bei 14 hatte ich Probleme) / Download Nightscout 13.0.1

Kopiere die Sourcen z.B. ins Home Verzechnis des Benutzers: / Copy sources in home directory

Falls noch nicht vorhanden, wget und unzip installieren: / If necessary install wget and unzip
    
    sudo apt-get wget unzip

    wget https://github.com/nightscout/cgm-remote-monitor/archive/13.0.1.zip

Entpacke die Sourcen:/ unpack sources

	sudo unzip /home/user/13.0.1.zip
	
Damit später einfacher ein Update installiert, oder eine andere Version zum testen gesetzt werden kann, wird ein symbolischer Link auf das aktuelle Sourcen Verzeichnis gesetzt. Außerdem schreibt sich "nightscout" leichter als "cgm-remote-monitor-13.0.1". ;-)

A symbolic link is set to the current source directory so that an update can be installed more easily later, or a different version can be set for testing. In addition, "nightscout" is easier to write than "cgm-remote-monitor-13.0.1". ;-) 

	sudo ln -s cgm-remote-monitor-13.0.1 nightscout
	sudo chmod 777 cgm-remote-monitor-13.0.1


7 - Swap aktivieren / activate swap

Zumindest für die Installation über npm ist die Swapdatei nötig, sonst kommt es zum Abbruch. Der Speicher des pi3 reicht sonst nicht aus.
Ob die Seite danach auch ohne Swap läuft, könnt ihr ja mal testen.

The swap file is required at least for the installation via npm, otherwise it will be aborted as the memory of the pi3 is insufficient.
You can test whether the site will run without swap afterwards. 

     sudo dd if=/dev/zero of=/swapfile bs=1M count=4048
     sudo chmod 0600 /swapfile
     sudo mkswap /swapfile 
     sudo swapon /swapfile 
     
Damit die Auslagerungsdatei auch nach einem Neustart noch benutzt wird, die Datei /etc/fstab in einem Editor mit Rootrechten öffnen und dort diese Zeile anhängen:

So that the swap file is still used after a restart, open the file /etc/fstab in an editor with root rights and append this line there: 

     sudo nano /etc/fstab
     /swapfile    none    swap    sw      0 0


8 - Nightscout installieren / Install Nightscout

Dabei auf eventuelle Fehlermeldungen achten. Bei mir musste npm noch einmal aktualisiert werden, damit es fehlerfrei durchlief 
(npm install -g npm@latest).

Pay attention to any error messages. For me, npm had to be updated again so that it ran without errors
(npm install -g npm @ latest). 

    cd ~/nightscout
    npm update
    sudo npm i npm@latest -g    (muss als root ausgeführt werden)
    npm install

Der Installationsvorgang sollte nun ohne Fehler durchlaufen. Kann bei einem pi3 eine Weile dauern.
Schauen ob evtl. Fehlermeldungen auftauchen und evtl. Abhängigkeiten nachinstalliert werden müssen.

The installation process should now run without errors. Can take a while with a pi3.
Check whether any error messages appear and whether any dependencies need to be reinstalled. 


9 - Konfiguration via Startskript (die Variante mit "my.env" funktionierte bei mir nicht) / Configuration via script

    cd ~/nightscout
    nano start.sh	

Dort werden alle Einstellungen (Konfig, Plugins, etc vorgenommen):

All settings (config, plugins, and so on) will be made there:

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

Das Skript jetz noch ausführbar machen: / Make the script executable:
        
    sudo chmod +x start.sh
        
Jetzt kann der Server im nightscout-Verzeichnis mit: / Now you can start the server with:

    ./start.sh    
gestartet werden. 


10 - Start/Stop mit Systemd-Service

Im Verzeichnis /etc/systemd/system/ oder /lib/systemd/sytem/ (geht wohl beides) wird eine Service-Datei erstellt:

A service file is created in the directory / etc / systemd / system / or / lib / systemd / sytem / (both is possible): 

    sudo nano nightscout.service

mit folgender Konfiguration: / with following config:

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

Dann Neustart von systemd: / restart of systemd

    sudo systemctl daemon-reload

Dann den Nightscout-Service starten und auch bei Neustart automatisch starten lassen:

Starting Nightscout-Service and make it automatic startup on booting:

    sudo systemctl enable nightscout.service (das ".service" kann man auch weglassen)
    sudo systemctl start nightscout

Überprüfung, ob der Service korrekt läuft, oder Fehler ausgibt: / Checking if service runs:

    sudo systemctl status nightscout



11 - Nginx Server

Wer jetzt noch weitermachen und seine Seite über das Internet aufrufen möchte, kann noch einen Reverse-Proxy mit Nginx aufsetzen:

If you want to go on and make your site accessable via Internet, you can set up a reverse proxy with Nginx: 

    sudo apt-get install nginx

Im Verzeichnis /etc/nginx/sites-available eine Datei mit folgendem Inhalt erstellen:

Create file in /etc/nginx/sies-available:

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

Then put a symlink to the website in the directory /etc/nginx/ sites-enabled: 

    sudo ln -s /etc/nginx/sites-available/nightscout /etc/nginx/sites-enabled/nightscout

Dann den Nginx-Service neu starten: / Restart Nginx-service:

    sudo systemctl restart nginx



12 - Jetzt noch mit Let's Encrypt verschlüsseln / Encrypt via Let's Encrypt

Let's Encrypt installieren:
    
    sudo apt-get install letsencrypt python-certbot-nginx
    sudo certbot

Let's Encrypt möchte Änderungen an der Seite vornehmen, das kann man zulassen.

Let's Encrypt wants to make changes to the page, that can be allowed.

Danach die Seite nochmal aufrufen: / Open site again:

    cd /etc/nxinx/sites-available
    sudo nano nightscout

Am Ende sieht meine Seite so aus: / Should look like this:

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

The last lines are the references to the keys. We now enter this in start.sh (see step 9): 

     export SSL_KEY=/etc/letsencrypt/live/meine.seite.url/privkey.pem   
     export SSL_CERT=/etc/letsencrypt/live/meine.seite.url/fullchain.pem

Dann noch einmal den nginx-Service und den nightscout-Sevice neu starten

Restart nginx-service and nightscout-service

    sudo systemctl restart nginx
    sudo systemctl restart nightscout

Und dann sollte Nightscout im Browser erreichbar sein. 
Nach einem Neustart des Pi kann es sein, dass der nginx und nightscout service nicht automatisch gestartet werden, da ein Fehler auftritt. 
Dann die beiden mit systemctl restart noch mal neu starten (erst nginx, dann nightscout), dann sollte es klappen. 
Scheinbar ist ein benötigter Service nicht schnell genug geladen, so dass die Fehlermeldung kommt. Muss ich bei Gelegenheit noch mal nachschauen, woran das liegt. 

After that the Nightscout should be accessible in the browser.
After restarting the Pi, the nginx and nightscout service may not start automatically because an error occurs.
Then restart the two with systemctl restart (first nginx, then nightscout), then it should work.
Apparently a required service is not loaded fast enough so that the error message appears. I do have to look again at this to see why. 
