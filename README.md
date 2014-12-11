**Work in progress:** Nothing to see here yet. This is highly experimental.

# GKV

GKV is a Git inspired key value store.

## Comparison to similar projects

The main goal of GKV is to create a simple database suitable for offline usage
across many devices with good support for syncing and conflict resolution
supporting up to 1 million records.

### Git

Each git commit references a full tree object. This works fine for git, as most
repositories are laid out in a hierarchical manner, causing the top level tree
object to be usually small. A key value store however may have hundreds of
thousands of top level keys without any hierarchies. For this reason GKV
commits reference partial indexes, which include only the key value pairs that
have changed. This greatly reduces the costs for writing and syncing data, but
introduces a O(N) cost for naive key lookups. However, this can be turned into
a one time costs by keeping a cache of the current index, and references to
full indexes from commits will be supported in the future as well.

Additionally git includes many advanced optimizations such as pack files which
makes it non-trivial to create native clients.

### CouchDB

To be written ...

## Objects

GKV uses the following object types for storing data:

* blob: Stores raw values.
* index: Maps key strings to value hashes. Similar to trees in git.
* commit: References previous commits and indexes.

The basic object format is given in ABNF with the following recurring rules:

```
number = 1*DIGIT
binary = *%x00-ff
hash   = 1*(DIGIT / "a" / "b" / "c" / "d" / "e" / "f")
time   = @TODO RFC3339 ABNF
```

### Blob

ABNF:

```
blob   = "blob " size %x00 value
size   = number
value  = binary
```

Example:

```
"blob 11\0Hello world"
```

### Index

ABNF:

```
index   = "index " size %x00 1*(keysize %x00 key valref "\n")
number  = 1*DIGIT
size    = number
keysize = number
key     = binary
valref  = hash
```

Example:

```
"index 9\03\0foobar\n"
```

### Commit

ABNF:

```
commit    = "commit" size %x00
            "time " time "\n"
            *1("partial " indexref "\n")
            *2("parent " commitref "\n")
size      = number
time      = rfc3339
indexref  = hash
commitref = hash
```

Example:

```
"commit 97\0partial c82a9efd857f436e0ececd7986cb8611b6b8f84e\nparent 119be3a4d2e8eef6fbf1e86d817fe58a452cf429\n"
```
