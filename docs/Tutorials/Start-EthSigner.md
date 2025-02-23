description: Getting started with EthSigner
<!--- END of page meta data -->

# Getting Started 

For file-based signing, EthSigner requires a V3 Keystore key file and a password file.

!!! tip 
    EthSigner also supports signing transactions with a key stored in an external vault (for example, 
    [Hashicorp Vault](../HowTo/Store-Keys/Use-Hashicorp.md)).  

## Prerequisites 

* [EthSigner](../HowTo/Get-Started/Install-Binaries.md)
* [Hyperledger Besu](https://besu.hyperledger.org/en/stable/HowTo/Get-Started/Install-Binaries/)
* [Node.js](https://nodejs.org/en/download/)
* [web3.js](https://github.com/ethereum/web3.js/)

!!! note
    The Ethereum client used in this documentation is Hyperledger Besu but EthSigner can be used with any Ethereum client.     


## Start Besu 

[Start Besu](https://besu.hyperledger.org/en/stable/HowTo/Get-Started/Starting-node/) with the 
[`--rpc-http-port`](https://besu.hyperledger.org/en/stable/Reference/CLI/CLI-Syntax/#rpc-http-port)
option set to `8590`. 

!!! example
    ```bash
    besu --network=dev --miner-enabled --miner-coinbase=0xfe3b557e8fb62b89f4916b721be55ceb828dbd73 --rpc-http-cors-origins="all" --host-whitelist=* --rpc-http-enabled --rpc-http-port=8590 --data-path=/tmp/tmpDatdir
    ```

## Create Password and Key Files 

Create a text file containing the password for the V3 Keystore key file to be created (for example, `passwordFile`). 

!!! attention "Password text file must not contain characters other than those used in your password"
    EthSigner reads the password file as binary and any character in the file is considered part
    of your password.
    
    _Some POSIX compliant editors automatically add an end-of-line in text files. If your editor adds an
    end-of-line character, the end-of-line is considered part of your password._
    
    Use the following command to ensure the password file is correct:
    ```bash
    echo -n "Type your password:";read -s password;echo -ne $password > passwordFile;
    ```
    Enter the password when prompted.

Use the [web3.js library](https://github.com/ethereum/web3.js/) to create a key file where: 

* `<AccountPrivateKey>` is the private key of the account with which EthSigner will sign transactions.  

* `<Password>` is the password for the key file being created. The password must match the password saved in the 
   password file created above (`passwordFile` in this example).

!!! example 

    ```javascript linenums="1" tab="Create Key File"
    const Web3 = require('web3')
    
    // Web3 initialization (should point to the JSON-RPC endpoint)
    const web3 = new Web3(new Web3.providers.HttpProvider('http://127.0.0.1:8590'))
    
    var V3KeyStore = web3.eth.accounts.encrypt("<AccountPrivateKey>", "<Password>");
    console.log(JSON.stringify(V3KeyStore));
    process.exit();
    ```
    
    ```javascript linenums="1" tab="Example"
    const Web3 = require('web3')
        
    // Web3 initialization (should point to the JSON-RPC endpoint)
    const web3 = new Web3(new Web3.providers.HttpProvider('http://127.0.0.1:8590'))
        
    var V3KeyStore = web3.eth.accounts.encrypt("0x8f2a55949038a9610f50fb23b5883af3b4ecb3c3bb792cbcefbd1542c692be63", "password");
    console.log(JSON.stringify(V3KeyStore));
    process.exit();
    ```
    
Copy and paste the example JS script to a file (for example, `createKeyFile.js`) and replace the placeholders. 

Use the JS script to display the text for the key file: 

```bash
node createKeyFile.js
```

Copy and paste the text to a file (for example, `keyFile`). The file is your V3 Keystore key file. 
    
## Start EthSigner

Start EthSigner with options specified as follows: 

* `chain-id` is the chain ID specified in the Besu genesis file. 

* `downstream-http-port` is the `rpc-http-port` specified for Besu (`8590` in this example). 

* `key-file` and `password-file` are the key and password files [created above](#create-password-and-key-files).  

!!! example
    ```
    ethsigner --chain-id=2018 --downstream-http-port=8590 file-based-signer --key-file=/mydirectory/keyFile --password-file=/mydirectory/passwordFile
    ```

## Confirm EthSigner is Up

Use the `upcheck` endpoint to confirm EthSigner is running.

!!! example
    ```bash tab="curl HTTP request"
    curl -X GET http://127.0.0.1:8545/upcheck
    ```
   
    ```json tab="Result"
    I'm up
    ```

## Confirm EthSigner Passing Requests to Besu 

Request the current block number using [`eth_blockNumber`](https://besu.hyperledger.org/en/stable/Reference/API-Methods/#eth_blocknumber) with the EthSigner JSON-RPC endpoint (`8545` in this example): 

```bash
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":51}' http://127.0.0.1:8545
```