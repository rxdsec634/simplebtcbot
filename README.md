# Binance Futures Testnet Trading Bot

Small Python CLI application for placing MARKET and LIMIT orders on Binance USDT-M Futures Testnet.

## Features

- Places MARKET and LIMIT orders on `https://testnet.binancefuture.com`
- Supports BUY and SELL sides
- Validates CLI input before sending requests
- Uses `python-binance` Futures methods (`futures_create_order`, `futures_exchange_info`, `futures_time`)
- Checks Binance exchange filters before placing orders
- Generates a traceable `newClientOrderId` when one is not supplied
- Syncs Binance server time before signed order calls
- Retries safe GET requests only; POST order placement is never auto-retried
- Logs API requests, responses, validation errors, API errors, and network failures as JSON lines
- Prints a clear order request summary and response summary

## Setup

1. Create and activate a Binance Futures Testnet account.
2. Generate API credentials from the Futures Testnet dashboard.
3. Make sure the testnet account has funds from the testnet faucet.
4. Create a Python virtual environment and install dependencies:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

5. Configure credentials. You can use environment variables:

```powershell
$env:BINANCE_API_KEY="your_testnet_api_key"
$env:BINANCE_API_SECRET="your_testnet_api_secret"
$env:BINANCE_FUTURES_BASE_URL="https://testnet.binancefuture.com"
```

Or copy `.env.example` to `.env` and fill in the values.

You can also let the CLI create `.env` for you. The secret is entered locally and is not printed:

```powershell
python -m trading_bot configure
```

Verify the account link:

```powershell
python -m trading_bot account-check
```

## Usage

Run the CLI from the repository root.

### MARKET order

```powershell
python -m trading_bot --symbol BTCUSDT --side BUY --type MARKET --quantity 0.001
```

### LIMIT order

```powershell
python -m trading_bot --symbol BTCUSDT --side SELL --type LIMIT --quantity 0.001 --price 90000 --time-in-force GTC
```

### Validate without sending

```powershell
python -m trading_bot --symbol BTCUSDT --side BUY --type MARKET --quantity 0.001 --dry-run
```

### Optional flags

- `--api-key` and `--api-secret`: pass credentials directly instead of environment variables
- `--base-url`: override the base URL, defaults to the Binance Futures Testnet URL
- `--log-file`: set a custom log path, defaults to `logs/trading_bot.log`
- `--position-side`: use `BOTH`, `LONG`, or `SHORT` when your account is configured for hedge mode
- `--reduce-only`: submit a reduce-only order
- `--client-order-id`: set a custom Binance client order id
- `--response-type`: choose `ACK` or `RESULT`, default is `RESULT`
- `--max-retries`: retry safe GET calls, defaults to `2`
- `--skip-exchange-validation`: skip pre-trade Binance filter validation
- `--no-time-sync`: skip server time sync before order placement

## Output

The CLI prints:

- Order request summary
- Order response summary: `orderId`, `status`, `executedQty`, and `avgPrice` when Binance returns it
- Raw response details
- Success or failure message

## Production-minded behavior

The CLI stays intentionally small, but the order path has guardrails:

- `python-binance` handles Binance request signing and endpoint formatting.
- The app loads exchange filters and validates symbol status, quantity step size, quantity min/max, LIMIT price tick size, LIMIT price min/max, and LIMIT minimum notional before submitting the order.
- Each order receives a `newClientOrderId`, either from `--client-order-id` or generated as `tb_<24 hex chars>`, so logs and Binance order history can be connected.
- Safe reads such as exchange info and time sync can be retried on transient failures.
- Actual POST order placement is not retried automatically, because a timeout can happen after Binance accepted the order.

## Logging

Runtime logs are written to `logs/trading_bot.log` as JSON lines. The log file includes request parameters and response payloads. API keys/secrets are not written.

Example log files are included:

- `logs/example_market_order.log`
- `logs/example_limit_order.log`

These examples show the expected structure. Real order logs are generated when you run the MARKET and LIMIT examples with valid testnet credentials.

## Project Structure

```text
trading_bot/
  bot/
    client.py
    exceptions.py
    exchange_filters.py
    logging_config.py
    orders.py
    validators.py
  cli.py
  __main__.py
tests/
  test_orders.py
  test_validators.py
requirements.txt
README.md
logs/
  example_market_order.log
  example_limit_order.log
```

## Assumptions

- This project targets Binance USDT-M Futures Testnet only.
- It does not change leverage, margin type, or account position mode.
- `MARKET` orders do not accept a price.
- `LIMIT` orders require `price` and default to `GTC`.
- Exchange-side filters such as minimum quantity, tick size, and step size are checked before submission unless `--skip-exchange-validation` is used.
- The app uses `python-binance` rather than handwritten REST signing.

## Notes

This application uses real signed testnet API calls. Keep credentials private, use testnet keys only, and verify account mode before submitting hedge-mode or reduce-only orders.
