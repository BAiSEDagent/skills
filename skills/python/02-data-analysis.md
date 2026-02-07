# On-Chain Data Analysis with Python

## Overview

Python excels at processing and analyzing blockchain data. This lesson covers using pandas for dataframe operations, processing event logs, calculating DeFi metrics, visualizing data with matplotlib, and building data pipelines for continuous analysis.

## Setup and Dependencies

Install required packages:

```bash
pip install web3 pandas numpy matplotlib seaborn python-dateutil
```

Basic imports:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from web3 import Web3
from datetime import datetime, timedelta
from typing import List, Dict, Any

# Set pandas display options
pd.set_option("display.max_columns", None)
pd.set_option("display.width", None)
pd.set_option("display.max_colwidth", 50)

# Set matplotlib style
plt.style.use("seaborn-v0_8-darkgrid")
sns.set_palette("husl")
```

## Processing Event Logs into DataFrames

Convert Web3 events to pandas DataFrame:

```python
from web3 import Web3
import pandas as pd
from datetime import datetime

w3 = Web3(Web3.HTTPProvider("https://mainnet.base.org"))
USDC_ADDRESS = "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913"

TRANSFER_EVENT_ABI = [{
    "anonymous": False,
    "inputs": [
        {"indexed": True, "name": "from", "type": "address"},
        {"indexed": True, "name": "to", "type": "address"},
        {"indexed": False, "name": "value", "type": "uint256"}
    ],
    "name": "Transfer",
    "type": "event"
}]

usdc = w3.eth.contract(address=Web3.to_checksum_address(USDC_ADDRESS), abi=TRANSFER_EVENT_ABI)

def events_to_dataframe(events, decimals=6):
    """
    Convert Web3 event logs to pandas DataFrame.
    """
    records = []
    for event in events:
        block = w3.eth.get_block(event["blockNumber"])
        timestamp = datetime.fromtimestamp(block["timestamp"])
        
        record = {
            "block_number": event["blockNumber"],
            "timestamp": timestamp,
            "transaction_hash": event["transactionHash"].hex(),
            "log_index": event["logIndex"],
            "from_address": event["args"]["from"],
            "to_address": event["args"]["to"],
            "value": event["args"]["value"] / (10 ** decimals)
        }
        records.append(record)
    
    df = pd.DataFrame(records)
    df["timestamp"] = pd.to_datetime(df["timestamp"])
    return df

# Fetch events from last 1000 blocks
latest_block = w3.eth.block_number
from_block = latest_block - 1000

transfer_filter = usdc.events.Transfer.create_filter(
    fromBlock=from_block,
    toBlock="latest"
)

events = transfer_filter.get_all_entries()
df = events_to_dataframe(events)

print(f"Loaded {len(df)} transfer events")
print(df.head())

# Basic statistics
print(f"\nTransfer Statistics:")
print(f"Total volume: {df['value'].sum():,.2f} USDC")
print(f"Average transfer: {df['value'].mean():,.2f} USDC")
print(f"Median transfer: {df['value'].median():,.2f} USDC")
print(f"Max transfer: {df['value'].max():,.2f} USDC")
print(f"Unique senders: {df['from_address'].nunique()}")
print(f"Unique receivers: {df['to_address'].nunique()}")
```

## Time-Series Analysis

Analyze transfer volume over time:

```python
import pandas as pd
import matplotlib.pyplot as plt

# Assuming df is loaded from previous example

# Resample to hourly buckets
df_hourly = df.set_index("timestamp").resample("1H").agg({
    "value": ["sum", "count", "mean"],
    "from_address": "nunique",
    "to_address": "nunique"
})

df_hourly.columns = ["total_volume", "transfer_count", "avg_transfer", "unique_senders", "unique_receivers"]
df_hourly = df_hourly.reset_index()

print(df_hourly.head())

# Plot volume over time
fig, axes = plt.subplots(2, 2, figsize=(15, 10))

# Total volume
axes[0, 0].plot(df_hourly["timestamp"], df_hourly["total_volume"], marker="o")
axes[0, 0].set_title("Hourly USDC Transfer Volume")
axes[0, 0].set_xlabel("Time")
axes[0, 0].set_ylabel("Volume (USDC)")
axes[0, 0].grid(True)

