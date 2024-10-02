# neon-evm-example

Deploy Solidity smart contracts to Solana using [Neon EVM](https://neonevm.org/).

## Running local test setup

The following will run a local Solana cluster, deploy Neon EVM to it and run the offchain services which expose a standard Ethereum like interface to the user:

`docker compose -f docker-compose.yml pull`

`docker compose -f docker-compose.yml up`

To later clean up everything `docker compose -f docker-compose.yml down`.

We follow the [guide](https://neonevm.org/docs/operating/basic) but deviate from this point on to interact with the EVM proxy in an automated way using foundry scripts.