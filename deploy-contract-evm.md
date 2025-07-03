## Cara Paling Mudah Deploy Contract di EVM
Deploy contract yang akan ku kasih ini bukan deploy via Remix, tapi via koding Javascript.

Jadi akan lebih baik jika sudah paham tentang dasar pemrograman di javascript.

Kalau gak bisa? ya lanjut aja. Namanya juga belajar ya kan?

### Apa aja yang perlu disiapkan?
- Tentu saja pemahaman koding (dikit aja yang penting bisa mikir)
- Laptop/PC/HP juga bisa
- Koneksi internet tentu saja
- Niat

### Mulai? 
Pertama, kita akan menggunakan [Hardhat](https://hardhat.org/) untuk deploy smart contract di EVM. Selain Hardhat emang bisa? ya bisa, tapi itu nanti saja.

Kita akan ambil contoh deploy contract di Helios, karena mereka baru saja mulai testnet dan ada task deploy contract.
### Hardhat
Install Hardhat dulu
```bash
npm install --global hardhat && npm i @nomicfoundation/hardhat-toolbox
```
Bikin folder untuk project deploy, di dalamnya bikin file dengan nama `hardhat.config.js` lalu isi dengan:
```javascript
require("@nomicfoundation/hardhat-toolbox");

module.exports = {
  solidity: "0.8.20",
  networks: {
    testDeploy: {
      url: "isi_dengan_url_rpc",
      accounts: ["isi_dengan_private_key"],
    },
  },
};
```
Perhatikan **testDeploy**, itu akan dipakai ketika mulai deploy nanti. Jadi perlu diingat! Sisanya perlu diisi sesuai dengan keterangan.

Bisa coba pakai RPC-nya Helios: https://testnet1.helioschainlabs.org

### Contract
Perhatikan contract sederhana ini
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MyContract {
    string public message;

    constructor(string memory _message) {
        message = _message;
    }

    function updateMessage(string memory _newMessage) public {
        message = _newMessage;
    }
}
```
`pragma solidity ^0.8.20;` itu wajib ada ketika menulis contract, kenapa? karena kode tersebut akan memberitahu bahwa kita sedang pakai solidity versi tersebut. Maka compiler harus menyesuaikan dengan versi solidity-nya.

`MyContract` adalah nama contract-nya. Terserah mau diganti apa aja bisa.

`string public message;` artinya kita sedang bikin variable dengan nama `message` yang bisa diakses siapa saja karena dideklarasikan sebagai public.

```
constructor(string memory _message) {
	message = _message;
}
```
Kode constructor di atas itu adalah fungsi contract yang akan dijalankan sekali ketika pertama kali contract di-deploy nanti. 

Isinya bisa apa saja, untuk kode di atas hanya akan memberikan nilai ke variable `message`

> File contract harus ditaruh di dalam folder 'contracts'
### Javascript
Buat file javascript (terserah apa saja namanya), lalu isi dengan kode ini:

```javascript
// misal nama file deploy.js
const hre = require("hardhat");

async function deployContract() {
	// fungsi utk ambil address wallet dari pk yg sudah diset di hardhat.config.js
	const [deployer] = await hre.ethers.getSigners();
	
	// ambil contract yg sudah dibikin di folder contracts
	// 'MyContract' adalah nama contract-nya
	const MyContract = await hre.ethers.getContractFactory("MyContract");
	
	// mulai deploy contract dg pesan yg ingin dikirim
	// ingat bahwa contract di atas dirancang utk menerima pesan dan diset ke variable 'message' di contract
	const contract = await MyContract.deploy("GM everyone!");

	// tunggu sampai deploy selesai
	await contract.waitForDeployment();

	// ambil address contract yg sudah dideploy
	const contractAddress = await contract.getAddress();

	console.log(`Ini address contract-nya:  ${contractAddress}`); 
}

// jalankakn fungsi deploy
deployContract().catch(err => {
  console.error("Deployment error:", err);
  process.exit(1);
});
```

Lalu jalankan perintah ini di terminal untuk deploy
```bash
npx hardhat compile
npx hardhat run --network testDeploy deploy.js
```

Perhatikan **testDeploy** di perintah tersebut. Itu adalah apa yang sudah kita set di **hardhat.config.js** sebelumnya

## Interaksi dengan smart contract yang sudah di-Deploy
Sekali di-deploy, kode smart contract kita sudah tidak bisa diubah. Hanya bisa kita update nilai di dalamnya. Misal, kita bisa mengupdate nilai dari `message`

```javascript
const contract = new ethers.Contract(
  "CONTRACT_ADDRESS",
  ["function updateMessage(string _newMessage) external"],
  wallet
);

await contract.updateMessage("Malas ah!");
```

Gimana? gampang kan? ya kali susah.

------
Follow bisa lah! https://x.com/rhmtfq
