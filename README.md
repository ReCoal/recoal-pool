cryptonote-nodejs-pool
======================

High performance Node.js (with native C addons) mining pool for CryptoNote based coins such as Bytecoin, DuckNote, Monero, QuazarCoin, Boolberry, Dashcoin, GRAFT, SUPERIORCOIN etc. Comes with lightweight example front-end script which uses the pool's AJAX API.



#### Table of Contents
* [Features](#features)
* [Community Support](#community--support)
* [Pools Using This Software](#pools-using-this-software)
* [Usage](#usage)
  * [Requirements](#requirements)
  * [Downloading & Installing](#1-downloading--installing)
  * [Configuration](#2-configuration)
  * [Starting the Pool](#3-start-the-pool)
  * [Host the front-end](#4-host-the-front-end)
  * [Customizing your website](#5-customize-your-website)
  * [SSL](#ssl)
  * [Upgrading](#upgrading)
* [JSON-RPC Commands from CLI](#json-rpc-commands-from-cli)
* [Monitoring Your Pool](#monitoring-your-pool)
* [Donations](#donations)
* [Credits](#credits)
* [License](#license)


Features
===

#### Optimized pool server
* TCP (stratum-like) protocol for server-push based jobs
  * Compared to old HTTP protocol, this has a higher hash rate, lower network/CPU server load, lower orphan
    block percent, and less error prone
* IP banning to prevent low-diff share attacks
* Socket flooding detection
* Share trust algorithm to reduce share validation hashing CPU load
* Clustering for vertical scaling
* Ability to configure multiple ports - each with their own difficulty
* Miner login (wallet address) validation
* Workers identification (specify worker name as the password)
* Variable difficulty / share limiter
* Set fixed difficulty on miner client by passing "address" param with "+[difficulty]" postfix
* Modular components for horizontal scaling (pool server, database, stats/API, payment processing, front-end)
* SSL support for both pool and API servers

#### Live statistics API
* Currency network/block difficulty
* Current block height
* Network hashrate
* Pool hashrate
* Each miners' individual stats (hashrate, shares submitted, pending balance, total paid, payout estimate, etc)
* Blocks found (pending, confirmed, and orphaned)
* Historic charts of pool's hashrate, miners count and coin difficulty
* Historic charts of users's hashrate and payments

#### Mined blocks explorer
* Mined blocks table with block status (pending, confirmed, and orphaned)
* Blocks luck (shares/difficulty) statistics
* Universal blocks and transactions explorer based on [chainradar.com](http://chainradar.com)

#### Smart payment processing
* Splintered transactions to deal with max transaction size
* Minimum payment threshold before balance will be paid out
* Minimum denomination for truncating payment amount precision to reduce size/complexity of block transactions
* Prevent "transaction is too big" error with "payments.maxTransactionAmount" option
* Option to enable dynamic transfer fee based on number of payees per transaction and option to have miner pay transfer fee instead of pool owner (applied to dynamic fee only)
* Control transactions priority with config.payments.priority (default: 0).
* Set payment ID on miner client when using "[address].[paymentID]" login
* Integrated payment ID addresses support for Exchanges

#### Admin panel
* Aggregated pool statistics
* Coin daemon & wallet RPC services stability monitoring
* Log files data access
* Users list with detailed statistics

#### Pool stability monitoring
* Detailed logging in process console & log files
* Coin daemon & wallet RPC services stability monitoring
* See logs data from admin panel

#### Extra features
* An easily extendable, responsive, light-weight front-end using API to display data
* Onishin's [keepalive function](https://github.com/perl5577/cpuminer-multi/commit/0c8aedb)
* Support for slush mining system (disabled by default)
* E-Mail Notifications on worker connected, disconnected (timeout) or banned (support MailGun, SMTP and Sendmail)


Community / Support
===

* [GitHub Issues for cryptonote-nodejs-pool](https://github.com/dvandal/cryptonote-nodejs-pool/issues)
* [CryptoNote Technology](https://cryptonote.org)
* [CryptoNote Forum](https://forum.cryptonote.org/)

#### Pools Using This Software

* https://graft.blockhashmining.com/


Usage
===

#### Requirements
* Coin daemon(s) (find the coin's repo and build latest version from source)
  * [ReCoal](https://github.com/ReCoal/recoal)
  * [ByteCoin](https://github.com/amjuarez/bytecoin)
  * [Monero](https://github.com/monero-project/bitmonero)
  * [GRAFT](https://github.com/graft-project/GraftNetwork)
  * [SUPERIORCOIN](https://github.com/TheSuperiorCoin/TheSuperiorCoin)
* [Node.js](http://nodejs.org/) v4.0+
  * For Ubuntu: `sudo apt-get install nodejs npm && ln -s /usr/bin/nodejs /usr/bin/node`
* [Redis](http://redis.io/) key-value store v2.6+ ([follow these instructions](http://redis.io/topics/quickstart))
* libssl required for the node-multi-hashing module
  * For Ubuntu: `sudo apt-get install libssl-dev`
* Boost is required for the cryptonote-util module
  * For Ubuntu: `sudo apt-get install libboost-all-dev`


##### Seriously
Those are legitimate requirements. If you use old versions of Node.js or Redis that may come with your system package manager then you will have problems. Follow the linked instructions to get the last stable versions.


[**Redis security warning**](http://redis.io/topics/security): be sure firewall access to redis - an easy way is to
include `bind 127.0.0.1` in your `redis.conf` file. Also it's a good idea to learn about and understand software that
you are using - a good place to start with redis is [data persistence](http://redis.io/topics/persistence).

#### 1) Downloading & Installing


Clone the repository and run `npm update` for all the dependencies to be installed:

```bash
git clone https://github.com/ReCoal/recoal-pool.git pool
cd pool

npm update
```

#### 2) Configuration

*Warning for Cryptonote coins other than Monero:* this software may or may not work with any given cryptonote coin. Be wary of altcoins that change the number of minimum coin units because you will have to reconfigure several config values to account for those changes. Unless you're offering a bounty reward - do not open an issue asking for help getting a coin other than Monero working with this software.

Copy the `config_examples/COIN.json` file of your choice to `config.json` then overview each options and change any to match your preferred setup.

Example Configuration :
```javascript
{
    "coin": "recoal",
    "symbol": "RECL",
    "coinUnits": 1000000000000,
    "coinDifficultyTarget": 120,
    "cnVariant": null,

    "logging": {
        "files": {
            "level": "info",
            "directory": "logs",
            "flushInterval": 5
        },
        "console": {
            "level": "info",
            "colors": true
        }
    },

    "poolServer": {
        "enabled": true,
        "clusterForks": "auto",
        "poolAddress": "__________________________YOUR_POOL_ADDRESS____________________________",
        "blockRefreshInterval": 1000,
        "minerTimeout": 900,
        "sslCert": "./cert.pem",
        "sslKey": "./privkey.pem",
        "sslCA": "./chain.pem",
        "ports": [
            {
                "port": 3333,
                "difficulty": 5000,
                "desc": "Low end hardware"
            },
            {
                "port": 4444,
                "difficulty": 15000,
                "desc": "Mid range hardware"
            },
            {
                "port": 5555,
                "difficulty": 25000,
                "desc": "High end hardware"
            },
            {
                "port": 7777,
                "difficulty": 500000,
                "desc": "Cloud-mining / NiceHash"
            },
            {
                "port": 9999,
                "difficulty": 20000,
                "desc": "SSL connection",
                "ssl": true
            }
        ],
        "varDiff": {
            "minDiff": 100,
            "maxDiff": 100000000,
            "targetTime": 60,
            "retargetTime": 30,
            "variancePercent": 30,
            "maxJump": 100
        },
        "paymentId": {
            "addressSeparator": "+"
        },
        "fixedDiff": {
            "enabled": true,
            "addressSeparator": "."
        },
        "shareTrust": {
            "enabled": true,
            "min": 10,
            "stepDown": 3,
            "threshold": 10,
            "penalty": 30
        },
        "banning": {
            "enabled": true,
            "time": 600,
            "invalidPercent": 25,
            "checkThreshold": 30
        },
        "slushMining": {
            "enabled": false,
            "weight": 300,
            "blockTime": 60,
            "lastBlockCheckRate": 1
         }
    },

    "payments": {
        "enabled": true,
        "interval": 300,
        "maxAddresses": 50,
        "mixin": 5,
        "priority": 0,
        "transferFee": 14000000000,
        "dynamicTransferFee": true,
        "minerPayFee" : true,
        "minPayment": 1000000000000,
        "maxTransactionAmount": 0,
        "denomination": 100000000000
    },

    "blockUnlocker": {
        "enabled": true,
        "interval": 30,
        "depth": 60,
        "poolFee": 1.8,
        "devDonation": 0.2
    },

    "api": {
        "enabled": true,
        "hashrateWindow": 600,
        "updateInterval": 5,
        "port": 8117,
        "blocks": 30,
        "payments": 30,
        "password": "your_password",
        "ssl": false,
        "sslPort": 8119,
        "sslCert": "./cert.pem",
        "sslKey": "./privkey.pem",
        "sslCA": "./chain.pem",
        "trustProxyIP": false
    },

    "daemon": {
        "host": "127.0.0.1",
        "port": 12121
    },

    "wallet": {
        "host": "127.0.0.1",
        "port": 12123
    },

    "redis": {
        "host": "127.0.0.1",
        "port": 6379,
        "auth": null,
        "cleanupInterval": 15
    },

    "email": {
        "enabled": false,
        "templateDir": "email_templates",
        "templates": ["worker_connected", "worker_banned", "worker_timeout"],
        "variables": {
            "POOL_HOST": "poolhost.com"
        },
        "fromAddress": "your@email.com",
        "transport": "sendmail",
        "sendmail": {
            "path": "/usr/sbin/sendmail"
        },
        "smtp": {
            "host": "smtp.example.com",
            "port": 587,
            "secure": false,
            "auth": {
                "user": "username",
                "pass": "password"
            }
        },
        "mailgun": {
            "key": "your-private-key",
            "domain": "mg.yourdomain"
        }
    },
    
    "monitoring": {
        "daemon": {
            "checkInterval": 60,
            "rpcMethod": "getblockcount"
        },
        "wallet": {
            "checkInterval": 60,
            "rpcMethod": "getbalance"
        }
    },

    "prices": {
        "source": "cryptonator",
        "currency": "USD"
    },
    
    "charts": {
        "pool": {
            "hashrate": {
                "enabled": true,
                "updateInterval": 60,
                "stepInterval": 1800,
                "maximumPeriod": 86400
            },
            "miners": {
                "enabled": true,
                "updateInterval": 60,
                "stepInterval": 1800,
                "maximumPeriod": 86400
            },
            "workers": {
                "enabled": true,
                "updateInterval": 60,
                "stepInterval": 1800,
                "maximumPeriod": 86400
            },
            "difficulty": {
                "enabled": true,
                "updateInterval": 1800,
                "stepInterval": 10800,
                "maximumPeriod": 604800
            },
            "price": {
                "enabled": true,
                "updateInterval": 1800,
                "stepInterval": 10800,
                "maximumPeriod": 604800
            },
            "profit": {
                "enabled": true,
                "updateInterval": 1800,
                "stepInterval": 10800,
                "maximumPeriod": 604800
            }
        },
        "user": {
            "hashrate": {
                "enabled": true,
                "updateInterval": 180,
                "stepInterval": 1800,
                "maximumPeriod": 86400
            },
            "payments": {
                "enabled": true
            }
        }
    }
}
```

#### 3) Start the pool

```bash
node init.js
```

The file `config.json` is used by default but a file can be specified using the `-config=file` command argument, for example:

```bash
node init.js -config=config_backup.json
```

This software contains four distinct modules:
* `pool` - Which opens ports for miners to connect and processes shares
* `api` - Used by the website to display network, pool and miners' data
* `unlocker` - Processes block candidates and increases miners' balances when blocks are unlocked
* `payments` - Sends out payments to miners according to their balances stored in redis


By default, running the `init.js` script will start up all four modules. You can optionally have the script start
only start a specific module by using the `-module=name` command argument, for example:

```bash
node init.js -module=api
```

[Example screenshot](http://i.imgur.com/SEgrI3b.png) of running the pool in single module mode with tmux.


#### 4) Host the front-end

Simply host the contents of the `website_example` directory on file server capable of serving simple static files.


Edit the variables in the `website_example/config.js` file to use your pool's specific configuration.
Variable explanations:

```javascript

/* Must point to the API setup in your config.json file. */
var api = "http://poolhost:8117";

/* Pool server host to instruct your miners to point to.  */
var poolHost = "poolhost.com";

/* Contact email address. */
var email = "support@poolhost.com";

/* Pool Telegram URL. */
var telegram = "https://t.me/YourPool";

/* Pool Discord URL */
var discord = "https://discordapp.com/invite/YourPool";

/* Market stat display params from https://www.cryptonator.com/widget */
var cryptonatorWidget = ["{symbol}-BTC", "{symbol}-USD", "{symbol}-EUR", "{symbol}-CAD"];

/* Default currency used by Estimate Mining Profit tool */
var defaultCurrency = 'USD';

/* Used for front-end block links. */
var blockchainExplorer = "http://chainradar.com/{symbol}/block/{id}";

/* Used by front-end transaction links. */
var transactionExplorer = "http://chainradar.com/{symbol}/transaction/{id}";

/* Any custom CSS theme for pool frontend */
var themeCss = "themes/light.css";

```

#### 5) Customize your website

The following files are included so that you can customize your pool website without having to make significant changes
to `index.html` or other front-end files thus reducing the difficulty of merging updates with your own changes:
* `custom.css` for creating your own pool style
* `custom.js` for changing the functionality of your pool website


Then simply serve the files via nginx, Apache, Google Drive, or anything that can host static content.

#### SSL

You can configure the API to be accessible via SSL using various methods. Find an example for nginx below:

* Using SSL api in `config.json`:

By using this you will need to update your `api` variable in the `website_example/config.js`. For example:  
`var api = "https://poolhost:8119";`

* Inside your SSL Listener, add the following:

``` javascript
location ~ ^/api/(.*) {
    proxy_pass http://127.0.0.1:8117/$1$is_args$args;
}
```

By adding this you will need to update your `api` variable in the `website_example/config.js` to include the /api. For example:  
`var api = "http://poolhost/api";`

You no longer need to include the port in the variable because of the proxy connection.

* Using his own subdomain, for example `api.poolhost.com`:

```bash
server {
    server_name api.poolhost.com
    listen 443 ssl http2;

    ssl_certificate /your/ssl/certificate;
    ssl_certificate_key /your/ssl/certificate_key;

    location / {
        more_set_headers 'Access-Control-Allow-Origin: *';
        proxy_pass http://127.0.01:8117;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

By adding this you will need to update your `api` variable in the `website_example/config.js`. For example:  
`var api = "//api.poolhost.com";`

You no longer need to include the port in the variable because of the proxy connection.


#### Upgrading
When updating to the latest code its important to not only `git pull` the latest from this repo, but to also update
the Node.js modules, and any config files that may have been changed.
* Inside your pool directory (where the init.js script is) do `git pull` to get the latest code.
* Remove the dependencies by deleting the `node_modules` directory with `rm -r node_modules`.
* Run `npm update` to force updating/reinstalling of the dependencies.
* Compare your `config.json` to the latest example ones in this repo or the ones in the setup instructions where each config field is explained. You may need to modify or add any new changes.

### JSON-RPC Commands from CLI

Documentation for JSON-RPC commands can be found here:
* Daemon https://wiki.bytecoin.org/wiki/Daemon_JSON_RPC_API
* Wallet https://wiki.bytecoin.org/wiki/Wallet_JSON_RPC_API


Curl can be used to use the JSON-RPC commands from command-line. Here is an example of calling `getblockheaderbyheight` for block 100:

```bash
curl 127.0.0.1:12121/json_rpc -d '{"method":"getblockheaderbyheight","params":{"height":100}}'
```


### Monitoring Your Pool

* To inspect and make changes to redis I suggest using [redis-commander](https://github.com/joeferner/redis-commander)
* To monitor server load for CPU, Network, IO, etc - I suggest using [New Relic](http://newrelic.com/)
* To keep your pool node script running in background, logging to file, and automatically restarting if it crashes - I suggest using [forever](https://github.com/nodejitsu/forever)


Donations
---------
* BTC: `19R69NNM4BAwSTcuJ8UjUCxcLBPyek9K8a`
* ETH: `0xb3272fb4e201d737cb2ffe82c46f674363c37057`
* LTC: `LbjuGwve9PwfSSkA4VZBANzuFNDfBc17Wx`
* XMR: `41iFUpFHqSnSoxJEADGPTihZXTtCpJUTLiaSh19Hdp5QePmVQ8DmdxgbfqCUUHH9CPC9t2Fwnwgg8cFs18jNvKUxAi4vrhJ`

Credits
---------

* [Pranav](https://github.com/pranav130100) - Lead Developer at ReCoal.
* [fancoder](//github.com/fancoder) - Developper on cryptonote-universal-pool project.

License
-------
Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html
