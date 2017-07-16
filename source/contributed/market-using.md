---
title: Using the Screeps Market and choosing prices
contributed:
    name: kotarou
    link: https://github.com/kotarou
    date: 2017-06-16
---

Interacting with the market in Screeps is a great way to improve your economy, but it can be overhwelming to get started.

This article discusses some basic theory in how to make effective market decisions, as well as outlining some reasons for interacting with the market.

This article assumes that you already know how to use functions like `Game.market.createOrder()` and `Game.market.deal()` to interact with the market, and that you already have some credits. For basic market information, or if you do not yet know how to do this, see the article [here](http://docs.screeps.com/market.html).


## Why use the market?

Any player who has a room at or above RCL 6 should consider using the market, as it can improve your empire in multiple ways:

### Economics:
- If you have labs and lab code working, you can purchase minerals that you would otherwise lack access to in order to run reactions.
- If you have excess minerals and/or energy, you can sell them for credits.

### Safety:
- Credits are not lost when you respawn, so selling resources guarantees you don't "lose it all" when attacked.
- You can buy boosts use them to fight bigger players near you, or expand faster.
- Using credits to buy boosts guarantees you have them when you need them, instead of wiating for lab reactions.

### Subscription tokens
- Sell minerals, buy tokens, play for free! 

### Flexibility
- You can convert a credit into whatever you need at the time, rather than having stockpiles of useless resource you don't use.

## Using the market effectively

When interacting with a virtual economy, such as the Screeps market, there are two things to consider:

1) The more urgent or impatient you are, the worse your deal will be
2) A worse deal now is better than no deal ever.

Screeps also adds two costs to dealing with the market:

1) The player that invokes `Game.market.createOrder()` will incurr a fee of 5% of the total transaction amount possible within the order - even if no deal occurs!
2) The player who executes `Game.market.deal()` will pay the energy cost to transfer the resources, regardless of whether they are a buyer or seller.

### Making an order vs dealing

If you are new to the markets, and do not have many credits, you will have to begin trading by fufilling other players' buy orders.

All base minerals (H,O,U,L,K,Z,X) trade in excess of 2 million units per day. With such a volume, a reasonably priced order is likely to be completed within a day or so, depending on your particular region - less central regions will be slower, by virtue of binng less established. However, orders you create cannot be guaranteed to complete immeditely, unless you have chosen an unsually competitive price.

When **fufilling** orders use the following code to work out the expected profit / cost from completing that order.

```javascript
const YOUR_ROOM;    // Wherever you wat the resource to arrive (buying) or leave (selling)
const AMOUNT;       // The amount you want to get rid of
const ENERGY_PRICE = 0.01;

const order         = Game.market.getOrderById(ORDER_ID);
const orderAmount   = Math.min(AMOUNT, order.remainingAmount)
const transferCost  = Game.market.calcTransactionCost(orderAmount, YOUR_ROOM, order.roomName);

// When selling
const totalProfit   = (order.price * AMOUNT) - (transferCost * ENERGY_PRICE);
// When buying
const totalCost     = (order.price * AMOUNT) +  transferCost * ENERGY_PRICE); 
```

Similarly, when **creating** orders, use the following code to work out the profit / cost.

```javascript
const YOUR_ROOM;    // Wherever you wat the resource to arrive (buying) or leave (selling)
const AMOUNT;       // The amount you want to get rid of
const SALE_PRICE    // Your desired sale price

// When selling
const expectedProfit = AMOUNT * SALE_PRICE * 0.95;
// When buying
const expectedCost   = AMOUNT * SALE_PRICE * 1.05;
```

