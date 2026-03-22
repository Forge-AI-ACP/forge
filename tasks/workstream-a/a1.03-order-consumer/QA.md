# QA: a1.03 — Order Consumer + Queueing

## Automated checks (ALL must pass)

### 1. Tests pass
- [ ] `poetry run pytest tests/test_order_consumer.py` exits 0

### 2. Idempotency
- [ ] Manually set an order to `execution_status: classifying` in Supabase
- [ ] Push same order_id to Redis queue twice
- [ ] Verify it is only processed once (check execution_logs — only 1 entry)

### 3. Dead letter queue
- [ ] Simulate 3 consecutive pipeline failures for a test order
- [ ] Verify order appears in `orders:failed` Redis key
- [ ] Verify `execution_status: failed` in Supabase orders table
- [ ] Verify Slack alert fired to #forge-alerts

### 4. Crash recovery
- [ ] Manually set an order to `execution_status: classifying` in Supabase (simulating mid-crash)
- [ ] Start the consumer fresh
- [ ] Verify `recover_inflight_orders()` resets it to `queued` and re-enqueues it

### 5. Signal handling
- [ ] Start consumer, send SIGTERM
- [ ] Verify it logs "Shutdown signal received" and exits cleanly

## Manual checks
- [ ] Consumer runs continuously without crashing on an empty queue
- [ ] Logs are structured JSON (not plain text)
- [ ] No order is ever silently dropped — every failure path logs + alerts
