## 6.25
The number of cache sets, S, is given by `S = C / (B * E)`. The number of tag bits, t, is given by `t = m - (s + b)`. The number of set index bits, s, is given by `s = log2(S)`. The number of block offset bits, b, is given by `b = log2(B)`

## 6.29

Suppose we have a system with the following properties:

-   The memory is byte addressable.
    
-   Memory accesses are to 1-byte words (not to 4-byte words).
    
-   Addresses are 12 bits wide.
    
-   The cache is two-way set associative (E = 2), with a 4-byte block size (B = 4) and four sets (S = 4).

A) 
| 11 | 10 | 9  | 8  | 7  | 6  | 5  | 4  | 3  | 2  | 1  | 0  |
|:-: |:-: |:-: |:-: |:-: |:-: |:-: |:-: |:-: |:-: |:-: |:-: |
| CT | CT | CT | CT | CT | CT | CT | CT | CI | CI | CO | CO |

### Why is 0 and 1 CO
The cache block offset is the least significant 2 bits of the address and can take on values from<mark style="background: #FFB8EBA6;"> 0 to 3, indicating which of the 4 bytes</mark> within a block is being accessed. In this case, it is limited to only 1 and 0 because the block size is 4 bytes and each byte can be accessed individually. The 2 bits of the cache block offset can represent 4 possible values, including 0, 1, 2, and 3. Since the values represent the byte-level granularity of the memory access, using only 1 and 0 is sufficient to access each of the 4 bytes within the block.

<mark style="background: #FFB8EBA6;">pink</mark>
The values from 0 to 3 for the cache block offset are used because the block size of the cache is 4 bytes. Each byte in the block can be individually accessed, so the cache block offset is used to determine which byte within the block is being accessed. <span style="background:#d3f8b6">The 2 bits of the cache block offset can represent 4 possible values, which are 0, 1, 2, and 3</span>. These values correspond to the byte position within the block, with 0 representing the first byte, 1 representing the second byte, 2 representing the third byte, and 3 representing the fourth byte. This allows the cache to determine the specific byte within the block that is being accessed and to retrieve or store the data at the correct byte-level granularity.

<span style="background:#d3f8b6">green:</span>
An example of how the 2 bits of the cache block offset can represent the 4 possible values of 0, 1, 2, and 3 is as follows:

Suppose we have a memory address `A` that is 12 bits wide and we want to determine the cache block offset for this address. The least significant 2 bits of the address, `A[1:0]`, represent the cache block offset. If `A[1:0]=00`, this represents the value 0, indicating that the first byte within the cache block is being accessed. If `A[1:0]=01`, this represents the value 1, indicating that the second byte within the cache block is being accessed. If `A[1:0]=10`, this represents the value 2, indicating that the third byte within the cache block is being accessed. If `A[1:0]=11`, this represents the value 3, indicating that the fourth byte within the cache block is being accessed.

So, the 2 bits of the cache block offset can represent the 4 possible values of 0, 1, 2, and 3, which correspond to the specific byte within the cache block that is being accessed.

### Why is 4,3,2 CI?
The cache set index determines which set in the cache is being accessed. The cache is two-way set associative, with 4 sets, so 2 bits are required to determine the set index. The next 2 bits of the address, bits 3 and 2, are used as the cache set index. The set index can take on values from 0 to 3, indicating which of the 4 sets in the cache the memory block should be stored in or searched for. In this case, 4, 3, 2, and 1 are not used as the cache set index because the cache set index is limited to 2 bits, and only values from 0 to 3 can be represented.

### Why is the rest CT?
The cache tag is used to compare with the tag stored in the cache to determine whether there is a cache hit or miss and to identify the unique memory block being accessed. The cache tag is the most significant 8 bits of the address, bits 11 to 4. The cache tag is stored in the cache along with the cache block data, and it is used to determine if the memory block being accessed is already stored in the cache. Since the cache tag is the most significant 8 bits of the address, the rest of the address bits, 11 to 4, are used as the cache tag.

### What is the cache tag used for?

The cache tag is used to compare with the tag stored in the cache to determine whether there is a cache hit or miss and to identify the unique memory block being accessed.

### What are the most significant bits of the address used for in the cache tag?

