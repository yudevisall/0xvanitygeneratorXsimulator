# 0xvanityGeneratorXsimulator
# TELEGRAM https://t.me/Dark0xDev


# Crypto Lab Monitor & Controlled Payroll Infrastructure

A controlled Ethereum laboratory infrastructure built with **Rust and Python** for blockchain transaction monitoring, address extraction, vanity-address generation, and controlled payroll-flow testing.

The system is designed for a controlled lab environment where the wallets and transactions used for testing are owned or authorized by the project team.

---

## 1. Overview

The infrastructure consists of two primary components:

* **Rust Monitor** — monitors Ethereum blockchain activity through an Alchemy WebSocket connection, identifies qualifying transactions, extracts destination addresses, and manages the vanity-address generation queue.
* **Python Payroll Engine** — consumes generated worker wallets and executes the controlled payroll workflow using Ethereum Mainnet.

### High-level flow

```text
Ethereum Mainnet
       │
       ▼
┌──────────────────────┐
│   Rust Blockchain    │
│       Monitor        │
└──────────┬───────────┘
           │
           │ Qualifying addresses
           ▼
  data/to_addresses.txt
           │
           ▼
┌──────────────────────┐
│  Vanity Generator    │
│       3 + 3          │
└──────────┬───────────┘
           │
           ▼
data/generated_addresses.txt
           │
           ▼
┌──────────────────────┐
│  Python Payroll      │
│       Engine         │
└──────────┬───────────┘
           │
           ▼
      Ethereum Mainnet
```

---

# 2. Architecture

## Rust Monitor

The Rust component is responsible for:

* Connecting to Ethereum Mainnet through WebSocket.
* Monitoring blockchain activity.
* Monitoring ETH, USDT and USDC transfers.
* Applying the configured USD transaction range.
* Extracting qualifying destination addresses.
* Writing addresses to the address queue.
* Watching the queue continuously.
* Passing addresses to the vanity generator.
* Generating 3-character prefix + 3-character suffix matches.
* Saving generated worker-wallet information.

The Rust monitor is the primary blockchain monitoring component.

---

## Python Payroll Engine

The Python component is responsible for:

* Loading generated worker wallets.
* Loading tax-wallet addresses.
* Connecting to Ethereum Mainnet.
* Obtaining the current ETH/USD price.
* Calculating payroll amounts.
* Building and signing transactions.
* Broadcasting transactions.
* Waiting for transaction receipts.
* Recording payroll activity.
* Maintaining payroll state.

The controlled payroll workflow is:

```text
CEO Wallet
    │
    │ Gross salary
    ▼
Worker Wallet
    │
    │ Tax amount
    ▼
Tax Wallet

Worker Wallet
    │
    └── Retains net salary
```

---

# 3. Project Structure

```text
crypto-lab-monitor/
├── README.md
├── Cargo.toml
├── Cargo.lock
├── .env
├── .gitignore
├── start_lab.sh
│
├── src/
│   └── main.rs
│
├── python/
│   ├── payroll.py
│   ├── blockchain.py
│   ├── storage.py
│   │
│   └── tests/
│       ├── test_ceo.py
│       └── test_rpc.py
│
├── data/
│   ├── to_addresses.txt
│   ├── generated_addresses.txt
│   ├── payroll_transactions.jsonl
│   ├── crypto_lab.db
│   └── monitor.db
│
└── lab_targets.txt
```

Runtime files such as databases, logs, state files and generated wallet data are created by the application when required.

---

# 4. Requirements

## Hardware

The system can run on a normal development computer.

For continuous 24/7 operation, a Linux VPS is recommended.

Recommended VPS configuration:

```text
CPU:      2 vCPU
RAM:      4 GB
Storage:  40 GB+
OS:       Ubuntu 24.04 LTS
```

---

## Software

Install:

* Git
* Rust
* Cargo
* Python 3.11+
* pip
* virtual environment support
* OpenSSL/system build tools where required

---

# 5. Clone the Repository

Clone the project:

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
```

Enter the project:

```bash
cd crypto-lab-monitor
```

The vanity generator repository should be available alongside the monitor project:

```text
vanity/
├── crypto-lab-monitor/
└── eth-vanity/
```

The generator must be built before running the complete infrastructure.

---

# 6. Rust Setup

Check Rust:

```bash
rustc --version
cargo --version
```

Build the monitor:

```bash
cargo build --release
```

Run the monitor:

```bash
cargo run --release
```

---

# 7. Vanity Generator

The project uses the `eth-vanity` generator for controlled address generation.

Build the generator:

```bash
cd ../eth-vanity
cargo build --release
```

The resulting executable should be available at:

```text
eth-vanity/target/release/eth-vanity
```

Return to the monitor:

```bash
cd ../crypto-lab-monitor
```

The generator performs a **3 + 3 hexadecimal pattern search**.

Example:

```text
Reference:
0x9b6......766c

