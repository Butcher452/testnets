## Installation Instructions
### for testnet participants

```bash
# Update system and install build tools
cd $HOME
sudo apt update
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils htop -y < "/dev/null"

# 2. Install Go
cd $HOME
wget -O go1.19.1.linux-amd64.tar.gz https://golang.org/dl/go1.19.1.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.19.1.linux-amd64.tar.gz && rm go1.19.1.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version

# 3. Build binaries
cd $HOME
git clone https://github.com/okp4/okp4d
cd okp4d
git checkout v3.0.0
make install
sudo cp $HOME/go/bin/okp4d /usr/local/bin
cd $HOME

# 4. Initialize the node and downloading the genesis file ( come up with your own node name and replace the variable value with your own data $okp4d_NODENAME)
okp4d init $okp4d_NODENAME --chain-id okp4-nemeton-1
wget -O $HOME/.okp4d/config/genesis.json "https://raw.githubusercontent.com/okp4/networks/main/chains/nemeton-1/genesis.json"

# Add peers
peers="a7f1dcf7441761b0e0e1f8c6fdc79d3904c22c01@38.242.150.63:36656,9c462b1c0ba63115bd70c3bd4f2935fcb93721d0@65.21.170.3:42656,2e85c1d08cfca6982c74ef2b67251aa459dd9b2f@65.109.85.170:43656,264256d32511c512a0a9d4098310a057c9999fd1@okp4.sergo.dev:12233,3c805c2dead7b7a3a1d3ba2399d4d62153322413@65.108.2.41:36656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.okp4d/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.okp4d/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.okp4d/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uknow\"/;" ~/.okp4d/config/app.toml

# 5. Set pruning
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.okp4d/config/app.toml

# 6. Create a service
echo "[Unit]
Description=okp4d Node
After=network.target
​
[Service]
User=$USER
Type=simple
ExecStart=$(which okp4d) start
Restart=on-failure
LimitNOFILE=65535
​
[Install]
WantedBy=multi-user.target" > $HOME/okp4d.service
sudo mv $HOME/okp4d.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable okp4d 

# 7. Start service and check the logs
sudo systemctl restart okp4d
sudo journalctl -u okp4d -f -o cat
```
