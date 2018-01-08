# Genolotse Banktools Server Anforderungen

Die Anwendung ist als klassische Client-Server Anwendung konzipiert. Im Backend verwenden wir PHP mit [Symfony](http://www.symfony.com), im Frontend setzen wir [React](https://reactjs.org) ein.

Die Banktools werden von uns installiert und gewartet. Die Absicherung des Servers, sowie Backup und Monitoring übernimmt der Serverbetreiber.

## Serveranforderungen

* Linux o.ä. (Wir empfehlen ein Ubuntu Linux)
* PHP 7.2 mit OpCode Cache
* MySQL 5.7
* Webserver (Wir empfehlen `nginx` mit `php-fpm`)
* Zusätzliche Pakete
  * Bitte http://symfony.com/doc/current/reference/requirements.html befolgen
  * pdo_mysql
  * vim

Wir stellen gerne ein [Ansible Playbook](http://docs.ansible.com/ansible/latest/index.html) bereit, das einen Ubuntu Server exemplarisch konfiguriert.


## Konfiguration Webserver

Bitte ersetzen Sie `%IHR_DOMAIN_NAME%` in diesem Dokument durch den Domainnamen Ihrer Bank.

* Die Konfiguration richtet sich nach der [Symfony Dokmentation](http://symfony.com/doc/current/cookbook/configuration/web_server_configuration.html)
* Das Document Root befindet sich unter `/var/www/tools.%IHR_DOMAIN_NAME%/current/web/`
* Die Hosts `tools.%IHR_DOMAIN_NAME%` und `toolsadmin.%IHR_DOMAIN_NAME%` zeigen beide auf den entsprechenden VHost
* Installieren Sie die SSL Zertifikate für `tools.%IHR_DOMAIN_NAME%` und `toolsadmin.%IHR_DOMAIN_NAME%`


### Bespielhafte nginx-Konfiguration

    server {
        listen                      443 ssl;
    
        ssl                         on;
        ssl_certificate             /usr/ssl/server.crt;
        ssl_certificate_key         /usr/ssl/server.key;
    
        server_name                 tools.%IHR_DOMAIN_NAME% toolsadmin.%IHR_DOMAIN_NAME%;
        root                        /var/www/tools.%IHR_DOMAIN_NAME%/web;
    
        error_log                   /var/log/nginx/tools.%IHR_DOMAIN_NAME%.error.log;
        access_log                  /var/log/nginx/tools.%IHR_DOMAIN_NAME%.access.log;
    
        client_max_body_size        100M;
        client_body_buffer_size     128k;
    
        rewrite                     ^/app\.php/?(.*)$ /$1 permanent;
    
        location / {
            index                   app.php;
            try_files               $uri @rewriteapp;
        }
    
        location @rewriteapp {
            rewrite                 ^(.*)$ /app_dev.php/$1 last;
        }
    
        location ~ ^/(app|app_dev|app_test|config)\.php(/|$) {
            fastcgi_pass            unix:/run/php/php7.2-fpm.sock;
            fastcgi_buffer_size     16k;
            fastcgi_buffers         4 16k;
            fastcgi_split_path_info ^(.+\.php)(/.*)$;
            include                 fastcgi_params;
            fastcgi_param           SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param           HTTPS           on;
        }
    }

Bitte testen Sie die Konfiguration des Virtuellen Hosts: 

Legen Sie dazu `/var/www/tools.%IHR_DOMAIN_NAME%/current/web/app.php` mit folgendem Inhalt an:

    <?php
    phpinfo();

und rufen Sie dies über `https://tools.%IHR_DOMAIN_NAME%/` auf, es sollte eine Infoseite mit allen aktuell gesetzten PHP Werten angezeigt werden.

Bitte stellen Sie ein Verfahren zur Verfügung den OpCode Cache über die Kommandozeile zu clearen. Wir empfehlen ein „sudo service php5-fpm restart“ im Hinblick auf php-fpm für unseren User zu erlauben.

## Konfiguration Datenbank

MySQL Datenbank „genolotse“ mit entsprechendem User und Passwort.

## Zugang per SSH

Zum deployen der Anwendung benötigen wir einen SSH Benutzer. Bitte hinterlegen Sie dazu folgenden Public Key in `~/.ssh/authorized_keys`:

    ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA1VyQaPaKeUbG96/FD8UcsbX4xECJWBg0PuYiD7+NcP9x4tThRvaMw9s7xTQTA+NF48LTyaGP5yKTUT90KI2sDtijf/GzXFNjIVv4lBo2nysNtA8PvoFqjVjDm973axEdk+iSG7hGW6/aXKgQ0LFg1wAl+a17DnVC/UQgO/CyGg/lvGuH5fyw1Yn8ZZRUtVx+dR4A9lEbohpvGmTLsnBjiq3szj1onYWKcZ3mShM+fL+nRcASzPP4tAkyQkeQxDILUliNsyRlfvus3odVkao7PEyhAGJAIie8jackTwSSpkYHzguF5lLj37wuCEMxhuQ4G639qLXv5GGrJJZv367/uw== hautzi@Christoph-Hautzingers-MacBook-Pro.local

Dieser Benutzer benötigt volle Schreibrechte auf `/var/www/tools.%IHR_DOMAIN_NAME%`, die Deploy-Verzeichnisstruktur wird von uns erzeugt.

Bitte stellen Sie Leserechte für unseren Benutzer auf sämtliche Logs (Webserver, PHP, Datenbank, Mail) sicher.

## SMTP Mailserver

Bitte stellen Sie uns einen SMTP Server zum versenden von Mails zu Verfügung und teilen Sie uns mit, mit welcher Absenderadresse/Absendername Mails aus der Anwendung versendet werden sollen.

## Final Steps

Zum Schluss bitte eine E-Mail mit

* Server-Adresse
* SSH-Username
* MySQL-Zugangsdaten
* SMTP-Mailserver Adresse
* Absenderkennung für Mails aus der Anwendung 

an `christoph.hautzinger@genolotse.de`, wir kümmern uns dann um das initiale Deployment der Anwendung.