When you are choosing to create a sell order or fufill a buy order, or likewise deciding to create a buy order or fufill a buy order, the above two sets of equations allow you to compare which action is better. It is very CPU intensive to run these on all orders within a given resource market, so using the market page [here](https://screeps.com/a/#!/market/all) and manually selecting a few orders to calculate is often a good idea. The results can be cached or manually written to Memory to be used later.

In general though, to gain credits you should make **sell** orders, and to gain resources you should make **buy** orders. Situations where this rule does not hold are unusual.  

## Selecting a Sell price

All sell prices should reflect a tradeoff between maximising profit and the amount of time the order will take to conclude. Orders that need to be completed sooner, for whatever reason, should be priced lower than otherwise suggested.

### Don't naively undercut the market

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

### Find the price wall

The simplest strategy for pricing is to find the point at which the number of orders spikes and creates a "wall", and slightly undercut it. Consider the hydrogen graph, below. 

![Sample price data for Hydrogen](./h_sell_wide.png "Sample price data for Hydrogen")

We can see a large spike at a price of 0.20, where the number of orders jumps from ~7000 at 0.19 to ~65,000 at 0.20. This is our price wall, and a second is likewise visible at 0.30, which makes sense - humans placing prices gravitate to rounded numbers. 

The candidate prices, for this graph, are thus 0.19 and 0.29 cresits per unit. If we look at the historical H prices, the average H sell price and standard deviation are approximately 0.16 ± 0.05 - as 0.19 is within this range it is the desired sale price.  

The useful thing about this pricing strategy is that it is self-balancing. If the market pumps more H into sell orders than is purchased, a new wall will form at 0.19, and the optmial price moves to 0.18, which *should* increase the amount of H players are willing to buy. If the market buys more H than is sold, then the wall at 0.20 will be depleted and a new wall will form at a higher price, which *should* decrease the amount of H players are willing to buy. However, since both of these situations involve moving the price via a changing "wall" or orders, the market price moves more slowly, more accurately reflecting player demand, compared to the naive undercutting approach.

If you are situated in the far regions of the map, your local market acitivty will likely be slower than players in the central regions, as neighbouring players will be less established. You may consider undercutting the wall by 0.02 instead, to reflect the maximal 0.01 energy per unit resource transfer cost.

## Selecting a buy price

This is basically the same as choosing a sell price. However, theres no real issue in overpricing this market - players automatially interact less with the buy market than the sell market, so you are less likely to see a cascade of scripts overcutting in this market. however, **Always keep distribution costs in mind**. 

As a player placing a buy order, keep in mind that the player fufilling your order will need to pay energy for distribution costs, according to the formula `Math.ceil( amount * ( 1 - Math.exp(-distanceBetweenRooms/30) ) )` which is in-game via the function `Game.market.calcTransactionCost(amount, roomName1, roomName2)`.

The pricing decision for making buy orders is similar to making a sell order - find the wall, and price slightly above it, accounting for distribution costs (a safe assumption - the further you are from the center of the map, the higher the average distribution cost will be). This in practice means that a buyer in e.g. W95N23 may pay 0.01 ~ 0.02 more per unit than a buyer in E1N1 - the average distance to sellers is higher in the first case, so they need to price more competitively to match. 

## Trading pitfalls and mistakes to watch out for

- A sell order placed too high, or a buy order placed too low will likely never fufill, wasting your 5% credit fee.
- The tier1 and tier2 boost markets have a sufficiently low trading volume that they are very risky. Be careful with any automatic scripts that interact with them.
- An order that is too large in volume may not be completely fufilled before the market moves, wasting credits. Make small orders (25000 to sell or 10000 to buy is good) which can be extended if they fufill to avoid wasting credits.
- Make sure you don't get the price and amount arguments swapped in `Game.market.createOrder(type, resource, price, amount, room)`!

<!--### Adjust prices for urgency

Sometimes you get invaded, and you have two decisions to make with your terminals.

1) Bring in boosts/energy and defend/repair your way to saftey.
2) Sell/transfer everything in the room's terminal to minimize loss.

In both cases, you are going to want the tranastions to conclude ASAP, especially with the 10 tick terminal cooldown. This is the only case where it is reasonable to undercut the entire sell market, or outprice the entire buy market. 

You may even consider fufilling otherwise unpalatable orders instead of making your own.-->
