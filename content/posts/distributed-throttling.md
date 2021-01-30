---
title: Distributed Throttling
date: 2021-01-30
---

Sometimes we need to limit a user's QPS(query per second) or bandwidth globally. The basic idea is simple, when a request comes in, before handling it, the service would ask for permission, or "token". If permitted, the request is handled as usual; if not permitted, the request would be rejected. But implementing it correctly takes some effort. I'll discuss my approach and thoughts behind it.

First things first, a distributed throttling service is a distributed system, it needs to scale horizontally. This part is easy, just start N throttling servers and hash token requests by user ID. Then all token requests of a user could hit the same throttling server, we could easily do some bookkeeping there and tell if the user should be throttled. Note these "bookkeeping" do not need to be persistent, keep them in memory would be fine.

Then we need to choose an algorithm for the "bookkeeping". The most commonly used algorithm is _token bucket_ (or _leaky bucket_). There're tons of articles discussing it, so I just put its pseudocode here.
```
if remaining_token + (now - last_token_acquire_timestamp) * token_refill_speed > acquired_token:
    remaining_token += (now - last_token_acquire_timestamp) * token_refill_speed - acquired_token
    last_token_acquire_timestamp = now
    return allow
else:
    return reject
```
The algorithm requires to update `remaining_token` and `last_token_acquire_timestamp` at the same time, so some kind of mutex or atomic operation is needed. [This article](https://blog.cloudflare.com/counting-things-a-lot-of-different-things/) from CloudFlare presents another algorithm, _sliding window_, but I consider it as a mutation of token bucket.

Another tricky part is API design. At first I designed only one interface:
```
acquire_token(user, item, request_amount, limit) -> (allow, backoff_time)
```
But the client needs to batch-apply for tokens to reduce pressure on throttling service. When a client acquires N tokens but turns out only consumes M, where M < N, another interface is needed so those extra tokens consumed could be refilled:
```
refill_token(user, item, refill_amount) -> ()
```

The last piece of advice would be monitoring your service closely after the throttling system is online, to make sure expected users are not shut out of the door.