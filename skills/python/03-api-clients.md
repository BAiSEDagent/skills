# Python API Clients for Web3 Services

## Overview

Building robust API clients is essential for integrating with Circle, Coinbase, and other Web3 services. This lesson covers async HTTP clients with httpx and aiohttp, implementing rate limiting and retry logic, handling webhooks, and building production-ready API client patterns.

## Setup and Dependencies

Install required packages:

```bash
pip install httpx aiohttp pydantic python-dotenv backoff tenacity
```

Basic imports:

```python
import httpx
import aiohttp
import asyncio
from typing import Optional, Dict, Any, List
from pydantic import BaseModel, Field
from datetime import datetime
import os
from dotenv import load_dotenv
import hashlib
import hmac
import json

load_dotenv()
```

## Async HTTP with httpx

Build a basic async API client:

```python
import httpx
import asyncio
from typing import Optional, Dict, Any

class AsyncAPIClient:
    def __init__(self, base_url: str, api_key: Optional[str] = None, timeout: float = 30.0):
        self.base_url = base_url.rstrip("/")
        self.api_key = api_key
        self.timeout = timeout
        self.client: Optional[httpx.AsyncClient] = None
    
    async def __aenter__(self):
        headers = {"User-Agent": "BAiSED-Agent/1.0"}
        if self.api_key:
            headers["Authorization"] = f"Bearer {self.api_key}"
        
        self.client = httpx.AsyncClient(
            base_url=self.base_url,
            headers=headers,
            timeout=self.timeout,
            follow_redirects=True
        )
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if self.client:
            await self.client.aclose()
    
    async def get(self, endpoint: str, params: Optional[Dict[str, Any]] = None) -> Dict[str, Any]:
        if not self.client:
            raise RuntimeError("Client not initialized. Use 'async with' context manager.")
        
        response = await self.client.get(endpoint, params=params)
        response.raise_for_status()
        return response.json()
    
    async def post(self, endpoint: str, json_data: Optional[Dict[str, Any]] = None) -> Dict[str, Any]:
        if not self.client:
            raise RuntimeError("Client not initialized. Use 'async with' context manager.")
        
        response = await self.client.post(endpoint, json=json_data)
        response.raise_for_status()
        return response.json()
    
    async def put(self, endpoint: str, json_data: Optional[Dict[str, Any]] = None) -> Dict[str, Any]:
        if not self.client:
            raise RuntimeError("Client not initialized. Use 'async with' context manager.")
        
        response = await self.client.put(endpoint, json=json_data)
        response.raise_for_status()
        return response.json()
    
    async def delete(self, endpoint: str) -> Dict[str, Any]:
        if not self.client:
            raise RuntimeError("Client not initialized. Use 'async with' context manager.")
        
        response = await self.client.delete(endpoint)
        response.raise_for_status()
        return response.json()

# Example usage
async def example_usage():
    async with AsyncAPIClient("https://api.example.com", api_key="test-key") as client:
        data = await client.get("/endpoint")
        print(data)

if __name__ == "__main__":
    asyncio.run(example_usage())
```

## Circle API Client

Build a production-ready Circle API client:

```python
import httpx
import asyncio
from typing import Optional, Dict, Any, List
from pydantic import BaseModel
from datetime import datetime
import os
import uuid

class CircleTransferRequest(BaseModel):
    idempotency_key: str
    source: Dict[str, str]
    destination: Dict[str, str]
    amount: Dict[str, str]

class CircleTransferResponse(BaseModel):
    id: str
    source: Dict[str, Any]
    destination: Dict[str, Any]
    amount: Dict[str, str]
    status: str
    create_date: datetime

class CircleAPIClient:
    BASE_URL = "https://api.circle.com/v1"
    
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.client: Optional[httpx.AsyncClient] = None
    
    async def __aenter__(self):
        self.client = httpx.AsyncClient(
            base_url=self.BASE_URL,
            headers={
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json",
                "Accept": "application/json"
            },
            timeout=30.0
        )
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if self.client:
            await self.client.aclose()
    
    async def create_transfer(self, 
                             source_wallet_id: str,
                             destination_address: str,
                             amount: str,
                             chain: str = "ETH",
                             idempotency_key: Optional[str] = None) -> CircleTransferResponse:
        """
        Create a USDC transfer on Base using Circle API.
        """
        if not idempotency_key:
            idempotency_key = str(uuid.uuid4())
        
        request_data = {
            "idempotencyKey": idempotency_key,
            "source": {
                "type": "wallet",
                "id": source_wallet_id
            },
            "destination": {
                "type": "blockchain",
                "address": destination_address,
                "chain": chain
            },
            "amount": {
                "amount": amount,
                "currency": "USD"
            }
        }
        
        response = await self.client.post("/transfers", json=request_data)
        response.raise_for_status()
        
        data = response.json()["data"]
        return CircleTransferResponse(**data)
    
    async def get_transfer(self, transfer_id: str) -> CircleTransferResponse:
        """
        Get transfer status by ID.
        """
        response = await self.client.get(f"/transfers/{transfer_id}")
        response.raise_for_status()
        
        data = response.json()["data"]
        return CircleTransferResponse(**data)
    
    async def list_transfers(self, 
                            from_date: Optional[datetime] = None,
                            to_date: Optional[datetime] = None,
                            page_size: int = 50) -> List[CircleTransferResponse]:
        """
        List transfers with optional date filters.
        """
        params = {"pageSize": page_size}
        
        if from_date:
            params["from"] = from_date.isoformat()
        if to_date:
            params["to"] = to_date.isoformat()
        
        response = await self.client.get("/transfers", params=params)
        response.raise_for_status()
        
        data = response.json()["data"]
        return [CircleTransferResponse(**item) for item in data]
    
    async def get_wallet_balance(self, wallet_id: str) -> Dict[str, Any]:
        """
        Get wallet balance.
        """
        response = await self.client.get(f"/wallets/{wallet_id}")
        response.raise_for_status()
        
        return response.json()["data"]

# Example usage
async def example_circle_usage():
    api_key = os.getenv("CIRCLE_API_KEY")
    
    async with CircleAPIClient(api_key) as client:
        # Create a transfer
        transfer = await client.create_transfer(
            source_wallet_id="wallet-123",
            destination_address="0x8004A169FB4a3325136EB29fA0ceB6D2e539a432",
            amount="10.00",
            chain="BASE"
        )
        print(f"Transfer created: {transfer.id}")
        print(f"Status: {transfer.status}")
        
        # Check transfer status
        await asyncio.sleep(5)
        updated_transfer = await client.get_transfer(transfer.id)
        print(f"Updated status: {updated_transfer.status}")
        
        # List recent transfers
        transfers = await client.list_transfers(page_size=10)
        print(f"Found {len(transfers)} recent transfers")

if __name__ == "__main__":
    asyncio.run(example_circle_usage())
```

## Rate Limiting and Retry Logic

Implement exponential backoff and rate limiting:

```python
import httpx
import asyncio
from typing import Optional, Dict, Any, Callable
import time
from functools import wraps
import logging

logger = logging.getLogger(__name__)

class RateLimiter:
    def __init__(self, calls_per_second: float):
        self.calls_per_second = calls_per_second
        self.min_interval = 1.0 / calls_per_second
        self.last_call_time = 0.0
        self.lock = asyncio.Lock()
    
    async def acquire(self):
        async with self.lock:
            current_time = time.time()
            time_since_last_call = current_time - self.last_call_time
            
            if time_since_last_call < self.min_interval:
                wait_time = self.min_interval - time_since_last_call
                await asyncio.sleep(wait_time)
            
            self.last_call_time = time.time()

def retry_with_backoff(max_retries: int = 3, 
                       initial_delay: float = 1.0,
                       backoff_factor: float = 2.0,
                       retry_on: tuple = (httpx.HTTPError,)):
    """
    Decorator for retrying async functions with exponential backoff.
    """
    def decorator(func: Callable):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            delay = initial_delay
            last_exception = None
            
            for attempt in range(max_retries):
                try:
                    return await func(*args, **kwargs)
                except retry_on as e:
                    last_exception = e
                    
                    if attempt < max_retries - 1:
                        logger.warning(f"Attempt {attempt + 1} failed: {e}. Retrying in {delay}s...")
                        await asyncio.sleep(delay)
                        delay *= backoff_factor
                    else:
                        logger.error(f"All {max_retries} attempts failed")
            
            raise last_exception
        return wrapper
    return decorator

class RateLimitedAPIClient:
    def __init__(self, base_url: str, api_key: str, rate_limit: float = 10.0):
        self.base_url = base_url
        self.api_key = api_key
        self.rate_limiter = RateLimiter(rate_limit)
        self.client: Optional[httpx.AsyncClient] = None
    
    async def __aenter__(self):
        self.client = httpx.AsyncClient(
            base_url=self.base_url,
            headers={"Authorization": f"Bearer {self.api_key}"},
            timeout=30.0
        )
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if self.client:
            await self.client.aclose()
    
    @retry_with_backoff(max_retries=3, initial_delay=1.0)
    async def get(self, endpoint: str, params: Optional[Dict[str, Any]] = None) -> Dict[str, Any]:
        await self.rate_limiter.acquire()
        
        response = await self.client.get(endpoint, params=params)
        
        if response.status_code == 429:  # Too Many Requests
            retry_after = int(response.headers.get("Retry-After", 60))
            logger.warning(f"Rate limited. Waiting {retry_after}s...")
            await asyncio.sleep(retry_after)
            return await self.get(endpoint, params)
        
        response.raise_for_status()
        return response.json()
    
    @retry_with_backoff(max_retries=3, initial_delay=1.0)
    async def post(self, endpoint: str, json_data: Optional[Dict[str, Any]] = None) -> Dict[str, Any]:
        await self.rate_limiter.acquire()
        
        response = await self.client.post(endpoint, json=json_data)
        
        if response.status_code == 429:
            retry_after = int(response.headers.get("Retry-After", 60))
            logger.warning(f"Rate limited. Waiting {retry_after}s...")
            await asyncio.sleep(retry_after)
            return await self.post(endpoint, json_data)
        
        response.raise_for_status()
        return response.json()

# Example usage with concurrent requests
async def example_rate_limited_requests():
    async with RateLimitedAPIClient("https://api.example.com", "api-key", rate_limit=5.0) as client:
        # Make 20 requests concurrently -- rate limiter will throttle
        tasks = [client.get("/endpoint") for _ in range(20)]
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        successful = sum(1 for r in results if not isinstance(r, Exception))
        failed = len(results) - successful
        
        print(f"Successful requests: {successful}")
        print(f"Failed requests: {failed}")

if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    asyncio.run(example_rate_limited_requests())
```