Generated pattern:
0x9b6...66c
```

The generated address and associated wallet information are stored in:

```text
data/generated_addresses.txt
```

---

# 8. Python Setup

Check Python:

```bash
python3 --version
```

Install the required Python packages:

```bash
pip3 install -r requirements.txt
```

If a requirements file is not provided, install the packages used by the Python modules according to the project environment.

---

# 9. Environment Configuration

Create the environment file:

```bash
touch .env
```

Example configuration:

```env
ALCHEMY_API_KEY=YOUR_ALCHEMY_API_KEY
ETH_WS_URL=YOUR_ALCHEMY_MAINNET_WEBSOCKET_URL

MIN_USD=100
MAX_USD=50000
MAX_SELECTED=300
WINDOW_HOURS=12

CEO_PRIVATE_KEY=YOUR_LAB_CEO_PRIVATE_KEY
RPC_URL=https://eth-mainnet.g.alchemy.com/v2/YOUR_ALCHEMY_API_KEY

WORKER_START=1
MAX_WORKERS=0

SKIP_PENDING_WORKERS=true

RECEIPT_TIMEOUT=180
POLL_INTERVAL=5
WATCH_INTERVAL=3
```

### Important

Never commit `.env` to GitHub.

The `.gitignore` file should contain:

```gitignore
.env
__pycache__/
*.pyc
target/
```

Private keys must remain local or inside the secured VPS environment.

---

# 10. Ethereum Network

The current infrastructure is configured for:

```text
Network: Ethereum Mainnet
Chain ID: 1
```

Before running transactions, verify the connection:

```bash
python3 python/blockchain.py
```

A successful read-only test should display information similar to:

```text
Connected:        True
Chain ID:         1
Block:            XXXXXXXX
CEO:              0x...
CEO balance:      X.XXXXXXXX ETH
ETH/USD:          $XXXX.XX
CEO nonce:        latest=X pending=X pending_count=0
```

The blockchain module test is read-only.

---

# 11. Running the Rust Monitor

From the project root:

```bash
cargo run --release
```

The monitor should display essential startup information:

```text
[START] Crypto Lab Monitor
[PRICE] ETH $.... | USDT $1.00 | USDC $1.00
[QUEUE] Watching data/to_addresses.txt
[WINDOW] #1 started
[WS] Connecting...
[WS] Subscriptions active
```

When a qualifying transaction is detected:

```text
[SELECTED] ETH $... | To: 0x... | TX: 0x...
[SAVED] TO address → 0x...
```

The address is then added to the generation queue.

---

# 12. Address Generation Queue

The Rust monitor continuously watches:

```text
data/to_addresses.txt
```

When a new address appears:

```text
[QUEUE] New reference → 0x...
[GEN] Searching 3+3 | Reference: 0x...
```

After successful generation:

```text
[GEN] Complete
[GEN] Saved → data/generated_addresses.txt
```

The generated worker information becomes available to the Python payroll engine.

---

# 13. Running the Payroll Engine

Start the payroll engine:

```bash
python3 python/payroll.py
```

Startup output should look similar to:

```text
=================================================================
              PAYROLL ENGINE STARTING
=================================================================
[RPC] Connected | Mainnet | Block XXXXXXXX
[DATA] Workers: X | Tax wallets: X | New workers: X
```

When a worker is available:

```text
=================================================================
[WORKER] Processing Worker #1
=================================================================
```

The payroll engine then performs the controlled payroll sequence.

---

# 14. Controlled Payroll Model

The current laboratory test uses:

```text
Gross salary: $1.00
Tax amount:   $0.10
```

Conceptually:

```text
CEO
 │
 │ $1.00
 ▼
Worker
 │
 │ $0.10 tax
 ▼
Tax Wallet
```

The worker retains the remaining amount after tax and transaction fees.

All transaction testing should use wallets controlled or explicitly authorized by the project team.

---

# 15. Testing

## Blockchain Test

Run:

```bash
python3 python/blockchain.py
```

This checks:

* RPC connectivity
* Ethereum chain ID
* latest block
* CEO address
* CEO balance
* ETH/USD price
* nonce
* pending transaction count

---

## Storage Test

Run:

```bash
python3 python/storage.py
```

This checks:

* worker loading
* tax-wallet loading
* state handling
* transaction-log handling
* storage integrity

The storage module test is read-only.

---

# 16. Running the Complete Local Infrastructure

The project includes:

```bash
./start_lab.sh
```

This starts:

```text
Rust Monitor
     +
