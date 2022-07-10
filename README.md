# better-sqlite3-int

The fastest and simplest library for SQLite3 in Node.js, [forked from upstream](https://github.com/WiseLibs/better-sqlite3) for better int support.

- Full transaction support
- High performance, efficiency, and safety
- Inserts integers as integers (not doubles) for up to 9x smaller column storage.
- Easy-to-use synchronous API *(better concurrency than an asynchronous API... yes, you read that correctly)*
- Support for user-defined functions, aggregates, virtual tables, and extensions
- 64-bit integers *(invisible until you need them)*
- Worker thread support *(for large/slow queries)*

## How other libraries compare

|   |select 1 row &nbsp;`get()`&nbsp;|select 100 rows &nbsp;&nbsp;`all()`&nbsp;&nbsp;|select 100 rows `iterate()` 1-by-1|insert 1 row `run()`|insert 100 rows in a transaction|
|---|---|---|---|---|---|
|better-sqlite3-int|1x|1x|1x|1x|1x|
|[sqlite](https://www.npmjs.com/package/sqlite) and [sqlite3](https://www.npmjs.com/package/sqlite3)|11.7x slower|2.9x slower|24.4x slower|2.8x slower|15.6x slower|

> You can verify these results by [running the benchmark yourself](./docs/benchmark.md).

## Installation

```bash
npm install better-sqlite3-int
```

> You must be using Node.js v10.20.1 or above. Prebuilt binaries are available for [LTS versions](https://nodejs.org/en/about/releases/). If you have trouble installing, check the [troubleshooting guide](./docs/troubleshooting.md).

## Usage

```js
const db = require('better-sqlite3-int')('foobar.db', options);

const row = db.prepare('SELECT * FROM users WHERE id = ?').get(userId);
console.log(row.firstName, row.lastName, row.email);
```

##### In ES6 module notation:

```js
import Database from 'better-sqlite3-int';
const db = new Database('foobar.db', options);
```

## Why should I use this instead of [better-sqlite3](https://github.com/WiseLibs/better-sqlite3)?

- `better-sqlite3` requires you to wrap numbers in Integer() classes to use them in statements as integers.
- `better-sqlite3` requires you to coerce booleans to to numbers to use them in statements.
- `better-sqlite3` stores all numbers as doubles by default, using 9 bytes on disk, instead of the [1-9 bytes that integers require](https://www.sqlite.org/fileformat.html#record_format).

## Why should I use this instead of [node-sqlite3](https://github.com/mapbox/node-sqlite3)?

- `node-sqlite3` uses asynchronous APIs for tasks that are either CPU-bound or serialized. That's not only bad design, but it wastes tons of resources. It also causes [mutex thrashing](https://en.wikipedia.org/wiki/Resource_contention) which has devastating effects on performance.
- `node-sqlite3` exposes low-level (C language) memory management functions. `better-sqlite3-int` does it the JavaScript way, allowing the garbage collector to worry about memory management.
- `better-sqlite3-int` is simpler to use, and it provides nice utilities for some operations that are very difficult or impossible in `node-sqlite3`.
- `better-sqlite3-int` is much faster than `node-sqlite3` in most cases, and just as fast in all other cases.

#### When is this library not appropriate?

In most cases, if you're attempting something that cannot be reasonably accomplished with `better-sqlite3-int`, it probably cannot be reasonably accomplished with SQLite3 in general. For example, if you're executing queries that take one second to complete, and you expect to have many concurrent users executing those queries, no amount of asynchronicity will save you from SQLite3's serialized nature. Fortunately, SQLite3 is very *very* fast. With proper indexing, we've been able to achieve upward of 2000 queries per second with 5-way-joins in a 60 GB database, where each query was handling 5–50 kilobytes of real data.

If you have a performance problem, the most likely causes are inefficient queries, improper indexing, or a lack of [WAL mode](./docs/performance.md)—not `better-sqlite3-int` itself. However, there are some cases where `better-sqlite3-int` could be inappropriate:

- If you expect a high volume of concurrent reads each returning many megabytes of data (i.e., videos)
- If you expect a high volume of concurrent writes (i.e., a social media site)
- If your database's size is near the terabyte range

For these situations, you should probably use a full-fledged RDBMS such as [PostgreSQL](https://www.postgresql.org/).

# Documentation

- [API documentation](./docs/api.md)
- [Performance](./docs/performance.md) (also see [benchmark results](./docs/benchmark.md))
- [64-bit integer support](./docs/integer.md)
- [Worker thread support](./docs/threads.md)
- [Unsafe mode (advanced)](./docs/unsafe.md)
- [SQLite3 compilation (advanced)](./docs/compilation.md)
- [Contribution rules](./docs/contribution.md)
- [Code of conduct](./docs/conduct.md)

# License

[MIT](./LICENSE)
