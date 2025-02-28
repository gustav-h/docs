# NuLink Worker Initialization and Running  

If install the Worker node via Docker, please initialize and run it using Docker as well.  And if install the Worker node via local installation, please read local operation part.  

**Make sure putting a small amount of BNB(test) to the worker account for sending one confirmation transaction.**


## Run Node via Docker (Recommended)  

There are there steps to complete starting a Worker node via Docker:
1. **Export Node Environment Variables**
2. **Initialize Node Configuration**
3. **Launch the Node**

### Export Node Environment Variables  
These environment variables are used to better simplify the Docker installation process.
```shell
# Password used for lock/unlock the private storage generated by NuLink Worker.You can choose some characters(8 minimum) as the password here. Do not forget it!!!
$ export NULINK_KEYSTORE_PASSWORD=<YOUR NULINK STOREAGE PASSWORD>

# Password used to unlock the keystore file of Worker account, you should have set it up when you create the Worker account using Geth or other sources. Make sure you enter the same one!!!
$ export NULINK_OPERATOR_ETH_PASSWORD=<YOUR WORKER ACCOUNT PASSWORD>
```

### Initialize Node Configuration  
This step creates and stores the NuLink worker node configuration, and only needs to be run once.

```shell
$ docker run -it --rm \
-p 9151:9151 \
-v </path/to/host/machine/directory>:/code \
-v </path/to/host/machine/directory>:/home/circleci/.local/share/nulink \
-e NULINK_KEYSTORE_PASSWORD \
nulink/nulink nulink ursula init \
--signer <ETH KEYSTORE URI> \
--eth-provider <NULINK PROVIDER URI>  \
--network <NULINK NETWORK NAME> \
--payment-provider <PAYMENT PROVIDER URI> \
--payment-network <PAYMENT NETWORK NAME> \
--operator-address <WORKER ADDRESS> \
--max-gas-price <GWEI>
```
Replace the following values with your own value:
- `</path/to/host/machine/directory>` - The host directory you create when install.
- `<ETH KEYSTORE URI>` - The path to the keystore file of the Worker account.
- `<NULINK PROVIDER URI>` - The URI of a local or hosted node where the Horus network launched.
- `<NULINK NETWORK NAME>` - The name of the network where  the Horus network launched. 
- `<PAYMENT PROVIDER URI>` - The URI of a local or hosted node where payment goes.
- `<PAYMENT NETWORK NAME>` - The name of the payment network. 
- `<OPERATOR ADDRESS>` - The address of the Worker account. [How to generate Worker account](./eth_account.md).
- `<GWEI>` (Optional) - The maximum price of gas to spend on any transaction.

Example Input:

```shell
$ docker run -it --rm \
-p 9151:9151 \
-v /root/nulink:/code \
-v /root/nulink:/home/circleci/.local/share/nulink \
-e NULINK_KEYSTORE_PASSWORD \
nulink/nulink nulink ursula init \
--signer keystore:///code/UTC--2022-09-13T01-14-32.465358210Z--8b1819341bec211a45a2186c4d0030681ccce0ee \
--eth-provider https://data-seed-prebsc-2-s2.binance.org:8545 \
--network horus \
--payment-provider https://data-seed-prebsc-2-s2.binance.org:8545 \
--payment-network bsc_testnet \
--operator-address 0x8B1819341BEc211a45a2186C4D0030681cccE0Ee \
--max-gas-price 100
```
Example Output:

```shell
# step 1
 Detected IPv4 address (8.219.186.125) - Is this the public-facing address of Ursula? [y/N]: y
 
 Please provide a password to lock Operator keys.
 Do not forget this password, and ideally store it using a password manager.
 
 # step 2
 Enter nulink keystore password (8 character minimum): xxxxxx
 Repeat for confirmation: xxxxxx
 
 Backup your seed words, you will not be able to view them again.
 
 xxxxxxxxxxxxxxxxxxxxxxxx
 
 # step 3
 Have you backed up your seed phrase? [y/N]: y
 
 # step 4
 Confirm seed words: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
 
 
Public Key:   02bb2067d21a677ce928967c0ece79a9
Path to Keystore: /home/circleci/.local/share/nulink/keystore

- You can share your public key with anyone. Others need it to interact with you.
- Never share secret keys with anyone! 
- Backup your keystore! Character keys are required to interact with the protocol!
- Remember your password! Without the password, it's impossible to decrypt the key!


Generated configuration file at default filepath /home/circleci/.local/share/nulink/ursula.json

* Review configuration  -> nulink ursula config
* Start working         -> nulink ursula run

```

### Launch the Node  
The following command will start the node. Make sure you use the same host directory as the configuration. 

