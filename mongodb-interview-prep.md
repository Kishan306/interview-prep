# The Complete MongoDB Interview Prep Sheet
### Beginner → Advanced | Document Model, Aggregation, Indexing, Replication, Sharding & Mongoose/TypeScript

**How to read this:**
- Each **main question** is numbered. Answer out loud before reading.
- **`↳ Follow-up:`** lines are the small "twist" an interviewer fires right after you answer — handle them inline. If you can't, you didn't fully know the main answer.
- Bigger twists become their own numbered questions (cross-referenced).
- Code uses the Node driver / Mongoose with TypeScript unless a point is shell-specific.

---

## TABLE OF CONTENTS

1. [Phase 1 — Fundamentals & The Document Model](#p1)
2. [Phase 2 — CRUD Operations](#p2)
3. [Phase 3 — Query Operators & Advanced Querying](#p3)
4. [Phase 4 — Indexing](#p4)
5. [Phase 5 — The Aggregation Framework](#p5)
6. [Phase 6 — Schema Design & Data Modeling](#p6)
7. [Phase 7 — Replication & High Availability](#p7)
8. [Phase 8 — Sharding & Horizontal Scaling](#p8)
9. [Phase 9 — Transactions & Consistency](#p9)
10. [Phase 10 — Performance & Optimization](#p10)
11. [Phase 11 — Mongoose & TypeScript](#p11)
12. [Phase 12 — Security](#p12)
13. [Phase 13 — Operations & Production](#p13)
14. [Phase 14 — Tricky / Gotcha & Rapid-Fire](#p14)

---

<a name="p1"></a>
## PHASE 1 — FUNDAMENTALS & THE DOCUMENT MODEL

**1. What is MongoDB?**
MongoDB is an open-source, **document-oriented NoSQL database**. Instead of tables and rows, it stores data as flexible, JSON-like **documents** grouped into **collections**. It's designed for developer productivity (data maps naturally to objects), horizontal scalability (sharding), and high availability (replica sets). It's schema-flexible — documents in the same collection can have different shapes — though you typically enforce structure at the application or schema-validation level.

`↳ Follow-up: Is MongoDB schemaless?` — It's **schema-flexible**, not truly schemaless. The database doesn't *require* a fixed schema, but real applications enforce one — via Mongoose schemas, JSON Schema validation (`$jsonSchema`), or application code. "No enforced schema by default" is more accurate than "no schema."

**2. What is a document, collection, and database in MongoDB?**
- **Document** — a single record, stored as BSON (binary JSON), the basic unit. Like a row, but can hold nested objects and arrays.
- **Collection** — a group of documents, like a table (but without an enforced schema).
- **Database** — a container of collections.
Mapping to SQL: database→database, collection→table, document→row, field→column.

**3. What is BSON and how does it differ from JSON?**
BSON ("Binary JSON") is the binary-encoded serialization MongoDB uses to store and transmit documents. It extends JSON with **additional types** (e.g., `ObjectId`, `Date`, `Decimal128`, `Binary`, `int32`/`int64`, `Timestamp`) and is designed to be **traversable and fast** to parse. JSON is text-based and human-readable; BSON is binary, typed, and includes length prefixes for efficient scanning.

`↳ Follow-up: Why does MongoDB use BSON instead of plain JSON?` — Speed and richer types. BSON's length-prefixed binary format is faster to scan/skip than parsing text JSON, and it supports types JSON lacks (native dates, 64-bit ints, decimal, binary, ObjectId) — critical for a database that needs precise numeric and date handling.

`↳ Follow-up: Why not use a double for currency?` — Doubles (JSON numbers) have floating-point rounding errors. For money, use **`Decimal128`** (exact decimal) or store integer minor units (cents). Never floats for currency.

**4. What is the `_id` field?**
Every document has a unique `_id` field acting as its **primary key**. If you don't provide one, MongoDB auto-generates an **`ObjectId`**. It's automatically indexed and immutable. `_id` is unique *within a collection*.

`↳ Follow-up: What is an ObjectId and what's inside it?` — A 12-byte identifier: a **4-byte timestamp** (creation time, in seconds), a **5-byte random value** (per process/machine), and a **3-byte incrementing counter**. This makes ObjectIds roughly **time-ordered** and globally unique without coordination — you can even extract the creation time (`objectId.getTimestamp()`).

`↳ Follow-up: Can you use your own value for _id?` — Yes — any unique value (a UUID, an email, a natural key). Just ensure uniqueness. Custom `_id`s can save an index if the natural key is already how you look things up, but lose ObjectId's time-ordering benefit.

`↳ Follow-up: Are ObjectIds a good shard key?` — Generally **no** for the default monotonic case — because they're time-increasing, all new writes hit the same shard (a "hotspot"). A **hashed** ObjectId shard key distributes writes evenly. (More in Phase 8.)

**5. When should you choose MongoDB over a relational database?**
Choose MongoDB when: data is **document-shaped** (nested, hierarchical, varying structure), the schema **evolves rapidly**, you need **horizontal scale** and high write throughput, or your access pattern is "fetch one rich object" rather than complex multi-table joins. Choose relational when you have highly **relational data with complex joins**, need strong multi-row ACID transactions as the norm, or rigid, well-known schemas with heavy ad-hoc querying.

`↳ Follow-up: Can MongoDB do joins?` — Yes, via the aggregation `$lookup` stage (a left outer join). But joins aren't its strength — the document model encourages **embedding** related data to avoid joins. Heavy reliance on `$lookup` often signals a relational schema forced into MongoDB.

`↳ Follow-up: Does MongoDB support ACID transactions?` — Yes — **multi-document ACID transactions** since v4.0 (replica sets) and v4.2 (sharded clusters). But single-document operations are *always* atomic, and good document design often makes multi-document transactions unnecessary. (More in Phase 9.)

**6. What does "MongoDB is single-document atomic" mean?**
Any operation on a **single document** — even updating multiple fields or nested arrays — is **atomic**: it either fully applies or not at all, with no partial state visible to other readers. This is a core design principle: if you model related data **within one document**, you get atomicity for free, without transactions. It's a major reason embedding is favored.

`↳ Follow-up: Why does this favor embedding?` — Because updating an order and its line items in one embedded document is atomic, whereas the same data split across collections would need a multi-document transaction to stay consistent. Modeling around single-document atomicity simplifies correctness.

**7. What storage engine does MongoDB use?**
**WiredTiger** (default since 3.2). It provides document-level concurrency control (multiple writes to different documents in a collection proceed concurrently), compression (snappy by default), and a B-tree based structure with an in-memory cache (the "working set"). Understanding WiredTiger's cache matters for performance — ideally your working set fits in RAM.

`↳ Follow-up: What is the "working set"?` — The portion of data + indexes actively accessed by your queries. For good performance it should fit in RAM (WiredTiger cache); if the working set exceeds RAM, MongoDB pages to disk and performance degrades sharply. Sizing RAM to the working set is a key operational concern.

**8. What are the data types in MongoDB?**
BSON types include: `String`, `Number` (`Int32`, `Int64`, `Double`, `Decimal128`), `Boolean`, `Date`, `ObjectId`, `Array`, `Object` (embedded document), `Null`, `Binary`, `Timestamp`, `Regex`, and `MinKey`/`MaxKey`. Each has a numeric type code, which matters for the `$type` operator and sort ordering across mixed types.

**9. What is the MongoDB connection string (URI)?**
A URI specifying how to connect: `mongodb://user:pass@host:port/dbname?options` or, for clusters/Atlas, `mongodb+srv://...` (which uses DNS SRV records to discover replica set members). Options include `replicaSet`, `retryWrites`, `w` (write concern), `readPreference`, and pool settings.
```ts
const uri = 'mongodb+srv://user:pass@cluster.mongodb.net/myapp?retryWrites=true&w=majority';
```

`↳ Follow-up: What does mongodb+srv do?` — It uses a DNS **SRV record** to look up the cluster's host list (so you don't hardcode every replica member) and a TXT record for default options. Common for MongoDB Atlas. The driver discovers the topology from one SRV lookup.

**10. How does the Node.js driver manage connections?**
The driver maintains a **connection pool** (default max 100) per `MongoClient`. You create **one** `MongoClient` for the app's lifetime and reuse it — never connect per request. The pool handles concurrency, and the driver auto-discovers and monitors the replica set topology, routing operations appropriately.

`↳ Follow-up: Why create only one MongoClient?` — Each client maintains its own pool and topology monitoring. Creating clients per request exhausts connections and adds handshake overhead. One shared client (a singleton) reused across the app is the correct pattern — same reasoning as any DB connection pool.

---

<a name="p2"></a>
## PHASE 2 — CRUD OPERATIONS

**11. What are the basic CRUD operations?**
- **Create:** `insertOne`, `insertMany`.
- **Read:** `find` (returns a cursor of many), `findOne` (single document).
- **Update:** `updateOne`, `updateMany`, `replaceOne`, `findOneAndUpdate`.
- **Delete:** `deleteOne`, `deleteMany`, `findOneAndDelete`.
```ts
await users.insertOne({ name: 'Kishan', city: 'Ahmedabad' });
const user = await users.findOne({ name: 'Kishan' });
await users.updateOne({ _id: id }, { $set: { city: 'Mumbai' } });
await users.deleteOne({ _id: id });
```

**12. What's the difference between `updateOne` and `replaceOne`?**
`updateOne` with update operators (`$set`, `$inc`) modifies **specific fields**, leaving the rest intact. `replaceOne` swaps the **entire document** (except `_id`) with a new one — any field not in the replacement is lost. Use `updateOne` for partial updates (the common case); `replaceOne` to overwrite wholesale.

`↳ Follow-up: What happens if you pass a plain object to updateOne without operators?` — Older drivers would replace the document; modern drivers **throw an error** ("update document requires atomic operators") to prevent the classic bug of accidentally wiping fields. Always use `$set` etc. for partial updates.

**13. What are the key update operators?**
- `$set` — set field values; `$unset` — remove fields.
- `$inc` — increment/decrement numerically (atomic); `$mul` — multiply.
- `$push`/`$pull`/`$addToSet`/`$pop` — array modifications.
- `$rename`, `$min`/`$max`, `$currentDate`.
- `$setOnInsert` — set fields only when an upsert creates a new doc.
```ts
await accounts.updateOne({ _id }, { $inc: { balance: -100 } }); // atomic, race-safe
await posts.updateOne({ _id }, { $push: { tags: 'mongodb' } });
await posts.updateOne({ _id }, { $addToSet: { tags: 'mongodb' } }); // no duplicates
```

`↳ Follow-up: $push vs $addToSet?` — `$push` always appends (allows duplicates); `$addToSet` adds only if the value isn't already present (set semantics). Use `$addToSet` when the array should hold unique values.

`↳ Follow-up: Why is $inc better than read-modify-write for a counter?` — `$inc` is a **single atomic operation** on the server — no race condition. Reading the value, adding in app code, and writing back creates a lost-update race when two requests interleave. Atomic operators (`$inc`, `$push`) push the mutation to the DB where it's safe. (Classic concurrency gotcha.)

**14. What is an upsert?**
An upsert (`{ upsert: true }`) updates a matching document, or **inserts** a new one if none matches. Useful for "create or update" logic in one atomic operation. Combine with `$setOnInsert` to set fields only on creation.
```ts
await counters.updateOne(
  { _id: 'pageViews' },
  { $inc: { count: 1 }, $setOnInsert: { createdAt: new Date() } },
  { upsert: true },
);
```

`↳ Follow-up: What's a concurrency risk with upserts?` — Two concurrent upserts for the same non-existent key can race; MongoDB relies on a **unique index** on the matched field to prevent duplicate inserts (one wins, the other errors/retries). Without a unique index, you can get duplicates. Always back upserts with a unique index on the query key.

**15. What is a projection?**
A projection limits which **fields** are returned, reducing network/memory cost. `1` includes a field, `0` excludes it (you can't mix include/exclude except for `_id`).
```ts
const u = await users.findOne({ _id }, { projection: { name: 1, email: 1, _id: 0 } });
```

`↳ Follow-up: Why does projection improve performance beyond less data transfer?` — It enables **covered queries** — if a query and its projection are fully satisfied by an index (all needed fields are in the index), MongoDB returns results **straight from the index without reading documents** from disk. Hugely faster. (More in Phase 4.)

**16. What is a cursor?**
`find()` returns a **cursor** — a pointer to the result set, fetched in **batches** rather than all at once (default batch ~101 docs, then larger batches). You iterate it (`for await...of`, `.toArray()`, `.next()`). Cursors enable processing large results without loading everything into memory and support `.sort()`, `.limit()`, `.skip()`.
```ts
const cursor = orders.find({ status: 'pending' }).sort({ createdAt: -1 }).limit(20);
for await (const order of cursor) { /* process */ }
```

`↳ Follow-up: Why avoid .toArray() on a huge result set?` — It loads **all** documents into memory at once, risking OOM. Iterate the cursor (streaming) for large sets so only a batch is in memory at a time.

**17. How does pagination work, and why is `skip()` slow at scale?**
`.skip(n).limit(m)` is the simple approach but **slow for large `n`** — MongoDB must scan and discard `n` documents every time. Better at scale: **range/keyset pagination** using an indexed field (often `_id` or a timestamp) as a cursor.
```ts
// Skip-based (slow at high offsets)
orders.find().sort({ _id: 1 }).skip(100000).limit(20);
// Keyset (fast — uses the index)
orders.find({ _id: { $gt: lastSeenId } }).sort({ _id: 1 }).limit(20);
```

`↳ Follow-up: Why is keyset pagination faster?` — It uses an indexed `WHERE _id > cursor`, so MongoDB **seeks directly** to the position via the index and reads only the page — constant time regardless of depth. `skip` reads and throws away everything before the offset. (Same principle as SQL keyset pagination.)

**18. What is the difference between `insertMany` ordered and unordered?**
By default `insertMany` is **ordered** — it stops at the first error. With `{ ordered: false }`, it attempts **all** inserts and reports errors at the end (faster, parallelizable, continues past failures). Use unordered for bulk loads where you want valid docs inserted despite some failures.

---

<a name="p3"></a>
## PHASE 3 — QUERY OPERATORS & ADVANCED QUERYING

**19. What are the comparison query operators?**
`$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in` (matches any value in an array), `$nin` (matches none). These build range and set filters.
```ts
products.find({ price: { $gte: 100, $lte: 500 } });
users.find({ role: { $in: ['admin', 'editor'] } });
```

**20. What are the logical operators?**
`$and`, `$or`, `$not`, `$nor`. Multiple conditions in one object are implicitly AND-ed; use `$or` for alternatives.
```ts
orders.find({ $or: [{ status: 'urgent' }, { total: { $gt: 1000 } }] });
```

`↳ Follow-up: When do you need explicit $and?` — When you have **multiple conditions on the same field** that can't be expressed in one sub-object, e.g. two separate `$or` clauses that must both hold, or repeated operators. Otherwise implicit AND (comma-separated fields) suffices.

**21. How do you query arrays?**
- Match if array **contains** a value: `{ tags: 'mongodb' }` (MongoDB matches if the array includes it).
- Match an exact array: `{ tags: ['a', 'b'] }` (order-sensitive).
- `$all` — array contains all listed values; `$size` — exact length; `$elemMatch` — a single element matches multiple criteria.
```ts
posts.find({ tags: { $all: ['mongodb', 'nodejs'] } });
results.find({ scores: { $elemMatch: { $gte: 80, $lt: 90 } } });
```

`↳ Follow-up: Why do you need $elemMatch?` — Without it, `{ scores: { $gte: 80, $lt: 90 } }` matches if **different** array elements satisfy each condition (one ≥80, another <90). `$elemMatch` requires **one single element** to satisfy *all* conditions together. Critical distinction for array-of-objects queries.

**22. How do you query embedded/nested documents?**
Use **dot notation** to reach nested fields: `{ 'address.city': 'Ahmedabad' }`. For matching a nested object exactly, pass the full sub-document (order-sensitive). Dot notation is the common, flexible approach.
```ts
users.find({ 'address.city': 'Ahmedabad', 'address.country': 'India' });
```

**23. What are element and evaluation operators?**
- **Element:** `$exists` (field present?), `$type` (BSON type match).
- **Evaluation:** `$regex` (pattern match), `$text` (text search), `$expr` (use aggregation expressions in queries — e.g., compare two fields), `$mod`, `$jsonSchema`.
```ts
users.find({ deletedAt: { $exists: false } });            // not soft-deleted
orders.find({ $expr: { $gt: ['$spent', '$budget'] } });   // compare two fields
```

`↳ Follow-up: What does $expr enable that normal queries can't?` — Comparing **two fields of the same document** (`spent > budget`) and using aggregation expressions inside a `find`. Standard query operators compare a field to a constant; `$expr` lets the comparison reference other fields.

`↳ Follow-up: Is $regex efficient?` — Only if it's **anchored at the start** (`/^prefix/`) and the field is indexed — then it can use the index for the prefix. Unanchored or case-insensitive regex (`/foo/i`) usually forces a **collection scan**. For real search, use a text index or Atlas Search instead.

**24. What is the difference between `find` returning no match vs `null`?**
`findOne` returns `null` when no document matches. `find` returns an empty cursor (`.toArray()` → `[]`). Always handle the `null`/empty case — a frequent source of runtime errors (e.g., accessing `.name` on `null`).

**25. How do sorting and collation work?**
`.sort({ field: 1 })` ascending, `-1` descending. Sorting on an **unindexed** field loads results into memory and is capped (32MB in-memory sort limit) — sort on indexed fields. **Collation** controls language-specific string comparison (case sensitivity, accents) — set it for correct alphabetical sorting in non-English locales.

`↳ Follow-up: What happens if an in-memory sort exceeds 32MB?` — The query **fails** (unless `allowDiskUse` is set in aggregation). The fix is an index supporting the sort, so MongoDB returns documents already in order without an in-memory sort. Sorting is a top reason to add a compound index.

---

<a name="p4"></a>
## PHASE 4 — INDEXING

**26. What is an index and why does it matter?**
An index is a data structure (a **B-tree**) storing a sorted subset of field values with pointers to documents, letting MongoDB find matching documents **without scanning the whole collection**. Without an index, a query does a **collection scan** (`COLLSCAN`) — O(n), slow at scale. With one, it does an **index scan** (`IXSCAN`) — roughly O(log n). Indexing is *the* single biggest lever on MongoDB performance. Every collection has a default index on `_id`.

`↳ Follow-up: What's the trade-off of adding indexes?` — Indexes speed reads but **slow writes** (every insert/update/delete must update all relevant indexes) and consume RAM and disk. Too many indexes hurt write throughput and waste memory. Index for your actual query patterns — no more.

**27. What are the main index types?**
- **Single-field** — on one field.
- **Compound** — on multiple fields (order matters — see ESR).
- **Multikey** — automatically created when you index an array field (one index entry per array element).
- **Text** — for text search across string fields.
- **Geospatial** (`2dsphere`) — for location queries.
- **Hashed** — hashes the value (used for hashed sharding, even write distribution).
- **Wildcard** — index unknown/arbitrary field paths.
- **TTL** — auto-expires documents after a time (see Phase 13).
- **Partial / Sparse** — index only a subset of documents.

**28. What is a compound index and the ESR rule?**
A compound index covers multiple fields in a specific order. **Field order is critical.** The **ESR rule** orders fields as: **Equality** first (fields matched exactly), then **Sort** fields, then **Range** fields (`$gt`/`$lt`). Following ESR lets one index serve filtering *and* sorting efficiently.
```ts
// Query: find active users in a city, sorted by signup date, age range
users.find({ status: 'active', city: 'Ahmedabad', age: { $gte: 18 } }).sort({ signupDate: -1 });
// Optimal index (ESR): Equality (status, city) → Sort (signupDate) → Range (age)
await users.createIndex({ status: 1, city: 1, signupDate: -1, age: 1 });
```

`↳ Follow-up: Why does Equality come before Sort before Range?` — Equality fields narrow to a contiguous index range; placing them first shrinks the scanned portion. The sort field next lets MongoDB read entries **already in sorted order** (no in-memory sort). Range last because it returns a span, which would otherwise break the sort ordering for subsequent fields. Wrong order → in-memory sorts or extra scanning.

`↳ Follow-up: Can a compound index on {a, b, c} serve a query on just {a}? On just {b}?` — On `{a}` **yes** (it's a left **prefix**). On just `{b}` **no** — you can't skip the leading field. This "prefix" rule is why field order in compound indexes is a design decision, and why one well-ordered compound index can replace several single-field ones.

**29. What is a multikey index?**
When you index a field that holds an **array**, MongoDB creates a **multikey index** automatically — one index entry per array element — so queries can match any element efficiently.
```ts
await posts.createIndex({ tags: 1 }); // multikey if tags is an array
posts.find({ tags: 'mongodb' });      // uses the index
```

`↳ Follow-up: Can you have a compound index on two array fields?` — **No.** MongoDB can't create a compound multikey index where **more than one** field is an array (the number of entries would explode combinatorially). At most one indexed field per compound index can be an array.

**30. What is a covered query?**
A query is **covered** when all the fields it needs (for both the filter and the projection) exist in an index, so MongoDB answers it **entirely from the index without fetching documents**. The fastest possible query. Requires excluding `_id` from the projection unless `_id` is in the index.
```ts
await users.createIndex({ email: 1, name: 1 });
users.find({ email: 'a@b.com' }, { projection: { name: 1, email: 1, _id: 0 } }); // covered
```

**31. How do you analyze query performance? What is `explain()`?**
`explain()` (or `explain('executionStats')`) shows the **query plan**: which index was used, whether it was a `COLLSCAN` vs `IXSCAN`, documents examined vs returned, and execution time. The key health signal is **`totalDocsExamined` vs `nReturned`** — they should be close. A huge ratio means the query scans far more than it returns → missing/poor index.
```ts
const plan = await users.find({ city: 'Ahmedabad' }).explain('executionStats');
// look at: winningPlan.stage (IXSCAN good, COLLSCAN bad),
//          executionStats.totalDocsExamined vs nReturned
```

`↳ Follow-up: What does a big gap between docsExamined and nReturned mean?` — The query is **inefficient** — scanning many documents to return few. Either no index supports it (collection scan) or the index isn't selective enough. Add or fix an index so examined ≈ returned.

`↳ Follow-up: What is the difference between COLLSCAN and IXSCAN in the plan?` — `COLLSCAN` = full collection scan (reads every document — bad for large collections). `IXSCAN` = index scan (seeks via the B-tree — good). Seeing `COLLSCAN` on a hot query is a red flag to add an index.

**32. What are partial and sparse indexes?**
- **Sparse** index: only indexes documents where the field **exists**, skipping those missing it — smaller index.
- **Partial** index: indexes only documents matching a **filter expression** — more flexible than sparse and generally preferred.
```ts
// Index only active users — smaller, targeted
await users.createIndex({ email: 1 }, { partialFilterExpression: { status: 'active' } });
```

`↳ Follow-up: Partial vs sparse — which to prefer?` — **Partial** — it's a superset of sparse's capability (you can express "field exists" as a partial filter) with more control. The docs recommend partial over sparse for new use cases.

**33. What is a unique index, and how does it interact with nulls?**
A unique index enforces no two documents share the same value for the field(s). Useful for emails, usernames. Note: by default, **missing fields count as `null`**, so a unique index allows only **one** document missing the field. Combine `unique` with `partial` (only index documents where the field exists) to allow many documents without the field.
```ts
await users.createIndex({ email: 1 }, { unique: true });
// Allow multiple docs without email but unique among those that have it:
await users.createIndex({ email: 1 }, { unique: true, partialFilterExpression: { email: { $exists: true } } });
```

**34. How do text indexes and text search work?**
A **text index** tokenizes string fields for word-based search via `$text`/`$search`, with stemming and stop-word handling. A collection can have **only one** text index (covering one or more fields).
```ts
await articles.createIndex({ title: 'text', body: 'text' });
articles.find({ $text: { $search: 'mongodb performance' } }, { score: { $meta: 'textScore' } });
```

`↳ Follow-up: When should you use Atlas Search instead of $text?` — For real-world search needs — relevance tuning, fuzzy matching, autocomplete, faceting — **Atlas Search** (built on Apache Lucene) is far more capable than the basic `$text` index. Use `$text` for simple keyword matching; Atlas Search for actual search features.

**35. What is index selectivity and why does it matter?**
Selectivity is how well an index narrows results — a **highly selective** field (many distinct values, like email) filters to few documents; a **low-selectivity** field (like a boolean `isActive`) barely narrows. Highly selective fields make better index leading fields. Indexing a low-selectivity field alone often isn't worth it (MongoDB may even prefer a scan).

---

<a name="p5"></a>
## PHASE 5 — THE AGGREGATION FRAMEWORK

**36. What is the aggregation framework?**
A pipeline-based data processing framework: documents flow through a sequence of **stages**, each transforming the stream (filter, group, reshape, join, compute), like Unix pipes. It's MongoDB's tool for analytics, reporting, and complex transformations that go beyond simple `find`.
```ts
const result = await orders.aggregate([
  { $match: { status: 'completed' } },              // filter
  { $group: { _id: '$customerId', total: { $sum: '$amount' } } }, // group + sum
  { $sort: { total: -1 } },                          // sort
  { $limit: 10 },                                    // top 10
]).toArray();
```

**37. What are the most important aggregation stages?**
- `$match` — filter (like `find`); put it **first** to use indexes and reduce the stream early.
- `$group` — group by a key and compute accumulators (`$sum`, `$avg`, `$min`, `$max`, `$push`, `$addToSet`).
- `$project` — reshape documents (include/exclude/compute fields).
- `$sort`, `$limit`, `$skip` — ordering and paging.
- `$lookup` — left outer join to another collection.
- `$unwind` — deconstruct an array field into one document per element.
- `$addFields`/`$set` — add computed fields without dropping others.
- `$count`, `$facet`, `$bucket`, `$replaceRoot`, `$out`/`$merge` (write results).

**38. Why does `$match` placement matter?**
Putting `$match` (and `$sort`) **early** — ideally first — lets the pipeline **use indexes** and shrink the document stream before expensive stages (`$group`, `$lookup`, `$unwind`). A `$match` after a `$group` can't use the original collection's indexes. Order stages to filter aggressively up front.

`↳ Follow-up: Does the aggregation pipeline use indexes?` — Yes, but mainly for the **leading** `$match` and `$sort` stages (and `$lookup`'s foreign field). Once data is transformed by `$group`/`$project`, subsequent stages operate in memory without the collection's indexes. So front-load index-usable filters.

**39. What is `$lookup` and its caveats?**
`$lookup` performs a **left outer join**, pulling matching documents from another collection into an array field.
```ts
orders.aggregate([
  { $lookup: { from: 'customers', localField: 'customerId', foreignField: '_id', as: 'customer' } },
  { $unwind: '$customer' }, // flatten the joined array (since it's 1:1)
]);
```

`↳ Follow-up: Performance concern with $lookup?` — It can be expensive — for each input document it queries the foreign collection. **Index the `foreignField`**, filter the input set first (`$match` before `$lookup`), and avoid `$lookup` on huge unfiltered collections. Heavy joining suggests the data might be better embedded or the schema reconsidered.

**40. What does `$unwind` do?**
`$unwind` deconstructs an array field, outputting **one document per array element** (the rest of the document duplicated). Essential for grouping or filtering by individual array elements.
```ts
// A post with tags:['a','b'] becomes two documents, one per tag
posts.aggregate([{ $unwind: '$tags' }, { $group: { _id: '$tags', count: { $sum: 1 } } }]);
```

`↳ Follow-up: What if the array is empty or missing — does the document survive $unwind?` — By default it's **dropped**. Use `{ $unwind: { path: '$tags', preserveNullAndEmptyArrays: true } }` to keep documents with empty/missing arrays. A common silent data-loss bug.

**41. What is `$facet` and when do you use it?**
`$facet` runs **multiple sub-pipelines in parallel** on the same input, returning all their results in one document — useful for dashboards (e.g., compute totals, top items, and a histogram in a single query) or "search results + total count + filters" responses.

**42. What are `$out` and `$merge`?**
Terminal stages that **write** the pipeline's output to a collection. `$out` replaces a collection entirely; `$merge` (more flexible) inserts/updates into an existing collection (great for incremental materialized views/rollups). Use them for precomputed reports/aggregates.

`↳ Follow-up: What's a materialized view pattern in MongoDB?` — Periodically run an aggregation and `$merge` results into a summary collection, so expensive analytics are precomputed and reads are cheap. You trade freshness for read speed — useful for dashboards.

**43. What is the aggregation memory limit, and what is `allowDiskUse`?**
Each pipeline stage is limited to **100MB** of RAM by default. Stages exceeding it (large `$group`/`$sort`) fail unless you set **`allowDiskUse: true`**, which lets them spill to temporary disk files (slower but unbounded). For big analytical pipelines, enable it.
```ts
orders.aggregate(pipeline, { allowDiskUse: true });
```

**44. Aggregation pipeline vs `map-reduce`?**
The **aggregation pipeline** is the modern, preferred tool — faster, more expressive, and optimized. **Map-reduce** is legacy (deprecated), slower, and runs JavaScript. Anything map-reduce did, the pipeline now does better. Mention aggregation as the answer; map-reduce only for historical awareness.

---

<a name="p6"></a>
## PHASE 6 — SCHEMA DESIGN & DATA MODELING

**45. What is the fundamental rule of MongoDB schema design?**
**Model your data around your application's access patterns (queries), not around normalized entities.** Unlike relational design (normalize first, join later), MongoDB design starts with "how will I read and write this data?" and structures documents so the most common operations are a single, efficient query. "Data that is accessed together should be stored together."

`↳ Follow-up: How is this different from relational modeling?` — Relational: normalize to eliminate redundancy, then join at query time. MongoDB: optimize for read/write patterns, often **denormalizing** (duplicating) data to avoid joins. You trade some redundancy and update complexity for fast, single-query reads.

**46. Embedding vs Referencing — what's the difference and when to use each?**
- **Embedding** (nest related data in one document): best for data accessed **together**, "contains"/one-to-few relationships, and when you want atomic single-document updates. Fast reads (one query), but documents can grow and data may be duplicated.
- **Referencing** (store an `_id` pointer, join with `$lookup`/`populate`): best for **large** or **independently-accessed** sub-data, many-to-many, or frequently-changing shared data. Avoids duplication and unbounded growth, at the cost of extra queries.

```ts
// Embedded — order with its line items (read together, atomic)
{ _id, customer, items: [{ sku, qty, price }], total }
// Referenced — post references author by id
{ _id, title, authorId: ObjectId('...') }
```

`↳ Follow-up: Give the rule of thumb.` — **Embed** for "one-to-few" and data read together; **reference** for "one-to-many/-squillions," large sub-documents, or data that's shared/updated independently. Ask: is this sub-data always fetched with the parent? Does it grow unbounded? Is it shared?

**47. What is the 16MB document size limit and why does it matter for design?**
A single BSON document **cannot exceed 16MB**. This caps **unbounded embedding** — you can't embed a comments array that grows forever, or a user's entire activity log. When a sub-collection can grow without bound, **reference** it instead (or use the outlier/bucket pattern). Designing around this limit is a core skill.

`↳ Follow-up: What's the "massive arrays" anti-pattern?` — Embedding an ever-growing array (e.g., all comments on a post, all events for a user) in one document. It approaches the 16MB limit, slows every read/write of that document, and fragments memory. Fix: reference into a separate collection, or bucket (group N items per document).

**48. What are common schema design patterns?**
- **Bucket pattern:** group many small time-series/IoT readings into "bucket" documents (e.g., one doc per sensor per hour) — fewer documents, better compression.
- **Outlier pattern:** handle the rare document that would otherwise blow up (e.g., a celebrity with millions of followers) differently from the norm.
- **Computed pattern:** precompute and store aggregates (totals, counts) instead of recalculating on every read.
- **Subset pattern:** embed only the **most-accessed subset** (e.g., 10 most recent reviews) and reference the rest.
- **Schema versioning:** add a `schemaVersion` field to evolve document shapes over time gracefully.
- **Extended reference:** duplicate a few frequently-needed fields from a referenced doc to avoid the join.

`↳ Follow-up: What is the extended reference pattern?` — Instead of referencing only an `_id` and joining, you **copy a few hot fields** (e.g., store `authorName` alongside `authorId` on a post) so common reads don't need a `$lookup`. You accept duplication and the need to update copies when the source changes — worth it when reads vastly outnumber those updates.

**49. How do you model one-to-many relationships?**
Three approaches: (1) **Embed** the "many" in the "one" (one-to-few, read together). (2) **Reference** from the "many" side (child stores parent `_id`) — for one-to-many where children are numerous/independent. (3) **Array of references** in the parent (for one-to-few where you need the list on the parent). Choose by cardinality and access pattern.

**50. How do you model many-to-many?**
Usually **references on both sides** or a dedicated relationship collection. E.g., students and courses: each student doc holds an array of `courseIds` (or vice versa), or a separate `enrollments` collection links them (better when the relationship itself has attributes like enrollment date/grade).

`↳ Follow-up: When use a separate relationship collection vs arrays of refs?` — Use a **relationship collection** when the link has its own data (enrollment date, role, status) or when either side's array would grow unbounded. Arrays of refs are fine for bounded many-to-many without link attributes.

**51. What are MongoDB schema anti-patterns?**
- **Massive unbounded arrays** (16MB risk).
- **Bloated documents** with rarely-used fields read on every query.
- **Too many collections** mimicking relational tables (over-normalization).
- **Unnecessary `$lookup`-heavy designs** (relational schema forced into Mongo).
- **Case-insensitive queries without collation/index** (forces scans).
- **Separating data always accessed together** into multiple collections (causes joins).
Atlas even flags several of these automatically.

`↳ Follow-up: How do you decide between normalizing and denormalizing?` — Read/write ratio and consistency needs. **Denormalize** (duplicate) when reads dominate and you can tolerate updating copies. **Normalize** (reference) when data changes often and must stay consistent, or when duplication would be large/unbounded. It's a trade-off, not a rule.

---

<a name="p7"></a>
## PHASE 7 — REPLICATION & HIGH AVAILABILITY

**52. What is a replica set?**
A replica set is a group of `mongod` instances maintaining the **same data** for redundancy and high availability. It has one **primary** (accepts all writes) and multiple **secondaries** (replicate the primary's data). If the primary fails, the set automatically **elects** a new primary, so the database stays available. This is MongoDB's core HA mechanism — production deployments always use replica sets.

`↳ Follow-up: Minimum nodes for a replica set with automatic failover?` — **Three** (typically primary + 2 secondaries, or primary + secondary + **arbiter**). You need an odd number of voting members to win elections (a majority); three is the practical minimum for automatic failover.

`↳ Follow-up: What is an arbiter?` — A voting-only member that holds **no data** — it participates in elections to break ties (maintain an odd vote count) without the storage cost of a full data-bearing node. Use sparingly; data-bearing members are preferred for durability.

**53. How does replication actually work (the oplog)?**
The primary records every write operation in a special capped collection called the **oplog** (operations log). Secondaries continuously **tail** the primary's oplog and apply the same operations to their own copies, staying in sync. The oplog is idempotent (operations can be reapplied safely) and is what makes replication, point-in-time recovery, and change streams possible.

`↳ Follow-up: What is replication lag?` — The delay between a write on the primary and its application on a secondary. High lag means secondaries serve stale data and failover may lose recent writes. Causes: slow secondaries, network latency, heavy write load. Monitor it (`rs.printSecondaryReplicationInfo()`).

`↳ Follow-up: What happens if the oplog is too small?` — If a secondary falls behind further than the oplog's time window (the oplog is capped and overwrites old entries), it can no longer catch up incrementally and needs a **full resync**. Size the oplog to cover your longest expected secondary downtime/maintenance.

**54. What is an election and how is a primary chosen?**
When the primary becomes unreachable, the remaining members hold an **election** (Raft-like protocol): a member needs votes from a **majority** of voting members to become primary. Factors include priority settings and how up-to-date each member is (the most current eligible member is favored). Elections typically complete in seconds, during which writes are unavailable.

`↳ Follow-up: Why does a replica set need a majority to elect?` — To prevent **split-brain** (two primaries). Requiring a strict majority ensures only one partition can elect a primary; a minority partition steps down to read-only. This is why an odd number of voting members matters.

**55. What is read preference?**
Read preference controls **which members** serve reads: `primary` (default — strong consistency), `primaryPreferred`, `secondary`, `secondaryPreferred`, `nearest`. Reading from secondaries can offload the primary and reduce latency, but risks **stale data** due to replication lag.
```ts
db.collection('users').find({}, { readPreference: 'secondaryPreferred' });
```

`↳ Follow-up: What's the risk of reading from secondaries?` — **Eventual consistency** — secondaries may lag, so you can read stale data (even read your own write as missing). Don't use secondary reads for read-after-write scenarios. Good for analytics/reporting that tolerate slight staleness.

**56. What is write concern?**
Write concern (`w`) specifies how many members must **acknowledge** a write before it's considered successful — controlling the durability/latency trade-off. `w: 1` (primary only — fast, less safe), `w: 'majority'` (a majority acknowledged — durable, survives failover), `w: 0` (fire-and-forget — no ack). Add `j: true` to require the write be journaled to disk.
```ts
await orders.insertOne(doc, { writeConcern: { w: 'majority', j: true } });
```

`↳ Follow-up: Why use w: 'majority'?` — It guarantees the write is on a **majority** of nodes, so it **survives a primary failover** without being rolled back. With `w: 1`, a write acknowledged only by the primary can be **lost** if that primary crashes before replicating. `majority` is the safe default for important data.

`↳ Follow-up: What is a rollback in a replica set?` — If a primary accepts writes that **don't** reach a majority, then crashes, and a new primary is elected without those writes, the old primary (on rejoining) must **roll back** its un-replicated writes to match the new primary. `w: 'majority'` prevents this for acknowledged writes.

**57. What is the relationship between read concern and write concern?**
**Write concern** = durability (how many nodes acked the write). **Read concern** = consistency/recency of data read (`local`, `available`, `majority`, `linearizable`, `snapshot`). `readConcern: 'majority'` returns only data acknowledged by a majority (won't be rolled back). Together they tune the consistency–performance balance.

---

<a name="p8"></a>
## PHASE 8 — SHARDING & HORIZONTAL SCALING

**58. What is sharding?**
Sharding is MongoDB's method of **horizontal scaling** — partitioning a collection's data across multiple servers (**shards**) so no single machine holds everything. It distributes both storage and load (reads/writes), enabling datasets and throughput beyond one machine's capacity. Each shard is itself a replica set (for HA). You shard when a single replica set can't handle the data volume or write load.

`↳ Follow-up: Replication vs sharding — what's the difference?` — **Replication** = copies of the *same* data for redundancy/HA (scales reads, provides failover). **Sharding** = *splitting* data across machines for capacity/write scaling. They're complementary: a sharded cluster's shards are each replica sets. Replication ≠ scaling writes; sharding does.

**59. What are the components of a sharded cluster?**
- **Shards** — each a replica set holding a subset of the data.
- **Config servers** — a replica set storing the cluster's **metadata** (which data ranges live on which shard).
- **`mongos` (query router)** — routes client queries to the correct shard(s) using the config metadata; the app connects to `mongos`, not shards directly.

**60. What is a shard key and why is it the most important decision?**
The shard key is the field(s) MongoDB uses to **partition** data across shards. It's chosen at shard time and (historically) hard to change, and it determines write/read distribution. A **bad shard key creates hotspots** (all traffic to one shard) or **scatter-gather** queries (every query hits all shards). Picking it well is the hardest, highest-stakes sharding decision.

`↳ Follow-up: What makes a good shard key?` — High **cardinality** (many distinct values), low **frequency** (no single value dominates), and **non-monotonic** (not ever-increasing) write distribution — ideally aligned with your common query filters so queries are **targeted** (hit one shard) not broadcast. A key matching your query pattern avoids scatter-gather.

**61. Ranged vs hashed sharding?**
- **Ranged:** data partitioned by **value ranges** of the shard key. Good for **range queries** (contiguous data on one shard), but a monotonically increasing key (timestamp, ObjectId) sends all new writes to one shard (**hotspot**).
- **Hashed:** MongoDB hashes the shard key, distributing writes **evenly** regardless of value. Great write distribution, but **range queries become scatter-gather** (related values land on different shards).

`↳ Follow-up: Why is a monotonically increasing shard key (like timestamp) bad for ranged sharding?` — All new documents have the **highest** key value, so they all route to the **same** shard (the one owning the top range) — that shard becomes a write hotspot while others sit idle. Hashed sharding or a compound key with a high-cardinality prefix fixes this.

**62. What are chunks and the balancer?**
MongoDB divides sharded data into **chunks** (contiguous shard-key ranges). The **balancer** is a background process that migrates chunks between shards to keep data **evenly distributed** as it grows. When a chunk gets too large, it **splits**. You generally let the balancer run automatically; it can be scheduled to off-peak windows.

**63. What is a targeted query vs a scatter-gather query?**
- **Targeted:** the query includes the **shard key**, so `mongos` routes it to the **specific shard(s)** holding the data — efficient.
- **Scatter-gather (broadcast):** the query lacks the shard key, so `mongos` must query **all shards** and merge results — slow, scales poorly. Designing queries to include the shard key keeps them targeted.

`↳ Follow-up: How does shard key choice affect query routing?` — If your common queries filter by the shard key, they're **targeted** (one shard). If they filter by other fields, every query is **scatter-gather**. This is why the shard key should align with your dominant query pattern — it's not just about write distribution.

**64. Can you change a shard key?**
Historically immutable. Modern MongoDB (4.4+) allows **refining** a shard key (adding suffix fields) and (5.0+) **resharding** a collection to an entirely new key — but resharding is a heavy, resource-intensive operation. The lesson stands: **choose carefully up front**, because fixing it later is expensive.

---

<a name="p9"></a>
## PHASE 9 — TRANSACTIONS & CONSISTENCY

**65. Does MongoDB support ACID transactions?**
Yes. **Single-document** operations are always atomic. **Multi-document, multi-collection ACID transactions** are supported since v4.0 (replica sets) and v4.2 (sharded clusters). They provide all-or-nothing semantics across multiple documents with snapshot isolation. But they carry overhead, so the idiomatic approach is to **design schemas (embedding) so most operations are single-document** and transactions are the exception.

```ts
const session = client.startSession();
try {
  await session.withTransaction(async () => {
    await accounts.updateOne({ _id: from }, { $inc: { balance: -100 } }, { session });
    await accounts.updateOne({ _id: to },   { $inc: { balance:  100 } }, { session });
  });
} finally {
  await session.endSession();
}
```

`↳ Follow-up: If single-doc ops are atomic, why design to avoid transactions?` — Transactions add **performance cost** (locks, coordination, especially across shards) and complexity. If related data is **embedded in one document**, you get atomicity for free with better performance. Reserve multi-document transactions for genuinely cross-document invariants (transfers between accounts).

`↳ Follow-up: What's a caveat of MongoDB transactions?` — They should be **short-lived** (default 60s limit) — long transactions hold resources and can abort. Also, large transactions hit oplog/size limits, and on sharded clusters they're more expensive. Keep them small and fast; don't do bulk work inside one.

**66. What consistency model does MongoDB provide?**
By default, reading from the **primary** gives **strong consistency** (read-after-write). Reading from secondaries gives **eventual consistency** (subject to replication lag). You tune the spectrum with **read concern** and **write concern**. MongoDB also offers **causal consistency** within a session (reads reflect prior writes in that session). So it's strongly consistent by default but configurable toward availability/performance.

`↳ Follow-up: What is causal consistency?` — Within a **client session**, operations observe a causally-consistent order — a read after your write sees that write, even across primary/secondary, because the session tracks operation times. Enable it for "read your own writes" guarantees without forcing all reads to the primary.

**67. How does MongoDB relate to the CAP theorem?**
Under network partition, MongoDB favors **consistency over availability** (CP) by default — the minority partition steps down to read-only and only the majority side accepts writes, preventing conflicting data. You can lean toward availability (e.g., secondary reads, lower write concern) at the cost of consistency. So MongoDB is **tunable**, defaulting to CP.

**68. What is optimistic concurrency control in MongoDB?**
A pattern to prevent lost updates without locking: include a **version field** (or the last-known value) in your update's filter, so the update only applies if the document hasn't changed since you read it. If the filter doesn't match (someone else updated it), the update affects 0 documents and you retry.
```ts
const res = await docs.updateOne(
  { _id, version: currentVersion },                 // only if unchanged
  { $set: { ...changes }, $inc: { version: 1 } },
);
if (res.matchedCount === 0) { /* conflict — re-read and retry */ }
```

`↳ Follow-up: When use this over a transaction?` — For **single-document** updates where you want to guard against concurrent modification (the lost-update problem) without the overhead of a transaction. It's lighter and works naturally with MongoDB's single-document atomicity. Mongoose supports this via `optimisticConcurrency`.

---

<a name="p10"></a>
## PHASE 10 — PERFORMANCE & OPTIMIZATION

**69. What's your general approach to optimizing a slow MongoDB query?**
1. **Profile/measure** — find the slow query (database profiler, slow query log, Atlas Performance Advisor).
2. **`explain()`** it — check for `COLLSCAN`, and the `docsExamined` vs `nReturned` ratio.
3. **Add/fix an index** — usually a missing or poorly-ordered (ESR) compound index.
4. **Reduce work** — project only needed fields, filter early, limit results.
5. **Reshape** — sometimes the schema (embedding/denormalizing) is the real fix.
6. **Re-measure**. Never optimize blind.

**70. What is the database profiler?**
A built-in tool that records operations exceeding a time threshold (or all ops) into the `system.profile` collection, capturing slow queries with their execution stats. Levels: 0 (off), 1 (slow ops above `slowms`), 2 (all). Use it to find which queries to optimize. Atlas surfaces this plus index suggestions automatically.

**71. What are the most common MongoDB performance problems?**
- **Missing indexes** → collection scans.
- **Unindexed sorts** → in-memory sort, 32MB cap failures.
- **Inefficient `$lookup`** / over-joining.
- **Large documents / unbounded arrays** → memory and I/O pressure.
- **Returning too much data** (no projection/limit).
- **Working set exceeding RAM** → constant disk paging.
- **Skip-based pagination** at high offsets.
- **N+1 query patterns** (especially via naive `populate`).

`↳ Follow-up: What is the N+1 problem in MongoDB/Mongoose?` — Fetching a list (1 query) then looping to fetch each item's reference separately (N queries) — e.g., 50 posts then 50 author lookups. Fix with `populate` (Mongoose batches it), a single `$lookup`, or an `$in` query fetching all references at once.

**72. How do you know if your working set fits in RAM?**
Monitor the WiredTiger cache usage and page faults / disk I/O. If queries that should be fast are doing heavy disk reads, or cache eviction is high, the working set likely exceeds RAM. Symptoms: rising latency under load, high disk activity. Fixes: add RAM, shard to spread the working set, or reduce data scanned (better indexes/projections).

**73. Why can too many indexes hurt?**
Every index must be **updated on every write** to the indexed fields, slowing inserts/updates/deletes, and each index consumes RAM (competing with the working set) and disk. Unused or redundant indexes are pure cost. Audit with `$indexStats` and drop indexes that aren't used.

`↳ Follow-up: How do you find unused indexes?` — `db.collection.aggregate([{ $indexStats: {} }])` shows access counts per index. Indexes with near-zero `accesses.ops` over a representative period are candidates to drop (after confirming they're not needed for rare-but-critical queries).

**74. What is the impact of document growth / fragmentation?**
When an update grows a document beyond its allocated space, WiredTiger handles it (less painful than the old MMAPv1 engine's in-place moves), but excessive growth still causes extra I/O and fragmentation over time. Designing documents close to their final size (or using the bucket/outlier patterns) keeps writes efficient.

**75. How do bulk operations improve performance?**
`bulkWrite` batches many inserts/updates/deletes into **one round-trip**, drastically reducing network overhead vs issuing them individually. Use ordered/unordered as appropriate. Essential for data imports and batch processing.
```ts
await users.bulkWrite([
  { updateOne: { filter: { _id: 1 }, update: { $set: { active: true } } } },
  { insertOne: { document: { name: 'New' } } },
], { ordered: false });
```

---

<a name="p11"></a>
## PHASE 11 — MONGOOSE & TYPESCRIPT

**76. What is Mongoose and what does it add over the native driver?**
Mongoose is an **ODM (Object Data Modeling)** library for MongoDB in Node.js. Over the raw driver it adds: **schemas** (structure + types), **validation**, **middleware/hooks** (pre/post save, etc.), **population** (reference resolution / pseudo-joins), **type casting**, **query helpers**, virtuals, and a more ergonomic model API. It brings structure and convenience to MongoDB's flexibility — at the cost of some overhead and abstraction.

`↳ Follow-up: When would you use the native driver instead of Mongoose?` — For maximum performance/control, simple apps that don't need schemas/validation, or heavy aggregation/bulk work where Mongoose's overhead and abstractions add little. Many teams use the driver directly with a validation layer (Zod) and TypeScript for type safety. Mongoose shines when you want schemas, hooks, and population out of the box.

**77. How do you define a typed Mongoose schema and model in TypeScript?**
Define a TS interface for the document, then a schema typed to it, then the model.
```ts
import { Schema, model, Document, Types } from 'mongoose';

interface IUser {
  email: string;
  name: string;
  age?: number;
  roles: string[];
  createdAt: Date;
}

const userSchema = new Schema<IUser>({
  email: { type: String, required: true, unique: true, lowercase: true, trim: true },
  name:  { type: String, required: true },
  age:   { type: Number, min: 0 },
  roles: { type: [String], default: ['user'] },
}, { timestamps: true }); // auto createdAt/updatedAt

export const User = model<IUser>('User', userSchema);

const user = await User.create({ email: 'a@b.com', name: 'Kishan', roles: ['user'] });
```

`↳ Follow-up: How do you keep the TS type and schema in sync automatically?` — Use **`InferSchemaType<typeof schema>`** to derive the TS type *from* the schema (single source of truth), or tools like `@typegoose/typegoose` (class-based, decorators) that generate the schema from a typed class. This avoids the schema and interface drifting apart.

**78. What is `populate` and how does it relate to `$lookup`?**
`populate` resolves **referenced** documents — replacing stored `ObjectId`s with the actual referenced documents. Under the hood it issues additional queries (or uses `$lookup`), batching them to avoid N+1. It's Mongoose's pseudo-join for referenced data.
```ts
const postSchema = new Schema({ title: String, author: { type: Schema.Types.ObjectId, ref: 'User' } });
const post = await Post.findById(id).populate<{ author: IUser }>('author');
post.author.name; // typed, resolved
```

`↳ Follow-up: populate vs $lookup — trade-offs?` — `populate` runs **separate queries** (the driver/Mongoose batches references with `$in`) — flexible, easy, but multiple round-trips. `$lookup` joins **inside one aggregation** on the server — fewer round-trips, but less flexible and can be heavy. For simple ref resolution, `populate`; for complex server-side joins/transforms, `$lookup`.

`↳ Follow-up: Does populate avoid N+1?` — Mostly — Mongoose batches the populated references into a single `$in` query per path (not one per parent). But populating across many paths/levels still multiplies queries. For deep/complex needs, a single aggregation `$lookup` can be more efficient.

**79. What are Mongoose middleware/hooks?**
Functions that run **before (`pre`) or after (`post`)** certain operations (`save`, `validate`, `findOneAndUpdate`, `remove`, aggregate). Used for: hashing passwords before save, setting timestamps, cascading deletes, logging, transforming data.
```ts
userSchema.pre('save', async function () {
  if (this.isModified('password')) this.password = await bcrypt.hash(this.password, 12);
});
```

`↳ Follow-up: A common gotcha with update hooks?` — `pre('save')` does **not** run on `updateOne`/`findOneAndUpdate` (those bypass the document and hit the DB directly). You must add separate `pre('findOneAndUpdate')` hooks, and `this` refers to the **query**, not the document. Password-hashing logic placed only in `save` silently won't run on updates — a real bug source.

**80. What validation does Mongoose provide?**
Built-in validators: `required`, `min`/`max`, `minlength`/`maxlength`, `enum`, `match` (regex), `unique` (note: not a true validator — it's an index hint), and **custom validators**. Validation runs on `save` and (optionally) on updates with `runValidators: true`.
```ts
age: { type: Number, min: [0, 'Age cannot be negative'], max: 120 },
status: { type: String, enum: ['active', 'banned'], default: 'active' },
```

`↳ Follow-up: Why is `unique: true` not really a validator?` — It doesn't validate in application code — it just tells Mongoose to **build a unique index**. Enforcement happens at the **database** level, surfacing as a duplicate-key error (E11000) on insert, *not* a Mongoose validation error. You must catch the DB error, and the index must actually exist.

**81. What are virtuals, lean queries, and when do you use `.lean()`?**
- **Virtuals:** computed properties not stored in MongoDB (e.g., `fullName` from `first`+`last`).
- **`.lean()`:** returns **plain JS objects** instead of full Mongoose documents — much faster and lighter (skips hydration, getters, virtuals, change tracking). Use for **read-only** queries where you don't need document methods.
```ts
const users = await User.find().lean(); // fast, plain objects, read-only
```

`↳ Follow-up: What do you lose with .lean()?` — Mongoose document features: virtuals, getters/setters, `save()`, validation, and instance methods. The result is a raw object. Perfect for read-heavy endpoints (APIs returning JSON), wrong when you need to modify and save the document.

**82. How do you handle transactions in Mongoose?**
Use a session and pass it to operations, ideally via `withTransaction`.
```ts
const session = await mongoose.startSession();
await session.withTransaction(async () => {
  await Account.updateOne({ _id: from }, { $inc: { balance: -100 } }, { session });
  await Account.updateOne({ _id: to },   { $inc: { balance:  100 } }, { session });
});
session.endSession();
```

**83. How do you connect Mongoose properly in a TypeScript app?**
Create **one** connection at startup, reuse it, handle errors/disconnects, and (in serverless) cache the connection across invocations to avoid exhausting the pool.
```ts
import mongoose from 'mongoose';
export async function connectDB(uri: string) {
  mongoose.set('strictQuery', true);
  await mongoose.connect(uri, { maxPoolSize: 20 });
  mongoose.connection.on('error', (err) => logger.error({ err }, 'mongo error'));
}
```

`↳ Follow-up: Why cache the connection in serverless?` — Each function invocation could otherwise open a new connection, exhausting MongoDB's limits under load. Cache the connection (e.g., on `global`) so warm invocations reuse it. (Same connection-explosion issue as any DB in serverless.)

---

<a name="p12"></a>
## PHASE 12 — SECURITY

**84. What is NoSQL injection and how do you prevent it?**
NoSQL injection exploits the fact that MongoDB queries are **objects** — if user input is placed directly into a query, an attacker can inject **operators**. Classic example: a login where `req.body.password` is `{ "$ne": null }`, turning `{ password: input }` into `{ password: { $ne: null } }` — matching any password.
```ts
// VULNERABLE — if req.body values are objects
await User.findOne({ email: req.body.email, password: req.body.password });
// Attacker sends { email: {"$gt":""}, password: {"$gt":""} } → matches a user
```
Prevent by: **validating/sanitizing input types** (ensure strings are strings — Zod/Joi), using `express-mongo-sanitize` to strip `$`/`.` keys, and never trusting that input is a primitive.

`↳ Follow-up: Why does this happen specifically with MongoDB?` — Because queries are **structured objects**, not strings — so injecting an operator object is enough; there's no SQL string to escape. The defense is **type validation** (reject non-string where a string is expected), which is why schema validation of input (Zod) matters as much as it does for SQL injection.

`↳ Follow-up: Does using Mongoose fully protect you?` — It helps — Mongoose **casts** values to the schema types, so a `String` field rejects an object. But query construction with raw user input (especially in `$where`, raw aggregation, or non-schema fields) can still be vulnerable. Validate input regardless.

**85. What is `$where` and why is it dangerous?**
`$where` lets you run **arbitrary JavaScript** for matching. It's slow (no index, runs JS per document) and a **security risk** (code injection if user input reaches it). Avoid it — use standard operators or `$expr` instead. Many deployments disable server-side JS entirely.

**86. How does authentication and authorization work in MongoDB?**
- **Authentication:** verifies identity — SCRAM (username/password, default), x.509 certificates, LDAP/Kerberos (enterprise), or Atlas's mechanisms.
- **Authorization (RBAC):** role-based access control — users are granted **roles** (`read`, `readWrite`, `dbAdmin`, `clusterAdmin`, or custom roles) scoped to databases. Follow **least privilege** — app users get only the access they need.

`↳ Follow-up: What's the most common MongoDB security failure?` — Historically, **exposing an unauthenticated MongoDB to the internet** (default bind/no auth in old versions) led to mass ransomware/data theft. Always: enable auth, bind to private networks, use a firewall/VPC, and never expose the DB port publicly.

**87. What encryption options exist?**
- **In transit:** **TLS/SSL** for client–server and intra-cluster traffic.
- **At rest:** WiredTiger encrypted storage engine (Enterprise/Atlas).
- **Client-Side Field Level Encryption (CSFLE) / Queryable Encryption:** encrypt specific sensitive fields **before** they reach the server, so even the DB admin/server can't read them — for PII/compliance.

`↳ Follow-up: What's special about Queryable Encryption?` — It lets you **query** (equality, and increasingly range) on **encrypted** fields without the server ever decrypting them — combining confidentiality with usability. A strong answer for "how do you protect highly sensitive fields like SSNs."

**88. What are MongoDB security best practices (checklist)?**
Enable authentication + RBAC (least privilege), use TLS everywhere, bind to private networks / use VPC + firewall, disable server-side JS (`$where`) if unused, validate/sanitize all input (NoSQL injection), keep MongoDB patched, audit access, encrypt sensitive fields, use strong unique credentials per app, and disable/secure default ports. In Atlas, use IP allowlists and database users with scoped roles.

---

<a name="p13"></a>
## PHASE 13 — OPERATIONS & PRODUCTION

**89. What are change streams?**
Change streams let applications **subscribe to real-time data changes** (inserts/updates/deletes) on a collection/database/cluster, built on the oplog. Use them for reactive features: cache invalidation, notifications, syncing to a search index/another system, event-driven workflows — without polling.
```ts
const stream = orders.watch([{ $match: { operationType: 'insert' } }]);
for await (const change of stream) { /* react to the new order */ }
```

`↳ Follow-up: Change streams vs polling?` — Polling repeatedly queries for changes (wasteful, laggy). Change streams **push** changes as they happen (efficient, near real-time), are **resumable** (via a resume token after disconnection), and respect majority read concern. Far superior for reacting to data changes.

**90. What is a TTL index?**
A **Time-To-Live** index automatically **deletes documents** after a specified time — MongoDB runs a background task expiring them. Perfect for sessions, logs, OTPs, caches, and any ephemeral data.
```ts
await sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 }); // expire after 1h
```

`↳ Follow-up: How precise is TTL expiration?` — Not exact — the background remover runs roughly **every 60 seconds**, so documents may persist up to a minute past expiry. Fine for cleanup, but don't rely on TTL for precise, security-critical timing (e.g., exact token invalidation) — check expiry in app logic too.

**91. How do you back up MongoDB?**
- **`mongodump`/`mongorestore`** — logical backups (BSON export); fine for smaller datasets.
- **Filesystem/volume snapshots** — point-in-time, faster for large data (must be consistent).
- **Atlas/Ops Manager continuous backups** — point-in-time recovery via oplog. 
Always test restores — an untested backup isn't a backup.

`↳ Follow-up: Why are mongodump backups problematic at scale?` — They read all data (impacting performance), aren't truly point-in-time across a busy cluster without care, and restores are slow for huge datasets. Large production systems use **snapshot-based** or managed continuous backups with oplog for point-in-time recovery.

**92. What is MongoDB Atlas?**
Atlas is MongoDB's fully-managed **cloud database service** (AWS/GCP/Azure). It handles provisioning, patching, backups, scaling, monitoring, security defaults, and adds features like Atlas Search (Lucene-based full-text), Performance Advisor (index suggestions), Charts, and serverless instances. It removes most operational burden — the common way to run MongoDB in production today.

**93. How do you monitor MongoDB in production?**
Track: **operation latency** and throughput, **slow queries** (profiler), **replication lag**, **connection count** vs pool limits, **WiredTiger cache** usage / page faults, **lock/queue** metrics, disk usage and IOPS, and index usage/hit rates. Tools: Atlas monitoring, `mongostat`/`mongotop`, Prometheus exporters, `db.serverStatus()`. Alert on replication lag, low cache hit rate, and rising query latency.

`↳ Follow-up: What metric most often signals a scaling problem?` — **Working set exceeding RAM** (rising page faults / falling cache hit ratio) and growing **replication lag** under write load. Both indicate you're outgrowing the current setup — time to add RAM, optimize queries/indexes, or shard.

**94. How do you do a zero-downtime schema migration in MongoDB?**
Because the schema is flexible, you migrate **incrementally**: deploy code that handles **both** old and new shapes, write new documents in the new shape, **backfill** old documents (in batches, or lazily on read — "migrate on read"), then once all are migrated, remove the old-shape handling. A `schemaVersion` field per document tracks progress. No big-bang `ALTER TABLE` needed.

`↳ Follow-up: What is "migrate on read" / lazy migration?` — Instead of a mass backfill, you transform each old document to the new shape **when it's next read** (and save it back). Spreads migration load over time, avoids a heavy batch job, but means old-shape handling code lives until all documents are touched.

---

<a name="p14"></a>
## PHASE 14 — TRICKY / GOTCHA & RAPID-FIRE

**95. Why might a unique index let two documents through with no value?**
A missing field is indexed as `null`. So a plain unique index permits only **one** document missing the field (all others collide on `null`). To allow many documents without the field while keeping uniqueness for those that have it, use a **partial** unique index (`partialFilterExpression: { field: { $exists: true } }`).

**96. What's the bug here (NoSQL injection)?**
```ts
app.post('/login', async (req, res) => {
  const user = await User.findOne({ username: req.body.username, password: req.body.password });
});
```
If body values are objects, an attacker sends `{ "username": {"$gt":""}, "password": {"$gt":""} }` to match a user without knowing credentials. Fix: validate inputs are **strings** (Zod), sanitize `$`/`.`, and never store/compare plaintext passwords (hash them).

**97. Why does this update accidentally wipe the document?**
```ts
await users.updateOne({ _id }, { name: 'New Name' }); // missing $set
```
Without `$set`, this is treated as a **replacement** intent — modern drivers throw, older ones replace the whole document, deleting all other fields. Always use `{ $set: { name: 'New Name' } }`.

**98. Why is `{ tags: ['a','b'] }` different from `{ tags: { $all: ['a','b'] } }`?**
`{ tags: ['a','b'] }` matches documents whose `tags` is **exactly** `['a','b']` (same elements, same order). `{ $all: ['a','b'] }` matches documents whose `tags` **contains both** `a` and `b` (in any order, among others). Confusing these is a common query bug.

**99. Why does this `$elemMatch`-less query over-match?**
```ts
scores.find({ grades: { $gte: 80, $lt: 90 } });
```
It matches if **different** array elements satisfy each condition (one element ≥80, a *different* one <90). To require a **single** element in `[80,90)`, use `{ grades: { $elemMatch: { $gte: 80, $lt: 90 } } }`.

**100. Why did my `pre('save')` password hook not run on an update?**
`updateOne`/`findOneAndUpdate` bypass the document lifecycle and don't trigger `save` hooks. Add a `pre('findOneAndUpdate')` hook (and handle `this.getUpdate()`), or always load-then-`save()` when you need document middleware. Silent cause of unhashed passwords on profile updates.

**101. Why is `0.1 + 0.2 !== 0.3`, and what does it mean for MongoDB money?**
Floating-point (BSON `Double`) can't represent those exactly. Store currency as **`Decimal128`** or integer minor units (paise/cents) — never `Double`. A classic correctness bug in financial data.

**102. Why is my sort failing with a memory error?**
An unindexed sort buffers results in memory and is capped at **32MB** (queries) — it fails past that. Add an index that supports the sort so MongoDB returns documents already ordered (no in-memory sort). In aggregation, `allowDiskUse: true` is the escape hatch.

**103. Rapid-fire one-liners:**
- **Single-document operations** are always atomic — design around it.
- **`_id`** is auto-indexed; **ObjectId** encodes a creation timestamp.
- **16MB** is the max document size — don't embed unbounded arrays.
- **ESR**: index order = Equality, Sort, Range.
- **Covered query** = answered entirely from the index (fast).
- **`explain()`**: watch `COLLSCAN` and docsExamined vs nReturned.
- **`$match` early** in aggregation to use indexes.
- **`$elemMatch`** to match one array element against multiple conditions.
- **Embed** data read together; **reference** large/independent/shared data.
- **Replica set** = HA + failover; **sharding** = horizontal scale.
- **`w: 'majority'`** survives failover; **`w: 1`** can be rolled back.
- **Shard key**: high cardinality, low frequency, non-monotonic, query-aligned.
- **`$inc`/`$push`** are atomic — avoid read-modify-write races.
- **`.lean()`** for fast read-only queries.
- **Validate input types** to prevent NoSQL injection.
- **TTL index** auto-expires; the remover runs ~every 60s.

---

## FINAL PREP TIPS (FROM THE OTHER SIDE OF THE TABLE)

1. **Schema design is the #1 MongoDB interview topic.** Be fluent in embedding vs referencing, the 16MB limit, single-document atomicity, and "model around access patterns." Most senior questions trace back here.
2. **Indexing is #2.** Know the ESR rule, compound-index prefixes, covered queries, and how to read `explain()`. If you can diagnose a slow query out loud, you'll stand out.
3. **Aggregation pipeline fluency signals depth.** Know the core stages, why `$match` goes first, `$lookup`/`$unwind` caveats, and the memory limit.
4. **Distinguish replication from sharding crisply.** One is HA/redundancy, the other is horizontal scaling — and explain shard-key choice (the hardest decision).
5. **Know the consistency knobs:** read preference, write concern (`majority` vs `1`), and what a rollback is. These separate people who've *operated* MongoDB from those who've only queried it.
6. **For MERN/Node roles, expect Mongoose + TypeScript.** Typed schemas, `populate` vs `$lookup`, the `pre('save')`-doesn't-run-on-update gotcha, `.lean()`, and NoSQL injection are high-probability questions.
7. **Have crisp security answers:** NoSQL injection (and why input validation defeats it), auth/RBAC, and "never expose an unauthenticated DB."
8. **Code the classics by hand:** a typed Mongoose schema with a password-hash hook, an aggregation top-N-by-group, an ESR compound index for a given query, an atomic `$inc` transfer, and a keyset-pagination query. These come up constantly.

> **Good luck - you've got this.** 
