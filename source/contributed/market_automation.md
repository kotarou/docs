---
title: From manual to automatic marketing
contributed:
    name: kotarou
    link: https://github.com/kotarou
    date: TODO
---

Interacting with the market in Screeps is a great way to improve your economy, but it can be difficult to get started with. This article assumes that you already know how to use `Game.market.createOrder` and `Game.market.deal` to interact with the market, and you already have some credits. For basic market information, or if you do not yet know how to do this, see the article [here](http://docs.screeps.com/market.html).

This article discusses a pathway of market interactions, beginning with easy manual interactions through to complex automatic trading.

## What type of interaction should I choose?

If you are a beginning player, with only a few rooms who does not automatically boost creeps: Go with **Fully Manual** interaction.\
If you have multiple rooms, and are running lab reactions, but do not have automatic boosting, or do not have much spare CPU, use **Semi-automatic** interaction.\
If you have the CPU to spare, or just want sexy market scripts, go **Fully automatic**.

## Some helper methods



## Manual marketing

The easiest way to interact with the market is manually. Every few days, look at the market [here](https://screeps.com/a/#!/market/all) and work out what prices are a good fit for you. The price guide [here](./market_using.md) is a good place to start if you are not sure about pricing.

You can then manually create your orders or complete deals. However, the default market method can be dangerous! Spot the difference:

` Game.market.createOrder(ORDER_SELL,SUBSCRIPTION_TOKEN,2000000,1) `
` Game.market.createOrder(ORDER_SELL,SUBSCRIPTION_TOKEN,1,2000000) `

This is a mistake that has happened to players before - selling subscription tokens at 1 credit, instead of selling one subscription token! And yes, they are charged the 5% fee for all of those tokens to be sold... an *expensive* mistake.

Lets replace this with a safer method;

```javascript
// A wrapper that adds a check for accidentally swapping an order's price and amount around
Game.market.prototype.createOrderSafe = function(orderType, resource, price, totalAmount, roomName) {
    if (totalAmount > price && resource === SUBSCRIPTION_TOKEN) {
        console.log(`You attempted to sell ${totalAmount} tokens @ ${price}c!`);
        return;
    }

    if (totalAmount < price) {
        console.log(`You attempted to sell ${totalAmount} of ${resource} @ ${price}c!`);
        return;
    }

    return Game.market.createOrder(orderType, resource, price, totalAmount, roomName);
}
```

<!--These safety methods are really useful for any manual interactions with the Screeps game and can be extended significantly. One that I have used in the past;

```javascript
// Ensure I am not competing with my own orders
Game.market.prototype.createOrderSafe2 = function(orderType, resource, price, totalAmount, roomName) {
    // Check for price mistake
    if (totalAmount < price) {
        console.log(`You attempted to sell ${totalAmount} of ${resource} at ${price}c!`);
        return;
    }
    
    let candidateOrder = _.filter(Game.market.orders, (order) => {
        return order.type === orderType && order.resourceType === resource && order.roomName === roomName && order.price === price;
    })[0] || undefined;

    // Check for duplicate order
    if (candidateOrder) {
        console.log(`Warning: ${orderType === ORDER_BUY ? "Buying" : "Selling"} ${resource} in ${roomName} has pre-existing corresponding orders.`);
        return Game.market.extendOrder(candidateOrder.id, totalAmount);
    }

    return Game.market.createOrder(orderType, resource, price, totalAmount, roomName);
    
}
```-->


The price / amount pitfall aside, manual marketing is fairly simple. You want to make orders with a size that allows reasonable deals to occur, without being so large that it won't fully complete before the market shifts / you need the deal to finish. This normally means around 25000 ~ 50000 units per sell order, or a minimum of 10000 for a buy order. If the entire set finished, you can simply call `Game.market.extendOrder(orderId, amount)` to increase the amount in the order - but you're not getting credits back if you have to adjust the price. 


## Semi-manual marketing

While working with the market smnaully is totally reasonable for a small player, at scale it gets tedious. Most rooms have fairly well-known parameters, so we can simply write a Memory object detailing what a room wants to bring in or sell, e.g.

```json
Memory.roomResourceData = {
    E26N43: {
        mines: "H",
        produces: ["OH"],
        needs: "O",
    },
    E29N45: {
        mines: "K",
        produces: ["KH"],
        needs:["H"],
    },
}

Memory.marketValues = {
    "H": {
        sellAt: 0.17,
        buyAt: 0.15,
        sellOrderId: 0,
        buyOrderId: 0
    },
    "O": {
        sellAt: 0.16,
        buyAt: 0.13,
        sellOrderId: 0,
        buyOrderId: 0
    },
}
``` 

It is then fairly simple to work out what you need to buy and sell on the market

```javascript
const BUY_WHEN_BELOW    = 500;
const SELL_WHEN_ABOVE   = 10000;
const MAX_ORDER_SIZE    = 5000;
// The Screeps lodash version is < 4x and thus we don't use `_.map`
const requiredResources = _.pluck(Memory.roomResourceData, 'needs');
const producedResources = _.pluck(Memory.roomResourceData, 'produces');
const minedResources    = _.pluck(Memory.roomResourceData, 'mines');

// Resources that we should be able to get this from our own rooms
let transferFromSelf    = [...producedResources, ...minedResources];        
// Resources that we want to get from the market   
let buyFromMarket       = _.difference(requiredResources, transferFromSelf);

// If we want to, we could check if we are empty on resources that would otherwise be transferred
// This is outside the scope of this article.

// We can then simply check the terminals of the relevant rooms

// If they don't have the resource they need, and it is in our buyFromMarket list
//  Check if we have a currently active order. 
//      If we do, just wait.
//      Otherwise, if we have an inactive one, extend with the stored price values
//      Otherwise, make one and stores its ID for later use.
// If it is in the transferFromSelf list, then just transfer it internally. 

// If they have too much of a resource
//  Check if we have a currently active order. 
//      If we do, just wait.
//      Otherwise, if we have an inactive one, extend with the stored price values
//      Otherwise, make one and stores its ID for later use.
// If it is in the transferFromSelf list, then transfer it internally if we need to.
```

The good thing about such a system is that it is very quick to set up, and as long as the chosen values are sane, reasonably safe against market movements - at most you lose the 5% fee on an order of MAX_ORDER_SIZE size. The bad thing is that it requires consistant attention - the sellAt and buyAt values need consistant picking.

## Fully automatic

This is where market interactions start costing CPU. Previously we were able to use the [market pages](https://screeps.com/a/#!/market/all) to work out good values for selling and buying, and have the empire automatically use these prices. 

To programatically work out what a good price is will require iterating over all the market orders in order to do calculations, and this eats CPU - so only run any of this code on occasion!

