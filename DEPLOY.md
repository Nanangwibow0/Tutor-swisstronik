# TASK 1

### DEPLOY CONTRACT

### Install : 
```
nano Deploy.sh

```
Pate scrip Berikut :

```
#!/bin/sh

# Function to handle errors
handle_error() {
    echo "Error occurred in script execution. Exiting."
    exit 1
}

# Trap any error
trap 'handle_error' ERR

# Update and upgrade the system (optional)
echo "Updating and upgrading the system..."
sudo apt-get update && sudo apt-get upgrade -y

# Initialize a new Node.js project if package.json doesn't exist
if [ ! -f package.json ]; then
  echo "Initializing npm project..."
  npm init -y
fi

# Install necessary packages
echo "Installing necessary packages..."
npm install --save-dev @nomicfoundation/hardhat-toolbox
npm install dotenv
npm install @swisstronik/utils

# Initialize a new Hardhat project if not already initialized
if [ ! -d "./contracts" ]; then
  echo "Initializing Hardhat project..."
  npx hardhat
fi

# Remove default Lock.sol contract to avoid compiler version mismatch
if [ -f "contracts/Lock.sol" ]; then
  echo "Removing default Lock.sol contract..."
  rm contracts/Lock.sol
fi

# Create .env file
echo "Creating .env file..."
read -p "Enter your private key: " PRIVATE_KEY
echo "PRIVATE_KEY=$PRIVATE_KEY" > .env
echo ".env file created."

# Configure Hardhat
echo "Configuring Hardhat..."
cat <<EOL > hardhat.config.js
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

module.exports = {
  solidity: "0.8.19",
  networks: {
    swisstronik: {
      url: "https://json-rpc.testnet.swisstronik.com/",
      accounts: [\`0x\${process.env.PRIVATE_KEY}\`],
    },
  },
};
EOL
echo "Hardhat configuration completed."

# Create Hello_swtr.sol contract
echo "Creating Hello_swtr.sol contract..."
mkdir -p contracts
cat <<EOL > contracts/Hello_swtr.sol
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.19;

//This contract is only intended for testing purposes

contract Swisstronik {
    string private message;

    /**
     * @dev Constructor is used to set the initial message for the contract
     * @param _message the message to associate with the message variable.
     */
    constructor(string memory _message) payable{
        message = _message;
    }

    /**
     * @dev setMessage() updates the stored message in the contract
     * @param _message the new message to replace the existing one
     */
    function setMessage(string memory _message) public {
        message = _message;
    }

    /**
     * @dev getMessage() retrieves the currently stored message in the contract
     * @return The message associated with the contract
     */
    function getMessage() public view returns(string memory){
        return message;
    }
}
EOL
echo "Hello_swtr.sol contract created."

# Compile the contract
echo "Compiling the contract..."
npx hardhat compile
echo "Contract compiled."

# Create deploy.js script
echo "Creating deploy.js script..."
mkdir -p scripts
cat <<EOL > scripts/deploy.js
const hre = require("hardhat");

async function main() {
  /**
   * @dev make sure the first argument has the same name as your contract in the Hello_swtr.sol file
   * @dev the second argument must be the message we want to set in the contract during the deployment process
   */
  const contract = await hre.ethers.deployContract("Swisstronik", ["Hello Swisstronik!!"]);

  await contract.waitForDeployment();

  console.log(\`Swisstronik contract deployed to \${contract.target}\`);
}

//DEFAULT BY HARDHAT:
// We recommend this pattern to be able to use async/await everywhere
// and properly handle errors.
main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "deploy.js script created."

# Deploy the contract
echo "Deploying the contract..."
npx hardhat run scripts/deploy.js --network swisstronik
echo "Contract deployed."

# Create setMessage.js script
echo "Creating setMessage.js script..."
cat <<EOL > scripts/setMessage.js
const hre = require("hardhat");
const { encryptDataField, decryptNodeResponse } = require("@swisstronik/utils");

const sendShieldedTransaction = async (signer, destination, data, value) => {
  const rpclink = hre.network.config.url;
  const [encryptedData] = await encryptDataField(rpclink, data);
  return await signer.sendTransaction({
    from: signer.address,
    to: destination,
    data: encryptedData,
    value,
  });
};

async function main() {
  const contractAddress = "0xf84Df872D385997aBc28E3f07A2E3cd707c9698a";
  const [signer] = await hre.ethers.getSigners();
  const contractFactory = await hre.ethers.getContractFactory("Swisstronik");
  const contract = contractFactory.attach(contractAddress);
  const functionName = "setMessage";
  const messageToSet = "Hello Swisstronik!!";
  const setMessageTx = await sendShieldedTransaction(signer, contractAddress, contract.interface.encodeFunctionData(functionName, [messageToSet]), 0);
  await setMessageTx.wait();
  console.log("Transaction Receipt: ", setMessageTx);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "setMessage.js script created."

# Create getMessage.js script
echo "Creating getMessage.js script..."
cat <<EOL > scripts/getMessage.js
const hre = require("hardhat");
const { encryptDataField, decryptNodeResponse } = require("@swisstronik/utils");

const sendShieldedQuery = async (provider, destination, data) => {
  const rpclink = hre.network.config.url;
  const [encryptedData, usedEncryptedKey] = await encryptDataField(rpclink, data);
  const response = await provider.call({
    to: destination,
    data: encryptedData,
  });
  return await decryptNodeResponse(rpclink, response, usedEncryptedKey);
};

async function main() {
  const contractAddress = "0xf84Df872D385997aBc28E3f07A2E3cd707c9698a";
  const [signer] = await hre.ethers.getSigners();
  const contractFactory = await hre.ethers.getContractFactory("Swisstronik");
  const contract = contractFactory.attach(contractAddress);
  const functionName = "getMessage";
  const responseMessage = await sendShieldedQuery(signer.provider, contractAddress, contract.interface.encodeFunctionData(functionName));
  console.log("Decoded response:", contract.interface.decodeFunctionResult(functionName, responseMessage)[0]);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "getMessage.js script created."

# Run setMessage.js script
echo "Running setMessage.js script..."
npx hardhat run scripts/setMessage.js --network swisstronik
echo "setMessage.js script completed."

# Run getMessage.js script
echo "Running getMessage.js script..."
npx hardhat run scripts/getMessage.js --network swisstronik
echo "getMessage.js script completed."

echo "All operations completed successfully."
```
CTRL+X+Y ENTER

### Jalankan :

```
chmod +x Deploy.sh
./Deploy.sh
```
Salin Cobtract Lalu Upload Ke repositori Github

### DONE
