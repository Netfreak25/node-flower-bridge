![Bridge](https://upload.wikimedia.org/wikipedia/commons/thumb/d/de/Suspension_bridge_icon.svg/2000px-Suspension_bridge_icon.svg.png)
![node-flower-bridge](http://img15.hostingpics.net/pics/880820nfb.png)
![Work](http://img15.hostingpics.net/pics/57414831fb.png)


[![Version node](https://img.shields.io/badge/node-4.x-brightgreen.svg)](https://nodejs.org/en/)
[![Issues](https://img.shields.io/badge/issues-open-orange.svg)](https://github.com/Parrot-Developers/node-flower-bridge/issues)
[![Forum](https://img.shields.io/badge/question-forum-blue.svg)](http://forum.developer.parrot.com/c/flower-power)

## Get your access API
* `username` `password`
	* Make sure you have an account created by your smartphone. 
* `client_id` `client_secret`
	* [Sign up to API here](https://api-flower-power-pot.parrot.com/api_access/signup), and got by **email** your *Access ID* (`client_id`) and your *Access secret* (`client_secret`).

## How to install
This program works with any *BLE-equipped/BLE-dongle-equipped* computers as well.
To install it on your raspberry is really easy. You require Node (with npm) and BLE libraries.

First, you need a raspberry with a *USB BLE dongle*. This raspberry must be up and running.
Then you need to install some required tools on your raspberry.

### Step 1: NodeJs
First, nodejs needs to be installed, proceed as following:
```bash
$ wget http://node-arm.herokuapp.com/node_latest_armhf.deb
$ sudo dpkg -i node_latest_armhf.deb
```

Then do a `node --version` to check if it worked.

### Step 2: BLE libraries
Then we need to install the BLE libraries:
```bash
$ sudo apt-get install libdbus-1-dev libdbus-glib-1-dev libglib2.0-dev libical-dev libreadline-dev libudev-dev libusb-dev glib2.0 bluetooth bluez libbluetooth-dev
```
The command `hciconfig` will be show your dongle (hci0):
```bash
$ sudo hciconfig hci0 up
```
You should be able to discover peripheral around you. To check it, do `sudo hcitool lescan` and if it shows you a list of surrounding BLE devices – or at least your Flower Power -- it works fine. If not, do `sudo apt-get install bluez` and try again.

### Step 3: Build the brigde
Now Nodejs and BLE libraries are installed.

#### Ready to use it
If you have cloned this project, install:
```bash
$ ./install.sh
```
Edit `credentials.json`:
```javascript
{
	"client_id":		"...",
	"client_secret":	"...",
	"username":		 "...",
	"password":		 "...",
	"url":				"..." // Url of API cloud
}
```
And walk on the brigde:
```bash
$ ./bridge display			: To have a output:
$ ./bridge background		 : To run the program in background
$ ./bridge restart			: To restart the program
$ ./bridge status			: To show if the program is running or not
$ ./bridge stop			   : To stop the program
$ ./bridge					: To have help
```

##### How it works
* Login Cloud
* Loop (every 15 minutes by default)
  * Get Inforamtions from Cloud
    * Your garden
    * Your user-config
  * For each of your FlowerPowers (1 by 1)
    * Scan to discover the Flower Power
    * Retrieve his history samples
    * Send his history samples to the Cloud
* End Loop

The program relive a new `Loop` only if all Flower Powers have been checked.

#### Quick started for developers
If you have this module in dependencies:
```bash
$ npm install node-flower-bridge
```
```javascript
var bridge = require('node-flower-bridge');
var credentials = {
	"client_id": "...",
	"client_secret": "...",
	"username": "...",
	"password": "..."
};

bridge.loginToApi(credentials, function(err, res) {
	if (err) return console.error(err);
	bridge.syncAll();
	bridge.live('...', 5);
	bridge.synchronize('...');
});

bridge.on('newProcess', function(flowerPower) {
	console.log(flowerPower.name, flowerPower.lastProcess);
});
bridge.on('info', function(info) {
	console.log(info.message);
});
bridge.on('error', function(error) {
	console.log(error.message);
});
```

The bridge is a continual queud. Method like `syncAll` `synchronize` or `live` push back to this queud.

##### Events
```js
'login' = {access_token, expires_in, refresh_token}
'info' = {message, date}
'error' = {message, date}
'newState' = state
'newProcess' = {name, lastProcess, process, date}
```

##### Api
```js
// Login in to the Cloud
bridge.loginToApi(credentials [, callback]);   // event: 'login'

// Get your garden configuration
bridge.getUser(callback);

// Make an automatic syncronization
var options = {
	delay: 15,      // loop delay
	priority: [],   // add a 'name'
};

brigde.automatic([options]); // Synchronize all flower power in your garden every 15 minutes by default
bridge.syncAll([options]); // Synchronize all flower power in your garden
bridge.synchronize(NAME); // Synchronize a flower power
bridge.live(NAME [, delay]); // Live for a flower power every 10 seconds by default
bridge.update(NAME, file); // Update the firmware [features: no file param = last firmware]
```
