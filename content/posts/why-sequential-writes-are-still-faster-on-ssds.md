---
title: Why Sequential Writes Are Still Faster on SSDs
date: 2020-02-23
---

Many people think of SSD as a random accessed device, like memory. While SSDs pretend to be randomly accessed, they are more complicated than that, so their performance patterns are sometimes confusing. To understand why we need to dig a little bit deeper.

An SSD is composed of a few [NAND chips](https://images.squarespace-cdn.com/content/v1/54a99adbe4b01ff9ee5b44ea/1431380764296-VTR3I6RBK4MOC99JCIMM/ke17ZwdGBToddI8pDm48kFAJvy9p_ltnkV8kJw5me6sUqsxRUqqbr1mOJYKfIPR7LoDQ9mXPOjoJoqy81S2I8N_N4V1vUb5AoIIIbLZhVYxCRW4BPu10St3TBAUQYVKcjmXuWhQ9U3oDT3qHu30IJHUbcIDPugzzAsF0I3RYW2ogakcQ3n1yAG9VJCzwHpYZ/nand.jpg?format=1500w). Those NAND chips support 3 basic operations:

1. Read a page. A page is the minimal addressable unit, usually 2KB or 4KB, and a read operation takes around 10us.
2. Write("program") a page. A write operation can only be done when the page is already erased, takes around 100us.
3. Erase a block. A block is usually 128 or 256 pages, and the erase operation takes a few ms.

Note that the unit of read/write operations are a page, while the unit of erase operation is a block. I guess this is because of cost reasons. So for disk write, a naive implementation needs to read the whole block, erase the block, then write updated data back to the block, which is unacceptable.

Furthermore, NAND flash has a certain amount of lifetime, ranging from 10000 to 100000 P/E (Program/Erase) cycles. Efforts must be taken to make sure blocks wear out uniformly, otherwise, the SSD would lose capacity.

Because of these, inside the firmware of SSDs lives another level of abstraction, the Flash Translation Layer (FTL). FTL helps to build an illusion of random access device. To achieve that, FTL employs an approach very similar to Log-Structured Merge (LSM) tree. Writes are always written to new, already erased pages, while in the background, garbage collects (GC) outdated data. And surely, FTL needs to keep a map from the user's logical address to physical address on SSD, both in-memory and persistently.

With the knowledge above, we could finally answer the question in the title, why sequential writes are faster than random writes on SSDs? Because for sequential writes:

- The address map table is smaller since new data are consecutive in larger chunks.
- GC operation is simpler, no "read, update, write-to-another-block" required, only metadata needs to be updated. Erase of the block is required anyway.