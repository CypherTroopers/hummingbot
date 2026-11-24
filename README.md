````markdown
# How to Run Hummingbot for CPH/USDT on MEXC

## Introduction
With Hummingbot, you can easily run an automated market-making bot for the CPH/USDT pair on your MEXC trading account and earn trading profits.
This guide walks you through each step, from installing the required software and creating a MEXC API key to launching the bot.

## Prerequisites and Installation
Prepare the following on the server (or PC) that will run the bot. This guide assumes an Ubuntu or Debian Linux server and that you are working as the root user.

**Requirements**
- A Linux server (Ubuntu 22.04 or later recommended, with at least 4 GB of RAM and 20 GB of disk space)
- Git (used to download Hummingbot)
- Make (used to set up and launch Hummingbot)
- Docker and Docker Compose (the environment Hummingbot runs in)
- A time synchronization tool (chrony), used to keep the server clock accurate

**1. Install the basic tools**
```bash
apt update
apt install -y git make curl ca-certificates chrony
```

**2. Install Docker**
```bash
curl -fsSL https://get.docker.com | sh
systemctl enable --now docker
```

**3. Enable time synchronization**
```bash
systemctl enable --now chrony
chronyc tracking
```
If the `System time` offset is 0.1 seconds or less, you're good to go.

**4. Verify the installation**
```bash
git --version
make --version
docker --version
docker compose version
```
If all four commands print a version, you're ready. If you see `docker: not found`, repeat Step 2.

**5. Check your server's location**
```bash
curl -s ipinfo.io
```
MEXC restricts trading from certain countries and regions. Orders placed from a server in a restricted region are rejected with the error `"The area does not support this business under the relevant laws","code":11444`. Before you start, make sure the `country` shown is a region where MEXC is available.

## Preparation: MEXC Account
You need a MEXC trading account to run the bot. If you don't have one yet, create one at [MEXC](https://www.mexc.com/) and complete identity verification (KYC).

## Create an API Key for the Bot on MEXC
The API key is what lets the bot access your MEXC account.

1. Log in to MEXC.
2. Open **API Management** from the profile icon in the top-right corner.
3. Select **Create API Key** and enter a name in the notes field (e.g. `hummingbot-cph`).
4. Under Spot permissions, check only **View Account Details** and **Trade**.
5. If possible, bind the API key to the IP address of the server running the bot (the one you checked in Step 5 above).
6. After completing email and two-factor authentication, your API key (Access Key) and Secret Key will be displayed.

There are two important points. First, the Secret Key is shown only once when it is created, so store it in a safe place. Second, and most importantly, **never enable the Withdraw permission.** This applies to API keys on every exchange.

As the operator, you need to deposit both CPH (for selling) and USDT (for buying) into your account.

The easiest way to install Hummingbot is with Docker. For details, see the [official Hummingbot installation guide](https://hummingbot.org/installation/docker/). The basic steps are as follows:

```bash
git clone https://github.com/hummingbot/hummingbot.git
cd hummingbot
make setup
make deploy
docker attach hummingbot
```

On first launch, you will set a Hummingbot password. Your API keys are encrypted and stored using this password. To leave the Hummingbot screen without stopping the bot, press `Ctrl+P` and then `Ctrl+Q`.

## Launching a Pure Market Making Bot

1. **Connect to the exchange.** Enter `connect mexc`.
2. **Enter your API key and Secret Key.** Paste the keys you created earlier. They will appear as `****` while you type. If they are correct, you will see "You are now connected to mexc."
3. **Create the bot.** Enter `create`.
4. **Choose a strategy.** Enter `pure_market_making`.
5. **Enter the exchange name.** Enter `mexc`.
6. **Enter the trading pair.** Enter `CPH-USDT`.
7. **Enter the buy order spread (bid_spread).** This is how far below the mid price, in percent, your buy order is placed. For example, `0.4` means 0.4% below.
8. **Enter the sell order spread (ask_spread).** Likewise, this is how far above the mid price your sell order is placed. A narrower spread leads to more fills, but make sure you still make a profit after fees. For CPH/USDT, a 1% spread placed orders outside the best bid and ask and rarely got filled, while 0.4% placed them at the best bid and ask.
9. **Enter the order refresh interval (order_refresh_time) in seconds.** For example, `30`. Orders are canceled and replaced at this interval.
10. **Enter the amount of CPH per order (order_amount).** For example, `1000`. Start with a small test amount and increase it as you see how things go.
11. **Choose whether to enable ping-pong.** `No` is fine to start with.
12. **Name and save the config file.** For example, `cph_usdt_test.yml`.

The setup is now complete.

## Starting the Bot
Enter `start` to launch the bot. Use `status` to check your current orders and balances, and `history` to check your trade history and profit and loss. Enter `stop` to stop the bot.

## Things to Watch Out For
- **Keep the server clock accurate.** MEXC is strict about timestamp drift in signed requests, and orders will be rejected if the clock is off (error code 700003). Enable time synchronization with chrony or a similar tool.
- **Regional restrictions.** Depending on where your server is located, orders may be rejected with error code 11444. Hummingbot's log only shows `KeyError: 'orderId'`, so if no orders are being placed, check for this possibility as well.
- **Delay after fills.** By default, the bot waits 60 seconds after a fill before placing the next order (filled_order_delay). Your orders disappear from the order book during this time, but this is normal behavior, not a malfunction.
- **Inventory imbalance.** When the market moves in one direction, orders on one side keep getting filled and you build up either CPH or USDT. Consider using inventory_skew to rebalance your inventory.
- **MEXC listing requirements.** If you need to provide order book depth, increase order_levels (the number of order levels) to place orders across multiple price levels.

## Summary
Once the bot is running, check the order book and trade history regularly, and fine-tune the spread and order amount to optimize performance. Profits depend on market movements and fees, and you may incur losses. Always start with a small amount.
````
