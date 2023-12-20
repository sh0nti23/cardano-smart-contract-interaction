# cardano-smart-contract-interaction     
The cardano-smart-contract-interaction script streamlines interactions with smart contracts on the Cardano blockchain, offering a practical tool for executing custom transactions and querying contract data.
Utilizing the Cardano-CLI and Cardano-Wallet, this script enables users to deploy smart contracts, submit transactions, and query contract information. It serves as a versatile foundation for developers and enthusiasts engaging with Cardano smart contracts.

#!/bin/bash

# Set your Cardano node and wallet information
CARDANO_NODE_URL="http://127.0.0.1:8100"
WALLET_ID="your_wallet_id_here"

# Deploy a smart contract (using Plutus Playground output)
deploy_smart_contract() {
    plutus-playground-generate-purescript > myContract.purs
    plutus-playground-generate-exe --target ./out --main MyContract --src myContract.purs
    plutus-playground-generate-script --target ./out/MyContract.plutus --contract MyContract
    cardano-cli transaction build \
        --alonzo-era \
        --testnet-magic 42 \
        --tx-in $(cardano-cli genesis initial-txin) \
        --tx-out $(cat $WALLET_ID.addr)+1000000 \
        --alonzo-era \
        --witness-override 1 \
        --fee 1000000 \
        --out-file matx.raw
    cardano-cli transaction witness \
        --alonzo-era \
        --testnet-magic 42 \
        --tx-body-file matx.raw \
        --out-file matx.witness \
        --signing-key-file $WALLET_ID.skey
    cardano-cli transaction assemble \
        --alonzo-era \
        --testnet-magic 42 \
        --tx-body-file matx.raw \
        --witness-file matx.witness \
        --out-file matx.tx
    cardano-cli transaction submit \
        --testnet-magic 42 \
        --tx-file matx.tx
}

# Submit a transaction to interact with the smart contract
submit_transaction() {
    cardano-wallet transaction create \
        --payment 1@"$WALLET_ID" \
        --withdrawal 0@"$WALLET_ID" \
        --metadata "$(cat metadata.json)" \
        --testnet-magic 42 \
        --transaction-dir ./tx \
        --time-to-live 600
}

# Query smart contract information
query_contract_info() {
    cardano-wallet transaction submit \
        --testnet-magic 42 \
        --transaction-dir ./tx
    cardano-cli transaction query \
        --testnet-magic 42 \
        --tx-hash "$(cat ./tx/tx1)" \
        --out-file query_output.json
    cat query_output.json
}

# Example Usage
deploy_smart_contract

# Wait for the contract to be confirmed and grab the contract address

# Now, interact with the deployed smart contract
submit_transaction

# Query smart contract information
query_contract_info

This script simplifies smart contract interactions on the Cardano blockchain using Cardano-CLI and Cardano-Wallet. Users can deploy smart contracts, submit transactions, and query contract information, making it a practical tool for developers and enthusiasts engaging with Cardano smart contracts.