# Transfer count
axes[0, 1].bar(df_hourly["timestamp"], df_hourly["transfer_count"], width=0.03)
axes[0, 1].set_title("Hourly Transfer Count")
axes[0, 1].set_xlabel("Time")
axes[0, 1].set_ylabel("Count")
axes[0, 1].grid(True)

# Average transfer size
axes[1, 0].plot(df_hourly["timestamp"], df_hourly["avg_transfer"], color="green", marker="s")
axes[1, 0].set_title("Average Transfer Size")
axes[1, 0].set_xlabel("Time")
axes[1, 0].set_ylabel("USDC")
axes[1, 0].grid(True)

# Unique users
axes[1, 1].plot(df_hourly["timestamp"], df_hourly["unique_senders"], label="Senders", marker="^")
axes[1, 1].plot(df_hourly["timestamp"], df_hourly["unique_receivers"], label="Receivers", marker="v")
axes[1, 1].set_title("Unique Users per Hour")
axes[1, 1].set_xlabel("Time")
axes[1, 1].set_ylabel("Count")
axes[1, 1].legend()
axes[1, 1].grid(True)

plt.tight_layout()
plt.savefig("usdc_analysis.png", dpi=300, bbox_inches="tight")
plt.show()
```

## DeFi Protocol Analytics

Analyze Aerodrome DEX activity:

```python
from web3 import Web3
import pandas as pd
import numpy as np

w3 = Web3(Web3.HTTPProvider("https://mainnet.base.org"))
AERODROME_ROUTER = "0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43"

SWAP_EVENT_ABI = [{
    "anonymous": False,
    "inputs": [
        {"indexed": True, "name": "sender", "type": "address"},
        {"indexed": False, "name": "amount0In", "type": "uint256"},
        {"indexed": False, "name": "amount1In", "type": "uint256"},
        {"indexed": False, "name": "amount0Out", "type": "uint256"},
        {"indexed": False, "name": "amount1Out", "type": "uint256"},
        {"indexed": True, "name": "to", "type": "address"}
    ],
    "name": "Swap",
    "type": "event"
}]

def analyze_swap_activity(pool_address, from_block, to_block, token0_decimals=18, token1_decimals=6):
    """
    Analyze swap activity for an Aerodrome pool.
    """
    pool = w3.eth.contract(address=Web3.to_checksum_address(pool_address), abi=SWAP_EVENT_ABI)
    
    swap_filter = pool.events.Swap.create_filter(
        fromBlock=from_block,
        toBlock=to_block
    )
    
    events = swap_filter.get_all_entries()
    
    records = []
    for event in events:
        block = w3.eth.get_block(event["blockNumber"])
        
        amount0_in = event["args"]["amount0In"] / (10 ** token0_decimals)
        amount1_in = event["args"]["amount1In"] / (10 ** token1_decimals)
        amount0_out = event["args"]["amount0Out"] / (10 ** token0_decimals)
        amount1_out = event["args"]["amount1Out"] / (10 ** token1_decimals)
        
        # Determine swap direction
        if amount0_in > 0:
            swap_direction = "token0_to_token1"
            input_amount = amount0_in
            output_amount = amount1_out
        else:
            swap_direction = "token1_to_token0"
            input_amount = amount1_in
            output_amount = amount0_out
        
        record = {
            "timestamp": pd.to_datetime(block["timestamp"], unit="s"),
            "block_number": event["blockNumber"],
            "transaction_hash": event["transactionHash"].hex(),
            "sender": event["args"]["sender"],
            "to": event["args"]["to"],
            "direction": swap_direction,
            "input_amount": input_amount,
            "output_amount": output_amount,
            "amount0_in": amount0_in,
            "amount1_in": amount1_in,
            "amount0_out": amount0_out,
            "amount1_out": amount1_out
        }
        records.append(record)
    
    df = pd.DataFrame(records)
    return df

# Example usage (replace with actual pool address)
pool_address = "0x1234567890123456789012345678901234567890"  # Example pool
latest_block = w3.eth.block_number

# Analyze last 500 blocks
df_swaps = analyze_swap_activity(pool_address, latest_block - 500, latest_block)

