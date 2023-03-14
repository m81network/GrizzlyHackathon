## FusionDrive
**NOTE: This code is not production ready and is a submission of the Solana Grizzly Hackathon**

----

**FusionDrive** is a business facing project aimed at increasing the revenues of Solana validators, enabling better decentralization of Web3 projects and accelerating the move to web3 by businesses. This repository contains a demo showing how this is made possible by using the Geyser Plugin Interface for Solana.

Further details can be found in the presentation at [https://github.com/m81network/GrizzlyHackathon/blob/master/Documentation/FusionDrive-M81-Network-Solana-Grizzly_Hackathon.pdf](https://github.com/m81network/GrizzlyHackathon/blob/master/Documentation/FusionDrive-M81-Network-Solana-Grizzly_Hackathon.pdf)

---

### How to run the project

##### Install Rust
You need the Rust toolchain to compile this project which you can install by following instructions for your platform at:
[https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install) .
This project is compiled using Rust version 1.66.1

#### Install a reverse-proxy for localhost
To display images in the email receipts sent through the Mail Service, you will need to expose localhost url port 6364 to the internet. This will enable email clients to fetch the resources.
Install bore-cli which is a very easy reverse proxy similar to ngrok.
Source code [https://github.com/ekzhang/bore](https://github.com/ekzhang/bore) .

###### Install using Rust `cargo`
```sh
cargo install bore-cli
```

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
5. **`client_config`** file
This file is used to configure the IP address from which the image assets will be served from. This is necessary when displaying email receipts from your favourite email client. It's TOML contents are:
```toml
ip = "bore.pub:41413" # Don't add http:// . Also https is not currently supported. The default IP is "127.0.0.1:6364"
email = "email_where_to_send_the_transaction_receipt" # Example "support@m81.network
```

**NOTE:** The values configuration files can be changed. They can be re-used as they are local to the `solana-test-validator` running on local host so their external exposure does not pose any security risk. 


#### Building and Running the Project

1. Switch to the `Geyser-Service-Program` directory and build the solana program. Building the S0lana program first generates the program keypair which is necessary for other steps to work.
    ```sh
    cargo build-sbf
    ```

2. Compile the Geyser plugin and start the
    - Compile the plugin
    ```sh
    cargo build --release -p fusion-engine-geyser
    ```
    - Switch to the `Config` directory and start the `solana-test-validator` with the plugin
    ```sh
    solana-test-validator --geyser-plugin-config ./FusionEngineGeyserPluginConfig.json
    ```
3. Switch to the `Geyser-Service-Program` directory and deploy the solana program. 
    ```sh
    solana program deploy ../target/deploy/geyser_service_program.so
    ```
4. Switch to the root directory of the project. Configure the `Config/mailer.toml` file with your SMTP mail settings and run the Email service from another terminal and keep it running in the foreground

    ```sh
    cargo run -p validator-mail-service -- Config/mailer.toml
    ```
5. Perform a Solana airdrop on the public key of the user in the `user_keypair.json`
    ```sh
    solana airdrop 10 -k Config/user_keypair.json
    ```
6. Start the Validator CDN Service from another terminal inside root directory of this repository.
    ```sh
    cargo run
    ```
7. Start the bore app in order to expose the CDN server to the internet for Email clients to fetch images required to be displayed in the receipt. The CDN server is running on port `6364`
   ```sh
   bore local 6364 --to bore.pub
   ```
   Some logs are displayed on the terminal. Copy the socket where bore app is listening. An example as displayed in the terminal is `listening at bore.pub:33019` and an example log is below
   ```sh
   2023-03-14T18:31:22.827379Z  INFO bore_cli::client: connected to server remote_port=33019
   2023-03-14T18:31:22.827483Z  INFO bore_cli::client: listening at bore.pub:33019
    ```
8. Modify the `Config/client_config.toml` file with this socket address you copied from the previous step. Also add the email address where you want the demo transaction receipt to be sent to. Example:
   ```toml
    ip = "bore.pub:33019" # The socket address you copied from step 7
    email = "support@m81.network"
   ```

9.  Run the RPC client in the `Geyser-Service-Client` directory to demo the email receipt notification and check your email address when the client prints a transaction signature.
    ```sh
    cargo run -p geyser-service-client -- mail
    ```

10.  Run the RPC client in the `Geyser-Service-Client` directory to demo the CDN network that can be used to display HTML receipts on a thin client
```sh
cargo run -p geyser-service-client -- mail
```

##### That's it.
You have shown how the Solana validator and nodes network can be used to decentralize web2 business facing services at the same time increasing earnings for validator and node operators using micro-transactions.
