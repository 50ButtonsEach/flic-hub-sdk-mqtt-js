# flic-hub-sdk-mqtt-js

This is an implementation of an MQTT client. It is lightweight and provides efficient and robust communication mechanisms as well as QOS. It provides a subset of the protocol. Only QOS 0 is truly supported, even though it can handle QOS > 0 (but has no guarantee built in). Encryption and keys are not supported.

It has been developed to use in the Flic Hub SDK with the Flic Hubs.

## Set up in the Flic Hub SDK
In order to use MQTT with the Flic Hub SDK you first need to create a new module. This automatically gives you a module.json and a main.js file. On the module folder you can do a right click and add a new file. Name it mqtt.js. Then, copy past the code from this mqtt.js file in here to the new file on the Flic Hub SDK. This file does not need any modifications. All your code should go into the main.js file. In there you can require the mqtt.js file and have then access to all the functionalities in there. How to use it in the main.js is described in the following.

## Setting up and connecting
Create an MQTT object as follows:

``` javascript
var server = "YOUR_SERVER_HERE";

var options = { // ALL OPTIONAL - the defaults are below
    client_id : "id",   // the client ID sent to MQTT, default random value
    keep_alive: 60,         // keep alive time in seconds
    port: 1883,             // port number
    clean_session: true,
    username: "username",   // default is undefined
    password: "password",   // default is undefined
    protocol_name: "MQTT",  // or MQIsdp, etc..
    protocol_level: 4,      // protocol level
  };

var mqtt = require("./mqtt").create(server, options /*optional*/);
// could also look like: var mqtt = require("./mqtt").create(server);

mqtt.on('connected', function(){
    mqtt.subscribe("test");
});

mqtt.on('publish', function(pub){
    console.log("Received a message with: ");
    console.log("topic: "+pub.topic);
    console.log("message: "+pub.message);
});

mqtt.connect();


```

## Disconnection
If you want to disconnect and close the socket you can use the ``` disconnect() ``` function.

``` javascript
mqtt.disconnect();
```

Or you can be notified when MQTT disconnects.
``` javascript
mqtt.on('disconnected', function(){
    console.log("Disconnected.");
});
```

## Reconnection
If the client has been disconnected due to command or communication failure, it emits the event ```'disconnected'``` or ```'error'```. If you want the client to reconnect after a disconnection, you should insert the following code snippet. This listens to the emitted events and tries to connect the client again to the broker. If it does not work this will be repeated over again.

``` javascript
mqtt.on('disconnected', function() {
	mqtt.connect();
});

// Since errors can be thrown very fast, you can include a timeout here
mqtt.on('error', function() {
  setTimeout(function (){
    mqtt.connect();
  }, 1000);
});
```


## Publish
The message can be published to the broker any time during a session. A publish command needs the topic it will be published to and the message that should be sent. Any client that has subscribed to that topic will receive this message.

``` javascript
var topic = "test";
var message = "I can publish";

mqtt.publish(topic, message);
```

This method publish can also take a third argument.
``` javascript
mqtt.publish(topic, message, {qos : int, retain : bool, dub : bool});
```
- qos: quality of service level (default 0)
- retain: server should retain this packet and publish it as soon as a client subscribes
- dup: this is a dublicate packet

## Subscribe / Unsubscribe
With this method a mqtt client can subscribe to certain topics. A topic filter can be just a specific topic or contain wildcards that matches groups and/or sub-groups of topics. An event, 'publish', is fired whenever a message is received by the client. The event emitter calls the listeners with an object containing all the relevant packet information: topic, message, (dup, qos and retain).

``` javascript
mqtt.subscribe("test/hub");

mqtt.on('publish', function(pub){
    console.log("topic: " + pub.topic);
    console.log("message: " + pub.message);
});

mqtt.unsubscribe("test/hub");
```

## Events
The following events can be emitted:
- ```connect``` and ```connected``` (sent at the same time)
- ```disconnected``` and ```close``` (sent at the same time)
- ```error(message)```
- ```publish({topic, message, dup, qos, retain})``` and ```message(topic, message)``` (sent at the same time)
- ```subscribed``` mqtt.subscribe suceeded
- ```subscribed_fail``` - mqtt.subscribe failed
- ```unsubscribed``` - mqtt.unsubscribe succeeded
- ```ping_reply``` - a reply to the ping was received
- ```puback``` - a packet to acknowledge for a ```publish``` was received (QOS > 0 only)
- ```pubcomp``` - a packet to acknowledge that a ```pubrel``` was received (QOS 2 only)

## Examples
### Example 1: Send a message with publish() after a button action.
``` javascript
var server = "YOUR_SERVER_HERE";
var topic = "flic";

var buttonManager = require("buttons");
var mqtt = require("./mqtt").create(server);

buttonManager.on("buttonSingleOrDoubleClickOrHold", function(obj) {
	var button = buttonManager.getButton(obj.bdaddr);
	var clickType = obj.isSingleClick ? "click" : obj.isDoubleClick ? "double_click" : "hold";
	mqtt.publish(topic + "/" + button.serialNumber, clickType);
});

mqtt.connect();
```

### Example 2: Use the options
``` javascript
var server = "YOUR_SERVER_HERE";

var options = { // ALL OPTIONAL - the defaults are below
    client_id : "hub name",   // the client ID sent to MQTT
    keep_alive: 60,         // keep alive time in seconds
    port: 1883,             // port number
    clean_session: true,
    username: "username",   // default is undefined
    password: "password",   // default is undefined
    protocol_name: "MQTT",  // or MQIsdp, etc..
    protocol_level: 4,      // protocol level
};

var mqtt = require("./mqtt").create(server, options /*optional*/);

// As soon as the hub is connected, it should subscribe to the "test" topic
  mqtt.on('connected', function(){
  mqtt.subscribe("test");
});

// If something has been published to the client, log it here
mqtt.on('publish', function(pub){
  console.log("Received a message with: ");
  console.log("topic: "+pub.topic);
  console.log("message: "+pub.message);
});

// If the client has been disconnected, log it here
mqtt.on('disconnected', function(){
  console.log("Disconnected.");
});

// If the client has been disconnected, try to connect the client again
mqtt.on('disconnected', function() {
	mqtt.connect();
});

// If the client throws an error, try to connect the client again
mqtt.on('error', function () {
  mqtt.connect();
});

function testpublish() {
  var topic = "test";
  var message = "I can publish";
  mqtt.publish(topic, message);
}

// If everything is set up, connect the mqtt client to the broker
mqtt.connect();

setInterval(testpublish, 10000); // publishes every 10 seconds
```
