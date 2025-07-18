---
title: "Bloom Filters: A probabilistic powerhouse"
date: 2025-07-07 00:34:00 +0530
last_modified_at: 2025-07-11 00:34:00 +0530
categories: [Article, DataStructures]
tags: [data-structures]
math: true
image:
  path: /assets/img/posts/bloom-filters.png
  lqip: data:image/webp;base64,UklGRroAAABXRUJQVlA4IK4AAAAwBQCdASoUABQAPm0wkkckIqGhKAqogA2JaQDA3YvWcLsQAf2XLP2nscqcS3IjWm6RsAD+/ZMDmSAXsZTNv4qnKniVVJ3OWIf9fzq...
  alt: Diagram showing how a Bloom Filter uses hash functions to set bits
description: Understand how Bloom Filters provide a space-efficient, probabilistic way to test set membership in large-scale systems.
---

## Understanding Bloom Filters: A Probabilistic Powerhouse

### Introduction

When dealing with large-scale data systems — especially where memory and speed matter — we often face a simple but important question:  
**“Have I seen this item before?”**

You might reach for a `HashSet` or database index — but what if memory is tight and occasional false positives are acceptable?  
That’s where **Bloom Filters** come in.

---

### What is a Bloom Filter?

A **Bloom Filter** is a *space-efficient probabilistic data structure* used to test **whether an element is a member of a set**. It can return:

- **Definitely not in the set**
- **Possibly in the set**

The trade-off? It **can return false positives**, but **never false negatives**.

---

### How Does It Work?

A Bloom filter is essentially:

- A **bit array** of size `m`, initialized to all 0s.
- `k` different **hash functions** that map input to positions in the bit array.

#### Insertion
To insert an element:
1. Hash the element with all `k` hash functions.
2. Set each corresponding bit to 1.

#### Lookup
To check membership:
1. Hash the element with the same `k` functions.
2. If *any* of the corresponding bits are 0 → **definitely not in the set**.
3. If *all* are 1 → **possibly in the set**.

Lets try to visualize this with the help of an interactive codepen. We can add a word, and test if a word exists with the help of the <kbd>Add</kbd>/<kbd>Test</kbd> buttons.

The filter will output one of the two messages. 
- `"word" is definitely not in the filter.` : When the filter is sure the word doesnot exists.
- `"word" might be in the filter.` : When the filter thinks the word might be present.

