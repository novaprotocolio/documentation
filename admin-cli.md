Nova provides a basic CLI to make configuring your DEX simple and easy. This document:

- You can also use [Admin API](admin-api.md#admin-api)

_Note that because this API controls important fundamental elements of Nova DEX, it is important to secure this API against unwanted access._

---

## CLI Guide (admin-cli)

If you are using docker-compose to run your nova relayer, you can login into the admin service by entering:

    	docker-compose exec admin sh

This enters the Admin CLI. Once you are logged in, you can use the commands detailed below to configure your DEX. To exit the CLI, type `exit`

### Commands

#### Show help

```bash
nova-dex-ctl help
```

#### Get dex status

```bash
nova-dex-ctl status
```

#### Manage markets

```bash
nova-dex-ctl market help
```

#### Get all markets

```bash
nova-dex-ctl market list
```

#### Create a new market

When creating a new market, you can choose to either:

- use default options for the majority of the parameters
- specify all parameters

To use default options, you only need to specify the base and quote token addresses for your trading pair. You can always edit these parameters later.

```bash
# Default market creation: specify the token addresses for your trading pair
nova-dex-ctl market new HOT-WWW \
  --baseTokenAddress=0x4c4fa7e8ea4cfcfc93deae2c0cff142a1dd3a218 \
  --quoteTokenAddress=0xbc3524faa62d0763818636d5e400f112279d6cc0

# create a new market and specify all attributes
nova-dex-ctl market new HOT-WWW \
  --baseTokenAddress=0x4c4fa7e8ea4cfcfc93deae2c0cff142a1dd3a218 \
  --quoteTokenAddress=0xbc3524faa62d0763818636d5e400f112279d6cc0 \
  --minOrderSize=0.1 \
  --pricePrecision=5 \
  --priceDecimals=5 \
  --amountDecimals=5 \
  --makerFeeRate=0.001 \
  --takerFeeRate=0.002 \
  --gasUsedEstimation=150000

```

#### Approve market tokens.

In order to make trades with a new token pair, the relayer must first set token allowance permissions for the new market. To do this, enter:

```bash
nova-dex-ctl market approve HOT-WETH
```

In the example above, the Relayer is approving both HOT and WETH (if not already approved) for trading.

#### Update a market

To change parameters in an existing market, specify the market and the parameter you wish to modify.

```bash
nova-dex-ctl market update HOT-WWW --amountDecimals=3
```

#### Publish a market

The market selection area on the frontend will only show markets that are published.

```bash
nova-dex-ctl market publish HOT-WETH
```

#### Unpublish a market

Unpublishing a market will make the market no exist in the frontend market selection area.

```bash
nova-dex-ctl market unpublish HOT-WETH

```

#### Change fee structure for a market

You can modify the fee structure for each market. Nova supports assymetric fee structures - so you can have the order maker and order taker automatically pay different fees.

```bash
# set the HOT-WETH market makerFee to 0.1% and takerFee to 0.3%

nova-dex-ctl market changeFees HOT-WETH "0.001" "0.003"
```