Python Payroll Watcher
```

The script can be stopped with:

```text
Ctrl+C
```

For development and debugging, running the Rust and Python components in separate terminals is also recommended.

---

# 17. Recommended Development Workflow

For debugging, run the components separately.

### Terminal 1 — Rust

```bash
cd ~/Desktop/vanity/crypto-lab-monitor
cargo run --release
```

### Terminal 2 — Python

```bash
cd ~/Desktop/vanity/crypto-lab-monitor
python3 python/payroll.py
```

This makes it easier to identify whether a problem originates from:

```text
Ethereum
   ↓
Rust Monitor
   ↓
Address Queue
   ↓
Vanity Generator
   ↓
Generated Worker
   ↓
Python Payroll
   ↓
Transaction
```

---

# 18. 24/7 VPS Deployment

For continuous operation, the infrastructure should run on a Linux VPS rather than relying on a personal computer.

Recommended architecture:

```text
GitHub
   │
   │ git clone / git pull
   ▼
Ubuntu VPS
   │
   ├── Rust Monitor
   ├── Vanity Generator
   ├── Address Queue
   ├── Worker Data
   └── Python Payroll Engine
            │
            ▼
       Ethereum Mainnet
```

The VPS should use `systemd` to supervise the Rust and Python processes independently.

This provides:

* automatic restart
* process supervision
* startup after reboot
* centralized logs
* independent monitoring of Rust and Python

---

# 19. Security

This project handles blockchain wallet information and transaction signing.

### Never commit:

```text
.env
private keys
wallet backups
seed phrases
production credentials
API secrets
```

Never paste private keys into:

* GitHub
* public documentation
* Discord
* Telegram
* issue trackers
* chat messages
* screenshots

Use dedicated laboratory wallets for testing.

For Mainnet testing, maintain only the funds required for the controlled experiment.

---

# 20. Troubleshooting

### `OutOfFunds`

Example:

```text
EVM error: OutOfFunds
```

The sending wallet does not have enough ETH to cover the transaction value plus gas.

Check:

```bash
python3 python/blockchain.py
```

---

### CoinGecko `429`

Example:

```text
429 Client Error: Too Many Requests
```

This indicates that the price API has rate-limited requests.

Avoid repeatedly restarting the payroll process or making unnecessary price requests.

---

### Workers show `0`

Check:

```bash
wc -c data/generated_addresses.txt
```

Then:

```bash
cat data/generated_addresses.txt
```

If the file is empty, the issue is upstream of the payroll engine.

---

### Rust generator not found

Verify:

```bash
ls -la ../eth-vanity/target/release/eth-vanity
```

If it does not exist:

```bash
cd ../eth-vanity
cargo build --release
```

---

### Wrong Ethereum network

Verify:

```bash
python3 python/blockchain.py
```

The expected chain ID is:

```text
1
```

---

# 21. Data Flow Summary

```text
Ethereum Mainnet
       │
       ▼
Rust WebSocket Monitor
       │
       ▼
Qualifying Transaction
       │
       ▼
to_addresses.txt
       │
       ▼
3+3 Vanity Generator
       │
       ▼
generated_addresses.txt
       │
       ▼
Python Worker Loader
       │
       ▼
Payroll Calculation
       │
       ▼
CEO → Worker
       │
       ▼
Worker → Tax Wallet
       │
       ▼
Transaction Records
```

---

# 22. Project Purpose

This infrastructure provides a controlled laboratory environment for studying:

* blockchain transaction monitoring
* Ethereum WebSocket infrastructure
* transaction filtering
* address extraction
* vanity address generation
* wallet management
* transaction construction
* transaction signing
* transaction broadcasting
* transaction confirmation
* payroll-style blockchain workflows
* automated blockchain infrastructure
* system reliability and 24/7 operation

All transaction experiments should remain within wallets and environments controlled or authorized by the project team.

---

## License

Add the project's applicable license here.

---

## Status

**Current network:** Ethereum Mainnet

**Monitoring:** Rust

**Payroll:** Python

**Address generation:** Rust + `eth-vanity`

**Blockchain RPC:** Alchemy

**Price data:** CoinGecko

**Deployment target:** Linux VPS + systemd
