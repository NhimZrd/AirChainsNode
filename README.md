# AirChainsNode
## Server recommendations:

- CPU: 4 cores
- RAM: 8 GB
- Storage: 200 GB SSD
- OS: Ubuntu 20.04 / Ubuntu 22.0

## Instructions for installing the AirChains node:
1. Install package updates:
```
sudo apt update && sudo apt upgrade -y
```
2. Downloading the script to install the node:

Set your own wallet name and node name(moniker). Leave the port as default(26)
```
source <(curl -s https://itrocket.net/api/testnet/airchains/autoinstall/)
```
**Wait for the end of the node software installation. At the end there will be logs, you can exit from them by CTRL+C key combination**

3. Check the synchronization of the node. Proceed to the next steps of the node installation only after the "catching_up" status changes from "true" to "false":
```
junctiond status 2>&1 | jq
```
4. Create a wallet and assign a password. After creation, save the passphrase to a safe place. Also, we will need the address of the wallet for the following steps to be successful, write it down somewhere:
```
junctiond keys add $WALLET
```
**If you have already created a node before, you can import your already created wallet using the seed phrase:**

```
junctiond keys add $WALLET --recover
```
5. Run the following commands on the server. The server will ask for the wallet password:
```
WALLET_ADDRESS=$(junctiond keys show $WALLET -a)
VALOPER_ADDRESS=$(junctiond keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
source $HOME/.bash_profile
```
7. Request tokens to create the validator. To do this go to the project's Discord, then find the "switchyard-faucet-bot" branch and request tokens:
```
$faucet <YOUR_WALLET_ADDRESS>
```
7. After sending tokens by bot, check their availability on your wallet with the following command:

junctiond q bank balances $WALLET_ADDRESS
You should see a certain amount of tokens on your balance sheet

8. Create a validator:
```
cd $HOME
echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(junctiond comet show-validator | grep -Po '\"key\":\s*\"\"\K[^"]*')\"}.
    \"amount\" \{ "1000000amf\",
    \"moniker." \{ "REPLACE_YOUR_VALIDATOR_NAME",
    \"identity\": \"\",

    \ "website\": \"\",
    \"security." \"\",
    \ "details\": \"\",
    \ "commission-rate." \"0.1\",
    \ "commission-max-rate." \"0.2\",
    \ "commission-max-change-rate." \"0.01\",
    \"min-self-delegation." \"1\"
}" > validator.json
```
```
junctiond tx staking create-validator validator.json \
    --from $WALLET \
    --chain-id junction \
	--fees 200amf \
```
**Enter the password and confirm the operation with the "Y" button, in response the server will give you the hash of your transaction**

9. Delegate the remaining tokens to yourself:
```
junctiond tx staking delegate $(junctiond keys show $WALLET --bech val -a) 1000000amf --from $WALLET --chain-id junction --fees 200amf -y
```
**At this point, the node installation is complete.**

## Below is a list of useful commands for interacting with the node:

Check logs:

```sudo journalctl -u junctiond -f```

Check the status of the node:

```junctiond status 2>&1 | jq```

Restart the node process:

```sudo systemctl restart junctiond```

Output a list of the node's wallets:

```junctiond keys list```

Check the wallet balance:

```junctiond q bank balances $WALLET_ADDRESS```

Delete a wallet:

```junctiond keys delete $WALLET```

Delegate tokens to another validator:

```junctiond tx distribution withdraw-rewards $VALOPER_ADDRESS --from $WALLET --commission --chain-id junction --fees 200amf -y ```
