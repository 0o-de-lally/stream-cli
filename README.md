# Stream CLI

🚧 **Alpha Testing Notice**

This project is currently in **alpha testing**. It may contain bugs, and features are subject to change. Use caution when running this program, and do not use it with accounts holding significant funds.

---

Stream CLI is a command-line interface tool designed to automate the execution of crypto coin trades based on your strategy. It allows you to schedule trades over a 24-hour period, executing them at random times within specified windows, and adhering to user-defined parameters such as daily amount, number of trades, and minimum acceptable price.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
  - [Initial Setup](#initial-setup)
- [How It Works](#how-it-works)
- [Transaction History](#transaction-history)
- [Terminating the Program](#terminating-the-program)
- [Logging](#logging)
- [Limitations](#limitations)
- [Environments](#environments)
- [Warnings](#warnings)
- [Contributions](#contributions)
- [License](#license)

## Features

- **Automated Trading**: Schedule and execute crypto trades based on your strategy.
- **Trading Summary**: Get detailed reports on stream transactions history.
- **Account Balance Query**: Easily query the balances of an account given its address.
- **Customizable Parameters**: Define daily amounts, number of trades, and minimum prices.
- **Randomized Execution Times**: Trades occur at random times within defined windows.
- **Osmosis Network Integration**: Currently supports the Osmosis blockchain network.
- **Detailed Transaction Logging**: Stores comprehensive transaction details for auditing.

## Installation

To install and run Stream CLI, follow these steps:

1. **Install dependencies**.

```bash
sudo apt install build-essential -y
sudo apt-get install pkg-config -y
sudo apt-get install libssl-dev -y
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

2. **Clone the repository**.

```bash
git clone https://github.com/soaresa/stream-cli.git
```

3. **Navigate to the repository root directory**.

```bash
cd stream-cli
```

4. **Build the program**.

```bash
cargo build
```

## Usage

Stream CLI provides three main commands:

- **`stream`**: Automate your trading strategy by scheduling trades.
- **`summary`**: Generate a summary report of all stream transactions.
- **`balance`**: Query the balances of an account given its address.

### **Stream Command**

To start the streaming trades, use the `stream` subcommand with the appropriate options. You can choose to specify either the amount of tokens you want to **obtain** (via `--daily-amount-out`) or the amount of tokens you want to **sell** (via `--daily-amount-in`).

#### Options:

- `--daily-amount-out`: The total amount of tokens you wish to **obtain** per day.
- `--daily-amount-in`: The total amount of tokens you wish to **sell** per day.
- `--daily-streams`: The number of trades to be executed over 24 hours.
- `--min-price`: The minimum price you are willing to pay per token.

#### Examples:

- **Obtain 20 tokens over 4 trades per day** at a minimum price of 0.1 per token:

```bash
cargo run -- stream --daily-amount-out 20.5 --daily-streams 4 --min-price 0.1
```

- **Sell 1000 tokens over 4 trades per day**, with a target minimum price of 0.1 per token:

```bash
cargo run -- stream --daily-amount-in 1000 --daily-streams 4 --min-price 0.1
```

In these examples, the program will either aim to **obtain** 20.5 tokens throughout the day or **sell** 1000 tokens, executing trades across 4 intervals, depending on which option is provided (`amount-in` or `amount-out`).

### **Summary Command**

To view a summary of all transactions across accounts, use the `summary` subcommand:

```bash
cargo run -- summary
```

This will display key metrics for each account, grouped by pool ID, token in, and token out.

**This summary includes:**

- **Pool ID**: The ID of the liquidity pool.
- **Token In/Out**: The tokens involved in the swap.
- **Transaction Counts**: Total, successful, and failed transactions.
- **Total Tokens In/Out**: Aggregated amounts of tokens exchanged.
- **Average Price**: The average swap price over successful transactions.
- **Total Gas**: The total gas used to execute all streams.
- **Swap Type Counts**: Number of `amount_in` and `amount_out` swaps.

**Example of Output:**

```bash
{
  "osmoyouraddresshere": {
    "15-TOSMO-TUSDC": {
      "pool_id": 15,
      "token_in": "TOSMO",
      "token_out": "TUSDC",
      "tx_total_count": 12,
      "tx_success_count": 12,
      "tx_failed_count": 0,
      "total_tokens_in": "TOSMO 399,996.123456",
      "total_tokens_out": "TUSDC 366,887.123456",
      "average_price": "TUSDC 1.123456",
      "total_gas_used": "TOSMO 1.987654",
      "swap_amount_in_count": 12,
      "swap_amount_out_count": 0
    }
  }
}
```

### **Balance Command**

To query the balances of an account, use the `balance` subcommand:

```bash
cargo run -- balance --address <your_account_address>
```

- `--address`: The account address to query.

**Example:**

```bash
cargo run -- balance --address osmo1youraddresshere
```

### Initial Setup

1. **Enter Your Mnemonic:**

   Upon starting, the program will prompt you to enter your mnemonic (seed phrase). This is required to access your account for executing trades.

2. **Confirm Parameters:**

   The program will display:

   - The parameters you have provided.
   - The account address corresponding to your mnemonic.

   Confirm the details by typing `y` to start the program.

## How It Works

- **Time Division:**

  - The program divides the 24-hour period into the specified number of trade windows (e.g., 4 windows for 4 trades).
  - For each window, it selects a random time to execute the trade.

- **Trade Execution:**

  - At the scheduled time, the program checks:
    - If the current price of the input token is greater than or equal to the user-defined minimum price.
    - If your account has sufficient balance to execute the trade.

- **Retry Mechanism:**

  - If the conditions are not met, the program retries every 5 seconds until the end of the current window.
  - If the trade cannot be executed within the window, it is skipped.
  - A new window begins with a new random trade time.

## Transaction History

- **Storage Location:**

  - Transactions are stored in `~/stream/test/osmosis_transactions.json` or `~/stream/prod/osmosis_transactions.json`, depending on the environment.

- **Transaction Details:**

  Each transaction record includes:

  - `txhash`
  - `timestamp`
  - `pool_id`
  - `token_in`
  - `token_out`
  - `amount`
  - `swap_type`
  - `min_price`
  - `tx_status` (broadcasted, executed, error, timeout)
  - `status_code`
  - `raw_log`
  - `gas_used`
  - `tokens_in`
  - `tokens_out`

- **Transaction Statuses:**

  - **broadcasted**: Transaction has been sent.
  - **executed**: Transaction was executed by validators.
  - **timeout**: No response received within 60 seconds.

  A transaction is considered **successfully executed** if the polling service confirms that it was processed by the validators and the `status_code` returned is `0`. In case of an error, a status_code different from zero is provided, and the raw_log is stored with more details about the error.

## Logging

The application supports configurable logging levels to give users more control over the amount of information displayed during execution. You can set the logging level by using the `RUST_LOG` environment variable.

Available log levels:

- `error`: Only shows error messages.
- `warn`: Shows warning messages and errors.
- `info`: Displays informational messages, warnings, and errors.
- `debug`: Provides detailed debugging information along with info, warnings, and errors.
- `trace`: Displays all logs, including trace-level messages for fine-grained debugging.

To set a log level, you can run the application like this:

```bash
RUST_LOG=info cargo run -- summary
```

This example sets the log level to `info`, showing all informational messages during the application's execution.

## Terminating the Program

To stop the program gracefully, press `Ctrl + C`. This will ensure the program completes its current operations before shutting down, preventing any unexpected interruptions or data loss.

## Limitations

- **Network Support**: Currently, Stream CLI only integrates with the Osmosis network.
- **Configuration**: The pool IDs, gas limits, and tokens for each environment are defined in the `src/config/prod.toml` and `src/config/test.toml` files.

## Environments

- **Setting the Environment:**

  - Modify the `ENVIRONMENT` variable in the `.env` file at the root of the project.
  - Possible values:
    - `prod`: Uses the `osmosis-1` chain.
    - `test`: Uses the `osmo-test-5` chain.

## Warnings

- **Experimental Software**:

  - This repository is an experiment and may contain bugs.
  - Use caution when running this program.

- **Security Notice**:

  - You will need to provide your mnemonic (seed phrase).
  - Ensure you are in a secure environment to prevent compromising your account.

- **Responsibility**:

  - You are responsible for any actions taken by this program.
  - Use at your own risk.

## Contributions

If you encounter any bugs or have suggestions for architectural or security improvements, please:

- **Open an Issue**: Describe the problem or enhancement in detail.
- **Submit a Pull Request**: Contribute code directly to the repository.

---

By following the instructions above, you can automate your crypto trading strategy using Stream CLI. Remember to use this tool responsibly and ensure that you understand the risks involved with automated trading.
