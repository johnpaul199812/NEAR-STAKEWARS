# NEAR-STAKEWARS
#How to set up a Shardnet Node for NEAR STAKEWARS

#Get a vps server at digitalocean with this link https://m.do.co/c/3d3f5749db15, 100$ in free credits to buy server 
![image](https://user-images.githubusercontent.com/42779023/180081719-6e4b4627-a6db-4059-9aeb-686aa88e0931.png)

#once yours vps is ready,you can ssh login with the droplet console
![image](https://user-images.githubusercontent.com/42779023/180082131-92930516-b38e-403e-bef2-bb65fcbd9ecd.png)
or you can use mobaxterm to ssh login, download mobaxterm here https://mobaxterm.mobatek.net/download.html

#now register with this form for participating in the testnet https://nearprotocol1001.typeform.com/to/Z39N7cU9

#Challenge 001#

#make sure the linux machine is up-to-date

sudo apt update && sudo apt upgrade -y

#install Node JS and NPM

curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash - 

sudo apt install build-essential nodejs 

PATH="$PATH"

#install NEAR-CLI

npm install -g near-cli

#Set up the environment

echo 'export NEAR_ENV=shardnet' >> ~/.bashrc

#Challenge 002#

#Install developer tools

sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo

#Install Python pip

sudo apt install python3-pip

#Set the configuration

USER_BASE_BIN=$(python3 -m site --user-base)/bin

export PATH="$USER_BASE_BIN:$PATH"

#Install Building env

sudo apt install clang build-essential make

#Install cargo and rust

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

#Now download and build binary

git clone https://github.com/near/nearcore 

cd nearcore 

git fetch 

git checkout 8448ad1ebf27731a43397686103aa5277e7f2fcf

cargo build -p neard --release --features shardnet

#check the version

~/nearcore/target/release/neard --version

#now you have to initialize the working directory, delete old files and download new config file and new genesis.json

~/nearcore/target/release/neard --home ~/.near init --chain-id shardnet --download-genesis 

rm ~/.near/config.json ~/.near/genesis.json

wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json 

wget -O ~/.near/genesis.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json

#Run the node

cd ~/nearcore

./target/release/neard --home ~/.near run

![image](https://user-images.githubusercontent.com/42779023/180088266-d6da46d1-1a73-4d27-822c-46ad5a892abe.png)

#you can see log outputs in your console. Your node should be able to find peers, download headers to 100%, and then download blocks.

#Activating the node as validator

#Now you will need to create a wallet https://wallet.shardnet.near.org/

#Authorize Wallet Locally

near login

![image](https://user-images.githubusercontent.com/42779023/180089139-d3501b3c-a2af-45bb-a955-62ad964f590f.png)

#Grant Access to Near CLI

![image](https://user-images.githubusercontent.com/42779023/180089741-2763afb9-17d9-4f7c-8325-29ef03d70b94.png)

![image](https://user-images.githubusercontent.com/42779023/180089798-784e5cc6-9453-4266-af25-cd87b8bd9cfd.png)

#paste your account id,e.g johnpaulnodes.shardnet.near

![image](https://user-images.githubusercontent.com/42779023/180089978-0628c944-ce62-44e6-93ed-f67770590571.png)
#you will see this, its fine

![image](https://user-images.githubusercontent.com/42779023/180090039-e0d9fbd3-bc94-4a9e-954a-b1522534d096.png)

#now paste your account id in the Cli and press ENTER

![image](https://user-images.githubusercontent.com/42779023/180090232-a702ed02-32b2-4fbc-9d9a-d75256a2660b.png)

![image](https://user-images.githubusercontent.com/42779023/180090313-304b526a-d972-43a5-9e3a-5421a28f7233.png)

#Copy your wallet json and make some changes

cd ~/.near-credentials/shardnet/

cp <wallet.json> ~/.near/validator_key.json

#NOTE:<wallet.json> ---> is your wallet name file e.g in mine johnpaulnodes.shardnet.near.json

#you should be able to open this file,copy the content

#now do this

nano validator_key.json

#then paste the content here, then edit the account ID and Private Key parameter accordingly

#should look like this

![image](https://user-images.githubusercontent.com/42779023/180093154-76819a03-6774-4ab1-8a2e-dc2a1ca52ba5.png)

#File content must be in the following pattern

{
"account_id":"johnpaulnodes.factory.shardnet.near",
"public_key":"ed25519:72wzgPZgDx2SgE7Nffd6xfXSe4RNv57JE2ZXZ8xGH",
"secret_key":"ed25519:55Xbx9zd7pbjDjmmWhMav7ozmreJWyYyiKBVFboG8UjCegUQkxH8bwQkeEuJHxD"
}

#then exit with CTRL+O press Enter, then CRTL+X press 

#Create a service file 

sudo tee /etc/systemd/system/neard.service > /dev/null <<EOF
                                                             
[Unit] 
                                                             
Description=NEARd Daemon Service 
                                                             
[Service] 
                                                             
Type=simple 
                                                             
User=$USER #Group=near 
                                                             
WorkingDirectory=$HOME/.near 
                                                             
ExecStart=$HOME/nearcore/target/release/neard run 
                                                             
Restart=on-failure 
                                                             
RestartSec=30 
                                                             
KillSignal=SIGINT 
                                                             
TimeoutStopSec=45 
                                                             
KillMode=mixed 
                                                             
[Install] 
                                                             
WantedBy=multi-user.target 
                                                             
EOF

#start the service and see logs
                                                             
sudo systemctl daemon-reload 
                                                             
sudo systemctl enable neard 
                                                             
sudo systemctl start neard
                                                             
journalctl -u neard -f

![image](https://user-images.githubusercontent.com/42779023/180093925-a6573675-a9c0-4bdd-bffb-0fd1d6719451.png)
                                                             
#now running

#Challenge 003#

#Mounting a staking pool
                                                             
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool id>", "owner_id": "<accountId>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<accountId>" --amount=30 --gas=300000000000000
                                                             
#NOTE: Change <pool id>, <accountId>, <public key>, <accountId> parameters e.g mine near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "johnpaulnodes", "owner_id": "johnpaulnodes.shardnet.near", "stake_public_key": "ed25519:72wzgPZgDx2SXSe4ba7RNv57JE2ZXZ8xGH", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="johnpaulnodes.shardnet.near" --amount=30 --gas=300000000000000 

#if its sucessful
  
#check your pool by running 
  
near proposals
  
#your node should be here
  
![image](https://user-images.githubusercontent.com/42779023/180095051-f7583a3a-eeb7-4d7b-8ebf-f4c9e6df9dd1.png)

#you can deposit and stake more depending on ur wallet balance
  
near call johnpaulnodes.factory.shardnet.near  deposit_and_stake --amount 1200 --accountId johnpaulnodes.shardnet.near --gas=300000000000000
  
#NOTE: johnpaulnodes.factory.shardnet.near and johnpaulnodes.shardnet.near should be replace by yours

#Challenge 004#

#monitor your node and be a validator confirming transactions on testnet and get >95% of uptime.
  
#Log Files, the command below will show you the log file
  
journalctl -n 100 -f -u neard | ccze -A

![image](https://user-images.githubusercontent.com/42779023/180096308-b91e92a9-d92c-47e3-8fe8-01c570e23978.png)
  
#Validator: A “Validator” will indicate you are an active validator
  
#97 validators: Total 97 validators on the network
  
#33 peers: You current have 33 peers. You need at least 3 peers to reach consensus and start validating

#Check your node version
  
curl -s http://127.0.0.1:3030/status | jq .version
  

#NOTE:install jq if you dont have it
  
sudo apt install curl jq

#Challenge 006#

#Create a cron task on the machine running node validator that allows ping to network automatically
  
  
#Create a new crontab, running every 5 minutes
  
crontab -e 
  
*/5 * * * *  export NEAR_ENV=shardnet && near call johnpaulnodes.factory.shardnet.near ping '{}' --accountId johnpaulnodes.shardnet.near --gas=300000000000000

  #NOTE: replace johnpaulnodes.factory.shardnet.near and johnpaulnodes.shardnet.near with yours
  
#check crontab to see if it is running
  
crontab -l
  
  #check wallet on web to see recent activity in interval of 5 minutes
  
  ![image](https://user-images.githubusercontent.com/42779023/180101376-d95dfe0d-9c86-445d-8580-ff39056d9a4e.png)

