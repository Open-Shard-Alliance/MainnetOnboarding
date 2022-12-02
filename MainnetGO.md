# NEAR Validator Guide🚀

# Setup using NEARCore

#### Step 1 – Installation required software & set the configuration

* Before you start, you might want to ensure your system is up to date.
```
sudo apt update && sudo apt upgrade -y
```
* Install Python
```
sudo apt install python3 git curl
```
* Install Building env
```
sudo apt install clang build-essential make
```
* Install Rust & Cargo
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
Press 1 and press enter
![img](/images/rust.png)

* Source the environment
```
source $HOME/.cargo/env
```
* Clone the NEARCore Repo
```
git clone https://github.com/nearprotocol/nearcore.git
```
* Set environment to the latest release tag. For the latest release tag, please check here: https://github.com/near/nearcore/releases.  Note: RC tags are for Testnet only.
```
export NEAR_RELEASE_VERSION=1.29.2
```
```
cd nearcore
git checkout $NEAR_RELEASE_VERSION
make release
```
* Install Nodejs and NPM
```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install build-essential nodejs
PATH="$PATH"
```
* Install Near CLI

Once NodeJs and NPM are installed you can now install NEAR-Cli.

Unless you are logged in as root, which is not recommended you will need to use `sudo` to install NEAR-Cli so that the near binary to /usr/local/bin
```
sudo npm install -g near-cli
```
* Launch this command so set the Near Mainnet Environment:
```
export NEAR_ENV=mainnet
```
* You can also run this command to set the Near testnet Environment persistent:
```
echo 'export NEAR_ENV=mainnet' >> ~/.bashrc
```
#### Step 2 – Create a wallet
MainNet: https://wallet.near.org/

#### Step 3 – Authorize Wallet Locally
A full access key needs to be installed locally to be able transactions via NEAR-Cli.

* You need to run this command:
```
near login
```
> Note: This command launches a web browser allowing for the authorization of a full access key to be copied locally.

1 – Copy the link in your browser

![img](/images/1.png)

2 – Grant Access to Near CLI

![img](/images/3.png)

3 – After Grant, you will see a page like this, go back to console

![img](/images/4.png)

4 – Enter your wallet and press Enter

![img](/images/5.png)

