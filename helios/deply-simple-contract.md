## Chronos Deploy - Helios
gSAD everyone! lansung aja. Kita bikin contract untuk deploy GM ala-ala onChainGM

----
### Cara Deploy-nya
Karena akan menggunakan sedikit koding Nodejs, maka perlu bikin folder dengan struktur seperti ini:
```bash
. 
├── contracts/
│   └── GmContract.sol       
├── scripts/
│   └── deployGM.js  # script js untuk deploy
├── .env  # Private key & RPC di sini nanti
├── hardhat.config.js # konfigurasi hardhat
└── package.json
```
Pastikan strukturnya seperti itu, lalu lanjut isi kodingnya.

Isi file **GmContract.sol** dengan kode ini

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract GmContract {
    string public gmMessage;
    uint public deployedAt;

    constructor(string memory _msg) {
        gmMessage = _msg;
        deployedAt = block.timestamp;
    }
}
```
Selanjutnya isi file **deployGM.js** dengan kode ini
```javascript
const hre = require("hardhat");
  
async  function  deployGmContract() {
	const now = new  Date();
	const gmMessage = `GM 🌞 ${now.toDateString()} - ${now.toLocaleTimeString()}`;

	const [deployer] = await hre.ethers.getSigners();
	console.log(`Deploying GM 🌞 from ${deployer.address}`);

	const GmContract = await hre.ethers.getContractFactory("GmContract");
	const contract = await GmContract.deploy(gmMessage);
	await contract.waitForDeployment();

	const contractAddress = await contract.getAddress();
	const explorerURL = `https://explorer.helioschainlabs.org/address/${contractAddress}`;

	console.log("✅ GM Contract deployed!");
	console.log("📍 Address:", contractAddress);
	console.log("🔗 Explorer:", explorerURL);
}

deployGmContract().catch(err => {
	console.error("❌ Deployment error:", err);
	process.exit(1);
});
```

Selanjutnya, buka file **hardhat.config.cjs** lalu isi dengan
```javascript
require('dotenv').config(); 
require("@nomicfoundation/hardhat-toolbox");

module.exports = { 
  solidity: "0.8.20",
  networks: {
    heliosTestnet: {
      url: process.env.RPC_URL,
      accounts: [process.env.PRIVATE_KEY]
    }
  }
};
```

Buka file **package.json** lalu isi dengan
```json
{
  "name": "helios-deployer",
  "version": "1.0.0",
  "description": "Deploy smart contract ke Helios EVM",
  "main": "scripts/deployGM.js",
  "scripts": {
    "deploy:gm": "npx hardhat run --network heliosTestnet scripts/deployGM.js"
  },
  "keywords": [
    "helios",
    "evm",
    "smart-contract",
    "gm",
    "hardhat"
  ],
  "author": "SAD",
  "license": "MIT",
  "dependencies": {
    "@openzeppelin/contracts": "^5.3.0",
    "dotenv": "^16.3.1",
    "ethers": "^6.8.1"
  },
  "devDependencies": {
    "@nomicfoundation/hardhat-toolbox": "^3.0.0",
    "hardhat": "^2.22.1"
  }
}
```
Buka .env lalu isi dengan
```
PRIVATE_KEY=0x3a4b9....
RPC_URL=https://testnet1.helioschainlabs.org
```

### Deploy gasss
Buka terminal di root folder, lalu jalankan perintah
```bash
npm install
```
lalu jalankan perintah
```bash
npm run deploy:gm
```
