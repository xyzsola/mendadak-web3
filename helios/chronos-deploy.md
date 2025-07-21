
If you don't know what [Schedule Execution](https://hub.helioschain.network/docs/innovate/advanced-use-cases/autonomous-chronos-tasks "https://hub.helioschain.network/docs/innovate/advanced-use-cases/autonomous-chronos-tasks") is, you can read it in Helios' docs.

Simply put, we can access certain functions in a contract we've previously created repeatedly at specific intervals, for example, every 30 minutes.

How this works is that when you first deploy this scheduled task, a **Cron Wallet** will be automatically created. This is a new wallet that is directly linked to the wallet used during deployment.

In addition to creating a Cron Wallet, you also need to deposit a certain amount of **HLS** (Helios Test Tokens) to be used as gas fees when the task is automatically run.

## Deploy simple contract
Clone this deployment repository
```bash
git clone https://github.com/xyzsola/helios-deploy
```
and run this command
```bash
cd helios-deploy && npm install
```
create file `.env` and fill with this
```
PRIVATE_KEY=your_wallet_private_key
RPC_URL=https://testnet1.helioschainlabs.org
```
Run this command to deploy contract
```bash
npm run deploy:contract
```
## Deploy Chronos
Run this command to deploy chronos
```bash
npm run deploy:chronos
```
If the message "successful deployment" appears, try checking whether https://testnet.helioschain.network/ is complete or not.
