# dazaar-lightning
Lightning payment api for Dazaar stream services

## Usage
### Options
```js
const lightningOpts = {
  lnddir: ..., // data directory of the lightning node
  rpcPort: ..., // only needed for LND
  address: ..., // format <host>:<port>
  network: ..., // mainnet / testnet / regtest
  implementation: ... // 'c-lightning' or 'lnd'
}

const paymentCard = {
  payto: 'dazaartest22',
  currency: 'LightningBTC', // may also be LightningSats
  amount: '0.002',
  unit: 'hours',
  interval: 1
}
```

### Seller
```js
const Payment = require('dazaar-ln-payment')
const market = require('dazaar/market')
const hypercore = require('hypercore')

const m = market('./tmp')
const feed = hypercore('./tmp/data')

let payee

// instantiate a seller for a feed and equip it
// with a validate function
const seller = m.sell(feed, {
  validate (remoteKey, cb) {
    payee.validate(remoteKey, cb)
  }
})

seller.ready(function (err) {
  payee = new Payment.seller(seller, paymentCard, lightningOpts)
  
  // payment now set up. dazaar logic follows ... 
})
```

### Buyer
On a separate machine with the  
```js
// instantiate a buyer for a specific feed 
const buyer = m.buy(seller.key)

// set up pay payment linked to the buyer
const payer = new Payment.buyer(buyer, lightningOpts)

// buy an amount of feed
payer..buy(800, cb)
```

## API
#### `const pay = dazaarLightning(actor, payment, options)`
Create a new lightning payment instance associate to an actor (seller/buyer). `actor` should be a dazaar buyer or seller, `payment` may either be a dazaar payment card, or a string specifying the per second rate in either `BTC` or `Sats`, such as `200 Sats/s`. Options include:
```js
{
  lnddir: ..., // data directory of the lightning node
  rpcPort: ..., // only needed for LND
  address: ..., // format <host>:<port>
  network: ..., // mainnet / testnet / regtest
  implementation: ... // 'c-lightning' or 'lnd'
}
```

#### `pay.buy(amount, cb)`
A buyer can pay a specified amount for the stream that this buyer is registered to. `amount` is specified in satoshis (1 x 10<sup>-8</sup> BTC). Because a new buyer is instatiated for each stream, there is no need to specify more than the amount to be purchased.

#### `pay.validate(buyerKey, cb)`
A seller can validate the time left for a given buyer. Returns `error` if there is no time left on the subscription. The method shall check whether the given buyer has a subscription set-up and instantiate one not already present.

#### `pay.destroy()`
Destroy the payment provider

## License
MIT
