## FusionDrive
**NOTE: This code is not production ready and is a submission of the Solana Grizzly Hackathon**

----

**FusionDrive** is a business facing project aimed at increasing the revenues of Solana validators, enabling better decentralization of Web3 projects and accelerating the move to web3 by businesses. This repository contains a demo showing how this is made possible by using the Geyser Plugin Interface for Solana.

Further details can be found in the presentation at [https://](https://)

---

### How to run the project

##### Install Rust
You need the Rust toolchain to compile this project which you can install by following instructions for your platform at:
[https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install) .
This project is compiled using Rust version 1.66.1

##### Install Solana Cli Tools
Solana's `solana-test-validator` and `sbf` tools are required to run the project. Install solana by following the instructions at:
[https://docs.solana.com/cli/install-solana-cli-tools](https://docs.solana.com/cli/install-solana-cli-tools)

#### The `Config` Directory
The `Config` directory in this repository provides a set of configuration files to run the geyser plugin and mail server if you wish to send an example email for the demo.
1. **The `mailer.toml` file**
This file contains the configuration settings for the SMTP mail server you can use to send a demo email as passed within the smart contract and it's structure is:
```toml
sender = ["SENDER_NAME", "sender_email_address"]
smtp_uri = "mail_server_uri" #Example `smtp.gmail.com` or `mail.infomaniak.com`
smtp_port = smtp_port #Example `587`
smtp_username = "smtp_username" #Most of the time it's the same as the `sender_email_address`
smtp_password = "smtp_password" #The password to login to the mail server in order to send emails

```
2. **`user_keypair.json`** File
This file contains the user's (payer's) Ed25519 Keypair
3. **`validator_bank.json`** - contains the checking account for the validator where a user would debit their public key using  a smart contract in order for the validator to provide services
4. **`FusionEngineGeyserPluginConfig.json`**
Contains an example configuration of the config file to pass to `solana-test-validator` when initializing it with the Geyser plugin. It's JSON5 contents are:
```json
{
    "libpath": "../target/release/libfusion_engine_geyser.so",
}
```
where the `libpath` is the file path to the compiled geyser plugin in the `target/release` directory and the `validator_config` which is the path to the config file that the geyser plugin will use during runtime.

**NOTE:** The values configuration files can be changed. The can be re-used as they are local to the `solana-test-validator` running on local host so their external exposure does not pose any security risk. 


#### Building and Running the Project

1. Compile the Geyser plugin and start the
    - Compile the plugin
    ```sh
    cargo build --release -p fusion-engine-geyser
    ```
    - Switch to the `Config` directory and start the `solana-test-validator` with the plugin
    ```sh
    solana-test-validator --geyser-plugin-config ./FusionEngineGeyserPluginConfig.json
    ```
2. Compile the Email service

    ```sh
    cargo build --release -p validator-mail-service
    ```
3. Perform a Solana airdrop on the public key of the user in the `user_keypair.json`
    ```sh
    solana airdrop 10 -k Config/user_keypair.json
    ```
4. Switch to the `Geyser-Service-Program` directory and build and deploy the solana program
    - **Build the Program**
    ```sh
    cargo build-sbf
    ```
    - **Deploy the Program**
    ```sh
    solana program deploy ../target/deploy/geyser_service_program.so
    ```
5. Switch to the `Config` directory
    ```sh
    cd Config
    ```
6. Configure the `mailer.toml` file with your SMTP mail settings
7. Start the Validator Mail Service from another terminal and switch to the root directory of this repository.
    ```sh
    ./target/release/validator-mail-service ./Config/mailer.toml
    ```
8. Run the RPC client in the `Geyser-Service-Client` directory
    ```sh
    cargo run -p geyser-service-client
    ```