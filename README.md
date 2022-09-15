> This library is no longer in active developement. Use [BOOT](https://github.com/Abstract-OS/BOOT) instead!

# Cosmos Rust Script

Smart contract scripting library to ease [CosmWasm](https://cosmwasm.com/) smart contract development and deployment.

> [cosm-script](cosm-script/README.md) is inspired by [terra-rust-api](https://github.com/PFC-Validator/terra-rust) and uses [cosmos-rust](https://github.com/cosmos/cosmos-rust) for [protocol buffer](https://developers.google.com/protocol-buffers/docs/overview) parsing.

[cw-plus-script](cw-plus-script/README.md) uses cosm-script to provide the standard type-safe interfaces to interact with cosmwasm-plus contracts.

The use of this software makes it easier to quickly deploy and iterate on your contracts. You should use this function responsibly when working on mainnet or testnet as ALL the code you upload to those networks takes up valuable space. Therefore I strongly suggest using a locally-hosted chain like [localterra](https://github.com/terra-money/LocalTerra), [local junod](https://docs.junonetwork.io/smart-contracts-and-junod-development/junod-local-dev-setup), etc. 
.
## How it works

Usually your contracts workspace will have a package that contains the structs that get filled by a provided JSON through the FFI on execution by the CosmWasm VM. 
We can easily access these endpoint structs (InstantiateMsg, ExecuteMsg, QueryMsg, ...) by adding that package as a dependency to the scripting workspace. 

In order to perform actions on the contract we need to specify these structs so the compiler can type-check our actions. This prevents us from executing a faulty message on a contract and it also handles converting the structs to their json format. The implementation for a CW20 token is shown below. The full file resides [here](cw-plus-script/src/cw20.rs)

```
// Wrapper stuct around the contract instance.
pub struct CW20<'a>(ContractInstance<'a>);

// Interface and instance traits allow for an auto-implementation of our Cosm traits.
impl Interface for CW20<'_> {
    type Exec = ExecuteMsg;
    type Init = InstantiateMsg;
    type Query = QueryMsg;
    type Migrate = NotImplemented;
}

impl Instance for CW20<'_> {
    fn instance(&self) -> &ContractInstance {
        &self.0
    }
}

impl CW20<'_> {
    /// Create a new CW20 ContractInstance. Uses "cw20" as code-id key.
    pub fn new<'a>(
        name: &'a str,
        sender: Wallet<'a>,
        deployment: &'a Deployment,
    ) -> anyhow::Result<CW20<'a>> {
        let mut instance = ContractInstance::new(name, sender, deployment)?;
        // We want all our CW20 tokens to use the same contract (code-id)!
        instance.overwrite_code_id_key("cw20");
        Ok(CW20(instance))
    }
    ...
}
```
After implementing these traits you have the ability to use all the functions provided by the WasmInstantiate, WasmExecute, WasmQuery and WasmMigrate traits. This way our script calls are very ergonomic: 

```
    let token_info = cw20_token.query(Cw20QueryMsg::TokenInfo {}).await?;
```

# Contributing
Feel free to open issues or PRs!