## Webhook Handlers

Build secure webhook receivers for Circle and other services:

```python
from fastapi import FastAPI, Request, HTTPException, Header
import hmac
import hashlib
import json
from typing import Optional
from pydantic import BaseModel
from datetime import datetime
import os

app = FastAPI()

class WebhookEvent(BaseModel):
    event_type: str
    event_id: str
    timestamp: datetime
    data: dict

def verify_webhook_signature(payload: bytes, signature: str, secret: str) -> bool:
    """
    Verify webhook signature using HMAC-SHA256.
    """
    expected_signature = hmac.new(
        secret.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(signature, expected_signature)

@app.post("/webhooks/circle")
async def circle_webhook(
    request: Request,
    x_signature: Optional[str] = Header(None)
):
    """
    Handle Circle webhook events.
    """
    webhook_secret = os.getenv("CIRCLE_WEBHOOK_SECRET")
    
    if not webhook_secret:
        raise HTTPException(status_code=500, detail="Webhook secret not configured")
    
    payload = await request.body()
    
    # Verify signature
    if x_signature:
        if not verify_webhook_signature(payload, x_signature, webhook_secret):
            raise HTTPException(status_code=401, detail="Invalid signature")
    
    # Parse event
    try:
        event_data = json.loads(payload)
        event = WebhookEvent(**event_data)
    except Exception as e:
        raise HTTPException(status_code=400, detail=f"Invalid payload: {e}")
    
    # Process event based on type
    if event.event_type == "transfer.completed":
        transfer_id = event.data.get("id")
        status = event.data.get("status")
        print(f"Transfer {transfer_id} completed with status: {status}")
        
        # Add your processing logic here
        # - Update database
        # - Trigger follow-up actions
        # - Send notifications
    
    elif event.event_type == "transfer.failed":
        transfer_id = event.data.get("id")
        error = event.data.get("error")
        print(f"Transfer {transfer_id} failed: {error}")
        
        # Handle failure
        # - Alert administrators
        # - Retry logic
        # - Update records
    
    return {"status": "received", "event_id": event.event_id}

@app.post("/webhooks/coinbase")
async def coinbase_webhook(
    request: Request,
    cb_signature: Optional[str] = Header(None, alias="CB-SIGNATURE")
):
    """
    Handle Coinbase Commerce webhook events.
    """
    webhook_secret = os.getenv("COINBASE_WEBHOOK_SECRET")
    
    if not webhook_secret:
        raise HTTPException(status_code=500, detail="Webhook secret not configured")
    
    payload = await request.body()
    
    # Verify Coinbase signature
    if cb_signature:
        expected_signature = hmac.new(
            webhook_secret.encode(),
            payload,
            hashlib.sha256
        ).hexdigest()
        
        if not hmac.compare_digest(cb_signature, expected_signature):
            raise HTTPException(status_code=401, detail="Invalid signature")
    
    event_data = json.loads(payload)
    
    event_type = event_data.get("type")
    event_id = event_data.get("id")
    
    print(f"Received Coinbase event: {event_type} (ID: {event_id})")
    
    # Process based on event type
    if event_type == "charge:confirmed":
        charge_id = event_data.get("data", {}).get("id")
        print(f"Charge {charge_id} confirmed")
    
    elif event_type == "charge:failed":
        charge_id = event_data.get("data", {}).get("id")
        print(f"Charge {charge_id} failed")
    
    return {"status": "received"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## aiohttp Client Pattern

Alternative implementation using aiohttp:

```python
import aiohttp
import asyncio
from typing import Optional, Dict, Any
import logging

