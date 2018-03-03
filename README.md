# Setup a Masternode with Ubuntu 16.04 VPS

In this guide you will learn how to prepare a TIME Masternode in Ubuntu version 16.04 VPS. Before you start, you need to meet the initial requirements.

This guide was done with the following environment:
* Local: Windows 10 64 bit.
* Remote: Ubuntu 16.04 64 bit.
* TIME Version: 1.0.0.

## Initial requirements

### Virtual Private Server (VPS)

A Masternode must be hosted on a [VPS](https://en.wikipedia.org/wiki/Virtual_private_server). Get a VPS from a provider like [DigitalOcean](https://www.digitalocean.com/), [Vultr](https://www.vultr.com/), [Linode](https://www.linode.com/), etc.

The recommended requirements for the master node would be the following:
* Operating system (OS): Linux.
* Linux Distribution: Ubuntu.
* Version: 16.04 (Xenial) 64 bit.
* Memory RAM: At least 1 GB RAM.
* Disk Size: At least 30 GB.

### TIME Transaction

In order to have a TIME masternode you must have a transaction of 10,000 TIME in your desktop Cold Wallet.

To carry this out you need to open your TIME Core wallet.

Now you will have to go to the "File" tab and select "Receiving Addresses...".

Click on the "New" button and write a label for your new address (for example, "mn01").

Right click on the address you just created and click on "Copy Address" and click on the "Close" button.

Now that you have the address created, you will send 10,000 TIME to your address. With this, you will have a transaction of 10,000 TIME in your wallet.

Click on the "Send" tab and on "Pay To", enter the address you just created. You will see that in "Label" the name that you put before will appear.

Then you must enter the value of "10000" in "Amount" and you must NOT select "Substract fee from amount". Finally, you must click on the "Send" button.

In the "Overview" tab, a "Payment to yourself" will appear. This transaction must be confirmed by the network, so you will have to wait for it to be confirmed (it may take five minutes of waiting depending on the state of the network and the commission chosen).

## Cold Wallet Setup Part 1

### Enabling TIME Core wallet advanced options

Open your TIME Core wallet on your desktop and now click on "Settings" tab, press on "Options..." and select "Wallet" tab.

The options that you have to select are the following:
* "Enable coin control features".
* "Show Masternodes Tab".

Press "OK" button. In order to enable the new options, you have to restart TIME Core wallet.

### Creating Masternode private key

Open your TIME Core wallet on your desktop and now click on "Tools" tab and press on "Debug console".

Now you have to enter the following command:

```
masternode genkey
```

You should see a long key that looks like:
```
3HaYBVUCYjEMeeH1Y4sBGLALQZE1Yc1K64xiqgX37tGBDQL8Xg
```

This is your Masternode `private key`. Keep it safe and do not share with anyone.

## VPS Setup

### VPS Connection

Depending on your local machine, you will need to use different steps:
* Windows users: You have to connect using PuTTY and it is recommended to [follow this guide](https://www.digitalocean.com/community/tutorials/how-to-log-into-your-droplet-with-putty-for-windows-users).
* Linux users: You have to connect using ssh.

### Masternode Installation

Copy and paste these commands into your VPS and hit enter. Remember that one box at a time:

```
apt-get -y update
```

```
apt-get -y upgrade
```

```
apt-get -y install software-properties-common
```

```
apt-add-repository -y ppa:bitcoin/bitcoin
```

```
apt-get -y update
```

```
apt-get -y install \
    wget \
    git \
    unzip \
    libevent-dev \
    libboost-dev \
    libboost-chrono-dev \
    libboost-filesystem-dev \
    libboost-program-options-dev \
    libboost-system-dev \
    libboost-test-dev \
    libboost-thread-dev \
    libminiupnpc-dev \
    build-essential \
    libtool \
    autotools-dev \
    automake \
    pkg-config \
    libssl-dev \
    libevent-dev \
    bsdmainutils \
    libzmq3-dev \
    nano
```

```
apt-get -y update
```

```
apt-get -y upgrade
```

```
apt-get -y install libdb4.8-dev
```

```
apt-get -y install libdb4.8++-dev
```

```
wget https://github.com/time-coin/timecoin.io/releases/download/1/timed
```

```
wget https://github.com/time-coin/timecoin.io/releases/download/1/time-cli
```

```
chmod +x ./timed
```

```
chmod +x ./time-cli
```

```
mkdir -p .timecore
```

```
nano .timecore/time.conf
```

Replace:

externalip=VPS_IP_ADDRESS
masternodeprivkey=WALLET_GENKEY

With your info!

```
rpcuser=YOUR_USERNAME_HERE
rpcpassword=GENERATED_PASSWORD_HERE
rpcallowip=127.0.0.1
listen=1
server=1
daemon=1
maxconnections=24
externalip=VPS_IP_ADDRESS
masternodeprivkey=WALLET_GENKEY
masternode=1

```

CTRL X to save it. Y for yes, then ENTER.

```
./timed &
```

We also want the wallet to start on boot, so you will want to add a cron to do this
```
crontab -e
```
```
@reboot /root/timed
```
CTRL X to save it. Y for yes, then ENTER.


Some hosts may have firewall blocks by default. The below will open up the required port
```
ufw allow 30000/tcp
```

```
apt-get -y install virtualenv python-pip
```

```
git clone https://github.com/time-coin/sentinel /sentinel
```

```
cd /sentinel
```

You now need to edit your sentinel.conf to look for your time.conf. 

```
nano sentinel.conf
````

Add the below to sentinel.conf
```
time_conf=/root/.timecore/time.conf
```
CTRL X to save it. Y for yes, then ENTER.


```
mkdir database
```

```
virtualenv venv
```

```
. venv/bin/activate
```

```
pip install -r requirements.txt
```

```
crontab -e
```

Hit 2. This will bring up an editor. Paste the following in it at the bottom.

```
* * * * * cd /sentinel && ./venv/bin/python bin/sentinel.py >/dev/null 2>&1
```

CTRL X to save it. Y for yes, then ENTER.

Use `watch time-cli getinfo` to check and wait until it's synced (look for blocks number and compare with a [block explorer](https://explorer.timec.io/)).

## Cold Wallet Setup Part 2 

1. On your local machine open your `masternode.conf` file.
   Depending on your operating system you will find it in:
   * Windows: `%APPDATA%\TIMECoinCore\`
   * Mac OS: `~/Library/Application Support/TIMECoinCore/`
   * Unix/Linux: `~/.timecore/`
   
   Leave the file open
2. Go to "Tools" -> "Debug console"
3. Run the following command: `masternode outputs`
4. You should see output like the following if you have a transaction with exactly 10,000 TIME:
```
{
    "12345678xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx": "0"
}
```
5. The value on the left is your `txid` and the right is the `vout`.
6. Add a line to the bottom of the already opened `masternode.conf` file using the IP of your VPS (with port 30000), `private key`, `txid` and `vout`:
```
mn1 1.2.3.4:24126 3xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx 12345678xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx 0 
```
7. Save the file, exit your wallet and reopen your wallet.
8. Go to the "Masternodes" tab.
9. Click "Start All".
10. You will see "WATCHDOG_EXPIRED". Just wait few minutes.

Congratulations! You have just owned a TIME Master Node. A master node helps improve the network and provide the network with certain anonymous and fast payments functions, also including governance functions.
