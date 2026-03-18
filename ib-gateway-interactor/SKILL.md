---
name: ib-gateway-interactor
description: Interact with an Interactive Brokers Gateway to perform trading, query market data, and manage orders. Use this skill when the user wants to write or execute code that connects to the IB Gateway, regardless of whether it's running in paper or live mode.
---

# IB Gateway Interactor Skill

This skill provides specialized procedures and code templates for interacting with an [IB Gateway Docker](https://github.com/gnzsnz/ib-gateway-docker) instance.

## Interaction Fundamentals

### 1. Connection Details
The Gateway exposes specific API ports based on its configuration. The standard ports used by this Docker environment are:
-   **Paper Trading**: `localhost:4002` (Internally 4004)
-   **Live Trading**: `localhost:4001` (Internally 4003)

### 2. Recommended Library
The most popular and idiomatic Python library for interacting with IB is `ib_insync`. 

## Common Interaction Tasks

### Task 1: Basic Connection & Account Info
Always verify the connection and fetch account summary before performing trades.

```python
from ib_insync import *

# Connect to the Gateway
ib = IB()
ib.connect('localhost', 4002, clientId=1) # Use 4002 for paper trading

# Fetch account info
print(f"Connected to Account: {ib.accountValues()[0].account}")
print(f"Total Cash Balance: {[v for v in ib.accountValues() if v.tag == 'TotalCashValue'][0].value}")

# Disconnect when finished
ib.disconnect()
```

### Task 2: Placing a Simulated Market Order (Paper Trading)
To trade, you must first define a `Contract` and then an `Order`.

```python
from ib_insync import *

ib = IB()
ib.connect('localhost', 4002, clientId=1)

# Define a Stock Contract
contract = Stock('AAPL', 'SMART', 'USD')
ib.qualifyContracts(contract)

# Place a Market Order for 10 shares
order = MarketOrder('BUY', 10)
trade = ib.placeOrder(contract, order)

# Wait for completion (simulated)
while not trade.isDone():
    ib.sleep(1)

print(f"Order status: {trade.orderStatus.status}")
ib.disconnect()
```

### Task 3: Querying Current Positions
Use this to check the state of the simulation.

```python
from ib_insync import *

ib = IB()
ib.connect('localhost', 4002, clientId=1)

positions = ib.positions()
for pos in positions:
    print(f"Contract: {pos.contract.symbol}, Amount: {pos.position}, Cost: {pos.avgCost}")

ib.disconnect()
```

## Best Practices
-   **ClientId**: Use a unique `clientId` for each agent or script (e.g., `clientId=1`).
-   **Qualify Contracts**: Always use `ib.qualifyContracts(contract)` to fill in missing details before placing an order.
-   **Wait for Events**: Use `ib.sleep(n)` or event handlers rather than standard `time.sleep(n)` to keep the event loop running.
-   **Security**: Do NOT hardcode credentials if they are required for connection (though IB Gateway handles this via the container's `.env`).