if len(df_swaps) > 0:
    print(f"Analyzed {len(df_swaps)} swaps")
    
    # Calculate swap statistics
    print(f"\nSwap Direction Distribution:")
    print(df_swaps["direction"].value_counts())
    
    print(f"\nVolume Statistics:")
    print(f"Total input volume: {df_swaps['input_amount'].sum():,.2f}")
    print(f"Total output volume: {df_swaps['output_amount'].sum():,.2f}")
    print(f"Average swap input: {df_swaps['input_amount'].mean():,.2f}")
    
    # Calculate effective price for each swap
    df_swaps["effective_price"] = df_swaps["output_amount"] / df_swaps["input_amount"]
    print(f"\nPrice Statistics:")
    print(f"Average effective price: {df_swaps['effective_price'].mean():.6f}")
    print(f"Price volatility (std dev): {df_swaps['effective_price'].std():.6f}")
else:
    print("No swaps found in block range")
```

## Address Activity Analysis

Analyze top addresses by activity:

```python
import pandas as pd
import matplotlib.pyplot as plt

# Assuming df is loaded from transfer events

# Calculate metrics per address
from_stats = df.groupby("from_address").agg({
    "value": ["sum", "count", "mean"],
    "to_address": "nunique"
}).reset_index()

from_stats.columns = ["address", "total_sent", "send_count", "avg_sent", "unique_receivers"]

to_stats = df.groupby("to_address").agg({
    "value": ["sum", "count", "mean"],
    "from_address": "nunique"
}).reset_index()

to_stats.columns = ["address", "total_received", "receive_count", "avg_received", "unique_senders"]

# Merge statistics
address_stats = pd.merge(from_stats, to_stats, on="address", how="outer").fillna(0)

# Calculate net flow
address_stats["net_flow"] = address_stats["total_received"] - address_stats["total_sent"]
address_stats["total_activity"] = address_stats["send_count"] + address_stats["receive_count"]

# Sort by total activity
top_addresses = address_stats.sort_values("total_activity", ascending=False).head(20)

print("Top 20 Most Active Addresses:")
print(top_addresses[["address", "total_sent", "total_received", "net_flow", "total_activity"]])

# Visualize top senders and receivers
fig, axes = plt.subplots(1, 2, figsize=(15, 6))

top_senders = address_stats.nlargest(10, "total_sent")
axes[0].barh(range(len(top_senders)), top_senders["total_sent"])
axes[0].set_yticks(range(len(top_senders)))
axes[0].set_yticklabels([addr[:10] + "..." for addr in top_senders["address"]])
axes[0].set_xlabel("Total Sent (USDC)")
axes[0].set_title("Top 10 Senders")
axes[0].invert_yaxis()

top_receivers = address_stats.nlargest(10, "total_received")
axes[1].barh(range(len(top_receivers)), top_receivers["total_received"], color="green")
axes[1].set_yticks(range(len(top_receivers)))
axes[1].set_yticklabels([addr[:10] + "..." for addr in top_receivers["address"]])
axes[1].set_xlabel("Total Received (USDC)")
axes[1].set_title("Top 10 Receivers")
axes[1].invert_yaxis()

plt.tight_layout()
plt.savefig("top_addresses.png", dpi=300, bbox_inches="tight")
plt.show()
```

## Moonwell Market Analysis

Analyze Moonwell lending market data:

```python
from web3 import Web3
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime, timedelta

w3 = Web3(Web3.HTTPProvider("https://mainnet.base.org"))
MOONWELL_USDC_MARKET = "0xEdc817A28E8B93B03976FBd4a3dDBc9f7D176c22"

MARKET_ABI = [
    {"constant": True, "inputs": [], "name": "totalBorrows", "outputs": [{"type": "uint256"}], "type": "function"},
    {"constant": True, "inputs": [], "name": "totalSupply", "outputs": [{"type": "uint256"}], "type": "function"},
    {"constant": True, "inputs": [], "name": "totalReserves", "outputs": [{"type": "uint256"}], "type": "function"},
    {"constant": True, "inputs": [], "name": "exchangeRateStored", "outputs": [{"type": "uint256"}], "type": "function"},
    {"constant": True, "inputs": [], "name": "borrowRatePerTimestamp", "outputs": [{"type": "uint256"}], "type": "function"},
    {"constant": True, "inputs": [], "name": "supplyRatePerTimestamp", "outputs": [{"type": "uint256"}], "type": "function"}
]

market = w3.eth.contract(address=Web3.to_checksum_address(MOONWELL_USDC_MARKET), abi=MARKET_ABI)

