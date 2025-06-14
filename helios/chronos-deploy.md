## Chronos Deploy - Helios
Tentang apa itu [Schedule Execution](https://hub.helioschain.network/docs/innovate/advanced-use-cases/autonomous-chronos-tasks "https://hub.helioschain.network/docs/innovate/advanced-use-cases/autonomous-chronos-tasks"), bisa baca di docs-nya Helios.  
  
Sederhananya, kita bisa akses fungsi tertentu di contract yg sudah kita bikin sebelumnya secara berulang dalam interval waktu tertentu. Misal, 30 menit sekali.  
  
Cara kerjanya adalah ketika pertama kali deploy schedule task ini, akan otomatis dibuatkan **Cron Wallet**. Sebuah wallet baru yang terikat langsung dengan wallet yg digunakan waktu deploy.  
  
Selain dibuatkan Cron Wallet, kita juga harus deposit sejumlah HLS (Helios Test Token) agar bisa dipakai sebagai gas fee ketika task akan dijalankan secara otomatis nanti.

> pastikan sudah bikin deploy GM dulu. [Check ini!]()

----
### Cara Deploy-nya
Karena akan menggunakan sedikit koding Nodejs, maka perlu bikin folder dengan struktur seperti ini:
```bash
.
├── abi/
│   └── chronos.json 
├── contracts/
│   └── Contract.sol       
├── scripts/
│   └── deployCron.js  # script js untuk deploy
├── .env  # Private key & RPC di sini nanti
├── hardhat.config.js # konfigurasi hardhat
└── package.json
```
Pastikan strukturnya seperti itu, lalu lanjut isi kodingnya.

Isi file **Contract.sol** dengan kode ini

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Contract {
    string public gmMessage;
    uint public deployedAt;
    uint public counter;

    constructor(string memory _msg) {
        gmMessage = _msg;
        deployedAt = block.timestamp;
    }

    function increment() external {
        counter += 1;
    }
}
```
Selanjutnya isi file **deployCron.js** dengan kode ini
```javascript
const dotenv = require("dotenv");
const inquirer = require("inquirer").default;
const fs = require("fs");
const path = require("path");
const hre = require("hardhat");
const cronABI = require("../abi/chronos.json"); 
const targetABI = require("../artifacts/contracts/GmContract.sol/GmContract.json");

dotenv.config();

async function deploy() {
  if (!fs.existsSync(logPath)) {
    console.error("❌ Tidak ada gm-log.json ditemukan. Deploy GM dulu!");
    return;
  }

  const [deployer] = await hre.ethers.getSigners();

  // Helios Chronos precompile
  const cronAddress = "0x0000000000000000000000000000000000000830"; 
  
  const cronContract = new hre.ethers.Contract(cronAddress, cronABI.abi, deployer);

  const logs = JSON.parse(fs.readFileSync(logPath, "utf8"));
  const last = logs[logs.length - 1];

  if (!last) {
    console.error("❌ Tidak ada log GM yang tersedia.");
    return;
  }

  // ambil contract address terakhir
  const contractAddress = last.address;

  const tx = await cronContract.createCron(
    contractAddress,
    JSON.stringify(targetABI.abi),
    "increment", // fungsi yang mau dipanggil
    [],          // argumen jika ada
    60,          // frequency: setiap 60 block (sekitar 3 menit)
    0,           // no expiration
    400_000,     // gas limit
    hre.ethers.parseUnits("2", "gwei"), // maxGasPrice
    hre.ethers.parseEther("0.1") // deposit 0.5 HLS
  );

  await tx.wait();
  console.log("Scheduled task created, transaction hash:", tx.hash);
}

deploy().catch(err => {
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
  "main": "scripts/deployCron.js",
  "scripts": {
    "deploy:cron": "npx hardhat run --network heliosTestnet scripts/deployCron.js"
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

Buka [ABI](https://github.com/helios-network/helios-core/blob/main/helios-chain/precompiles/chronos/abi.json), copy isinya lalu paste di file **abi/chronos.json**

### Deploy gasss
Buka terminal di root folder, lalu jalankan perintah
```bash
npm install
```
lalu jalankan perintah
```bash
npm run deploy:cron
```
