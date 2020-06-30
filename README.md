# Google IoT Core integration for Mongoose OS

This library is an extension to the original library that implements the integration of Mongoose OS with Google's Cloud IoT Core.

## Additions:

- New function to send state changes
- C handlers to adapt config and command messages so mJS can easily register a function to receive them
- Expanded mJS API to include these.
- If you also use rpc-gcp (RPC over GCP), your commands handler will not be triggered by RPC messages as long as those use their own subfolder

## Try it

You can test it just by including this library instead of the original one
```
libs:
  - origin: https://github.com/CikaElectronica/gcp2             # modified version of GCP libraries
    name: gcp
```

## Usage:

```
load('api_gcp.js');

// send telemetry (events) works as before
Timer.set(5000 /* milliseconds */, Timer.REPEAT, function() {
    if(GCP.isConnected()){
        let msg=JSON.stringify({t: Timer.fmt("%F %T", Timer.now())});
        Log.print(Log.INFO, 'Send event:' + ((GCP.sendEvent(msg)) ? 'OK' : 'FAIL') + ' msg:' +  msg);
    }
}, null);

// send state change
GPIO.set_button_handler(button_pin, GPIO.PULL_NONE, GPIO.INT_EDGE_POS, 100, function() {
    if(GCP.isConnected()){
        let msg = JSON.stringify({free: Sys.free_ram(), total: Sys.total_ram()}); 
        Log.print(Log.INFO, 'Send state:' + ((GCP.sendState(msg)) ? 'OK' : 'FAIL') + ' msg:' +  msg);
    }
}, null);

// receive config changes
GCP.config(function (confdata, ud) {
    if(!confdata)
      return;
    let obj = JSON.parse(confdata) || {out1: 0};
    Log.print(Log.INFO, 'Received config: ' + confdata + ' output: ' + ((obj.out1)? 'ON':'OFF'));
}, null);

// receive commands
GCP.command(function (cmddata, subfolder, ud) {
    if(!cmddata)
      return;
    let cmd = JSON.parse(cmddata) || {};
    Log.print(Log.INFO, 'Received command: ' + cmddata);
    if(subfolder)
      Log.print(Log.INFO, 'for subfolder:: ' + subfolder);
}, null);

// handle connect/disconnect
Event.addHandler(Event.GCP_CONNECT, function (ev, evdata, ud) {
    Log.print(Log.INFO, 'Connected to GCP');
}, null);
Event.addHandler(Event.GCP_CLOSE, function (ev, evdata, ud) {
    Log.print(Log.INFO, 'Disconnected from GCP');
}, null);
```