def sample_market_state(market_contract, num_samples=100, block_interval=100):
    """
    Sample market state over time.
    """
    latest_block = w3.eth.block_number
    samples = []
    
    for i in range(num_samples):
        block_number = latest_block - (i * block_interval)
        
        try:
            total_borrows = market_contract.functions.totalBorrows().call(block_identifier=block_number)
            total_supply = market_contract.functions.totalSupply().call(block_identifier=block_number)
            exchange_rate = market_contract.functions.exchangeRateStored().call(block_identifier=block_number)
            
            block = w3.eth.get_block(block_number)
            timestamp = datetime.fromtimestamp(block["timestamp"])
            
            # Calculate underlying supply from mToken supply
            underlying_supply = (total_supply * exchange_rate) / 1e18
            
            utilization = (total_borrows / underlying_supply * 100) if underlying_supply > 0 else 0
            
            sample = {
                "timestamp": timestamp,
                "block_number": block_number,
                "total_borrows": total_borrows / 1e6,  # USDC has 6 decimals
                "underlying_supply": underlying_supply / 1e6,
                "utilization_rate": utilization
            }
            samples.append(sample)
        except Exception as e:
            print(f"Error sampling block {block_number}: {e}")
            continue
    
    df = pd.DataFrame(samples)
    df = df.sort_values("timestamp")
    return df

print("Sampling Moonwell USDC market state...")
df_market = sample_market_state(market, num_samples=50, block_interval=200)

print(f"\nCurrent Market State:")
latest = df_market.iloc[-1]
print(f"Total Supply: ${latest['underlying_supply']:,.2f}")
print(f"Total Borrows: ${latest['total_borrows']:,.2f}")
print(f"Utilization Rate: {latest['utilization_rate']:.2f}%")

# Plot market metrics
fig, axes = plt.subplots(2, 1, figsize=(12, 8))

axes[0].plot(df_market["timestamp"], df_market["underlying_supply"], label="Supply", marker="o")
axes[0].plot(df_market["timestamp"], df_market["total_borrows"], label="Borrows", marker="s")
axes[0].set_title("Moonwell USDC Market - Supply and Borrows")
axes[0].set_xlabel("Time")
axes[0].set_ylabel("Amount (USDC)")
axes[0].legend()
axes[0].grid(True)

axes[1].plot(df_market["timestamp"], df_market["utilization_rate"], color="red", marker="^")
axes[1].axhline(y=80, color="orange", linestyle="--", label="High Utilization (80%)")
axes[1].set_title("Moonwell USDC Market - Utilization Rate")
axes[1].set_xlabel("Time")
axes[1].set_ylabel("Utilization (%)")
axes[1].legend()
axes[1].grid(True)

plt.tight_layout()
plt.savefig("moonwell_analysis.png", dpi=300, bbox_inches="tight")
plt.show()
```

## Building a Data Pipeline

Create a continuous data collection pipeline:

```python
import pandas as pd
from web3 import Web3
import time
import sqlite3
from datetime import datetime

w3 = Web3(Web3.HTTPProvider("https://mainnet.base.org"))
USDC_ADDRESS = "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913"

