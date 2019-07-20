## Quickstart

### Prerequisites

The only required software that you must have installed are `docker` and `docker-compose`.

If you don't already have them installed, you can follow [this link](https://docs.docker.com/compose/install/) to install them (free).

### Initial Setup

1.  **Clone this repo**

        git clone https://gitlab.com/tubackkhoa/novalex-dex-starterkit

2.  **Change your working directory**

        cd novalex-dex-starterkit

3.  **Build and launch your nova relayer**

        docker-compose up -d

    This step may takes a few minutes.
    When complete, it will start all necessary services.

    It will use ports `3000`, `3001`, `3002`, `6379`, `8043`, and `8545` on your computer. Please make sure theses ports are available.

4.  **Check out your relayer**

    Open http://localhost:3000/ on your browser to see your exchange in action!

    ***

    ![web-screen-shot](assets/novalex_dex_scaffold_screenshot.jpg)

5.  **Change web and backend code**
    docker-compose stop web
    cd web
    API_HOST=192.168.1.3 yarn start

    docker-compose stop admin
    cd admin  
    make all

    Or run group of services: `docker-compose up -d redis ethereum-node db watcher engine launcher maker ws admin api web`

## Testdrive Your DEX

Now that your DEX is up on your local server, let's try it out a bit.

1. **Connect a wallet**

   You can connect to a wallet by clicking the button at right-top corner. We have already prepared an address for you.

   You can find it under the `Browser Wallet` type. The address is:

- public key: `0x31ebd457b999bf99759602f5ece5aa5033cb56b3`
- private key: `0xb7a0c9d2786fc4dd080ea5d619d36771aeb0c8c26c290afd3451b92ba2b7bc2c`

2. **Test out the trading flow**

   You might have noticed that we setup a simple market making bot on the HOT-DAI market. Try to make some trades on this active market.

## Configuring Your DEX

Our Starterkit come with a powerful API and easy Command Line Interface (CLI) for configuring your DEX.

1.  **Login to the CLI**

        docker-compose exec admin sh

2.  **View the CLI manual to see a list of functions you can perform**

    See [admin cli manual](admin-cli.md#cli-guide-admin-cli)

3.  **Try creating a new market**

        nova-dex-ctl market new HOT-WWW \
          --baseTokenAddress=0x4c4fa7e8ea4cfcfc93deae2c0cff142a1dd3a218 \
          --quoteTokenAddress=0xbc3524faa62d0763818636d5e400f112279d6cc0

- The base token is the first symbol (HOT above), the quote token is the second symbol (WWW above).
- You could try this with different token symbols and contract addresses.
- This creates a market with "default parameters" for fees, decimals, etc.

        nova-dex-ctl market publish HOT-WWW

- This makes the market viewable on the frontend

4.  **Exit the CLI**

        exit

    This will exit out of the CLI and go back to the original terminal.

### Questions?

You now have a fully functioning DEX on your local system, complete with a CLI for easy customization.

1. **Support Channels**

   We're happy to help you through the setup process at anytime. Our [Nova Relayer Slack Channel](https://novalextechnologies.slack.com/messages/CHX7K06L8/) is designed specifically for this. Come say hi and show off your DEX!

   Our team has thorough exchange experience and would be happy to talk over details of what it takes to operate a successful relayer.

2. **Deploying your DEX**

   Check out our [Developer Documentation](config-nginx.md).

---

# Additional Info

## Useful Docker Commands

1.  **Display the status of all services running**

         docker-compose ps

    This command displays the status of all services running in docker. It's helpful for troubleshooting and for understanding the combination of components that goes into running your DEX.

2.  **Stopping your DEX**

         docker-compose stop

    This command will stop all of the current services running in docker.

3.  **Restarting your DEX**

         docker-compose up -d

    The same command that you ran to start it the first time can be used for subsequent restarts. Always run the pull command first, as the docker-compose up command will not run without an image.

4.  **View logs**

        # view logs of the service that defined in docker-compose.yml services
        # e.g. view watcher log
        docker-compose logs --tail=20 -f watcher
        # e.g. view api log
        docker-compose logs --tail=20 -f api

    Much like the status, viewing the logs can give you an idea of the specific details involved in each service.

5.  **Update this repo**

        git pull origin master

6.  **Completely clean the old state (data will be deleted)**

        docker-compose down -v

## What all comes in this Starterkit?

- Frontend:
  - A Basic Exchange Web UI
- Backend:
  - Http Server to serve API data
  - Websocket Server to handle keepalive connections and serve realtime data
  - Matching Engine to send matching orders to the nova smart contracts
  - Blockchain Monitor to watch for transaction changes on the blockchain
- [PostgresSQL](https://www.postgresql.org) database
- [ganache-cli](https://github.com/trufflesuite/ganache-cli) to run a local ethereum node and support for ropsten and mainnet

## F&Q

- [How to run on other network and run from source?](change-network.md)
- [How to configure for external access?](config-nginx.md)

## How to setup environment for local development

See [setup dev env manual](setup-dev-env.md).

## License

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details
