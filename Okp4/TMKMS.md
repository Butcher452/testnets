### Install TMKMS for OKP4
#### It is assumed that you will have two servers. The first a node synchronized with the network and a second server with tmkms  installed and configured.

```bash
# 1. Start by opening the node you intend to run TMKMS (not the node you validate on) and install the following dependencies:
sudo apt update
sudo apt install git build-essential ufw curl jq snapd libusb-1.0-0-dev -y

# 2. install Cargo ( choose 1 option ) 
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

# 3. In this example, we will be compiling from the github source code using the --features=softsign  flag, however you may use --features=yubihsm if you want to use a yubikey. ( if you want to compile a specific version, you should specify the value of the option --version=......, for example --version=0.12.2 )
cd $HOME
git clone https://github.com/iqlusioninc/tmkms.git
cd $HOME/tmkms
cargo install tmkms --features=softsign --version=0.12.2

# 4. The tmkms init command can be used to generate a directory containing the configuration files needed to run the KMS. Run the following:
tmkms init $HOME/tmkms_okp4

# 5. Now we will transfer your priv_validator_key.json from your validator to your VM running TMKMS. Then, import the private validator key into tmkms:
tmkms softsign import $HOME/priv_validator_key.json  $HOME/tmkms_okp4/secrets/priv_validator_key
Please note at this point, you could delete the priv_validator_key.json from both your validator node and tmkms node and store it safely offline in case of an emergency. This newly created priv_validator_key will be what TMKMS will use to sign for your validator.

# 6. Now, modify the tmkms.toml file
nano $HOME/tmkms_okp4/tmkms.toml

# 7. This is just an example. If you use tmkms on other networks, you should substitute your values. ( chain id , account_key_prefix , state_file, path , addr , secret_key , protocol_version !!! )
# Tendermint KMS configuration file
​
## Chain Configuration
​
### Cosmos Hub Network
​
[[chain]]
id = "okp4-nemeton-1"
key_format = { type = "bech32", account_key_prefix = "okp4pub", consensus_key_prefix = "okp4valconspub" }
state_file = "/home/okp4/tmkms_okp4/state/priv_validator_state.json"
​
## Signing Provider Configuration
​
### Software-based Signer Configuration
​
[[providers.softsign]]
chain_ids = ["okp4-nemeton-1"]
key_type = "consensus"
path = "/home/okp4/tmkms_okp4/secrets/priv_validator_key"
​
## Validator Configuration
​
[[validator]]
chain_id = "okp4-nemeton-1"
addr = "tcp://xx.xx.xx.xx:26659"
secret_key = "/home/okp4/tmkms_okp4/secrets/kms-identity.key"
protocol_version = "v0.34"
reconnect = true
check the Tendermint version that is running. ( protocol_version )
okp4d tendermint version
9. Now, modify your validators config.toml to use the port you selected in the tmkms.toml file
nano $HOME/.okp4d/config/config.toml
priv_validator_laddr = "tcp://0.0.0.0:26659"
It is also recommended to comment out the priv_validator_key_file line and the priv_validator_state_file line:
# Path to the JSON file containing the private key to use as a validator in the consensus protocol
# priv_validator_key_file = "config/priv_validator_key.json"
​
# Path to the JSON file containing the last sign state of a validator
# priv_validator_state_file = "data/priv_validator_state.json"
10. Next, stop the validator. Move back to your VM running TMKMS and start it:
tmkms start -c $HOME/tmkms_okp4/tmkms.toml
You will see error logs like the following:
2022-12-31T00:36:09.335756Z  INFO tmkms::commands::start: tmkms 0.12.1 starting up...
2022-12-31T00:36:09.335887Z  INFO tmkms::keyring: [keyring:softsign] added consensus Ed25519 key: okp4valconspub1zcjduepqutdquajscs4696rvl0x9ntes585ywjt7xwx3t2udma5xtfjywfqq903wfp
2022-12-31T00:36:09.336079Z  INFO tmkms::connection::tcp: KMS node ID: f7c81f67b322bb902c1f55d4a8c59706ca0c52f4
2022-12-31T00:36:09.336374Z ERROR tmkms::client: [okp4-nemeton-1@tcp://89.163.151.253:26659] I/O error: Connection refused (os error 111)
2022-12-31T00:36:10.336542Z  INFO tmkms::connection::tcp: KMS node ID: f7c81f67b322bb902c1f55d4a8c59706ca0c52f4
2022-12-31T00:36:10.336711Z ERROR tmkms::client: [okp4-nemeton-1@tcp://89.163.151.253:26659] I/O error: Connection refused (os error 111)
2022-12-31T00:36:11.336884Z  INFO tmkms::connection::tcp: KMS node ID: f7c81f67b322bb902c1f55d4a8c59706ca0c52f4
2022-12-31T00:36:11.337051Z ERROR tmkms::client: [okp4-nemeton-1@tcp://89.163.151.253:26659] I/O error: Connection refused (os error 111)
2022-12-31T00:36:12.337271Z  INFO tmkms::connection::tcp: KMS node ID: f7c81f67b322bb902c1f55d4a8c59706ca0c52f4
2022-12-31T00:36:12.337526Z ERROR tmkms::client: [okp4-nemeton-1@tcp://89.163.151.253:26659] I/O error: Connection refused (os error 111)
2022-12-31T00:36:13.337727Z  INFO tmkms::connection::tcp: KMS node ID: f7c81f67b322bb902c1f55d4a8c59706ca0c52f4
2022-12-31T00:36:13.337900Z ERROR tmkms::client: [okp4-nemeton-1@tcp://89.163.151.253:26659] I/O error: Connection refused (os error 111)
11. Restart the OK 4 node so that the binary re-reads the configuration changes made above.
sudo systemctl restart okp4d
Now tmkms logs should look like this. Tmkms established connections, waited for synchronization to complete, and started signing blocks.
2022-12-31T00:38:49.853892Z  INFO tmkms::connection::tcp: KMS node ID: f7c81f67b322bb902c1f55d4a8c59706ca0c52f4
2022-12-31T00:38:49.854045Z ERROR tmkms::client: [okp4-nemeton-1@tcp://89.163.151.253:26659] I/O error: Connection refused (os error 111)
2022-12-31T00:38:50.854222Z  INFO tmkms::connection::tcp: KMS node ID: f7c81f67b322bb902c1f55d4a8c59706ca0c52f4
2022-12-31T00:38:50.855411Z  INFO tmkms::session: [okp4-nemeton-1@tcp://89.163.151.253:26659] connected to validator successfully
2022-12-31T00:38:50.855455Z  WARN tmkms::session: [okp4-nemeton-1@tcp://89.163.151.253:26659]: unverified validator peer ID! (4a6958e1211ee64d4a1d380a2a267867c8f1c6cb)
2022-12-31T00:39:13.266634Z  INFO tmkms::session: [okp4-nemeton-1@tcp://89.163.151.253:26659] signed PreCommit:<nil> at h/r/s 242376/0/2 (0 ms)
2022-12-31T00:39:13.597237Z  INFO tmkms::session: [okp4-nemeton-1@tcp://89.163.151.253:26659] signed PreCommit:<nil> at h/r/s 242377/0/2 (0 ms)
2022-12-31T00:39:14.066509Z  INFO tmkms::session: [okp4-nemeton-1@tcp://89.163.151.253:26659] signed PreCommit:<nil> at h/r/s 242378/0/2 (0 ms)
2022-12-31T00:39:17.530149Z  INFO tmkms::session: [okp4-nemeton-1@tcp://89.163.151.253:26659] signed PreVote:6BC6B77784 at h/r/s 242379/0/1 (0 ms)
2022-12-31T00:39:17.803117Z  INFO tmkms::session: [okp4-nemeton-1@tcp://89.163.151.253:26659] signed PreCommit:6BC6B77784 at h/r/s 242379/0/2 (0 ms)
2022-12-31T00:39:23.259515Z  INFO tmkms::session: [okp4-nemeton-1@tcp://89.163.151.253:26659] signed PreVote:124CE9752D at h/r/s 242380/0/1 (0 ms)
2022-12-31T00:39:23.521055Z  INFO tmkms::session: [okp4-nemeton-1@tcp://89.163.151.253:26659] signed PreCommit:124CE9752D at h/r/s 242380/0/2 (0 ms)
It remains to create a service for the process tmkms 
sudo tee <<EOF >/dev/null /etc/systemd/system/tmkms.service
[Unit]
Description=tmkms okp4 service
After=network.target
StartLimitIntervalSec=0
[Service]
Type=simple
Restart=always
RestartSec=10
User=$USER
ExecStart=$(which tmkms ) start -c $HOME/tmkms_okp4/tmkms.toml
LimitNOFILE=1024
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable tmkms
sudo systemctl restart tmkms
journalctl -u tmkms -f --no-hostname -o cat
```
