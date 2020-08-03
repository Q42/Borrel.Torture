# Installing commands for the Selenium Node
If we ever lose the image to the Selenium Nodes please execute the following commands to install a Selenium Node from scratch, this is tested on Debian 10.

## Selenium Grid Node
These commands will freshly install Selenium Grid

```
sudo su
apt-get update
apt-get install -y gnupg curl wget unzip libxi6 libgconf-2-4 default-jdk xorg xvfb gtk2-engines-pixbuf dbus-x11 xfonts-base xfonts-100dpi xfonts-75dpi xfonts-cyrillic xfonts-scalable
curl -sS -o - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -
echo "deb [arch=amd64]  http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list
apt-get -y update
apt-get -y install google-chrome-stable
curl -O https://chromedriver.storage.googleapis.com/84.0.4147.30/chromedriver_linux64.zip
unzip chromedriver_linux64.zip -d /usr/local/bin/
curl https://selenium-release.storage.googleapis.com/3.141/selenium-server-standalone-3.141.59.jar --output /usr/local/bin/selenium-server-standalone.jar
mkdir -p /usr/share/jitsi-meet-torture/resources
curl https://media.xiph.org/video/derf/y4m/FourPeople_1280x720_60.y4m --output /usr/share/jitsi-meet-torture/resources/FourPeople_1280x720_60.y4m
```

## Auto starting Selenium Node on startup
To start a the Node automatically on each startup we're going to create a system service for it.

First create the startup script itself starting the Java application. Execute `nano /usr/local/bin/selenium-node-start.sh` and paste the following code

```
#!/bin/bash

Xvfb -ac :99 -screen 0 1280x1024x16 & export DISPLAY=:99
java -Dwebdriver.chrome.driver=/usr/local/bin/chromedriver -jar /usr/local/bin/selenium-server-standalone.jar -role node -maxSession 1 -hub http://jitsi-load-test-hub.borrel.app:4444/grid/register -browser browserName=chrome=chrome,version=84,platform=Linux,maxInstances=1
```

Save the file and create the service itself that calls the script on startup. Execute `nano /etc/systemd/system/selenium-node.service` and pase the following code

```
[Unit]
After=network.target

[Service]
ExecStart=/usr/local/bin/selenium-node-start.sh

[Install]
WantedBy=default.target
```

Save the file and we're ready to give these files startup rights and enable the service on startup:

```
chmod -x /usr/local/bin/selenium-node-start.sh
chmod -x /etc/systemd/system/selenium-node.service
systemctl daemon-reload
systemctl enable selenium-node.service
```