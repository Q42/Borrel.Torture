# Borrel Torture Test
This repository is a fork from the original [Jitsi Meet Torture](https://github.com/jitsi/jitsi-meet-torture). It has been forked because there are several changes have been made to the original code. These changes ensure smooth setup with multiple Selenium Grid nodes and easy integration with the Borrel room itself.

## Changes compared to original
The following changes have been made to the original code;

- All the nodes have the name 'Bot' in order to detect them in Borrel,
- Web participants spawn an headless and non-sandboxed Chrome instance. It needs to be headless because our Google Cloud nodes don't have GPU's, which they also do not need to serve a video,
- The resource path is pointing to the actual resource path on our nodes, this is something that has to be done in the original files.

## Starting a torture test
In order to start a torture test, follow the steps described below.

### Starting premade Selenium Grid Hub in Google Cloud
For the torture test we use Selenium Grid to manage the test itself. The Selenium Grid has a Hub which tells all Nodes what to do and which room to connect to. In the Borrel Google Cloud Console you will find a premade Hub instance. This is probably stopped right now so make sure to spin this up.

To save some cash we did not (yet) opt for a fixed IP on the Hub. Therefore we need to **update the DNS record in TransIP** for now to point to the correct server. This will probably change in the future if we decide to let Google manage our namesevers. Once this is all done you can validate if the hub is running by navigating to:

`http://<SELENIUM_GRID_HUB_IP>:4444/grid/console`

This will show the currently connected nodes, which are probably none because we havent spun these up yet.

### Spinning up our Selenium Nodes (bots)
The bots will start up and try to connect to `https://jitsi-load-test-hub.borrel.app`, which is the Selenium Hub. They will keep trying to reconnect every few seconds until they find the hub. There is an 'Instance Template' and appropriate image in Google Cloud. This in template can be used to start an 'Instance Group' this instance group is a number of servers that can autoscale of have a fixed number of servers. This makes it easy to scale the number of nodes we want.

Each node can mock 1 user connecting. So if you want mock 100 users to join Jitsi Meet, make sure you spin up 100 Google Cloud servers. To create an instance group go to the appropriate page in Google Cloud and select the `jitsi-load-test-node-small` Instance Template. This template is called 'small' because it is the cheapest server only capable of spawning 1 Chrome instance at the time. In theory can spawn multiple instances of Chrome but this can cause bandwidth of processing bottlenecks on the Node server.

### Starting the actual test
This is where this repo comes in, either check it out locally (make sure you have Java SDK installed) or remote connect into the Selenium Hub. This already has the repo checked out at `/usr/share/borrel-torture`. In this folder you can start a torture test with the following command:

```
./scripts/malleus.sh --conferences=4 --participants=25 --senders=25 --audio-senders=25 --duration=120 --room-name-prefix=LoadTest --hub-url=http://jitsi-load-test-hub.borrel.app:4444/wd/hub --instance-url=https://jitsi-vpn.borrel.app
```

The command is pretty self-explanatory. But in short this will start 4 conference rooms with 25 participants each. Please note that this will require 100 Selenium Nodes to be spawned, so please note that his is not 25 participants as you might think. 

### Freshly Installing Hub and Nodes
If either of the images in Google Cloud become broken or lost, follow the appropriate guides in the readmes below.

- Selenium Hub Readme: [Freshly install Selenium Hub](README_SELENIUM_HUB.md)
- Selenium Node Readme: [Freshly install Selenium Node](README_SELENIUM_NODE.md)

## Notes
Because of all nodes connect instantly, this is not the easiest way to test autoscaling on the Jitsi videobridges. An alternative would be to send multiple commands with fewer participants at a time and having a manual delay in between. The Selenium Hub is smart enough to only direct nodes that are idle at the moment.
