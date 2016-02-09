[![NPM version][npm-image]][npm-url] [![Circle CI](https://circleci.com/gh/signalfx/signalfx-nodejs.svg?style=svg)](https://circleci.com/gh/signalfx/signalfx-nodejs)
# Node.js client library for SignalFx

This is a programmatic interface in JavaScript for SignalFx's metadata and
ingest APIs. It is meant to provide a base for communicating with
SignalFx APIs that can be easily leveraged by scripts and applications
to interact with SignalFx or report metric and event data to SignalFx.


## Installation

To install using npm:
```sh
$ npm install signalfx --save-dev
```


## Usage

### API access token

To use this library, you need a SignalFx API access token, which can be
obtained from the SignalFx organization you want to report data into.

### Create client

There are two ways to create a client object: 

+ The default constructor `SignalFx`. This constructor uses Protobuf to send data to SignalFx. If it cannot send Protobuf, it falls back to sending JSON. 
+ The JSON constructor `SignalFxJson`. This constructor uses JSON format to send data to SignalFx.

```js
var signalfx = require('signalfx');

// Create default client
var client = new signalFx.SignalFx('MY_SIGNALFX_TOKEN' [, options]);
// or create JSON client
var clientJson = new signalFx.SignalFxJson('MY_SIGNALFX_TOKEN' [, options]);
```
Object `options` is an optional map and may contains following fields:
+ **enableAmazonUniqueId** - boolean, `false` by default. If `true`, library will retrieve Amazon unique identifier and set it as `AWSUniqueId` dimension for each datapoint and event. Use this option only if your application deployed to Amazon  
+ **dimensions** - object, pre-defined dimensions for each datapoint and event. This object has key-value format `{ dimension_name: dimension_value, ...}`
+ **ingestEndpoint** -  string, custom url to send datapoints in format http://custom.domain/api/path
+ **timeout** - number, sending datapoints timeout in ms (default is 1000ms)
+ **batchSize** - number, batch size to group sending datapoints 
+ **userAgents** - array of strings, items from this array will be added to 'user-agent' header separated by comma

### Reporting data

This example shows how to report metrics to SignalFx, as gauges, counters, or cumulative counters. 

```js
var signalfx = require('signalfx');

var client = new signalFx.SignalFx('MY_SIGNALFX_TOKEN');

client.send({
           cumulative_counters:[
             {  metric: 'myfunc.calls_cumulative', 
                value: 10,
                timestamp: 1442960607000},
             ...
           ],
           gauges:[
             {  metric: 'myfunc.time', 
                value: 532,
                timestamp: 1442960607000},
             ...
           ],
           counters:[
             {  metric: 'myfunc.calls', 
                value: 42,
                timestamp: 1442960607000},
             ...
           ]});
```
The `timestamp` must be a millisecond precision timestamp; the number of milliseconds elapsed since Epoch. The `timestamp` field is optional, but strongly recommended. If not specified, it will be set by SignalFx's ingest servers automatically; in this situation, the timestamp of your datapoints will not accurately represent the time of their measurement (network latency, batching, etc. will all impact when those datapoints actually make it to SignalFx).

### Sending multi-dimensional data

Reporting dimensions for the data is also optional, and can be accomplished by specifying a `dimensions` parameter on each datapoint containing a dictionary of string to string key/value pairs representing the dimensions:

```js
var signalfx = require('signalfx');

var client = new signalFx.SignalFx('MY_SIGNALFX_TOKEN');

client.send({
          cumulative_counters:[
            { 'metric': 'myfunc.calls_cumulative', 
              'value': 10, 
              'dimensions': {'host': 'server1', 'host_ip': '1.2.3.4'}},
            ...
          ],
          gauges:[
            { 'metric': 'myfunc.time', 
              'value': 532, 
              'dimensions': {'host': 'server1', 'host_ip': '1.2.3.4'}},
            ...
          ],
          counters:[
            { 'metric': 'myfunc.calls', 
              'value': 42, 
              'dimensions': {'host': 'server1', 'host_ip': '1.2.3.4'}},
            ...
          ]});
```

### Sending events

Events can be send to SignalFx via the `sendEvent` function. The
event param objects must be specified. `Event` param object is an optional map and may contains following fields:

+ **eventType** (string) - Required field. The event type (name of the event time series).
+ **category** (int) - the category of event. Choose one from EVENT_CATEGORIES list. 
Different categories of events are supported.Available categories of events are `USER_DEFINED`, `ALERT`, `AUDIT`, `JOB`, 
`COLLECTD`, `SERVICE_DISCOVERY`, `EXCEPTION`. For mode details see 
`proto/signal_fx_protocol_buffers.proto` file. Value by default is `USER_DEFINED`
+ **dimensions**  (dict) - a map of event dimensions, empty dictionary by default
+ **properties**  (dict) - a map of extra properties on that event, empty dictionary by default
+ **timestamp** (int64) - a timestamp, by default is current time
 Also please specify event category: for that get
option from dictionary `client.EVENT_CATEGORIES`.

```js
var signalfx = require('signalfx');

var client = new signalFx.SignalFx('MY_SIGNALFX_TOKEN');

client.sendEvent({
          category: '[event_category]',
          eventType: '[event_type]',
          dimensions: {
              'host': 'myhost',
              'service': 'myservice',
              'instance': 'myinstance'
          },
          properties: {
              'version': 'event_version'},
          timestamp: timestamp})
```

## Examples

Complete code example for Reporting data
```js
var signalfx = require('signalfx');

var myToken = 'MY_SIGNALFX_TOKEN';

var client = new signalFx.SignalFx(myToken);
var gauges = [{
        metric: 'test.cpu',
        value: 10
      }];

var counters = [{
        metric: 'cpu_cnt',
        value:  2
      }];
      
client.send({gauges: gauges, counters: counters});
```

Complete code example for Sending events
```js
var signalfx = require('signalfx');

var myToken = '[MY_SIGNALFX_TOKEN]';

var client = new signalFx.SignalFx(myToken);

var eventCategory = client.EVENT_CATEGORIES.USER_DEFINED
var eventType = 'deployment'
var dimensions = {
     host: 'myhost',
     service: 'myservice',
     instance: 'myinstance'
  };
var properties = {version: '[EVENT-VERSION]'};

client.sendEvent({category: eventCategory, 
            eventType: eventType, 
            dimensions: dimensions, 
            properties:properties});
```
See `example/generic_usage.js` for a complete code example for Reporting data.
Set your SignalFx token and run example 

```sh
$ node path/to/example/generic_usage.js
```

## License

Apache Software License v2 © [SignalFx](https://signalfx.com)

[npm-image]: https://badge.fury.io/js/signalfx.svg
[npm-url]: https://npmjs.org/package/signalfx