The most significant 8 bits of the address, bits 11 to 4, are used for the cache tag.

### How is the cache tag stored in the cache?

The cache tag is stored in the cache along with the cache block data.

### What happens if the cache tag of the memory access matches the tag stored in the cache?

If the cache tag of the memory access matches the tag stored in the cache, a cache hit is declared and the data can be returned from the cache.

### What happens if the cache tag of the memory access does not match the tag stored in the cache?

If the cache tag of the memory access does not match the tag stored in the cache, a cache miss is declared, and the data must be fetched from memory.

### What is the cache set index used for?

The cache set index determines which set in the cache is being accessed.

### How many bits are required to determine the set index in the two-way set associative cache with 4 sets?

2 bits are required to determine the set index in the two-way set associative cache with 4 sets.

### What is the use of the cache block offset?

The cache block offset determines which byte within the cache block is being accessed and determines the byte-level granularity of the memory access.

```c
                                     Cache
   +--------------------------------------------------+
   |                    Set 0                         |
   | +------------------+   +------------------+      |
   | | Block 0          |   | Block 1          |      |
   | | +--------------+ |   | +--------------+ |      |
   | | | Tag (8 bits) | |   | | Tag (8 bits) | |      |
   | | +--------------+ |   | +--------------+ |      |
   | | | Data (4 bytes)| |   | | Data (4 bytes)| |    |
   | | +--------------+ |   | +--------------+ |      |
   | +------------------+   +------------------+      |
   |                    Set 1                         |
   | +------------------+   +------------------+      |
   | | Block 0          |   | Block 1          |      |
   | | +--------------+ |   | +--------------+ |      |
   | | | Tag (8 bits) | |   | | Tag (8 bits) | |      |
   | | +--------------+ |   | +--------------+ |      |
   | | | Data (4 bytes)| |   | | Data (4 bytes)| |    |
   | | +--------------+ |   | +--------------+ |      |
   | +------------------+   +------------------+      |
   |                    Set 2                         |
   | +------------------+   +------------------+      |
   | | Block 0          |   | Block 1          |      |
   | | +--------------+ |   | +--------------+ |      |
   | | | Tag (8 bits) | |   | | Tag (8 bits) | |      |
   | | +--------------+ |   | +--------------+ |      |
   | | | Data (4 bytes)| |   | | Data (4 bytes)| |    |
   | | +--------------+ |   | +--------------+ |      |
   | +------------------+   +------------------+      |
   |                    Set N                         |
   |---------------------------------------------------

```

## 6.30

A) 
The size of the cache, `C`, is determined by the number of blocks it can store, which is determined by the number of sets `S`, the associativity `E`, and the block size `B`.

In this case, the cache is 4-way set associative with `E = 4`, which means there are 4 blocks in each set. The cache has `S = 8` sets, so there are a total of `S * E = 8 * 4 = 32` blocks in the cache.

The block size is `B = 4B`, so the total size of the cache is `C = E * B * S = 4 * 4 * 8 = 128B`.

So, the size of the cache in this system is 128 bytes.

B)
```c
  12   11   10    9    8    7    6    5    4    3    2    1   0
+----+----+----+----+----+----+----+----+----+----+----+----+----+
| CT | CT | CT | CT | CT | CT | CT | CT | CI | CI | CI | CO | CO |
+----+----+----+----+----+----+----+----+----+----+----+----+----+
```

To determine the fields in the address format that correspond to the cache tag (CT), cache set index (CI), and cache block offset (CO), you need to consider the properties of the cache described in the problem statement.

In this case, we know that the memory is byte addressable, with addresses that are 13 bits wide. We also know that the cache is 4-way set associative, with a 4-byte block size, and 8 sets.

From this information, we can use the following formula to determine the number of bits required for each field:

CI = log2(S) = log2(8) = 3 bits

CO = log2(B) = log2(4) = 2 bits

CT = m - (CI + CO) = 13 - (3 + 2) = 8 bits

Therefore, the address format would be divided as follows:

-   The most significant 8 bits, bits 12 to 5, are used for the cache tag (CT).
-   The next 3 bits, bits 4 to 2, are used for the cache set index (CI).
-   The least significant 2 bits, bits 1 and 0, are used for the cache block offset (CO).


## 6.31