logger = logging.getLogger(__name__)

class AiohttpAPIClient:
    def __init__(self, base_url: str, api_key: Optional[str] = None):
        self.base_url = base_url.rstrip("/")
        self.api_key = api_key
        self.session: Optional[aiohttp.ClientSession] = None
    
    async def __aenter__(self):
        headers = {"User-Agent": "BAiSED-Agent/1.0"}
        if self.api_key:
            headers["Authorization"] = f"Bearer {self.api_key}"
        
        timeout = aiohttp.ClientTimeout(total=30)
        self.session = aiohttp.ClientSession(
            base_url=self.base_url,
            headers=headers,
            timeout=timeout
        )
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if self.session:
            await self.session.close()
    
    async def get(self, endpoint: str, params: Optional[Dict[str, Any]] = None) -> Dict[str, Any]:
        if not self.session:
            raise RuntimeError("Session not initialized")
        
        async with self.session.get(endpoint, params=params) as response:
            response.raise_for_status()
            return await response.json()
    
    async def post(self, endpoint: str, json_data: Optional[Dict[str, Any]] = None) -> Dict[str, Any]:
        if not self.session:
            raise RuntimeError("Session not initialized")
        
        async with self.session.post(endpoint, json=json_data) as response:
            response.raise_for_status()
            return await response.json()
    
    async def stream_get(self, endpoint: str, params: Optional[Dict[str, Any]] = None):
        """
        Stream large responses line by line.
        """
        if not self.session:
            raise RuntimeError("Session not initialized")
        
        async with self.session.get(endpoint, params=params) as response:
            response.raise_for_status()
            
            async for line in response.content:
                yield line

# Example usage with streaming
async def example_streaming():
    async with AiohttpAPIClient("https://api.example.com", api_key="test") as client:
        line_count = 0
        async for line in client.stream_get("/large-dataset"):
            line_count += 1
            if line_count % 1000 == 0:
                print(f"Processed {line_count} lines")
        
        print(f"Total lines: {line_count}")

if __name__ == "__main__":
    asyncio.run(example_streaming())
```

## Common Patterns

**Connection Pooling:**

```python
import httpx
import asyncio

class PooledAPIClient:
    def __init__(self, base_url: str, max_connections: int = 100):
        self.base_url = base_url
        self.limits = httpx.Limits(
            max_keepalive_connections=max_connections,
            max_connections=max_connections
        )
    
    async def __aenter__(self):
        self.client = httpx.AsyncClient(
            base_url=self.base_url,
            limits=self.limits
        )
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.client.aclose()
    
    async def get(self, endpoint: str):
        response = await self.client.get(endpoint)
        response.raise_for_status()
        return response.json()
```

**Request Caching:**

```python
from functools import lru_cache
import time

class CachedAPIClient:
    def __init__(self, base_url: str, cache_ttl: int = 300):
        self.base_url = base_url
        self.cache_ttl = cache_ttl
        self.cache = {}
    
    def _get_cache_key(self, endpoint: str, params: dict) -> str:
        params_str = str(sorted(params.items())) if params else ""
        return f"{endpoint}:{params_str}"
    
    async def get_cached(self, endpoint: str, params: dict = None):
        cache_key = self._get_cache_key(endpoint, params or {})
        
        if cache_key in self.cache:
            data, timestamp = self.cache[cache_key]
            if time.time() - timestamp < self.cache_ttl:
                return data
        
        # Cache miss or expired
        data = await self.get(endpoint, params)
        self.cache[cache_key] = (data, time.time())
        return data
```

## Gotchas

**Connection Cleanup:** Always use context managers (async with) to ensure proper connection cleanup. Unclosed connections can leak resources.

**Timeout Configuration:** Set appropriate timeouts for all requests. Default infinite timeouts can hang your application indefinitely.

**Error Response Bodies:** When an API returns an error status, read the response body before raising exceptions. It often contains useful debugging information.

**JSON Encoding:** Not all Python objects serialize to JSON automatically. Use custom encoders or pydantic models for complex types.

**Webhook Replay Attacks:** Always verify webhook signatures and consider implementing replay protection with timestamp checks.

**Rate Limit Headers:** Check response headers for rate limit information (X-RateLimit-Remaining, X-RateLimit-Reset) and adjust accordingly.

**Async Context:** Never use blocking operations (requests library, time.sleep) inside async functions. Use asyncio.sleep and async-compatible libraries.

**Certificate Verification:** Never disable SSL verification in production. If using self-signed certificates, explicitly trust them rather than disabling verification.

**Idempotency:** Always use idempotency keys for create operations to prevent duplicate transactions from retries.

**Concurrent Requests:** Be mindful of API rate limits when making concurrent requests. Implement proper throttling to avoid being blocked.