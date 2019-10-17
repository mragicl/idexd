# IDEXd: IDEX Staking Tier 3 [beta]
# TEST VERSION FOR A RASPBERRY PI 


## Introduction

IDEXd is software that enables the [IDEX](https://idex.market/) community to stake IDEX tokens, serve parts of the IDEX production infrastructure, and earn fees for participation. IDEXd is the first part of a comprehensive plan to decentralize the centralized components of IDEX. For complete coverage, motivation, and roadmap, see our most recent post on [IDEX Staking](https://medium.com/idex/aura-staking-pos-earn-trade-fees-36319229ceae).

#### A note about versions

**IDEXd is currently beta software.** It is under development and subject to frequent changes, upgrades, and bug fixes. We appreciate your help in providing feedback and working around rough edges as we build towards 1.0.

## Requirements

### Staking

In order to be eligible to participate in IDEX Staking, you must have a wallet that holds a minimum of 10,000 IDEX for a minimum of 7 days. Dropping below a 10,000 IDEX balance, even for a brief period, will reset the incubation period.

### Hardware / VPS

Test version for Raspberry Pi. Tested with Raspberry Pi 4.
You will need > 20GB of disk space, so get a large memory card.

### Software

* Tested with Raspbian GNU/Linux 10 (buster)
* All dependencies are installed as described below. Please start with a freshly install copy of Rasbpian Buster image.



## Getting IDEXd

IDEXd is distrbuted via the @idexio/idexd-cli npm package with dependencies on Docker and Docker Compose.

## Getting Started

All of the requrements provide first-rate installation documentation, but we've collected the key steps to get up-and-running here. Start with a freshly installed copy of Raspbian GNU/Linux 10 (buster). Remark: Raspbian Buster Light suffices.


### Install Docker

1. Update packages
```
sudo apt update
sudo apt upgrade
```
2. Install dependencies that allow `apt` to install packages via https
```
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common git
```
3. Install docker from docker.com:
```
curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh 
```

4. Confirm Docker is running
```
sudo systemctl status docker
```
The output should look similar to:
```
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2019-09-30 21:20:20 CEST; 19min ago
     Docs: https://docs.docker.com
 Main PID: 430 (dockerd)
    Tasks: 13
   Memory: 89.6M
   CGroup: /system.slice/docker.service
           └─430 /usr/bin/dockerd -H unix:// --containerd=/run/containerd/containerd.sock
```
5. Add your user to the `docker` group to avoid permissions issues
```
sudo usermod -aG docker ${USER}
```
6. Log out and log back in and `docker` [commands](https://docs.docker.com/) will be available from the prompt

### Install Docker Compose
1. Install dependencies
```
sudo apt-get install libffi-dev python python-pip
```
2. Install docker compose with apt:
```
sudo apt-get install docker-compose
```

### Install nvm, Node.js & npm

1. Install build depenency packages
```
sudo apt install build-essential python
```
2. Install `nvm`
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
```
3. Log out and log back in and `nvm` [commands](https://github.com/creationix/nvm#usage) will be available from the prompt
4. Install Node.js
```
source ~/.bashrc
nvm install 10.15
```

### Install and launch IDEXd

1. Install `idexd-cli` to start and manage IDEXd
```
npm install -g @idexio/idexd-cli
```
2. Get the patch needed to make @idexio/idex-cli work for a raspberry pi with armhf architecture:
```
cd ~/
wget https://raw.githubusercontent.com/mragicl/idexd/master/patch/0001_RPI.patch
cd ~/.nvm/versions/node/v10.15.3/lib/node_modules/@idexio/idexd-cli/
patch -s -p0 < ~/0001_RPI.patch
```


3. Configure a staking wallet
```
idex config
```
IDEXd provides prompts asking for a staking wallet that contains at least 10,000 IDEX for 7 days. Go to [MyEtherWallet](https://www.myetherwallet.com/interface/sign-message) or your preferred wallet software to sign the challenge and provide the `sig` value to prove that you control the wallet.

IDEXd employs a cold-wallet design so that staked funds never need to leave the staking wallet for maximum security.

4. Get the raspberry pi compatible idexd docker container:
```
docker pull mragicl/idexd
```


5. Sync your IDEX Node and start serving traffic. Remark: idex comes with parity. I was not able to make it run on the raspberry pi 4, so I used a rpc endpoint from infura:
```
idex start --rpc <RPC endpoint URL including port>
```
IDEXd connects to the Ethereum network and downloads IDEX trade history data to serve to IDEX users. Depending on the resources of the underlying computer or VPS and network, the initial syncing process may take a few minutes to several hours.

TODO: documentation on forcing a full sync vs fast sync

#### A note on connectivity

Long time stability with the raspberry pi has not been tested thoroughly. You will need to open port 8443 on your router in order to be able to serve requests from IDEX users.
In order to serve data to IDEX users, IDEXd Nodes must be reachable from the public internet. Most home and office connections are not publicly reachable by default, so you may need to take steps like opening up specific ports on your router. IDEXd requires public TCP access to port 8443, and has limits on how frequently a node can change IP addresses.



## Managing IDEXd

IDEXd is design to require minimal maintenance once it is live. For details on managing IDEXd
```
idex
```
to display documentation on the `idexd-cli`'s capabilities.

### Common management tasks

### Public endpoints

To inspect the current total AURA staked, you may query the public endpoint:
https://reporting.idex.market/aurad/staked

#### Examining logs

Logs are the best source of information to understand what's happening with IDEXd under the hood. To follow the IDEXd logs
```
docker logs -f docker_idexd_1
```

#### Stopping IDEXd

```
idex stop
```

#### Restarting IDEXd

```
idex restart
```

#### Upgrading IDEXd

To upgrade IDEXd, stop the service, upgrade `@idexio/idexd-cli`, apply the patch,  and restart the service.
```
$ idex stop
$ npm install -g @idexio/idexd-cli
$ cd ~/.nvm/versions/node/v10.15.3/lib/node_modules/@idexio/idexd-cli/
$ patch -s -p0 < ~/0001_RPI.patch
$ idex start
```
Occasionally an upgrade may require running `idex config` before it can start serving traffic.



## License

IDEXd is licensed under the [GNU Lesser General Public License v3.0](https://www.gnu.org/licenses/lgpl-3.0.en.html).


### Sponsoring
This is just a little update to make idexd work for the raspberry pi. 

ETH: 
0x6c54eA14109f3E97cdfC02b0C5AbE88e190BDf18 

IOTA: 
9PDEUUFUTPLBNGORASBZGYXLWC9KLWWPGZFF9T9AHUKMLHHEVDWJJDUNXFJNADDHKT9ZKCNCVEY9MJRTZEWUHGASKY

BTC: 
35N7cEkjiKGrMyETDU61KBtWWc7wRBYAXv
