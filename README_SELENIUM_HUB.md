# Install commands for the Selenium Hub
If we ever lose the image to the Selenium Hub please execute the following commands to install a Selenium Hub from scratch, this is tested on Debian 10.

## Selenium Grid Hub
These commands will freshly install Selenium Grid

```
sudo su
apt-get -y  update
apt-get -y install default-jre default-jdk maven git
export JAVA_HOME=$(readlink -f /usr/bin/javac | sed "s:/bin/javac::")
curl https://selenium-release.storage.googleapis.com/3.141/selenium-server-standalone-3.141.59.jar --output /usr/local/bin/selenium-server-standalone.jar
chmod -x /usr/local/bin/selenium-server-standalone.jar
```

## Creating HTTPS certificate
In order for the nodes to connect to the hub using a secure connection we need to create a certificate and sign it.

```
var=jitsi-load-test-hub.borrel.app
hostnamectl set-hostname $var
apt-get -y  update
apt-get -y install software-properties-common certbot
certbot certonly --standalone
```

## Auto starting Selenium Hub on startup
To start a the Hub automatically on each startup we're going to create a system service for it.

First create the startup script itself starting the Java application. Execute `nano /usr/local/bin/selenium-hub-start.sh` and paste the following code

```
#!/bin/bash

java -jar /usr/local/bin/selenium-server-standalone.jar -role hub
```

Save the file and create the service itself that calls the script on startup. Execute `nano /etc/systemd/system/selenium-hub.service` and pase the following code

```
[Unit]
After=network.target

[Service]
ExecStart=/bin/bash /usr/local/bin/selenium-hub-start.sh

[Install]
WantedBy=default.target
```

Save the file and we're ready to give these files startup rights and enable the service on startup:

```
chmod -x /usr/local/bin/selenium-hub-start.sh
chmod 644 /etc/systemd/system/selenium-hub.service
systemctl daemon-reload
systemctl enable selenium-hub.service
```

## Optionally checkout Jitsi Meet Torture
We can choose to run Jitsi Meet Torture on the server instead of on a users PC. To do so check out this custom Borrel Torture using these commands

```
git clone https://github.com/Q42/Borrel.Torture.git /usr/share/borrel-torture
```

## Notes
One important thing to notice is to configure the firewall to accept incomming connections on port `4444`. There is a firewall rule that does this called `jitsi-stress-test-node` in Google Cloud Console.
