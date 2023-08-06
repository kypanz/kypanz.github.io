---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
exclude: true
---

# Blockchain filter in real time

Script in Nodejs to filter pending transaction greater than specific amount in realtime

- Filter BSC transactions by value in real time using `websocket`

- save the transactions `greater` tan `x` value in `memory`


Github Repository [here](https://github.com/kypanz/blockchain-transactions-filter-in-console-with-charts)


## The code

`index.js`

```javascript
var asciichart = require('asciichart');
var clear = require('clear');
const readline = require('readline');
const { ethers } = require('ethers');
require('dotenv').config();

// Connection for the blockchain node
const URL_NODE = `wss://bsc.getblock.io/${process.env.API_KEY}/mainnet/`;
const provider = new ethers.providers.WebSocketProvider(URL_NODE);

// Data handle
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

// Data output
const data = [0];
const history = [];
let lastBlock = 0;
let lastTransaction = '';
let lastGreaterThanEther = '';
let valueToFilter = 0;
let seconds = 0;

function refresh() {
    try {
        if (data.length >= 100) {
            data.shift();
        }
    } catch (error) {
        console.log('error on refresh');
    }
}

function comparation(valuePrice) {
    try {
        const oneEther = ethers.utils.parseUnits(valueToFilter, 'ether');
        const oneEtherinWei = ethers.utils.parseUnits(oneEther.toString(), 'wei');
        const bnValue = ethers.BigNumber.from(valuePrice);
        return bnValue.gt(ethers.BigNumber.from(oneEtherinWei));
    } catch (error) {
        console.log('error on comparation');
    }
}

function captureData() {
    provider.on('pending', async (transactionHash) => {
        const tx = await provider.getTransaction(transactionHash);
        if (tx && tx.value) {
            const valuePrice = tx.value.toString();
            const isAcutalPriceGreaterThanOneEther = comparation(valuePrice);
            if (isAcutalPriceGreaterThanOneEther) {
                lastGreaterThanEther = ethers.utils.formatEther(valuePrice);
                lastTransaction = `https://www.bscscan.com/tx/${transactionHash}`;
                history.push({ tx : lastTransaction, amount : ethers.utils.formatUnits(valuePrice.toString(), 'ether') });
            }
            refresh();
            data.push(Number(valuePrice));
        } else {
            refresh();
            data.push(0);
        }
    });

    provider.on('block', (blockNumber) => {
        lastBlock = blockNumber;
    });
}

function settingValueToFilter() {
    return new Promise((resolve) => {
        rl.question('Enter the value of BNB to filter => ', (inputValue) => {
            valueToFilter = inputValue;
            rl.close();
            resolve();
        });
    })
}

function renderChart() {
    const padding = '';
    const minLength = 12;
    const config = {
        offset: 3,
        height: 15,
        colors: [asciichart.green],
        format: function (x) {
            let inEther = ethers.utils.formatUnits(x.toString(), 'ether');
            if (inEther.length < minLength) {
                while (inEther.length < minLength) {
                    inEther += '0';
                }
            }
            return (padding + inEther).substring(0, minLength)
        }
    }
    const chartString = asciichart.plot(data, config);
    console.log(chartString);
}

function renderMessages() {
    console.log('------------------ About --------------------------------');
    console.log('[ Created by ] => kypanz');
    console.log('Happy hacking :)');
    console.log('------------------ Filter status ------------------------');
    console.log(`[ Greater than ] => ${valueToFilter} BNB`);
    console.log(`[ Actual block ] => ${lastBlock}`);
    console.log(`[ Live seconds ] => ${seconds}`);
    console.log('------------------ Last Capture -------------------------');
    console.log(`[ Pending transaction ] => ${lastTransaction}`);
    console.log(`[ Value ] ${lastGreaterThanEther} BNB`);
    console.log('------------------ History --------------------------------');
    for (let index = 0; index < history.length; index++) {
        console.log(`[ tx ] => ${history[index].tx} | [ amount ] => ${history[index].amount}`);
    }
}


async function main() {
    await settingValueToFilter();
    captureData();
    setInterval(() => {
        seconds++;
        clear();
        renderChart();
        renderMessages();
    }, 1000 * 1);
}

main();
```


## Tested on
```shell
node => v16.20.1, v19.3.0
npm => 8.19.4, 9.7.2
```

## Installation
```shell
npm install
```

## Configuration
Rename the `.env.example` file to `.env` and configure your `https://getblock.io/` api_key

## Run
```
node index.js
```

## Captures
![image](https://github.com/kypanz/blockchain-transactions-filter-in-console-with-charts/assets/37570367/b2963a0e-2567-472a-8bdd-122e7a303737)
![image](https://github.com/kypanz/blockchain-transactions-filter-in-console-with-charts/assets/37570367/4d27c1e4-3a12-436f-b07c-7c967113353b)
![image](https://github.com/kypanz/blockchain-transactions-filter-in-console-with-charts/assets/37570367/59b012f1-84a3-43fb-b7e6-abbc1cd8d022)

