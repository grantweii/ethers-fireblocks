# <h1 align="center"> ethers-fireblocks </h1>

 Provides [ethers](https://github.com/gakonst/ethers-rs)-compatible Signer and Middleware
 implementations for the [Fireblocks Vaults API](https://fireblocks.com).

## Documentation

Clone the repository and run `cd ethers-fireblocks/ && cargo doc --open`

## Add ethers-fireblocks to your repository

```toml
[dependencies]

ethers-fireblocks = { git = "https://github.com/gakonst/ethers-fireblocks" }
```

To use the example, you must have the following env vars set:

 ```
export FIREBLOCKS_API_SECRET_PATH=<path to your fireblocks.key>
export FIREBLOCKS_API_KEY=<your fireblocks api key>
export FIREBLOCKS_SOURCE_VAULT_ACCOUNT=<the vault id being used for sending txs>
```

## Example Usage

 ```rust
use ethers::core::types::{Address, TransactionRequest};
use ethers::fireblocks::{Config, FireblocksMiddleware, FireblocksSigner};
use ethers::providers::{Middleware, Provider};
use std::convert::TryFrom;

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let wallet_id = "1"; // Our wallet id
    let chain_id = 3; // Ropsten
    let cfg = Config::new(
        &std::env::var("FIREBLOCKS_API_SECRET_PATH").expect("fireblocks secret not set"),
        &std::env::var("FIREBLOCKS_API_KEY").expect("fireblocks api key not set"),
        wallet_id,
        chain_id,
    )?;

    // Create the signer (it can also be used with ethers_signers::Wallet)
    let mut signer = FireblocksSigner::new(cfg).await;

    // Instantiate an Ethers provider
    let provider = Provider::try_from("http://localhost:8545")?;
    // Wrap the provider with the fireblocks middleware
    let provider = FireblocksMiddleware::new(provider, signer);

    // Any state altering transactions issued will be signed using
    // Fireblocks. Wait for your push notification and approve on your phone...
    let address: Address = "cbe74e21b070a979b9d6426b11e876d4cb618daf".parse()?;
    let tx = TransactionRequest::new().to(address);
    let pending_tx = provider.send_transaction(tx, None).await?;
    // Everything else follows the normal ethers-rs APIs
    // e.g. we can get the receipt after 6 confs
    let receipt = pending_tx.confirmations(6).await?;

    Ok(())
}
 ```
