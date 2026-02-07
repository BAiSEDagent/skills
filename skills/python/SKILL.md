---
name: python
description: Python for Web3 backend development, data analysis, and API integration on Base
trigger_phrases:
  - "write python code"
  - "use web3.py"
  - "analyze blockchain data with python"
  - "build an api client"
  - "process events with pandas"
version: 1.0.0
---

# Python for Base Development

## Guidelines

Python is essential for building agent backends, analyzing on-chain data, and integrating with external APIs. Use Python when:

- Building data pipelines to process blockchain events and transactions
- Performing quantitative analysis on DeFi protocols and market data
- Creating async API clients for Circle, Coinbase, or other Web3 services
- Developing agent backends that need to interact with Base contracts
- Processing large datasets of on-chain activity for ML or analytics
- Building monitoring and alerting systems for protocol health
- Creating webhook handlers for real-time event processing

For Web3 interactions, use web3.py with Base RPC endpoints. Connect to Base Mainnet at https://mainnet.base.org (chain ID 8453). Always use async patterns (asyncio, aiohttp, httpx) for production systems to handle concurrent operations efficiently.

When analyzing on-chain data, leverage pandas for dataframe operations, numpy for numerical computations, and matplotlib or plotly for visualization. Structure your data pipelines to handle event logs, transaction traces, and state changes systematically.

For API integrations, implement robust error handling, exponential backoff retry logic, and rate limiting. Use httpx or aiohttp for async HTTP operations. Always validate API responses and handle edge cases.

Security considerations:
- Never hardcode private keys -- use environment variables or secure vaults
- Validate all external data inputs including API responses and user inputs
- Use type hints and pydantic models for data validation
- Implement request signing for authenticated API calls
- Log operations comprehensively but never log sensitive data

Python complements Solidity and Foundry workflows. Use Python for:
- Pre-deployment analysis and simulation
- Post-deployment monitoring and data collection
- Integration testing with live contracts
- Building user-facing applications and dashboards

## Examples

**Example 1: Analyze USDC Transfer Volume**
Trigger: "Analyze USDC transfer volume on Base over the last 24 hours"
Action:
1. Connect to Base RPC using web3.py
2. Fetch Transfer events from USDC contract (0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913)
3. Load events into pandas DataFrame
4. Aggregate by hour and calculate total volume
5. Generate matplotlib chart showing hourly volume trends

**Example 2: Build Circle API Client**
Trigger: "Create an async Python client for Circle USDC transfers"
Action:
1. Use httpx.AsyncClient with base URL and API key authentication
2. Implement methods for transfer creation, status checking, and webhook verification
3. Add exponential backoff retry logic for failed requests
4. Include request signing for authenticated endpoints
5. Return pydantic models for type-safe response handling

**Example 3: Monitor Moonwell Market Health**
Trigger: "Monitor Moonwell USDC market utilization and alert on high usage"
Action:
1. Connect to Moonwell USDC market contract (0xEdc817A28E8B93B03976FBd4a3dDBc9f7D176c22)
2. Read totalBorrows, totalSupply, and calculate utilization rate
3. Set up event listener for Borrow and Repay events
4. Send alerts via webhook when utilization exceeds threshold
5. Log historical data to time-series database for trend analysis

## Resources

- web3.py Documentation: https://web3py.readthedocs.io/
- Base Network Documentation: https://docs.base.org/
- pandas Documentation: https://pandas.pydata.org/docs/
- httpx Documentation: https://www.python-httpx.org/
- aiohttp Documentation: https://docs.aiohttp.org/
- pydantic Documentation: https://docs.pydantic.dev/
- Ethereum Python Ecosystem: https://ethereum.org/en/developers/docs/programming-languages/python/
- Base Contract Addresses: https://docs.base.org/docs/contracts

## Cross-References

- foundry: Use Python for analyzing Foundry test results and deployment scripts
- solidity: Python complements Solidity by providing data analysis and integration capabilities
- circle: Build Python clients for Circle APIs including USDC transfers and CCTP
- moonwell: Analyze Moonwell market data and build monitoring systems
- aerodrome: Process Aerodrome swap events and calculate pool analytics