# Node.js CLEXI
Node.js CLEXI is a lightweight **cl**ient **ex**tension **i**nterface that enhances connected clients with functions of the underlying operating system using a duplex, realtime Websocket connection.
Current extensions include:
* clexi-broadcaster - a simple Websocket message broadcaster
* clexi-http-events - receives events at the 'event' endpoint and broadcasts them via the Websocket connection
* ble-beacon-scanner - scans for Bluetooth BLE beacons and broadcasts their data

CLEXI works as a web-server as well and can host the client application if required (e.g. just put the files in the www folder).

## Using as node module

Install via `npm install clexi` (tested under Linux, please see requirements section below) then build your own server like this:
```
'use strict'
const settings = {
	ssl: false
}
const Clexi = require('clexi')(settings);
Clexi.start();
```
This will run fastify with Websocket and static file support and expose the CLEXI xtensions.
Custom settings can be used to implement additional, self-made xtensions as well, e.g.:
```
const settings = {
	port: 9000,
	hostname: '0.0.0.0',
	id: 'my-clexi-server-123',
	wwwPath: '/home/pi/clexi-www',
	customXtensionsPath: '/home/pi/clexi-xtensions',
	customXtensions: ['my-xtension']
}
```
See client and extensions section below to get more info.

## Using the 'clexi-http-events' extension
As noted above this extension receives events at the 'event' endpoint and broadcasts them via the Websocket connection. The 'event' endpoint accepts HTTP GET and POST requests in the following format (curl example):
```
#POST example
curl -X POST \
  https://raspberrypi.local:8443/event/myEventName \
  -H 'Content-Type: application/json' \
  -d '{
	"answer": "42"
}'
#GET example
curl -X GET \
  'https://raspberrypi.local:8443/event/myEventName?answer=42'
```
CLEXI will then broadcast the data as following message object to all Websocket clients:
```
{"name":"myEventName","data":{"answer":"42"}}
```

## Raspberry Pi 3 installation

Requirements:  
* Node.js (tested with 9.11.2 and 10.15.3)
* Some Linux packages: `sudo apt-get install bluetooth bluez libbluetooth-dev libudev-dev libnss3-tools libcap2-bin openssl`
* SSL certificates for HTTPS (you can use the included script to generate self-signed)
  
Clone the repository to the folder 'clexi', enter that folder and run `npm install`:  
```
git clone https://github.com/bytemind-de/nodejs-client-extension-interface.git clexi
cd clexi
npm install
```  
Decide which hostname you want to use for your server. Default is `localhost` but I usually prefer `raspberrypi.local` (default hostname of RPi) to make CLEXI available to all devices in the network.  
You can change your hostname via the raspi-config tool.
Now generate some self-signed SSL certificates for your CLEXI server:  
```
sh generate_ssl_cert.sh
```  
The tool will ask you for some info. By pressing RETURN you can keep most of the default values, just for `common name` choose your hostname (or 'localhost').  
Next step is to adjust the CLEXI settings. Use a text editor of your choice, I prefer nano:
```
nano settings.json
```  
Here you can change the default port of your server and set the hostname to the SAME name you used for the SSL certificate (e.g. raspberrypi.local). This is important because you might not be able to reach the server otherwhise.  
Now you can run your server :-)  
```
sudo node server.js
```  
You should see a confirmation that the server is running and that extensions have been loaded (and hopefully no error ^^).  
The `sudo` command is required for Bluetooth control. If you want to run the server without sudo you have to grant the node binary cap_net_raw privileges:  
```
sudo setcap cap_net_raw+eip $(eval readlink -f `which node`)
```  
Finally to check if everything worked out fine visit the test-page in your browser, e.g. `https://raspberrypi.local:8443/index.html` (depending on your settings). When using a self-signed SSL certificate this also helps to tell the browser to trust them (usually required once).
  
## Client installation

### Clexi.js for browsers

Copy latest Clexi.js library from this repository and include it in your page head, e.g.:
```
<script type="text/javascript" src="lib/clexi-0.8.0.js" charset="UTF-8"></script>
```
Make sure your server is running and reachable, then connect like this:
```
var hostURL = "wss://raspberrypi.local:8443";
ClexiJS.serverId = 'my-clexi-server-123'; 	//must be empty or identical to 'id' entry in server settings
  
ClexiJS.subscribeTo('ble-beacon-scanner', function(e){
	console.log('BLE Beacon event: ' + JSON.stringify(e));
}, function(e){
	console.log('BLE Beacon response: ' + JSON.stringify(e));
}, function(e){
	console.log('BLE Beacon error: ' + JSON.stringify(e));
});
  
ClexiJS.connect(hostURL, function(e){
	console.log('connected');
	
	//start BLE beacon scanner
	ClexiJS.send('ble-beacon-scanner', {
		ctrl: "start"
	});
	
}, function(e){
	console.log('closed');
}, function(err){
	console.log('error');
});
```
  
For more [examples](www/index.html) check the `www` folder of this repository.

## Adding your own extensions

* Check the `xtensions` folder for references (it's pretty simple ;-))
* Build your own extension and put the file in the same folder
* Open `settings.json` and add your file to the xtensions array by using the filename without ending, e.g.: my-extension.js -> my-extension
* Subscribe inside your client app to your extension using the same name
* Done :-)

## Version history

See [changelog](CHANGELOG.md)
