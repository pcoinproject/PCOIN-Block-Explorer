# PCOIN Block Explorer

![GitHub release (latest by date)](https://img.shields.io/github/v/release/pcoinproject/PCOIN-Block-Explorer?color=0A80AB&label=Version)
![GitHub last commit](https://img.shields.io/github/last-commit/pcoinproject/PCOIN-Block-Explorer)
![GitHub](https://img.shields.io/github/license/pcoinproject/PCOIN-Block-Explorer?color=0A80AB)

Customized version of [eIquidus](https://github.com/team-exor/eiquidus)

### Installation

#### Quick Install Instructions

##### Pre-Install

The following prerequisites must be installed before using the explorer:

- [Node.js](https://nodejs.org/en/) (v14.15.4 or newer recommended)
- [MongoDB](https://www.mongodb.com/) (v4.4.3 or newer recommended)
- [Git](https://git-scm.com/downloads) (v2.36.0 or newer recommended)
- A fully synchronized *pcoind* wallet daemon.
 **NOTE:** In most cases, the blockchain must be synced with the `txindex` feature enabled to have access to all transactions. See the [Wallet Settings](#wallet-settings) section for more details.

##### Database Setup

Open the MongoDB cli:

```
mongo
```

Select database:

**NOTE:** `explorer` is the name of the database where you will be storing local explorer data. You can change this to any name you want, but you must make sure that you set the same name in the `settings.json` file for the `dbsettings.database` setting.

```
use explorer
```

Create a new user with read/write access:

```
db.createUser( { user: "DBAdmin", pwd: "Ndsp2d77crrk33eBX!L", roles: [ "readWrite" ] } )
```

##### Download Source Code

```
git clone https://github.com/pcoinproject/PCOIN-Block-Explorer explorer
```

##### Install Node Modules

```
cd explorer && npm install --only=prod
```

##### Configure Explorer Settings

```
nano./settings.json
```

*Make required changes in settings.json*

**NOTE:** You can further customize the site by adding css rules to the `public/css/custom.scss` file.

### Start/Stop the Explorer

#### Start Explorer (Use for Testing)

You can launch the explorer in a terminal window that will output all warnings and error msgs with one of the following cmds (be sure to run from within the explorer directory):

```
npm start
```

or (useful for crontab):

```
cd /path/to/explorer && /path/to/npm run prestart && /path/to/node --stack-size=10000 ./bin/cluster
```

**NOTE:** mongod must be running to start the explorer.

The explorer defaults to cluster mode by forking an instance of its process to each cpu core, which results in increased performance and stability. Load balancing gets automatically taken care of and any instances that for some reason die, will be restarted automatically. If desired, a single instance can be launched with:

```
npm run start-instance
```

or (useful for crontab):

```
cd /path/to/explorer && /path/to/npm run prestart && /path/to/node --stack-size=10000 ./bin/instance
```

#### Stop Explorer (Use for Testing)

To stop the explorer running with `npm start` you can end the process with the key combination `CTRL+C` in the terminal that is running the explorer, or from another terminal you can use one of the following cmds (be sure to run from within the explorer directory):

```
npm stop
```

or (useful for crontab):

```
cd /path/to/explorer && /path/to/node ./scripts/stop_explorer.js
```

#### Start Explorer Using PM2 (Recommended for Production)

[PM2](https://www.npmjs.com/package/pm2) is a process manager for Node.js applications with a built-in load balancer that allows you to always keep the explorer alive and running even if it crashes. Once you have configured the explorer to work properly in a production environment, it is recommended to use PM2 to start and stop the explorer instead of `npm start` and `npm stop` to keep the explorer constantly running without the need to always keep a terminal window open.

You can start the explorer using PM2 with one of the following terminal cmds (be sure to run from within the explorer directory):

```
npm run start-pm2
```

or (useful for crontab):

```
cd /path/to/explorer && /path/to/npm run prestart "pm2" && /path/to/pm2 start ./bin/instance -i 0 --node-args="--stack-size=10000"
```

**NOTE:** Use the following cmd to find the install path for PM2 (Linux only):

```
which pm2
```

#### Start Explorer Using PM2 and Log Viewer

Alternatively, you can start the explorer using PM2 and automatically open the log viewer which will allow for viewing all warnings and error msgs as they come up by using one of the following terminal cmds (be sure to run from within the explorer directory):

```
npm run start-pm2-debug
```

or (useful for crontab):

```
cd /path/to/explorer && /path/to/npm run prestart "pm2" && /path/to/pm2 start ./bin/instance -i 0 --node-args="--stack-size=10000" && /path/to/pm2 logs
```

#### Stop Explorer Using PM2 (Recommended for Production)

To stop the explorer when it is running via PM2 you can use one of the following terminal cmds (be sure to run from within the explorer directory):

```
npm run stop-pm2
```

or (useful for crontab):

```
cd /path/to/explorer && /path/to/pm2 stop ./bin/instance
```

#### Start Explorer Using Forever (Alternate Production Option)

[Forever](https://www.npmjs.com/package/forever) is an alternative to PM2 which is another useful Node.js module that is used to always keep the explorer alive and running even if the explorer crashes or stops. Once you have configured the explorer to work properly in a production environment, forever can be used as an alternative to PM2 to start and stop the explorer instead of `npm start` and `npm stop` to keep the explorer constantly running without the need to always keep a terminal window open.

You can start the explorer using forever with one of the following terminal cmds (be sure to run from within the explorer directory):

```
npm run start-forever
```

or (useful for crontab):

```
cd /path/to/explorer && /path/to/npm run prestart && /path/to/forever start ./bin/cluster
```

**NOTE:** Use the following cmd to find the install path for forever (Linux only):

```
which forever
```

#### Stop Explorer Using Forever (Alternate Production Option)

To stop the explorer when it is running via forever you can use one of the following terminal cmds (be sure to run from within the explorer directory):

```
npm run stop-forever
```

or (useful for crontab):

```
cd /path/to/explorer && /path/to/forever stop ./bin/cluster
```

### Syncing Databases with the Blockchain

sync.js (located in scripts/) is used for updating the local databases. This script must be called from the explorers root directory.

```
Usage: /path/to/node scripts/sync.js [mode]

Mode: (required)
update           Updates index from last sync to current block
check            Checks index for (and adds) any missing transactions/addresses
                 Optional parameter: block number to start checking from
reindex          Clears index then resyncs from genesis to current block
reindex-rich     Clears and recreates the richlist data
reindex-txcount  Rescan and flatten the tx count value for faster access
reindex-last     Rescan and flatten the last blockindex value for faster access
market           Updates market summaries, orderbooks, trade history + charts
peers            Updates peer info based on local wallet connections
masternodes      Updates the list of active masternodes on the network

Notes:
- 'current block' is the latest created block when script is executed.
- The market + peers databases only support (& defaults to) reindex mode.
- If check mode finds missing data (other than new data since last sync),
  this likely means that sync.update_timeout in settings.json is set too low.
```

*It is recommended to do the initial syncing of your blockchain, markets, peers and masternodes using the manual commands below to ensure there are no sync issues. When you are sure that everything is syncing correctly, you should then install the necessary scripts to a crontab at 1+ minute intervals as indicated below*

#### Commands for Manually Syncing Databases

A number of npm scripts are included with the explorer for easy syncing of the various explorer databases. The following scripts are the main commands used for syncing the explorer with your blockchain:

- `npm run sync-blocks`: Connect to the wallet daemon to pull blocks/transactions into the explorer, starting from genesis to current block. Repeat calls of this command will remember the last block downloaded, to allow continuous syncing of new blocks.
- `npm run sync-markets`: Connect to the various exchange apis as defined in the `settings.json` file to provide market related data such as market summaries, orderbooks, trade history and charts.
- `npm run sync-peers`: Connect to the wallet daemon and pull in data regarding connected nodes.
- `npm run sync-masternodes`: Connect to the wallet daemon and pull in the list of active masternodes on the network. *\*only applicable to masternode coins*

A small handful of useful scripts are also included to assist in solving various issues you may experience with the explorer:

- `npm run check-blocks`: Recheck all previously synced blocks by comparing against the wallet daemon to look for and add any missing transactions/addresses. Optional parameter: block number to start checking from. Example: `npm run check-blocks 1000` will begin the check starting at block 1000. :warning: **WARNING:** This can take a very long time depending on the length of the blockchain and is generally not recommended unless absolutely necessary. Furthermore, while you are checking for missing data, you will be unable to sync new blocks into the explorer until the check command has finished. If you do find missing transactions with this check (other than new data since last sync), this likely means that `sync.update_timeout` in `settings.json` is set too low.
- `npm run reindex`: Delete all blocks, transactions and addresses, and resync from genesis to current block. :warning: **WARNING:** This will wipe out all blockchain-related data from the explorer. It is recommended to [backup the explorer database](#backup-database-script) before continuing with this command.
- `npm run reindex-rich`: Clears and recreates the richlist data for the top 100 coin holders page. Rarely needed, but can be useful for debugging or if you are certain the richlist data is incorrect for some reason.
- `npm run reindex-txcount`: Recalculate the count of transactions stored in `stats.txes` by recounting the txes stored in the mongo database. Rarely needed, but can be useful for debugging or if you notice the main list of transactions is showing the wrong number of entries. If this value is off for some reason, you will not be able to page back to the 1st blocks on the main list of transactions for example.
- `npm run reindex-last`: Lookup the last transaction in the mongo database and reset the `stats.last` value to that most recent block index. Rarely needed, but can be useful for debugging. The `stats.last` value is used to remember which block the last sync left off on to resume syncing from the next block.

Also see the [Useful Scripts](#useful-scripts) section for more helpful scripts.

#### Sample Crontab

*Example crontab; update index every minute, market data every 2 minutes, peers and masternodes every 5 minutes*

Easier crontab syntax using npm scripts, but may not work on some systems depending on permissions and how nodejs was installed:

```
*/1 * * * * cd /path/to/explorer && npm run sync-blocks > /dev/null 2>&1
*/2 * * * * cd /path/to/explorer && npm run sync-markets > /dev/null 2>&1
*/5 * * * * cd /path/to/explorer && npm run sync-peers > /dev/null 2>&1
*/5 * * * * cd /path/to/explorer && npm run sync-masternodes > /dev/null 2>&1
```

Or, run the crontab by calling the sync script directly, which should work better in the event you have problems running the npm scripts from a crontab:

```
*/1 * * * * cd /path/to/explorer && /path/to/node scripts/sync.js update > /dev/null 2>&1
*/2 * * * * cd /path/to/explorer && /path/to/node scripts/sync.js market > /dev/null 2>&1
*/5 * * * * cd /path/to/explorer && /path/to/node scripts/sync.js peers > /dev/null 2>&1
*/5 * * * * cd /path/to/explorer && /path/to/node scripts/sync.js masternodes > /dev/null 2>&1
```

### Wallet Settings

The wallet connected to explorer must be running with the following flags:

```
-daemon -txindex
```

You may either call your coins daemon using this syntax:

```
coind -daemon -txindex
```

or else you can add the settings to your coins config file (recommended):

```
daemon=1
txindex=1
```

### License

Copyright (c) 2021-2022, PCOIN Dev Team<br />
Copyright (c) 2019-2021, The Exor Community<br />
Copyright (c) 2017, The Chaincoin Community<br />
Copyright (c) 2015, Iquidus Technology<br />
Copyright (c) 2015, Luke Williams<br />
All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

* Neither the name of Iquidus Technology nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