**Remark1: You need to [claim](https://testnet.binance.org/faucet-smart) some BNB(test) token for Worker account as gas fee.**

**Remark2: If you encounter error when starting Worker node,  first please check that the port 9151 has not been occupied by other process. If still not working, please check there is only one configuration json file in the </path/to/host/machine/directory>**

```shell
$ docker run --restart on-failure -d \
--name ursula \
-p 9151:9151 \
-v </path/to/host/machine/directory>:/code \
-v </path/to/host/machine/directory>:/home/circleci/.local/share/nulink \
-e NULINK_KEYSTORE_PASSWORD \
-e NULINK_OPERATOR_ETH_PASSWORD \
nulink/nulink nulink ursula run --no-block-until-ready
```

Example Input：
```shell
$ docker run --restart on-failure -d \
--name ursula \
-p 9151:9151 \
-v /root/nulink:/code \
-v /root/nulink:/home/circleci/.local/share/nulink \
-e NULINK_KEYSTORE_PASSWORD \
-e NULINK_OPERATOR_ETH_PASSWORD \
nulink/nulink nulink ursula run --no-block-until-ready
```
Example Output:

```shell
aa3a0f6376b566473cbcde46b0e772feb4d3658188d2cbb424a1e94588d6d8eb
```




### Check Node URI for Worker Account
The following command describes how to view worker addresses.

```shell
    docker logs -f <docker name>
```
Example Input:
```shell
    $ docker logs -f ursula
```
Example Output:
```shell
Authenticating Ursula
Loaded Ursula (horus)
✓ External IP matches configuration
Starting services
✓ Node Discovery (Horus)
✓ Work Tracking
✓ Start Operator Bonded Tracker
✓ Rest Server https://8.219.186.125:9151
Working ~ Keep Ursula Online!

```

Now the Node URI(e.g: https://8.219.186.125:9151) and Worker address(e.g: 0x8B1819341BEc211a45a2186C4D0030681cccE0Ee)  are ready for bonding operation.

## Run Node via Local Operation  

There are two steps to complete starting a Worker node via local operation:
1. **Initialize Node Configuration**
2. **Launch the Node**

### Initialize Node Configuration

This step creates and stores the worker node configuration, and only needs to be run once.

```shell
nulink ursula init      \
--signer <ETH KEYSTORE URI>           \
--network <NULINK NETWORK NAME>           \
--eth-provider <NULINK PROVIDER URI>      \
--payment-provider <PAYMENT PROVIDER URI>  \
--payment-network <PAYMENT NETWORK NAME>   \
--operator-address <OPERATOR ADDRESS> \
--max-gas-price <GWEI>
```


- `<ETH KEYSTORE URI>` - The path to the keystore file of the Worker account..
- `<NULINK PROVIDER URI>` - The URI of a local or hosted node where the Horus network launched.
- `<NULINK NETWORK NAME>` - The name of the network where the Horus network launched. 
- `<PAYMENT PROVIDER URI>` - The URI of a local or hosted node where payment goes.
- `<PAYMENT NETWORK NAME>` - The name of the payment network. 
- `<OPERATOR ADDRESS>` - The address of the Worker account. [How to generate Worker account](./eth_account.md)
- `<GWEI>` (Optional) - The maximum price of gas to spend on any transaction.

Example Input:

```shell
nulink ursula init \
--signer keystore:///root/nulink/keystore \
--eth-provider https://data-seed-prebsc-2-s2.binance.org:8545 \
--network horus \
--payment-provider https://data-seed-prebsc-2-s2.binance.org:8545 \
--payment-network bsc_testnet \
--operator-address 0x8a20d379A4C08c482a617A81a39EB426B6EB8642 \
--max-gas-price 2000
```

Example Output:

```shell
# step 1
Detected IPv4 address (8.219.188.70) - Is this the public-facing address of Worker? [y/N]: y

Please provide a password to lock Operator keys.
Do not forget this password, and ideally store it using a password manager.

# step 2
Enter nulink keystore password (8 character minimum): xxxxxx
Repeat for confirmation: xxxxxx

Backup your seed words, you will not be able to view them again.

hammer fatal jazz era hurt shoulder stand story find move earn  much actor animal stamp know vital odor coin electric torch quick siege tonight

# step 3
Have you backed up your seed phrase? [y/N]: y

# step 4
Confirm seed words: hammer fatal jazz era hurt shoulder stand story find move earn  much actor animal stamp know vital odor coin electric torch quick siege tonight


Generated keystore


Public Key:   02dacea4c7f5563004af37f282ca10f7
Path to Keystore: /root/.local/share/nulink/keystore

- You can share your public key with anyone. Others need it to interact with you.
- Never share secret keys with anyone! 
- Backup your keystore! Character keys are required to interact with the protocol!
- Remember your password! Without the password, it's impossible to decrypt the key!


Generated configuration file at non-default filepath /root/.local/share/nulink/ursula-02dacea4.json
* NOTE: for a non-default configuration filepath use `--config-file "/root/.local/share/nulink/ursula-02dacea4.json"` with subsequent `ursula` CLI commands

* Review configuration  -> nulink ursula config
* Start working         -> nulink ursula run
```

### Launch the Node

Run worker node using the initialized configuration.

```shell
screen -S nulink-worker // use this command if you want to run the worker node in a screen session

nulink ursula run --no-block-until-ready
```

Enter the above startup command and press enter to prompt for the passwords of ETH account and NuLink keystore, which are set in the initialization phase.

Note: **_operator account needs to have tokens on the corresponding chain_**


Example Input:

```shell
nulink ursula run --no-block-until-ready
```

Example Output:

```shell
Enter ethereum account password (0x7bD7B1266868B34dA4929501FfEA4ac737dA0E93):
Enter nulink keystore password:
Authenticating Ursula
Loaded Ursula (horus)
✓ External IP matches configuration
Starting services
✓ Node Discovery (Horus)
✓ Operator 0x7bD7B1266868B34dA4929501FfEA4ac737dA0E93 is funded with 0.499959405 ETH
✓ Operator 0x7bD7B1266868B34dA4929501FfEA4ac737dA0E93 is bonded to staking provider 0xf3D6ad89E34b1Cf8325EA614fa901eA4F34Be14a
✓ Operator already confirmed.  Not starting worktracker.
✓ Start Operator Bonded Tracker
✓ Rest Server https://8.219.188.70:9157
Working ~ Keep Ursula Online!
```

Now the Worker address(e.g: 0x7bD7B1266868B34dA4929501FfEA4ac737dA0E93) and Node URI(e.g: https://8.219.188.70:9157)  are ready for bonding operation.



Remark: If you use a LAN address in the first step, please use --no-ip-checkup. If you start the node without bonding, use --no-block-until-ready

