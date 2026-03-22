# Task a1.03 — Order Consumer + Queueing

**Owner:** Achyut
**Estimated time:** 3-4 hours
**Goal:** A durable, crash-safe order consumer that picks up paid orders and routes them to pipelines. This is the most critical piece of infrastructure — if it loses an order, a customer paid and got nothing.

---

## The 4 rules this must follow (NON-NEGOTIABLE)

### Rule 1: Redis is the durable queue — not Supabase Realtime alone

Supabase Realtime triggers the enqueue. Redis holds the job durably.
If Railway crashes mid-processing, Redis still has the job. Worker picks it up on restart.

```python
# When a new paid order is detected via Supabase Realtime:
async def enqueue_order(order_id: str):
    redis = get_redis()
    # Only enqueue if not already queued (prevent duplicates)
    existing = await redis.get(f"order:{order_id}:status")
    if existing:
        return  # already in queue, skip
    await redis.lpush("orders:pending", order_id)
    await redis.set(f"order:{order_id}:status", "queued", ex=86400)  # 24hr TTL
```

### Rule 2: Idempotency check before processing

Before touching an order, check its execution_status in Supabase.
If it's anything other than 'queued' — skip it. Do not process twice.

```python
async def process_order(order_id: str):
    supabase = get_supabase()
    order = await supabase.table("orders").select("*").eq("id", order_id).single().execute()

    # IDEMPOTENCY CHECK — critical
    if order.data["execution_status"] != "queued":
        logger.info(f"Order {order_id} already processing/processed, skipping")
        return

    # Immediately claim it to prevent double-processing
    await supabase.table("orders").update({
        "execution_status": "classifying"
    }).eq("id", order_id).eq("execution_status", "queued").execute()
    # Note: .eq("execution_status", "queued") on the update is intentional
    # If two workers race, only one will match — the other gets 0 rows updated

    # Now safe to process
    await route_to_pipeline(order.data)
```

### Rule 3: Dead letter queue for failed jobs

If a pipeline fails after 3 retries, move to dead letter queue.
Never silently drop a failed order. Always alert.

```python
async def handle_failure(order_id: str, error: str, attempt: int):
    if attempt >= 3:
        redis = get_redis()
        await redis.lpush("orders:failed", order_id)
        await redis.set(f"order:{order_id}:error", error)

        # Update Supabase
        supabase = get_supabase()
        await supabase.table("orders").update({
            "execution_status": "failed"
        }).eq("id", order_id).execute()

        # Alert the team
        await alert(f"Order {order_id} failed after 3 attempts: {error}")
    else:
        # Re-enqueue for retry
        redis = get_redis()
        await redis.lpush("orders:pending", order_id)
        await redis.set(f"order:{order_id}:attempts", attempt + 1)
```

### Rule 4: Worker must survive Railway restarts

The consumer runs as an infinite loop with proper signal handling.
On startup, it checks Redis for any jobs that were in-flight when Railway last crashed.

```python
async def recover_inflight_orders():
    """On startup, find any orders stuck in 'classifying' or 'running' state
    that were abandoned mid-process. Re-enqueue them."""
    supabase = get_supabase()
    stuck = await supabase.table("orders").select("id").in_(
        "execution_status", ["classifying", "running"]
    ).execute()

    for order in stuck.data:
        logger.warning(f"Recovering stuck order: {order['id']}")
        # Reset to queued and re-enqueue
        await supabase.table("orders").update({
            "execution_status": "queued"
        }).eq("id", order["id"]).execute()
        await enqueue_order(order["id"])

async def main():
    logger.info("Order consumer starting up...")
    await recover_inflight_orders()  # ALWAYS run on startup

    while True:
        try:
            redis = get_redis()
            # Block for up to 5 seconds waiting for a job
            result = await redis.brpop("orders:pending", timeout=5)
            if result:
                _, order_id = result
                await process_order(order_id)
        except Exception as e:
            logger.error(f"Consumer error: {e}")
            await asyncio.sleep(1)  # brief pause before retry
```

---

## Full file: `src/workers/order_consumer.py`

Implement the above 4 rules into a single worker file:

```python
import asyncio
import logging
import signal
from src.utils.supabase import get_supabase
from src.utils.redis import get_redis
from src.utils.slack import alert
from src.pipelines.router import route_to_pipeline

logger = logging.getLogger(__name__)
running = True

def handle_shutdown(sig, frame):
    global running
    logger.info("Shutdown signal received, finishing current job...")
    running = False

signal.signal(signal.SIGTERM, handle_shutdown)
signal.signal(signal.SIGINT, handle_shutdown)

# ... all 4 functions above ...

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Also build: `src/pipelines/router.py`

Routes an order to the correct pipeline based on `task_type` and `assigned_pipeline`:

```python
from src.pipelines.generic import GenericPipeline
from src.pipelines.automation import AutomationPipeline
from src.pipelines.content import ContentPipeline

PIPELINE_MAP = {
    "automation": AutomationPipeline,
    "content": ContentPipeline,
    "agent": GenericPipeline,      # placeholder until agent pipeline built
    "integration": GenericPipeline,
    "data": GenericPipeline,
    "other": GenericPipeline,
}

async def route_to_pipeline(order: dict):
    task_type = order.get("task_type", "other")
    pipeline_class = PIPELINE_MAP.get(task_type, GenericPipeline)
    pipeline = pipeline_class()
    await pipeline.run(order)
```

---

## Files to create

- `src/workers/order_consumer.py`
- `src/pipelines/router.py`
- `src/pipelines/base.py` (BasePipeline abstract class — see CLAUDE.md)
- `src/pipelines/generic.py` (catch-all pipeline stub)
- `tests/test_order_consumer.py`

---

## Files to reference

- `CLAUDE.md` → Pipeline architecture section for BasePipeline structure
- `contracts/database.sql` → orders table columns and execution_status enum values
