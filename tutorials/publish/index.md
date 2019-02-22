---
# Page settings
layout: default
keywords:
comments: true

# Hero section
title: How to Publish a SingularityNET Service
description: Getting your service to the SingularityNET Marketplace

# extralink box
extralink:
    title: All Docs
    title_url: '/docs/all'
    external_url: false
    description: Find an overview of our full documentation here.

# Developer Newsletter
dev_news: true

# Micro navigation
micro_nav: true

# Page navigation
page_nav:
    prev:
        content: Back to tutorials
        url: '/tutorials'
    next:
        content: View all docs
        url: '/docs/all'
---

[naming-standards]: https://dev.singularitynet.io/docs/all/naming-standard/

## Step 1. Get Started

Run this tutorial from a bash terminal.

In this tutorial we will publish an example service in SingularityNET using Kovan Test Network.

To prepare your system for this tutorial you have two options:

1. Installing all the the requirements on your system (go to [Step 1a](#step-1a-setup-your-system)).
2. Using a pre-configured [Docker](https://www.docker.com/) image (go to [Step 1b](#step-1b-setup-a-docker-container)).

## Step 1a. Setup your system

#### Requirements

- [Python 3.6+](https://www.python.org/downloads/)
- [Node 8+ with npm](https://nodejs.org/en/download/)
- [SNET CLI](https://github.com/singnet/snet-cli/releases)
    - libudev
    - libusb 1.0
- [SNET Daemon](https://github.com/singnet/snet-daemon/releases)

For Ubuntu >= 18.04 users:

```
sudo apt-get update
sudo apt-get install wget git
sudo apt-get install python3 python3-pip
sudo apt-get install nodejs npm
sudo apt-get install libudev-dev libusb-1.0-0-dev
sudo pip3 install snet-cli

# !!! Change version to the latest snet-daemon from releases link above
SNETD_VERSION="v0.1.6"

wget https://github.com/singnet/snet-daemon/releases/download/$SNETD_VERSION/snetd-linux-amd64
chmod a+x snetd-linux-amd64
sudo mv snetd-linux-amd64 /usr/bin/snetd
```

Now you can proceed to [Step 2](#step-2-setup-environment-variables).

## Step 1b. Setup a Docker container

-------------------------------
_Before following, make sure you've installed_

* _Docker ([https://www.docker.com/](https://www.docker.com/))_

_If you aren't familiar with Docker you may want to take a look at its official [Get Started Guide](https://docs.docker.com/get-started/)._

-------------------------------

Setup a `ubuntu:18.04` docker container using provided `Dockerfile`.

```
docker build -t snet_publish_service https://github.com/singnet/dev-portal.git#master:/tutorials/docker
```

Setup environment variables (variables are explained later in the tutorial as they're used)

```
ORGANIZATION_ID="$USER"-org
ORGANIZATION_NAME="The $USER's Organization"

SERVICE_ID=example-service
SERVICE_NAME="SNET Example Service"
SERVICE_IP=127.0.0.1
SERVICE_PORT=7000

DAEMON_HOST=0.0.0.0
USER_ID = $USER
```

Now you can simply run a docker container (with proper port mapping).

XXX
Tab 1: standard docker run

```
docker run \
XXX variaveis
-p $SERVICE_PORT:$SERVICE_PORT \
-ti snet_publish_service bash
```
XXX
Tab 2: (optional) persistence on host machine

Services store some sensible information in the filesystem. So you may want to share a couple of folders in your host machine with the Docker container to avoid then from vanishing if you loose the Docker container.

- etcd storage folder (XXX link para o daemon direto para a secao do etcd)
- SNET CLI configuration folder (XXX link para a Cli)

```
# To secure payments
ETCD_HOST=$HOME/singnet/etcd/example-service/
ETCD_CONTAINER=/opt/singnet/etcd/

# To make your snet's configs persistent between multiple containers
SNET_HOST=$HOME/.snet
SNET_CONTAINER=/root/.snet

docker run \
XXX variaveis
    --name MY_SNET_SERVICE \
    -p $SERVICE_PORT:$SERVICE_PORT \
    -v $ETCD_HOST:$ETCD_CONTAINER \
    -v $SNET_HOST:$SNET_CONTAINER \
    -ti snet_publish_service bash
```

From this point on we follow the tutorial in the Docker container's prompt.

## Step 3. Setup `SNET CLI` and create your identity

XXX
Tab 1: mnemonic key setup

Select a Mnemonic of your choice. MY_MNEMONIC is a string which will be used as seed to generate a public/private key pair. (see XXX here for details)

```
snet identity create $USER_ID mnemonic --mnemonic "MY_MNEMONIC"
```
Tab 2: (optional) other key options  

You can create your identity by using a previously created key.

`SNET CLI` supports theese other identity types:

* key - a text private key
* rpc - JSON-RPC key server
* ledger - hardware wallet
* trezor - hardware wallet

XXX see details on how to use them here (Documentacao da Cli direto na secao apropriada)

## Step 4. Get ETH and AGI

You need some ETH and AGI tokens. You can get them for free using your Github's account here:

* AGI: [https://faucet.singularitynet.io/](https://faucet.singularitynet.io/)
* ETH: [https://faucet.kovan.network/](https://faucet.kovan.network/)

Get the address of your account using ```snet account print``` command.

## Step 5. Create an organization

In order to be able to publish a service you need to be the owner or a member of an organization.

You can create a new organization with:
 
```
snet organization create "$ORGANIZATION_NAME" --org-id $ORGANIZATION_ID
```

In case of an already taken `ORGANIZATION_ID` replace it with a different id of your choice.
Make sure you follow our [naming standardisation guidelines][naming-standards]. If you use a different organization id (other than the one we provided in step 2, update ORGANIZATION_ID properly as it is used later in this tutorial.

```
ORGANIZATION_ID="new-org-id"
```

If you want to join an existing organization (e.g. `snet`), ask the owner to add your public key (account) into it before proceeding.

## Step 6. Download and configure example-service

In this tutorial we'll use a simple service from [SingularityNET Example Service](https://github.com/singnet/example-service).

* Clone the git repository:

```
git clone https://github.com/singnet/example-service.git
cd example-service
```

* Install the dependencies and compile the protobuf file:

```
pip3 install -r requirements.txt
sh buildproto.sh
```

Service is ready to run, but first we need to publish it in SingularityNET and configure the `SNET DAEMON`.

## Step 7. Prepare service metadata to publish the service

First we need to create a service metadata file. You can do it by runing:

```
snet service metadata-init SERVICE_PROTOBUF_DIR SERVICE_DISPLAY_NAME PAYMENT_ADDRESS --endpoints SERVICE_ENDPOINT --fixed-price FIXED_PRICE
```

You need to specify the following parameters:
* `SERVICE_PROTOBUF_DIR` - Directory which contains protobuf files of your service: ```service/service_spec/``` in case of our example service.
* `SERVICE_DISPLAY_NAME` - Display name of your service. You can choose any name you want. 
* `PAYMENT_ADDRESS` - Ethereum account which will receive payments for this service. You should set it to your ethereum account. You can use ```snet account print``` to see your account.
* `SERVICE_ENDPOINT` - Endpoint which will be used to connect to your service.
* `FIXED_PRICE` - Price in AGI for a single call to your service. We will set the price to 1 COG (remember that 1 AGI = 10^8 COGS).

For example:

```
ACCOUNT=`snet account print`
snet service metadata-init service/service_spec/ "$SERVICE_NAME" $ACCOUNT --endpoints http://$SERVICE_IP:$SERVICE_PORT --fixed-price 0.00000001

# Describe your service and add an URL for further service information.
snet service metadata-add-description --json '{"description": "Description of my Service.", "url": "https://service.users.guide"}'
```

This command will create a JSON configuration file: ```service_metadata.json```.

See details of service metadata in [mpe-metadata](https://dev.singularitynet.io/docs/all/mpe/mpe-metadata/).

## Step 8. Publish the service in SingularityNET

Now you can publish your service using the following command (```service_metadata.json``` is used implicitly):

```
snet service publish $ORGANIZATION_ID $SERVICE_ID
```

Check if your service has been properly published:

```
snet organization list-services $ORGANIZATION_ID
```

Optionally you can un-publish the service:

```
snet service delete $ORGANIZATION_ID $SERVICE_ID
```

## Step 9. Run the service (and SNET Daemon)

Create a SNET Daemon configuration file named `snetd.config.json`. 

```
cat > snetd.config.json << EOF
{
   "DAEMON_END_POINT": "$DAEMON_HOST:$DAEMON_PORT",
   "ETHEREUM_JSON_RPC_ENDPOINT": "https://kovan.infura.io",
   "IPFS_END_POINT": "http://ipfs.singularitynet.io:80",
   "REGISTRY_ADDRESS_KEY": "0xe331bf20044a5b24c1a744abc90c1fd711d2c08d",
   "PASSTHROUGH_ENABLED": true,
   "PASSTHROUGH_ENDPOINT": "http://localhost:7003",
   "ORGANIZATION_ID": "$ORGANIZATION_ID",
   "SERVICE_ID": "$SERVICE_ID",
   "PAYMENT_CHANNEL_STORAGE_SERVER": {
       "DATA_DIR": "/opt/singnet/etcd/"
   },
   "LOG": {
       "LEVEL": "debug",
       "OUTPUT": {
          "TYPE": "stdout"
       }
   }
}
EOF
```

Running the service will make a instance of SNET Daemon to be spawned automatically.

```
python3 run_example_service.py
```

At this point your service should be up and running. 

## Step 10. Call your service using `SNET CLI`

Open a new terminal in your host machine and attach the docker container to it using

```docker exec -it MY_SNET_SERVICE bash```

At this point you can use several SNET CLI commands to interact with your account and the Kovan network.
(see XXX for a list of all available commands).

Check your balance and setup a MPE channel to run your service.

```
# check your balance
snet account balance

# deposit funds (10 COG) into MultiPartyEscrow (`MPE`) contract:
snet account deposit 0.00000010

# check your balance - 10 Cogs have been moved to MPE
snet account balance

# open a payment channel to your service:
snet channel open-init $ORGANIZATION_ID $SERVICE_ID 0.0000001 +10days

# check your balance - 10 Cogs have been moved from MPE to the channel
snet account balance

# Look for the channel balance (CHANNEL_ID have been printed by 'snet channel open-init')
snet client get-channel-state CHANNEL_ID $SERVICE_IP:$SERVICE_PORT
```

```snet channel open-init``` opened and initialized a channel with 10 cogs for ORGANIZATION_ID/SERVICE_ID with expiration of 10 days (57600 blocks in the future with 15 sec/blocks). It prints the id of the created channel. Record it for use in the the following commands.

Call your service using:

```
snet client call $ORGANIZATION_ID $SERVICE_ID mul '{"a":12,"b":7}'
```

The MPE channel have changed. See its funds using

```
# 1 Cog have been spent (signed) 
snet client get-channel-state CHANNEL_ID $SERVICE_IP:$SERVICE_PORT
```

At this point you've spent 1 Cog (service cost is defined in step XXX 7) of your MPE channel calling the service. You can keep calling the service until your MPE channel run out of funds.

As the service provider, you can claim spent Cogs at any time using

```
snet treasurer claim-all --endpoint $SERVICE_IP:$SERVICE_PORT -y

# Claimed funds are know in MPE
snet account balance

# Move funds from MPE to your account
snet account withdraw AMOUNT_IN_AGI -y
snet account balance
```

## Step 11. (optional) claiming unused funds from MPE channel

As the service user, _you CAN'T claim unsused funds before the channel expires_. Once it did, you can claim the funds using ```snet treasurer claim-expired```

```
# Shows spent/unspent AGIs in the MPE channel
snet client get-channel-state CHANNEL_ID $SERVICE_IP:$SERVICE_PORT
snet account balance

# Move funds from all expired channels to MPE
snet treasurer claim-expired
snet account balance

# Move funds from MPE to user's account
snet account withdraw AMOUNT_IN_AGI -y
snet account balance
```
