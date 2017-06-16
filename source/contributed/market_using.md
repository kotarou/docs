---
title: Using the Screeps Market and choosing prices
contributed:
    name: kotarou
    link: https://github.com/kotarou
    date: TODO
---

Interacting with the market in Screeps is a great way to improve your economy, but it can be difficult to get started with. There are also a many mistakes and inefficiences to be avoided in order to maximise your profits. This article assumes that you already know how to use `Game.market.createOrder` and `Game.market.deal` to interact with the market, and you already have some credits. For basic market information, or if you do not yet know how to do this, see the article [here](http://docs.screeps.com/market.html).

This article discusses some basic theory in how to make effective market decisions, as well as why you should even bother. 

## Why use the market?

There are several major advantages to using the market;

- You can buy minerals for reactions that you do not currently mine.
- You can sell excess minerals and energy for credits to be used later.
- If someone attacks you;
    - You can't lose credits but you may lose whatever is sitting in storage.
    - You can buy boosts to defend yourself with, without having to wait for lab reactions to occur.
- You can buy subscription tokens with credits and play for free!
- Credits can be considered as implicit resources - instead of storing every resource/boost you could ever need in terminals, you can purchase them when and as required.

## Choosing BUY and SELL prices

When you create an order, or fufill a existing deal, whether manually or programatically, there are two fundamental tradeoffs to consider:

1) A seller needs to mazimize unit price (i.e. their profit) against the speed with which the market will fufill their deal.
2) A buyer likewise wants to minimize the cost per unit purchased against the speed with which they are able to purchase resources.

Given that a seller incurs a 5% credit fee for posting a transaction, and the buyer incurs a energy cost for the distribtuion, there are two more factors to consider:

1) A sell order that is too high, or a buy order that is too low is likely to never complete, losing the 5% fee with no expected return.
2) Fufilling any order will cost energy to distrubte - this cost may affect value considerations, especially for long orders.

### When to sell, when to buy

As a rule of thumb, unless you have more energy than you know what to do with, the corresponding market has gone bonkers, or urgency is a factor:
- If you want to get rid of a resource, only create sell orders.
- If you want to gain a resource, only create buy orders.

Notable exception: Subscription tokens. These are a low volume, high value resource, and follow very different rules wile the market is as slow as it currently is. Unitl the price of a token drops to ~1 million credits, this market is seller driven.

### Selling: Don't undercut the market

The bad way to price a sell order is to find the lowest price on the market and undercut it by 0.01 credits. This can cause some issues - consider the theoretical markest siutation below;

ORDER_ID | Price | Amount |  Location
--- | --- | --- | --- |
0007 | 0.07 | 1000 | E7N7
0006 | 0.08 | 1000 | E6N6
0005 | 0.09 | 1000 | E5N5
0004 | 0.10 | 1000 | E4N4
0003 | 0.11 | 2000 | E3N3
0002 | 0.12 | 345345 | E2N2
0001 | 0.13 | 1000 | E1N1

In this example, 5 rooms have each undercut the market's lowest price by 0.01, each presumably hoping for a fast trade. This means that the seller `0007` is getting 0.04 less per unit than they otherwise could - they are losing almost half of their potential profit! In any given day, a base resource will see in excess of a million units traded. Resources like Hydrogen will often see over 4million units sold - this means that the marginally more expensive orders, like `0002` above will likely conclude within the day anyway. 

### Selling: Find the price wall

The simplest strategy for pricing is to find the point at which the number of orders spikes and creates a "wall", and slightly undercut it. Consider the hydrogran graph, below. 

![Sample price data for Hydrogen](./h_sell_wide.png "Sample price data for Hydrogen")

We can see a large spike at a price of 0.20, where the number of orders jumps from ~7000 at 0.19 to ~65,000 at 0.20. This is our price wall, and a second is likewise visible at 0.30, which makes sense - humans placing prices gravitate to rounded numbers. If we look at the historical H prices, this wall is within the average H sell range, which hovers around 0.16 ± 0.05. 

So, the general optmial price for H in this situation is 0.19.

The nice thing about this pricing strategy is that it is self-balancing. If the market pumps more H into sell orders than is purchased, a new wall will form at 0.19, and people will price at 0.18 to sell. If the market buys more H than is sold, then the wall at 0.20 will be depleted and a new wall will from at a higher price. However, these effects will not drive the price down overnight, as occurs with the previous situation.

### Buying: Find the wall, adjust for distribution costs

As a buyer, keep in mind that the player fufilling your order will need to pay energy for distribution costs, according to the formula `Math.ceil( amount * ( 1 - Math.exp(-distanceBetweenRooms/30) ) )` which is in-game via the function `Game.market.calcTransactionCost(amount, roomName1, roomName2)`. In a reasonable market, a buy order's price + average distribution cost + 5% should approximate the lowest sell order's price.

The pricing decision for making buy orders is similar to making a sell order - find the wall, and price slightly above it, accounting for distribution costs (a safe assumption - the further you are from the center of the map, the higher the average distribution cost will be). This in practice means that a buyer in e.g. W95N23 may pay 0.01 ~ 0.02 more per unit than a buyer in E1N1 - the average distance to sellers is higher in the first case, so they need to price more competitively to match. 

### Adjust prices for urgency

Sometimes you get invaded, and you have two decisions to make with your terminals.

1) Bring in boosts/energy and defend/repair your way to saftey.
2) Sell/transfer everything in the room's terminal to minimize loss.

In both cases, you are going to want the tranastions to conclude ASAP, especially with the 10 tick terminal cooldown. This is the only case where it is reasonable to undercut the entire sell market, or outprice the entire buy market. 

You may even consider fufilling otherwise unpalatable orders instead of making your own.
