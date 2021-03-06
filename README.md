# Marta.js

This library is a wrapper for the
[MARTA Realtime RESTful APIs](https://www.itsmarta.com/app-developer-resources.aspx).

It's designed to work both in node.js and the browser.

Because the documentation of the upstream APIs is limited, this library alters some naming
conventions from what the API returns in an effort to make the data easier to understand.

It uses [moment](https://momentjs.com/docs) for times and durations, and it's written in TypeScript
to aid in defining data structures.

You can [request an API key from MARTA](https://www.itsmarta.com/developer-reg-rtt.aspx) to use for
the Realtime Rail data. The Realtime Bus data does not require an API key.

# Usage

    npm install --save marta-js

## Promises vs Callbacks

All methods support both [promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) and callbacks.

Promise mode:

```js
railApi.getRealtimeTrainArrivals().then(function (arrival) {
  // ...
}).catch(function (error) {
  console.error(error)
})
```

Callback mode:

```js
railApi.getRealtimeTrainArrivals(function (error, arrival) {
  if (error) {
    console.error(error)
  } else {
    // ...
  }
})
```

## In the browser

This library depends on a native ES6 Promise implementation to be
[supported](http://caniuse.com/promises). If your environment doesn't support ES6 Promises,
for example older browsers, you can [polyfill](https://github.com/jakearchibald/es6-promise).

This library also depends on [moment-timezone](https://momentjs.com/timezone/docs/), which
is a fairly large library, so it is not bundled in the browser output blob.

NOTE: currently the Realtime APIs don't work from the browser, as the API does not support
CORS. Hopefully this will be resolved soon.

```html
<!-- polyfil for Promises -->
<script src="https://cdn.jsdelivr.net/npm/es6-promise@4/dist/es6-promise.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/es6-promise@4/dist/es6-promise.auto.min.js"></script>
<!-- moment-timezone -->
  <script src="node_modules/moment/moment.js"></script>
  <script src="node_modules/moment-timezone/builds/moment-timezone-with-data.min.js"></script>
<!-- marta-js -->
<script src="node_modules/marta-js/dist/marta.js"></script>
<script>
  // library is exposed as a global "Marta":
  const busApi = new window.Marta.RealtimeBusApi()
</script>
```

# Methods

## Realtime Rail

### Definition

```typescript
type RailArrival = {
  destination: string
  direction: 'North' | 'South' | 'East' | 'West'
  eventTime: Moment
  line: 'RED' | 'GOLD' | 'GREEN' | 'BLUE'
  nextArrival: Moment
  station: Station
  trainId: string
  waitingTimeSeconds: number
  waitingTime: Moment.Duration
  waitingState: 'Boarding' | 'Arriving' | string
}
```

### Example

```js
{
  line: 'GOLD', // which train line
  destination: 'Airport', // Name of the train line
  direction: 'South', // Direction of the train line
  eventTime: moment("2017-07-25T21:45:42.000"), // Time at which this update was received
  station: 'AIRPORT STATION', // Name of the station the times are relative to
  nextArrival: moment("2017-07-25T19:45:42.000"), // Time the train will arrive at the station
  trainId: '303506', // A unique identifier for the train
  waitingTimeSeconds: 1608, // how long in seconds until the train arrives at the station
  waitingTime: moment.duration(1608, 'seconds'), // the waiting time, but as a duration object
  waitingState: '26 min' // a string representing the waiting time. Could be also be "Boarding" or "Arriving"
}
```

### `RealtimeRailApi#getArrivals`

This is the same data that powers the realtime train time monitors in the station. For all stations,
it will tell you how long until the next several trains arrive.

```js
const RealtimeRailApi = require('marta-js').RealtimeRailApi
const railApi = new RealtimeRailApi('xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx') // your api key
railApi.getArrivals(function (error, arrivals) {
  // ...
})
```

### `RealtimeRailApi#getArrivalsForStation`

This is the same as `getArrivals`, but filtered to only the specified station.

```js
const RealtimeRailApi = require('marta-js').RealtimeRailApi
const railApi = new RealtimeRailApi('xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx') // your api key
railApi.getArrivalsForStation('FIVE POINTS STATION', function (error, arrivals) {
  // ...
})
```

## Realtime Bus

### Definition

```typescript
type BusArrival = {
  adherence: Moment.Duration
  blockId: string
  blockAbbriviation: string
  direction: 'North' | 'South' | 'East' | 'West'
  latitude: number
  longitude: number
  eventTime: Moment
  route: BusRoute
  stopId: string
  timepoint: string
  tripId: string
  busId: string
}
```

### Example

```js
{
  route: '12', // the route the bus is on
  direction: 'North', // the direction the bus is headed
  tripId: '5572278',
  busId: '1453', // a unique id for the bus
  stopId: '901718', // a unique id of the next stop
  // the current location of the bus
  latitude: 33.8206244,
  longitude: -84.4163082,
  eventTime: moment("2017-07-25T21:46:12.000"), // Time at which this update was received
  // Identifies the current time of arrival compared to scheduled arrival time.
  // Negative number indicates bus is running late. Note: this is opposite of what
  // comes back from the API, because I think it makes more sense.
  adherence: moment.duration(-2, 'minutes'),
  blockId: '82', // TODO: ???
  blockAbbriviation: '12-10',  // TODO: ???
  timepoint: 'Howell Mill Rd at Trabert Ave NW',  // TODO: ???
}
```

### `RealtimeBusApi#getArrivals`

This API returns the lastest location update from each bus, it's route, and if it is on-time.

```js
const RealtimeBusApi = require('marta-js').RealtimeBusApi
const busApi = new RealtimeBusApi()
busApi.getArrivals(function (error, arrivals) {
  // ...
})
```

### `RealtimeBusApi#getArrivalsForRoute`

This is the same as `getArrivals`, but filtered to only the specified route.

```js
const RealtimeBusApi = require('marta-js').RealtimeBusApi
const busApi = new RealtimeBusApi()
busApi.getArrivalsForRoute('12', function (error, arrivals) {
  // ...
})
```

# License

[MIT](LICENSE)