{% raw %}
<p class="codepen" data-height="512" data-default-tab="result" data-slug-hash="raOBqQx" data-pen-title="Bloom Filter" data-user="Sandeep-Mahanty" style="height: 512px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/Sandeep-Mahanty/pen/raOBqQx">
  Bloom Filter</a> by Sandeep Mahanty (<a href="https://codepen.io/Sandeep-Mahanty">@Sandeep-Mahanty</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://public.codepenassets.com/embed/index.js"></script>
{% endraw %}
---

### Choosing the Right Hash Function

A core component of a Bloom filter’s performance lies in the **quality and speed of its hash functions**. Each element added to the filter is passed through multiple hash functions, which determine which bits to set (or check) in the bit array. The choice of these hash functions significantly affects the false positive rate and performance and hence fast hash functions like [murmur](https://en.wikipedia.org/wiki/MurmurHash) are generally preferred over cryptographic hash functions.

#### Why Not Use SHA or MD5 ?

Hash functions like `SHA-256`, `SHA-1`, or `MD5` are **cryptographic hash functions**, designed for **security** rather than speed. They provide strong guarantees like:

- Resistance to collisions  
- Irreversibility  
- Uniform distribution  

While that’s excellent for **password hashing** or **digital signatures**, it’s overkill for Bloom filters. Cryptographic hashes are **computationally expensive**, which becomes a bottleneck when we need to hash large volumes of data rapidly.

#### Why MurmurHash (or Similar) is Preferred ?

**MurmurHash** is a **non-cryptographic** hash function designed for **speed and simplicity**, making it ideal for use in Bloom filters. Its advantages include:

- **Fast** — Much faster than cryptographic hashes, especially in tight loops  
- **Uniform output** — Good distribution of hash values with minimal collisions  
- **Deterministic and portable** — Same input → same output across platforms  
- **Low CPU overhead** — Critical when performing multiple hash operations per item  

Bloom filters don’t need security — they need **consistent and efficient hashing**, and MurmurHash strikes that balance perfectly.

#### Alternative Hashes

Other popular non-cryptographic hash functions used in Bloom filters include:

- [xxHash](https://github.com/Cyan4973/xxHash) — ultra-fast, used in systems like Apache Kafka  
- [FNV](https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo_hash_function) — simple and widely used  
- [CityHash](https://github.com/google/cityhash) — optimized for speed and cache-friendliness by Google. 


### Trade-offs: False Positives

Bloom filters are fast and memory-efficient, but they **can’t remove elements**, and **false positives** are possible.

The false positive rate depends on:

- Number of hash functions `k`
- Size of bit array `m`
- Number of inserted elements `n`

The probability of a false positive is:

$$
\begin{equation}
  P \approx \left(1 - e^{- \frac{kn}{m}} \right)^k
  \label{eq:bloom-false-positive}
\end{equation}
$$

### When Should You Use a Bloom Filter?

#### Use cases:
- Caching layers (check if it's worth looking up an item)
- Distributed systems (avoid expensive queries)
- Email spam detection
- Web crawler visited URLs

#### Ideal when:
- You don’t need to delete items.
- You can tolerate false positives.
- Memory usage matters more than perfect accuracy.

### Real world uses of Bloom Filter
**Cassandra** the distributed highly available and high write throughput NoSQL database employs bloom filters extensively in its read path. Cassandra keeps data flushed into SSTables (String Sorted Tables) on disk to quickly read data. Bloom filters help here by efficiently determining if the data being requested exists in a particular SSTable file. [Bloom Filters](https://cassandra.apache.org/doc/5.0/cassandra/managing/operating/bloom_filters.html)

Similarly Apache HBase and ScyllaDB are also example of some databases which use bloom filters.

**CDNs** like Akamai leverage bloom filters to quickly check if an object is present in the cache or not.

### Java Implementation

```java
  public class BloomFilter {
    private BitSet bitset;
    private int bitSize;
    private int hashCount;

    public BloomFilter(int bitSize, int hashCount) {
        this.bitSize = bitSize;
        this.hashCount = hashCount;
        this.bitset = new BitSet(bitSize);
    }

    public void add(String item) {
        for (int i = 0; i < hashCount; i++) {
            int hash = getHash(item, i);
            bitset.set(Math.abs(hash % bitSize));
        }
    }

    public boolean mightContain(String item) {
        for (int i = 0; i < hashCount; i++) {
            int hash = getHash(item, i);
            if (!bitset.get(Math.abs(hash % bitSize))) return false;
        }
        return true;
    }

    private int getHash(String item, int seed) {
        return item.hashCode() ^ (seed * 0x5bd1e995);
    }
}
```

### Final Thoughts

> Bloom Filters are a great tool in the arsenal of backend engineers, especially when you're operating at scale and every byte of memory counts. This post was an attempt at presenting the content in a simplified manner.

### Further Reading
- Excellent report on bloom filter by James Blustein, 
Amal El-Maazawi -  [Bloom Filters - A Tutorial, Analysis, and Survey](https://cdn.dal.ca/content/dam/dalhousie/pdf/faculty/computerscience/technical-reports/CS-2002-10.pdf)
- [Bloom filter - Wikipedia ](https://en.wikipedia.org/wiki/Bloom_filter?utm_source=chatgpt.com)
- Excellent explanation post By Bill Mill [@llimllib](https://github.com/llimllib) - [Bloom Filters by Example](https://llimllib.github.io/bloomfilter-tutorial/)