class TransferDataPipeline:
    def __init__(self, db_path="transfers.db"):
        self.db_path = db_path
        self.init_database()
        self.last_processed_block = self.get_last_processed_block()
    
    def init_database(self):
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS transfers (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                block_number INTEGER,
                timestamp TEXT,
                transaction_hash TEXT,
                log_index INTEGER,
                from_address TEXT,
                to_address TEXT,
                value REAL,
                UNIQUE(transaction_hash, log_index)
            )
        """)
        cursor.execute("""
            CREATE INDEX IF NOT EXISTS idx_block_number ON transfers(block_number)
        """)
        cursor.execute("""
            CREATE INDEX IF NOT EXISTS idx_timestamp ON transfers(timestamp)
        """)
        cursor.execute("""
            CREATE INDEX IF NOT EXISTS idx_from_address ON transfers(from_address)
        """)
        cursor.execute("""
            CREATE INDEX IF NOT EXISTS idx_to_address ON transfers(to_address)
        """)
        conn.commit()
        conn.close()
    
    def get_last_processed_block(self):
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        cursor.execute("SELECT MAX(block_number) FROM transfers")
        result = cursor.fetchone()[0]
        conn.close()
        return result if result else 0
    
    def process_block_range(self, from_block, to_block):
        TRANSFER_EVENT_ABI = [{
            "anonymous": False,
            "inputs": [
                {"indexed": True, "name": "from", "type": "address"},
                {"indexed": True, "name": "to", "type": "address"},
                {"indexed": False, "name": "value", "type": "uint256"}
            ],
            "name": "Transfer",
            "type": "event"
        }]
        
        usdc = w3.eth.contract(address=Web3.to_checksum_address(USDC_ADDRESS), abi=TRANSFER_EVENT_ABI)
        
        transfer_filter = usdc.events.Transfer.create_filter(
            fromBlock=from_block,
            toBlock=to_block
        )
        
        events = transfer_filter.get_all_entries()
        
        if not events:
            return 0
        
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        inserted = 0
        for event in events:
            block = w3.eth.get_block(event["blockNumber"])
            timestamp = datetime.fromtimestamp(block["timestamp"]).isoformat()
            
            try:
                cursor.execute("""
                    INSERT INTO transfers 
                    (block_number, timestamp, transaction_hash, log_index, from_address, to_address, value)
                    VALUES (?, ?, ?, ?, ?, ?, ?)
                """, (
                    event["blockNumber"],
                    timestamp,
                    event["transactionHash"].hex(),
                    event["logIndex"],
                    event["args"]["from"],
                    event["args"]["to"],
                    event["args"]["value"] / 1e6
                ))
                inserted += 1
            except sqlite3.IntegrityError:
                pass  # Duplicate entry
        
        conn.commit()
        conn.close()
        return inserted
    
    def run(self, poll_interval=15):
        print(f"Starting pipeline from block {self.last_processed_block}")
        
        while True:
            try:
                latest_block = w3.eth.block_number
                
                if latest_block > self.last_processed_block:
                    from_block = self.last_processed_block + 1
                    to_block = min(from_block + 500, latest_block)  # Process 500 blocks at a time
                    
                    inserted = self.process_block_range(from_block, to_block)
                    print(f"Processed blocks {from_block} to {to_block}: {inserted} new transfers")
                    
                    self.last_processed_block = to_block
                
                time.sleep(poll_interval)
            except Exception as e:
                print(f"Error in pipeline: {e}")
                time.sleep(poll_interval)

# Example usage
if __name__ == "__main__":
    pipeline = TransferDataPipeline()
    # pipeline.run()  # Uncomment to start continuous collection
    
    # Query collected data
    conn = sqlite3.connect("transfers.db")
    df = pd.read_sql_query("""
        SELECT * FROM transfers 
        ORDER BY block_number DESC 
        LIMIT 100
    """, conn)
    conn.close()
    
    print(f"\nLatest 100 transfers:")
    print(df.head())
```

## Common Patterns

**Rolling Window Analysis:**

```python
import pandas as pd

# Calculate 24-hour rolling volume
df["timestamp"] = pd.to_datetime(df["timestamp"])
df = df.set_index("timestamp").sort_index()

df["volume_24h"] = df["value"].rolling(window="24H").sum()
df["count_24h"] = df["value"].rolling(window="24H").count()
df["avg_24h"] = df["value"].rolling(window="24H").mean()

print(df[["value", "volume_24h", "count_24h", "avg_24h"]].tail())
```

**Correlation Analysis:**

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Analyze correlation between different metrics
metrics_df = df[["value", "volume_24h", "count_24h"]].dropna()
correlation_matrix = metrics_df.corr()

plt.figure(figsize=(8, 6))
sns.heatmap(correlation_matrix, annot=True, cmap="coolwarm", center=0)
plt.title("Metric Correlation Matrix")
plt.tight_layout()
plt.savefig("correlation.png", dpi=300)
plt.show()
```

## Gotchas

**Memory Management:** Loading large event datasets into memory can exhaust RAM. Process data in chunks and use generators when possible.

**Block Timestamps:** Block timestamps are set by miners and can be slightly inaccurate. Don't rely on them for precise timing in financial calculations.

**Data Consistency:** Events can be reordered during block reorganizations. Always check finality before making critical decisions based on recent data.

**Timezone Handling:** Always work in UTC and convert to local timezones only for display purposes. Use pandas datetime with UTC timezone awareness.

**Decimal Precision:** Use Python Decimal class for financial calculations requiring exact precision. Float arithmetic can introduce rounding errors.

**Database Locking:** SQLite can have locking issues with concurrent writes. Use WAL mode or consider PostgreSQL for production pipelines.

**API Rate Limits:** Fetching historical blocks for timestamps hits RPC rate limits quickly. Cache block timestamp mappings locally.