# Globally Unique ID Generator

[![godoc](http://img.shields.io/badge/godoc-reference-blue.svg?style=flat)](https://godoc.org/github.com/robxyy/xid) [![license](http://img.shields.io/badge/license-MIT-red.svg?style=flat)](https://raw.githubusercontent.com/robxyy/xid/master/LICENSE) [![Build Status](https://travis-ci.org/robxyy/xid.svg?branch=master)](https://travis-ci.org/robxyy/xid) [![Coverage](http://gocover.io/_badge/github.com/robxyy/xid)](http://gocover.io/github.com/robxyy/xid)

> ⚠️ _This xid is not compatible with Mongo Object ID_

Package xid is a globally unique id generator library, ready to safely be used directly in your server code.

~~Xid uses the Mongo Object ID algorithm to generate globally unique ids with a different serialization (base64) to make it shorter when transported as a string:
https://docs.mongodb.org/manual/reference/object-id/~~

- 5-byte value representing the seconds since the Unix epoch,
- 3-byte machine identifier,
- 2-byte process id, and
- 3-byte counter, starting with a random value.

The string representation is using base32 hex (w/o padding) for better space efficiency
when stored in that form (21 bytes). The hex variant of base32 is used to retain the
sortable property of the id.

Xid doesn't use base64 because case sensitivity and the 2 non alphanum chars may be an
issue when transported as a string between various systems. Base36 wasn't retained either
because 1/ it's not standard 2/ the resulting size is not predictable (not bit aligned)
and 3/ it would not remain sortable. To validate a base32 `xid`, expect a 21 chars long,
all lowercase sequence of `a` to `v` letters and `0` to `9` numbers (`[0-9a-v]{21}`).

UUIDs are 16 bytes (128 bits) and 36 chars as string representation. Twitter Snowflake
ids are 8 bytes (64 bits) but require machine/data-center configuration and/or central
generator servers. xid stands in between with 13 bytes (104 bits) and a more compact
URL-safe string representation (21 chars). No configuration or central generator server
is required so it can be used directly in server's code.

| Name        | Binary Size | String Size    | Features
|-------------|-------------|----------------|----------------
| [UUID]      | 16 bytes    | 36 chars       | configuration free, not sortable
| [shortuuid] | 16 bytes    | 22 chars       | configuration free, not sortable
| [Snowflake] | 8 bytes     | up to 20 chars | needs machine/DC configuration, needs central server, sortable
| [MongoID]   | 12 bytes    | 24 chars       | configuration free, sortable
| xid         | 13 bytes    | 21 chars       | configuration free, sortable

[UUID]: https://en.wikipedia.org/wiki/Universally_unique_identifier
[shortuuid]: https://github.com/stochastic-technologies/shortuuid
[Snowflake]: https://blog.twitter.com/2010/announcing-snowflake
[MongoID]: https://docs.mongodb.org/manual/reference/object-id/

Features:

- Size: 13 bytes (104 bits), smaller than UUID, larger than snowflake
- Base32 hex encoded by default (21 chars when transported as printable string, still sortable)
- Non configured, you don't need set a unique machine and/or data center id
- K-ordered
- Embedded time with 1 second precision
- Unicity guaranteed for 16,777,216 (24 bits) unique ids per second and per host/process
- Lock-free (i.e.: unlike UUIDv1 and v2)

~~Best used with [zerolog](https://github.com/rs/zerolog)'s
[RequestIDHandler](https://godoc.org/github.com/rs/zerolog/hlog#RequestIDHandler).~~

Notes:

- Xid is dependent on the system time, a monotonic counter and so is not cryptographically secure. If unpredictability of IDs is important, you should not use Xids. It is worth noting that most other UUID-like implementations are also not cryptographically secure. You should use libraries that rely on cryptographically secure sources (like /dev/urandom on unix, crypto/rand in golang), if you want a truly random ID generator.

References:

- http://www.slideshare.net/davegardnerisme/unique-id-generation-in-distributed-systems
- https://en.wikipedia.org/wiki/Universally_unique_identifier
- https://blog.twitter.com/2010/announcing-snowflake
- ~~Python port by [Graham Abbott](https://github.com/graham): https://github.com/graham/python_xid~~
- ~~Scala port by [Egor Kolotaev](https://github.com/kolotaev): https://github.com/kolotaev/ride~~
- ~~Rust port by [Jérôme Renard](https://github.com/jeromer/): https://github.com/jeromer/libxid~~
- ~~Ruby port by [Valar](https://github.com/valarpirai/): https://github.com/valarpirai/ruby_xid~~
- ~~Java port by [0xShamil](https://github.com/0xShamil/): https://github.com/0xShamil/java-xid~~
- ~~Dart port by [Peter Bwire](https://github.com/pitabwire): https://pub.dev/packages/xid~~

## Install

    go get github.com/robxyy/xid

## Usage

```go
guid := xid.New()

println(guid.String())
// Output: 016ohoarc3q8dp1884msi
```

Get `xid` embedded info:

```go
guid.Machine()
guid.Pid()
guid.Time()
guid.Counter()
```

## Benchmark

Benchmark against Go [Maxim Bublis](https://github.com/satori)'s [UUID](https://github.com/satori/go.uuid).

```
BenchmarkXID        	20000000	        91.1 ns/op	      32 B/op	       1 allocs/op
BenchmarkXID-2      	20000000	        55.9 ns/op	      32 B/op	       1 allocs/op
BenchmarkXID-4      	50000000	        32.3 ns/op	      32 B/op	       1 allocs/op
BenchmarkUUIDv1     	10000000	       204 ns/op	      48 B/op	       1 allocs/op
BenchmarkUUIDv1-2   	10000000	       160 ns/op	      48 B/op	       1 allocs/op
BenchmarkUUIDv1-4   	10000000	       195 ns/op	      48 B/op	       1 allocs/op
BenchmarkUUIDv4     	 1000000	      1503 ns/op	      64 B/op	       2 allocs/op
BenchmarkUUIDv4-2   	 1000000	      1427 ns/op	      64 B/op	       2 allocs/op
BenchmarkUUIDv4-4   	 1000000	      1452 ns/op	      64 B/op	       2 allocs/op
```

Note: UUIDv1 requires a global lock, hence the performance degradation as we add more CPUs.

## Licenses

All source code is licensed under the [MIT License](https://raw.github.com/robxyy/xid/master/LICENSE).
