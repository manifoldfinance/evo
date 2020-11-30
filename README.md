<br>

<img src="https://raw.githubusercontent.com/sambacha/evo-whitepaper/master/cover_img.png">
<br>

| EVOProtocol | Embededd Volumetric Optionality Protocol | v1.0.0+0 |
| ----------- | ---------------------------------------- | -------- |


## Embedded Volumetric Optionality Protocol

### Abstract

EVO Protocol is a dynamically adjusting ERC-compatible protocol that adjusts based on _volume_
<br>

EVO tokens are minted and burned on-demand by deposit and withdraw operations directly via the contract.

> Initiated Protocol Operations

-   Deposit <br>
-   Withdraw <br>
-   Transfer <br>

These operations contribute to `transfer rates`.
`Transfer rates` are tracked both in `aggregate` and `individually` (i.e. per address).
The `period of time` for tracking is the last `25 days`.

### Time and Period

> [V2 Upgrade will include upgrading the time and date to a new libray](https://github.com/bokkypoobah/BokkyPooBahsDateTimeLibrary)

Time and Period should be defined on a `per market` basis. Meaning you should choose what is computed to be the _optimal_ time period based on historical analysis.

Multiples of 4,6, etc are suggested

-   For Example
    `25` days has`36000 minutes`, which divided by `block_time=4` gives `9000`

GasEVO is determined both in `aggregate` (dynamically) and `individually` for each address based on transactional (i.e. volumetric transactional information) stored and updated through the smart contract during the previous transactions.
<br>

All three operations such as `deposit`, `withdraw` and `transfer` can equally contribute to the `transfer` rates that are tracked totally and individually(as per holder) by the smart contract for the period of the last `25 days`. <br>
The token price is determined dynamically(and individually for each holder) based on the information stored or updated in the smart contract during previous transactions:

![](https://raw.githubusercontent.com/gist/sambacha/2cd97b61b0a29dd18f0d12fb0029ee73/raw/67c4785230a544558263beb4ede534ad2b3a0bc4/equation.svg)

## Utility

<!-- EN: specifcaiton -->
<!-- DE: spezifikation -->

> Note: This is specific to the implementation based on the reference specification , as described in the whitepaper (./latex/\*_/_.tex)

Given enough liquidity, `GasEVO` has a way to compute the `exchange rate` towards the base instrument (ETH). <br>

Like this, movements of the bigger or significant volumes can be interpreted as market trends (i.e. `gwei` pricing.) <br>

By utilizing small volume movements and disincentivizing the larger ones without compensation to holders every exceeding `bulge bracket` trade of the token is tracked by the smart contract and higher "transactional" fees are applied (re: withdraw, or 'consumption').

> Note: We describe `transactional` fees sometimes as an `interest` fee. This language is marked as _depreciated_ as this confers and/or implies a rate of return that is somewhat deterministic, this however is not the case per se as it is entirely possible that all trades could be below the `transfer rate` during a period/epoch.

Transference of funds _below_ daily volume threshold does not impose any interest fee. <br>

When the threshold has been exceeded some percentage of tokens gets burned, for the transfer, for `deposit` or for `withdraw` of the base instrument (ETH). <br>

Thresholds are tracked individually per address as the average rate and have a function by which they operate on. <br>

---
title: 'EVO Protocol Reference Implementation Overview'
---

EVO Protocol Reference Implementation Overview
===
> [time=Wed, Oct 7, 2020 4:07 AM]




## Table of Contents

[TOC]


### Protocol Overview

EVO tokens are minted and burned **on-demand** by deposit and withdraw operations directly via the contract.

#### Initiated Protocol Operations

* Deposit
* Withdraw
* Transfer

These **operations** contribute to the *transfer rates*. Transfer rates are tracked both **in aggregate** and **individually** (i.e. *per address*). The *period* of time for tracking is the last `25 days`

> 25 days has `36000 minutes`, which *divided by* `block_time=4` gives `9000`

**GasEVO** is determined both in aggregate (dynamically) and individually for each address based on transactional (i.e. volumetric transactional information) stored and updated through the smart contract during the previous transactions. 


All three operations such as deposit, withdraw and transfer can equally contribute to the transfer rates that are tracked totally and individually(as per holder) by the smart contract for the period of the last 25 days.The token price is determined dynamically(and individually for each holder) based on the information stored or updated in the smart contract during previous transactions:


{equation.gasevo}

$$
P_{t+1}(h, a):=\sqrt{\frac{D_{t}}{S_{t}}}+I_{t+1}^{\prime}(h, a)
$$



The above equation will compute the price for a holder $$h$$ to purchase a certain amount of `EVO` tokens in exchange for a base deposit in `ETH/WETH` at the given discrete time - $$t +1$$ , where $$Dt$$ stands for the deposit of `ETH` in the smart contract at previous time - point and $$St$$ stands for the total supply of `EVO` tokens so far.

The first component with the token - base ratio $$Dt/St$$ under the square root is the *indicative price* and **does not depend on the purchase/transferred** amount, ``$a$.``

Ergo, the component $$I_{t+1}^{\prime}(h, a)$$ is called the *discounted interest rate* and it can grow *proportionally* to a within a range of $$[0, 0.24]$$ of ``$$a$$.``

Higher interest payouts can slow down, **deaccelerate**, the price movement. Interest rate determines how fast, or **accleration**, such price can change depending on the market demand & supply pressure for EVO-based tokens. Interest[#] is computed individually for each EVO holder. 

*Note* that all interest payments are contributed to the same common deposit `Dt` on the smart contract, which is supporting the indicative price. This means that interest is shared by all holders that *choose not to trade their tokens* at the moment.

An ERC20 smart contract will contain the information about the balance of every address,

##### Address Information (i.e. wallets)

$$B(h) s.t. Bt + 1(h, a): = Bt(h) + a$$.

In addition to the individual balances, GasEVO contract keeps track about how much each holder has transferred in the last epoch (i.e. 25days)

##### Total average transfer rate for an address

$$avg(Rt + 1(h, a)): = avg(Rt(h)) + a$$

##### Total average daily transfer rate for all holders 
$$avg(R¯ t + 1(h, a)): = avg(R¯t(h)) + a$$.


### Calculations
More formally calculation of the individual interest rate as well as the applied ownership discount can be described in following steps:

For: $$l := 4 , m := 26$$ are the *low* and *high* *transfer rate constants* and 

$$\beta=\frac{\operatorname{avg}\left(B_{t+1}(h, a)\right)}{S_{t+1}}$$, the *future balance ratio*, we resolve $$\tau=\frac{\operatorname{avg}\left(R_{t+1}(h, a)\right)}{\operatorname{avg}\left(\bar{R}_{t+1}(h, a)\right)}$$ is the *future transfer ratio* and $$\theta=\frac{B_{t}(h)}{S_{t}}$$ is the ownership ratio at a *discrete point in `block time`* then we resolve the **interest rate**;

$$
P_{t+1}(h, a):=\sqrt{\frac{D_{t}}{S_{t}}}+I_{t+1}^{\prime}(h, a)
$$

thereby applying the ownership ratio for discount 

$$l_{t+1}^{\prime}(h, a):=\frac{a \times \sqrt{l * \max \left(\min \left(\theta, l^{2}\right), 1\right)}}{100}$$

whereas %$$I$$ is the *discount**, thereby computing the *discounted interest* as,

$$I_{t+1}^{\prime}(h, a):=\max \left(I_{t+1}(h, a), l_{t+1}^{\prime}(h, a)\right)-l_{t+1}^{\prime}(h, a)$$

Price dynamics of equation (1) depends on the transactions volume conducted by all of the involved market participants and bounded by $$O(sqrt(n))$$.

Therefore it can be expected that the demand for EVO Protocol based tokens like GasEVO will be able to represent the *demand* for the value storage, whereas GasEVO represents the value of storage as a derivative function of the underlying asset, Ethereum (i.e. gwei, or as a fixed unit of account for contracting)



---

## Appendix

### Window Calculation

> informative
> 525600 / 4 = 131400
> 131400 / 25
> windows = 5256


###### tags: ``{erc20.balance_wallet}``

### Address Balance
$$B(h) s.t.$$

$B_{t+1}(h, a):=B_{t}(h)+a$

### Adress Balance
$$B(h) s.t. B_{t+1}(h, a):=B_{t}(h)+a$$

### Discrete Transfer Rate 
###### tags: ``{calculation.25day_transfer_rate_discrete}``


### Equations
<!-- 
$$\operatorname{avg}\left(R_{t+1}(h, a)\right):=\operatorname{avg}\left(R_{t}(h)\right)+a$$
-->

### Aggreagte Transfer Rate
###### tags: ``{calculation.transfer_rate_aggregate}``
$$\operatorname{avg}\left(\bar{R}_{t+1}(h, a)\right):=\operatorname{avg}\left(\bar{R}_{t}(h)\right)+a$$

### Future Balance Ratio
###### tags: ``{equations.future_balance:ratio}``
$$\beta=\frac{\operatorname{avg}\left(B_{t+1}(h, a)\right)}{S_{t+1}}$$

###### tags: ``{equations.future-transfer:ratio}``
$$\tau=\frac{\operatorname{avg}\left(R_{t+1}(h, a)\right)}{\operatorname{avg}\left(\bar{R}_{t+1}(h, a)\right)}$$

###### tags:``{equations.ownership:ratio(DISCRETE_POINT)}``
$$\theta=\frac{B_{t}(h)}{S_{t}}$$

###### tags: {equations.gasevo}

``P_(t+1)(h,a):=sqrt((D_(t))/(S_(t)))+I_(t+1)^(')(h,a)``

$$
P_{t+1}(h, a):=\sqrt{\frac{D_{t}}{S_{t}}}+I_{t+1}^{\prime}(h, a)
$$

### 2 Interest Rate
###### tags: {equations.interest_rate}
<!--
$$I_{t+1}(h, a):=\frac{a \times \min (\operatorname{avg}(\beta, \tau), m)}{100}$$
-->

$$I_{t+1}(h, a):=\frac{a \times \min (\operatorname{avg}(\beta, \tau), m)}{100}$$

### 3 Ownership Rate
###### tags: {equations.ownership_ratio}
<!--
l_{t+1}^{\prime}(h, a):=\frac{a \times \sqrt{l * \max \left(\min \left(\theta, l^{2}\right), 1\right)}}{100}
-->

$$l_{t+1}^{\prime}(h, a):=\frac{a \times \sqrt{l * \max \left(\min \left(\theta, l^{2}\right), 1\right)}}{100}$$

### 4 Discounted Interest Rate
###### tags: {equations.discounted_interest_rate}
<!-- 
I_{t+1}^{\prime}(h, a):=\max \left(I_{t+1}(h, a), l_{t+1}^{\prime}(h, a)\right)-l_{t+1}^{\prime}(h, a)
-->

$$I_{t+1}^{\prime}(h, a):=\max \left(I_{t+1}(h, a), l_{t+1}^{\prime}(h, a)\right)-l_{t+1}^{\prime}(h, a)$$

###### tags: {equations.discounted_interest_rate_qed}
<!--
I_{t+1}^{\prime}(h, a) \in[0,0.24]
-->

$$I_{t+1}^{\prime}(h, a) \in[0,0.24]$$

###### tags: {equation.stress_test}

> appendix scenario: *Firesale* 


$$\sqrt{\frac{\left(m-\max \left(l^{\prime}\right)\right) * D_{t}}{100}},$$ $$where; 
m:=26, l:=8$ and S_{t}=B_{t}(h)=1$$

## Errata


### Volumetric Manifolds

> A constructed mechanism for facilitation of efficient and effective contract$^[1]$ trading 

$$
G:=(V, E, w)
$$



## Appendix and FAQ

:::info
**Find this document incomplete?** Leave a comment!
:::


## Security

please contact: `<mailto: sam@manifoldfinance.com>`for bugs/security issues, thank you.

## License

SPDX-License-Identifier: SSPL-1.0