#### Step 4 – Initialize & Start the Node
* Download the latest genesis, config files:
```
mkdir ~/.near
cd ~/.near
wget -c https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/mainnet/genesis.json
wget -c https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/mainnet/config.json
```
* Download the latest snapshot from [the snapshot page](https://near-nodes.io/intro/node-data-snapshots).

* Initialize NEAR:
```
target/release/neard init --chain-id="mainnet" --account-id=<staking pool id>
```
##### Create `validator_key.json`
* Generate the Key file:
```
near generate-key <pool_id>
```
* Copy the file generated to Mainnet folder.Make sure to replace YOUR_WALLET by your accountId
```
cp ~/.near-credentials/YOUR_WALLET.json ~/.near/mainnet/validator_key.json
```
* Edit “account_id” => xx.poolv1.near, where xx is your PoolName
* Change “private_key” to “secret_key”
> Note: The account_id must match the staking pool contract name or you will not be able to sign blocks.
> File content must be something like :
> {
>   "account_id": "xx.poolv1.near",
>   "public_key": "ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G",
>   "secret_key": "ed25519:****"
> }
```
near generate-key <your_accountId>
vi ~/.near/{your_accountId}.json
```
Rename "private_key" to "secret_key", then move the file to ~/.near.

```
mv the file to ~/.near
```
Update "account_id" to be the staking_pool_id

> Note: The account_id must match the staking pool contract name or you will not be able to sign blocks.

* Start the Node
```
target/release/neard run
```
* Setup Systemd
Command:

```
sudo vi /etc/systemd/system/neard.service
```
Paste:

```[Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=<USER>
#Group=near
WorkingDirectory=/home/<USER>/.near
ExecStart=/home/<USER>/nearcore/target/release/neard run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

> Note: Change USER to your paths

Command:

```
sudo systemctl enable neard
```
Command:

```
sudo systemctl start neard
```
If you need to make a change to service because of an error in the file. It has to be reloaded:

```
sudo systemctl reload neard
```
##### Watch logs
Command:

```
journalctl -n 100 -f -u neard
```
Make log output in pretty print

Command:

```
sudo apt install ccze
```
View Logs with color

Command:

```
journalctl -n 100 -f -u neard | ccze -A
```
### Becoming a Validator
In order to become a validator and enter the validator set, a minimum set of success criteria must be met.

* The node must be fully synced
* The `validator_key.json` must be in place
* The contract must be initialized with the public_key in `validator_key.json`
* The account_id must be set to the staking pool contract id
* There must be enough delegations to meet the minimum seat price. See the seat price [here](https://explorer.mainnet.near.org/nodes/validators).
* A proposal must be submitted by pinging the contract
* Once a proposal is accepted a validator must wait 2-3 epoch to enter the validator set
* Once in the validator set the validator must produce great than 90% of assigned blocks

Check running status of validator node. If “Validator” is showing up, your pool is selected in the current validators list.

### Submitting Pool Information
Adding pool information helps delegators and also helps with outreach for upgrades and other important announcements: https://github.com/zavodil/near-pool-details.
The available fields to add are: https://github.com/zavodil/near-pool-details/blob/master/FIELDS.md.

The identifying information that we ask the validators:to provide are:
- Name
- Description
- URL
- Country and country code
- Email (for support)
- Telegram, Discord, or Twitter

Command:

```
near call name.near update_field '{"pool_id": "<pool_id>.poolv1.near", "name": "url", "value": "https://yoururl.com"}' --accountId=<accountId>.near  --gas=200000000000000
```
```
near call name.near update_field '{"pool_id": "<pool_id>.poolv1.near", "name": "twitter", "value": "<twitter>"}' --accountId=<account id>.near  --gas=200000000000000
```
```
near view name.near get_all_fields '{"from_index": 0, "limit": 3}'
```
```
near view name.near get_fields_by_pool '{"pool_id": "<pool_id>.poolv1.near"}'
```
---

## STAKING POOLS
NEAR uses a staking pool factory with a whitelisted staking contract to ensure delegators’ funds are safe. In order to run a validator on NEAR, a staking pool must be deployed to a NEAR account and integrated into a NEAR validator node. Delegators must use a UI or the command line to stake to the pool. A staking pool is a smart contract that is deployed to a NEAR account.

### Deploy a Staking Pool Contract
#### Deploy a Staking Pool
Calls the staking pool factory, creates a new staking pool with the specified name, and deploys it to the indicated accountId.

For Mainnet
```
near call poolv1.near create_staking_pool '{"staking_pool_id": "<pool id>", "owner_id": "<accountId>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}}' --accountId="<accountId>" --amount=30 --gas=300000000000000
```

From the example above, you need to replace:

* **Pool ID**: Staking pool name, the factory automatically adds its name to this parameter, creating {pool_id}.{staking_pool_factory}
Examples:   

- `XXX.poolv1.near` for mainnet

* **Owner ID**: The NEAR account that will manage the staking pool. Usually your main NEAR account.
* **Public Key**: The public key in your **validator_key.json** file.
* **5**: The fee the pool will charge (e.g. in this case 5 over 100 is 5% of fees).
* **Account Id**: The NEAR account deploying the staking pool.

> Be sure to have at least 30 NEAR available, it is the minimum required for storage.

To change the pool parameters, such as changing the amount of commission charged to 1% in the example below, use this command:
```
near call <pool_name> update_reward_fee_fraction '{"reward_fee_fraction": {"numerator": 1, "denominator": 100}}' --accountId <account_id> --gas=300000000000000
```


You will see something like this:

![img](/images/pool.png)

If there is a “True” at the End. Your pool is created.

**You have now configure your Staking pool.**


#### Configure your staking pool contract
Replace:

* Pool Name – pool_id.staking_pool_factory
* Owner Id
* Public Key
* Reward Fraction
* Account Id
```
near call <pool name> new '{"owner_id": "<owner id>", "<public key>": "<public key>", "reward_fee_fraction": {"numerator": <reward fraction>, "denominator": 100}}' --accountId <accountId>
```
#### Manage your staking pool contract
> HINT: Copy/Paste everything after this line into a text editor and use search and replace. Once your pool is deployed, you can issue the commands below:


##### Retrieve the owner ID of the staking pool

Command:

```
near view {pool_id}.{staking_pool_factory} get_owner_id '{}'
```
##### Issue this command to retrieve the public key the network has for your validator
Command:

```
near view {pool_id}.{staking_pool_factory} get_staking_key '{}'
```
##### If the public key does not match you can update the staking key like this (replace the pubkey below with the key in your validator.json file)

```
near call {pool_id}.{staking_pool_factory} update_staking_key '{"stake_public_key": "<public key>"}' --accountId <accountId>
```

### Working with Staking Pools
> NOTE: Your validator must be fully synced before issuing a proposal or depositing funds.

### Proposals

In order to get a validator seat you must first submit a proposal with an appropriate amount of stake. Proposals are sent for epoch +2. Meaning if you send a proposal now, if approved, you would get the seat in 3 epochs. You should submit a proposal every epoch to ensure your seat. To send a proposal we use the ping command. A proposal is also sent if a stake or unstake command is sent to the staking pool contract.

To note, a ping also updates the staking balances for your delegators. A ping should be issued each epoch to keep reported rewards current on the pool contract. You could set up a ping using a cron job or use [Cron Cat](https://cron.cat/).
