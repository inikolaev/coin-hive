# Coin-Hive [![Build Status](https://travis-ci.org/cazala/coin-hive.svg?branch=master)](https://travis-ci.org/cazala/coin-hive)

Mine cryptocurrency [Monero (XMR)](https://getmonero.org/) using [CoinHive](https://coinhive.com/) from node.js

**New:** Now you can [run this miner on any stratum based pool](https://github.com/cazala/coin-hive#faq).

## Disclaimer

This project is not endorsed by or affiliated with `coinhive.com` in any way.

## Install

```
npm install -g coin-hive
```

## Usage

```js
const CoinHive = require('coin-hive');

(async () => {

  // Create miner
  const miner = await CoinHive('ZM4gjqQ0jh0jbZ3tZDByOXAjyotDbo00'); // Coin-Hive's Site Key

  // Start miner
  await miner.start();

  // Listen on events
  miner.on('found', () => console.log('Found!'))
  miner.on('accepted', () => console.log('Accepted!'))
  miner.on('update', data => console.log(`
    Hashes per second: ${data.hashesPerSecond}
    Total hashes: ${data.totalHashes}
    Accepted hashes: ${data.acceptedHashes}
  `));

  // Stop miner
  setTimeout(async () => await miner.stop(), 60000);
})();
```

## CLI

```
Usage: coin-hive <site-key>

<site-key>: Your CoinHive Site Key

Options:

  --username        Set a username for the miner
  --interval        Interval between updates (logs)
  --port            Port for the miner server
  --host            Host for the miner server
  --threads         Number of threads for the miner
  --throttle        Set the fraction of time that threads should be idle
  --proxy           Proxy socket 5/4, for example: socks5://127.0.0.1:9050
  --puppeteer-url   URL where puppeteer will point to, by default is miner server (host:port)
  --miner-url       URL of CoinHive's JavaScript miner, can be set to use a proxy
  --pool-host       A custom stratum pool host, it must be used in combination with --pool-port
  --pool-port       A custom stratum pool port, it must be used in combination with --pool-host
```

## API

- `CoinHive(siteKey[, options])`: Returns a promise of a `Miner` instance. It requires a [Coin-Hive Site Key](https://coin-hive.com/settings/sites). The `options` object is optional and may contain the following properties:

  - `username`: Set a username for the miner. See [CoinHive.User](https://coinhive.com/documentation/miner#coinhive-user).

  - `interval`: Interval between `update` events in ms. Default is `1000`.

  - `port`: Port for the miner server. Default is `3002`.

  - `host`: Host for the miner server. Default is `localhost`.

  - `threads`: Number of threads. Default is `navigator.hardwareConcurrency` (number of CPU cores).

  - `throttle`: Set the fraction of time that threads should be idle. Default is 0 (full speed)

  - `proxy`: Puppeteer's proxy socket 5/4 (ie: `socks5://127.0.0.1:9050`).

  - `pool`: This allows you to use a different pool. It has to be an [Stratum](https://en.bitcoin.it/wiki/Stratum_mining_protocol) based pool. This object must contain the following properties:

      - `host`: The pool's host.

      - `port`: The pool's port.

- `miner.start()`: Connect to the pool and start mining. Returns a promise that will resolve once the miner is started.

- `miner.stop()`: Stop mining and disconnect from the pool. Returns a promise that will resolve once the miner is stopped.

- `miner.kill()`: Stop mining, disconnect from the pool, shutdown the server and close the headless browser. Returns a promise that will resolve once the miner is dead.

- `miner.on(event, callback)`: Specify a callback for an event. The event types are:

  - `update`: Informs `hashesPerSecond`, `totalHashes` and `acceptedHashes`.

  - `open`:	The connection to our mining pool was opened. Usually happens shortly after miner.start() was called.

  - `authed`:	The miner successfully authed with the mining pool and the siteKey was verified. Usually happens right after open.

  - `close`:	The connection to the pool was closed. Usually happens when miner.stop() was called.

  - `error`:	An error occured. In case of a connection error, the miner will automatically try to reconnect to the pool.

  - `job`:	A new mining job was received from the pool.

  - `found`:	A hash meeting the pool's difficulty (currently 256) was found and will be send to the pool.

  - `accepted`:	A hash that was sent to the pool was accepted.

- `miner.rpc(methodName, argsArray)`: This method allows you to interact with the Coin-Hive miner instance. It returns a Promise that resolves the the value of the remote method that was called. The miner intance API can be [found here](https://coin-hive.com/documentation/miner#miner-is-running). Here's an example:

```js
var miner = await CoinHive('SITE_KEY');
await miner.rpc('isRunning'); // false
await miner.start();
await miner.rpc('isRunning'); // true
await miner.rpc('getThrottle'); // 0
await miner.rpc('setThrottle', [0.5]);
await miner.rpc('getThrottle'); // 0.5
```

## Environment Variables

All the following environment variables can be used to configure the miner from the outside:

- `COINHIVE_SITE_KEY`: Coin-Hive's Site Key

- `COINHIVE_USERNAME`: Set a username to the miner. See [CoinHive.User](https://coinhive.com/documentation/miner#coinhive-user).

- `COINHIVE_INTERVAL`: The interval on which the miner reports an update

- `COINHIVE_THREADS`: Number of threads

- `COINHIVE_THROTTLE`: Set the fraction of time that threads should be idle

- `COINHIVE_PORT`: The port that will be used to launch the server, and where puppeteer will point to

- `COINHIVE_HOST`: The host that will be used to launch the server, and where puppeteer will point to

- `COINHIVE_PUPPETEER_URL`: In case you don't want to point puppeteer to the local server, you can use this to make it point somewhere else where the miner is served (ie: `COINHIVE_PUPPETEER_URL=http://coin-hive.herokuapp.com`)

- `COINHIVE_MINER_URL`: Set the CoinHive JavaScript Miner url. By defualt this is `https://coinhive.com/lib/coinhive.min.js`. You can set this to use a [CoinHive Proxy](https://github.com/cazala/coin-hive-proxy).

- `COINHIVE_PROXY`: Puppeteer's proxy socket 5/4 (ie: `COINHIVE_PROXY=socks5://127.0.0.1:9050`)

- `COINHIVE_POOL_HOST`: A custom stratum pool host, it must be used in combination with `COINHIVE_POOL_PORT`.

- `COINHIVE_POOL_PORT`: A custom stratum pool port, it must be used in combination with `COINHIVE_POOL_HOST`.

## FAQ

**Can I run this on a different pool than CoinHive's?**

Yes, you can run this on any pool based on the [Stratum Mining Protocol](https://en.bitcoin.it/wiki/Stratum_mining_protocol).

```js
const CoinHive = require('coin-hive');
(async () => {
  const miner = await CoinHive('<YOUR-MONERO-ADDRESS>', {
    pool: {
      host: 'xmr-eu1.nanopool.org',
      port: 14444
    }
  });
  await miner.start();
  miner.on('found', () => console.log('Found!'))
  miner.on('accepted', () => console.log('Accepted!'))
  miner.on('update', data => console.log(`
    Hashes per second: ${data.hashesPerSecond}
    Total hashes: ${data.totalHashes}
    Accepted hashes: ${data.acceptedHashes}
  `));
})();
```

Now your CoinHive miner would be mining on `nanopool.org` XMR pool, using your monero address.

You can also do this using the CLI:

```
coin-hive <YOUR-MONERO-ADDRESS> --pool-host=xmr-eu1.nanopool.org --pool-port=14444
```

**Can I run this on Heroku?**

Yes, but since Puppeteer requires some additional dependencies that aren't included on the Linux box that Heroku spins up for you, you need to go to your app's `Settings > Buildpacks`  first and add this url:

```
https://github.com/jontewks/puppeteer-heroku-buildpack
```

On the next deploy, your app will also install the dependencies that Puppeteer needs to run.

**Which version of Node.js do I need?**

Node v8+
