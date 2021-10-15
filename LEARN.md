# Cloning a real life project - PoolTogether ($100M marketcap)
In this Quest, we’ll be looking at how we can use the things we’ve learnt to create games that people from around the world can play. 

We’ll create a simple game using a protocol called PoolTogether. 

By going through this quest you’ll learn how to integrate with existing projects and the tooling that real projects use.

For going through this quest, you need to have completed the basics of Ethereum track on [https://questbook.app](https://questbook.app)
## Little background on PoolTogether
PoolTogether is a lottery game. People can pool in money into a pot. This pot is deposited to yield generators like Compoound. The interest aka yield from this deposit is then awarded to a winner of the lottery. 

PoolTogether calls itself a no-loss game. Because, you can always withdraw money from the pool as is. However, the longer you stay, the more likely you are to win lotteries!

You can play around with it at pooltegether.com.

You can also checkout the project’s market cap here : [https://etherscan.io/token/0x0cec1a9154ff802e7934fc916ed7ca50bde6844e](https://etherscan.io/token/0x0cec1a9154ff802e7934fc916ed7ca50bde6844e)

At the time of this writing, this super simple game has a market cap of $100M!

PoolTogether has just a few simple operations at a high level.

- User puts money into a pot
- A randomly selected winner is given the interest derived from yields
- User withdraws their money and their prize
## Setting up PoolTogether
First up lets dive straight into the project. We will be cloning the PoolTogether official repository. 

Fire up a terminal and get started!

```

~/ $ git clone https://github.com/pooltogether/pooltogether-pool-contracts.git

~/ $ cd pooltogether-pool-contracts

```

Now install all the dependencies. You’ll notice that there are a lot of dependencies in the `package.json` file. We needn’t bother about all of them, but we will introduce you to the most important ones. Install the packages by running : 

```

~/pooltogether-pool-contracts $ yarn

```
## Getting ready to deploy
PoolTogether works on the fact that if you deposit money, PoolTogether will in turn deposit that money in Compound or a similar yield generating protocol. 

To quickly deploy PoolTogether contracts on a local blockchain, run 

```

$ yarn start

```

What this does is, it compiles all the contracts in the `contracts/`

You’ll also see in the console that it deploys multiple contracts one after another. But where is it deploying this contract? Local? Ropsten? Mainnet?

PoolTogether uses a software called hardhat. 

Hardhat is the best blockchain development tooling there is. If you’ve been using Remix all this while, this is going to be a major improvement. 

Once `yarn start` has executed, you can pull up another terminal. Here open hardhat.networks.js. Under the object `networks.localhost` update the `chainId` to `31337`

Every network has a chainId. We’ve seen networks like mainnet, Ropsten etc in the previous quests. `chainId` `1` is reserved for mainnet. `localhost` chains usually use the `chainId` `31337`.

Now connect to this blockchain from the console using 

```

$ npx hardhat console --network localhost

```

This will connect to the blockchain running under the name `localhost` as defined in the `hardhat.networks.js` file.
## deploying contracts
The hardhat console is a powerful `js` based console. 

To see all the deployed contracts run 

```

await deployments.all()

```

To see only the names of the Contracts deployed 

```

Object.Keys(await deployments.all())

```

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/9631985a-78d8-4cc9-befe-674a5937928e.jpg)

You will see that there is an entry called cDai. If you’d remember from the quest on “getting a real interest rate on compound” we used cETH to deposit and withdraw ETH. In this example, PoolTogether has deployed a copy of Compound protocol with a single currency support, that is cDAI, and also the DAI contract itself. These wont be required when we’re deploying to mainnets, because those contracts would already exist there (DAI and Compound)

Because, if you remember, PoolTogether has to interact with the Compound protocol to earn the yield that can then be distributed to the winner of the prize.
## Interacting with Deployed Contracts
Now that the contracts are deployed let’s try interacting with some of them.

Let’s play around with a Contract we might already be familiar with - Dai. Dai is an ERC20 token. So it will have all the standard functions like name, symbol, balanceOf and so on. 

On the hardhat console

```

> DaiContractDeployed= (await deployments.all())\[“Dai”\]

```

Now, let’s see what all functions are there in this contract

```

> dai.abi

```

If you remember from previous quests, to interact with a contract you need 2 things. 

1. The interface using which you’ll interact with the contract
2. The address of the contract

```

> DaiContract = await ethers.getContractFactory(“Dai”)

````

Since hardhat was used to compile and deploy these contracts, we can use an internal function to get the interface. You can verify the interface using

```

> DaiContract.interface

```

We’ll get the address of the deployed contract using

```

> DaiContractDeployed.address

```

Using these we’ll be able to start interacting with the contract

```

dai = await DaiContract.attach(DaiContractDeployed.address)

```

Now you can interact with this contract :)

```

> await dai.name()

> await dai.balanceOf(“0x89Ce0f71D7387a580c6C07032f74f393a65d77F4”)

```

However, the above are read-only functions. You don’t have to write anything to the blockchain. However if you want to write something to the blockchain, you must sign the transactions with an account.

First, we’ll need to get the signer. Hardhat, when deployed locally, creates 10 test accounts that you can use for testing prefilled with some eth. 

```

> signers = await ethers.getSigners()

> signer = signers\[0\]

```

You can see the address of the signer using 

```

> signer.address

````

Hardhat will automatically handle the private keys for these 10 signers. 

Now we need to attach this signer to the dai contract

```

> daiWithSigner = dai.connect(signer)

```

Now you can use this to call state changing functions like approve. 

Let’s check the allowance of Dai before any approval

```

> await dai.allowance(signer.address, “0x89Ce0f71D7387a580c6C07032f74f393a65d77F4”)

```

This will check what is the allowance given to the address `0x89Ce` by `signer`

Now let’s approve some balance, since approve requires a state change (writing to the blockchain), we’ll need to use `daiWithSigner`

```

> await daiWithSigner.approve(“0x89Ce0f71D7387a580c6C07032f74f393a65d77F4”, 100)

> 

 await dai.allowance(signer.address, “0x89Ce0f71D7387a580c6C07032f74f393a65d77F4”)

```

This will return 

```

BigNumber { _hex: '0x64', _isBigNumber: true } 

```

`0x64` is the hex equivalent of `100`!

So, yay, you’ve now started interacting with the blockchain from the console.
## What next?
Now that you can interact with the DAI contract, i’ll let you in on a secret. 

This DAI contract (only on the localhost), has a secret function called `mint`

You can use `dai.mint(address, number of dai)` to freely mint DAI tokens into the said address. 

Write a script to have a 100 DAI balance for all the dummy accounts created by hardhat

In the next Quest, we’ll look at how to interact with this Project using a UI
