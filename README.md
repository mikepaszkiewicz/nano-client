# Nano Client

Dead simple, promise-based client for interacting and building services on top of the NANO network, a next-generation cryptocurrency created by Colin LeMahieu with nearly instant transactions and no fees. [Learn more on the official repo](https://nanode.co/node-api) ⚡️

If you've worked with NANO before, you probably have experienced it's learning curve. The community is amazingly helpful and growing fast,
but the documentation and guides to working with it currently leave a lot to be desired.

**This library is designed to get anyone, even a total beginner, up and running building services on NANO in just a few minutes.** At it's core,
this package is a wrapper around the official [RPC protocol](https://github.com/nanocurrency/raiblocks/wiki/RPC-protocol) that does a few things:

1. abstracts away some of the idiosyncracies and quirks with the RPC API,
2. exposes some top-level methods like `Nano.send()` and `Nano.recieve()`, so you don't have to know about creating, signing, publishing, etc.

Want to build with NANO, but not running a full node? [Sign up for node access and get your API key at nanode.co](https://nanode.co/node-api)

## Install

`npm install nanode`

## Usage

This library is built with TypeScript, and I highly reccommend you take advantage of your code editor's Intellisense features. All fields on requests and responses for the RPC are strings - and for now, the same is true for this library.

The full code for these snippets can be found /examples directory

### Connect

Initiate the client with your API key and a valid RPC url. Optionally pass your address and private key to the constructor.

Not sure what your private key is? Call `nano.get_deterministic_key()` with your account's seed to get all relevant account information.

```typescript
//examples/init.ts
import Nano from 'nanode'
const nano = new Nano({
  api_key: process.env.API_KEY,
  url: 'https://api.nanode.co',
  origin_address: process.env.SENDER_WALLET, // wallet for the 'controlling' account
  origin_key: process.env.SENDER_WALLET_PRIVATE_KEY // key for the 'controlling' account
})
```

### Open account

Now that we're connected, we need to 'open' the account. Opening an account is a bit of a catch-22 - it requires any amount to be sent to the address from a funded account before account is actually open.

```typescript
//examples/open.ts

import Nano from './init'
//create a new key
const target = await Nano.key.create()

console.log(`Initializing account ${target.account} with funds..`)

//send new account init funds from our walet
const hash = await Nano.send('0.01', target.account)

//open block with send's hash
const hash = await Nano.open(
  hash,
  process.env.DEFAULT_REP,
  target.private,
  target.public
)
console.log(`Open block published with hash: ${hash}`)
```

### Send and receive

Sending and receiving are simple one liners. For receive, we have to pass in the hash of the block we're receiving

```typescript
//examples/send.ts, examples/receive.ts
import Nano from './init'

const wallet_addr =
  'xrb_3hk1e77fbkr67fwzswc31so7zi76g7poek9fwu1jhqxrn3r9wiohwt5hi4p1'

const hash = await Nano.send('0.01', wallet_addr)

return await Nano.receive(
  hash,
  process.env.RECIPIENT_WALLET_PRIVATE_KEY,
  wallet_addr
)
```

## Full list of methods

If you aren't sure about some of the arguments, they're available as types in your editor, or in the official RPC guide. Full TypeDoc documentation is on the way!

### Top-level calls

if you only need to send and recieve NANO, **these methods should technically be the only ones you need**, per the examples above:

* `nano.open()`
* `nano.send()`
* `nano.receive()`
* `nano.change()`

### Accounts

Account methods take a single account string or in some cases, an array of accounts.

* `nano.accounts.get()`
* `nano.accounts.balance()`
* `nano.accounts.balances()`
* `nano.accounts.block_count()`
* `nano.accounts.frontiers()`
* `nano.accounts.history()`
* `nano.accounts.info()`
* `nano.accounts.key()`
* `nano.accounts.ledger()`
* `nano.accounts.pending()`
* `nano.accounts.representative()`
* `nano.accounts.weight()`

### Blocks

Has methods to get information about blocks:

* `nano.blocks.account(hash: string)`
* `nano.blocks.count(byType?: boolean)`
* `nano.blocks.chain(hash: string, count?: number)`
* `nano.blocks.history(hash: string, count?: number)`
* `nano.blocks.info(hashOrHahes: string | string[], details?: boolean)`
* `nano.blocks.pending(hash: string)`
* `nano.blocks.successors(block: string, count?: number)`

Methods to construct blocks:

* `nano.blocks.createOpen(block: OpenBlock)`
* `nano.blocks.createSend(block: SendBlock)`
* `nano.blocks.createReceive(block: ReceiveBlock)`
* `nano.blocks.createChange(block: ChangeBlock)`

And a method to publish a block to the network:

* `nano.blocks.publish(block: string)`

### Convert

Allows you to convert `rai`, `krai`, and `mrai` amounts to and from their raw values.

* `nano.convert.toRaw(amount: string, denomination: 'rai' | 'krai' | 'mrai')`
* `nano.convert.fromRaw(amount: string, denomination: 'rai' | 'krai' | 'mrai')`

### Work

Allows you to generate and validate Proof of Work for a given block hash.

* `nano.work.generate(hash: string)`
* `nano.work.validate(work: string, hash: string)`

## Todos

* Use BigNumber, etc. to allow passing numbers
* TypeDoc site + add remaining method documentation
* Better argument checking / error handling
* Tests!
