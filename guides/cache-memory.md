---
layout: default
title: Cache Memory
nav_order: 7
parent: Guides
permalink: /guides/cache-memory
---

# Cache Memory

## Overview
In Computer Science, a cache is construct that is used to speed up access to data. Often data is stored somewhere, say in memory, on the disk, or in a database. To keep the costs of large storage reasonable, access time to the data is slower than what it could be if a more expensive technology was used for the data storage. A cache is designed to be much smaller than the primary data storage, but uses more expensive technology to make access fast. A cache will keep recently accessed data so that future requests for the data are served quickly. Caches can be very effective if the access pattern to data favors recently requested data items and also data items that are physically near one and other. While we are going to focus on cache memory found in computer processors, caches are widely used in other areas of computer systems, including operating system kernels, databases, and cloud computing. This guide will explain cache memory concepts and look at three types of cache memory structures: direct mapped, fully associative, and set associative.

## Cache Memory Concepts
Processors need to access data that resides in memory. This memory is sometimes called main memory or RAM. When we execute programs, the memory consists of instructions, global data, heap data, and stack data. A RISC-V processor uses the PC (program counter) register to point to the next instruction in memory to execute. Executing programs use load and store instructions to access data in memory, such as ld, sd, lw, and sw. The purpose of cache memory is to speed up access to main memory by holding recently used data in the cache. A cache can hold either data (called a D-Cache), instructions, (called an I-Cache), or both (called a Unified Cache).

A cache memory will take an address as input and decide if the data associated with the address is in the cache. If the data is in the cache, we call this a **hit**, and the data can be returned immediately without going to main memory. If the data associated with the address is not in the cache, we call this a **miss**. Each time the processor requests data from the cache, we call this a memory reference, or ref. We can keep track of each of these metrics during the use of a cache. If we do this, we can then compute two important values: the **hit rate** and the **miss rate**. We compute these values as follows:

```
hit_rate = num_of_hits / num_of_refs
miss_rate = num_of_misses / num_of_refs
```
and
```
hit_rate = 1 - (miss_rate)
miss_rate =  1 - (hit_rate)
```

Our goal is to build caches that achieve high hit rates. In fact, cache memories are wildly successful and can achieve extremely high hit rates in practice, e.g., greater than 97% for many types of programs. The reason for this is that executing code adheres to principles of locality: the **principle of temporal locality** and the **principle of spatial locality**. The principle of temporal locality states that if a data element is accessed, there is a good chance it will be accessed again in the near future. Think about an array index (i). If we have a loop with many iterations, the array index will need to be accessed on each iteration. Similarly, if we call a function in the loop, the code for that function will be accessed each time through the loop. The same can be said for recursive functions. The principle of spatial locality states that if a data element is accessed, there is a good chance a close neighbor of the data element will be access in the near future. Traversing an array in a loop is a good example. If we access `array[i]` there is a good chance will access `array[i+1]`. Code also exhibits a high degree of spatial locality. If we execute an instruction at the current PC, there is a good chance we will execute instruction at `PC + 4` (the next instruction) unless the instruction at PC is a branch or jump. Spatial locality suggests that when we access a data element we should also load some of that data elements neighbors into the cache at the same time reduce the latency of going to main memory for smaller data element access.

## Cache Mechanisms
In order for a cache memory to operate properly it needs to support some basic mechanisms. At a high level, a cache memory will receive a memory address request (memory reference) from the processor. It needs to determine if the data associated with the memory address is in the cache (a hit). If so, the cache needs to respond with the data requested. If the data is not in the cache (a miss) the cache needs to retrieve the data from main memory, populate the cache with this data, and also return the data back to the processor. When the cache retrieves data from memory it needs to find a place in the cache for this data. In many cases, since the cache is much smaller than main memory, the cache will need to remove some existing data in order to bring the new data into the cache. We call the process of choosing which data to remove the **cache replacement policy**. When we select data to be removed or overwritten in the cache, we call this **eviction**. 

To support spatial locality, we need to define the size of data that is moved between main memory and the cache when we populate the cache with data. We call this data size the **block size**. The block size will be typically be defined in terms of number of bytes or words.

When the processor updates data in the cache, i.e., with sd and sw, we have to decide when the update will be reflected in main memory. We can chose to update main memory at the same time that the update is made in the cache. This technique is called **write-though**. We can also delay the update to main memory until we need to evict the updated data on a cache miss. This technique is called **write-back** and is the most common approach because it avoids having to write to memory more than is necessary. However, the write-back approach does introduce challenges for multi-processors and multi-core processors in which each processor has its own cache, but wants to see the most up to data value in memory. To solve this problem, caches need to be coordinated and employ a **cache coherency protocol** to keep caches in sync with memory updates. 

Note that instruction memory is generally read only. That is, once code is loaded into memory, it is not modified for the duration of the executing program. In this case we can assume there will be no updates to the code in memory, and thus no need for write-through or write-back updates.

## Addresses and Memory
A memory request from the processor to the cache is just an address. In modern computers an address is usually a byte address. This means that each unique address points to a different byte (8-bits) in memory. You can think of memory as an array of bytes:

```
uint8_t mem[N];
```

Where N is the size of memory and the index into the array is an address. However, typically, a cache does not hold individual bytes, but instead, groups of bytes. Recall from the discussion above, to take advantage of spatial locality, a cache will define a block size so that when we move data from memory to the cache we do it a block at a time, where a block will be multiple bytes or words.

We will start will considering caches whose block size is one word (32 bits, which is 4 bytes). In this way, we will assume that the processor will only generate requests for word values. We will also assume that when the processor requests a word from memory that the address for the word will be **word-aligned**. That is, the address will be a multiple of 4. This is generally the case for most word data like instruction words (32 bits) and int values (32 bits). With these assumptions we can view memory as a sequence of words, where each word occupies four bytes.

For example, word 0 occupies mem[0], mem[1], mem[2], mem[3]. Then word 1 occupies mem[4], mem[5], mem[6], mem[7]. And so on. If we have a byte address, we can get the word address like this:
```
addr_word = addr / 4;
```

To get a byte address from the word address we can do:
```
addr = addr_word * 4;
```

## Direct-Mapped Caches

A direct-mapped cache is considered the simplest form of a cache memory because every memory address maps to a specific physical location in the cache. This makes the replacement policy very simple. If the data associated with an address is not in the cache, then you evict the data in the position that the address maps to. 

The most important aspect of a direct mapped cache is the mapping function. Let's consider a direct mapped cache that can hold 4 words of data:
```
uint32_t cache[4];
```

In this way, we are saying that our cache holds 4 words of data. We refer to a position in the cache as a cache slot. In order to map an address to cache slot, we need to first get the word address then compute the following:
```
slot_index = addr_word % 4;
```

This is a very basic hash function that uses the remainder of the division of the word address by the cache size to determine the position in the cache. Using this mapping function the following word addresses all map to the same cache slot number, slot number 0: 0, 4, 8, 12, 16, etc. Similarly the following word addresses all map to cache slot number 1: 1, 5, 9, 13, 17, etc.

Using the modulo operator is one way to get the slot number based on the cache size. However, we can also use bit manipulation to get the same value.

First, another way to get the word address from a byte address is the following:
```
addr_word = addr >> 2;
```

We know that a right shift by 1 is the same as dividing by 2, so a right shift by 2 is the same as dividing by 4. Now let's consider the slot_index. We know we can use modulus (%) to get the remainder, but another way to get the remainder is to look at the lowest bits in the word address to get the slot_index like this:
```
slot_index = addr_word & 0b11;
```

Why does this work? Take a look at the binary form of the following word addresses in both decimal and binary form (we only show the bottom 5 bits of each word address):
```
dec bin
 0  00000 <-- slot 0
 1  00001 <-- slot 1
 2  00010 <-- slot 2
 3  00011 <-- slot 3
 4  00100 <-- slot 0
 5  00101 <-- slot 1
 6  00110 <-- slot 2
 7  00111 <-- slot 3
 8  01000 <-- slot 0
 9  01001 <-- slot 1
10  01010 <-- slot 2
```

Notice that the last two bits of each word address is exactly the same as addr_word % 4. So, we can just mask off those last two bits to get the slot_index. If we increase the number slots in our cache, this technique still works. For example, if we increase our cache to hold 8 slots, we can now get the slot_num using addr_word % 8 or we can mask off the last 3 bits (2^3 = 8) to get the slot number. This holds for all cache sizes that are a power of 2, which is usually the case.

Now we have a way to map an address to a cache slot index. Our next challenge is to figure out if the address and data we are looking for is actually in the cache. Since multiple address map to the same cache slot, we need a way to know the requested address and data are in the cache. To do this we need to store additional information in the cache slot. So now, we can view a cache slot as a struct:
```
struct cache_slot_st {
    uint64_t addr;
    uint32_t data;
}

struct cache cache_slot_st[N];
```

That is, we associate the requesting address with the 32 bit word. This technically works, but we don't need the entire byte address to determine if we have a hit. Look at the word addresses that map to slot 0 from above:
```
dec bin
 0  00000 <-- slot 0
 4  00100 <-- slot 0
 8  01000 <-- slot 0
12  01100 <-- slot 0
```

Look at the upper 3 bits for each of these word addresses; 000, 001, 010, 011. They are all unique. While we are only showing 5 bits, all the word address that map to slot 0 will have unique values for the upper bits. We call the unique upper bits the **tag**.

We can now view a byte address in the following way for a 64 bit byte address:
```
tag[63:4] slot_index[3:2] byte_offset[1:0]
```

The byte_offset is just the lower two bits that we loose when we compute the word address. If we wanted to access a specific byte from a word in the cache, we would need to use the byte_offset.

In addition to the data and tag, we need one other piece of information in every cache slot. Typically when we power up a computer or start a new program, the cache does not have valid data. On power up, the computer zeros out all values in the cache (and other memory elements as well). With the cache zeroed out and without any memory requests, there is no valid data in the cache. However a tag of value 0 and data of value 0 are both legitimate values, so without something else, the cache would return the value zero for a request with tag equal to 0. To accommodate this initialization problem we introduce a valid field to each cache slot. So now, our cache struct looks like this:
 ```
 struct cache_slot_st {
    uint64_t tag;
    uint32_t data;
    uint32_t valid;
}

struct cache_slot_st cache[N];
```

If valid is 0, the cache slot does not hold valid data.  A request that maps to this slot will result in a miss and a subsequent memory access. If valid is 1, then we need to compare the tag of the memory request address to the tag in the cache slot. If they are equal, then we have a cache hit.

Here is the pseudo code for handling an addr request to a 4 word direct-mapped cache:
```
addr_tag = addr >> 4; // remove byte bits (2) and index bits (2)
index_mask = 0b11;
slot_index = (addr >> 2) & index_mask;
slot = cache[slot_index];
if (slot.valid == 1 && slot.tag == addr_tag) {
  // hit
  return slot.data;
} else {
  // miss
  slot.data = *((uint32_t *) addr);
  slot.tag = addr_tag;
  slot.valid = 1;
  return slot.data;
}
```

## Fully-Associative Caches

Before describing set-associative caches, it is useful to first understand how fully-associative caches work. In a fully-associative cache, an address can map to any cache slot, not just one like in the direct-mapped cache. In a direct-mapped cache it is possible to have cache slots that go unused because no addresses mapped to the unused slots during program execution. This usually does not happen for small caches, but can happen as the size of the cache increases. To avoid a situation in which we have unused cache slots a fully associative cache is the most flexible. However, this flexibility comes at a cost. When we get an address request we need to compare the address tag with all the slot tags. In code this requires a loop to check the tag in each cache slot. In hardware this comparison can be done in parallel, but it requires quite a bit of logic to achieve this. While fully-associative caches are too expense for regular cache memory, they are used in very specific situations, such as in hardware support for virtual memory.

We can use the same structure we did for direct mapped caches:
```
 struct cache_slot_st {
    uint64_t tag;
    uint32_t data;
    uint32_t valid;
    uint64_t timestamp;
}

struct cache_slot_st cache[N];
```

However, we now view the request address like this:
```
tag[63:2] byte_offset[1:0]
```

That is, we no longer need a slot_index because the address can be mapped to any slot in the cache. Here is the pseudo code to handling an addr request to a 4 word fully-associative cache:
```
num_refs += 1;
addr_tag = addr >> 2; // remove byte offset bits (2)
for (i = 0; i < 4; i++) {
  slot = cache[i];
  if (slot.valid == 1 && slot.tag == addr_tag) {
      // hit
      slot.timestamp = num_refs;
      return slot.data;
  }
}
// miss
slot = find_lru_slot(cache);
slot.data = *((uint32_t *) addr);
slot.tag = addr_tag;
slot.valid = 1;
slot.timestamp = num_refs;
return slot.data;
```

On a miss, we need to find a slot to use for the new request. Because an address can map to any slot in the cache, we should first look for an unused cache slot. We can look for the first slot whose valid field is 0, as this means the slot is unused. If all the slots are used (valid) we need a way to choose a slot to evict. A good choice is to pick the slot that has been least recently used. In software we could use a linked list to keep track of recently used slots by moving the currently access slot to the beginning of the list. Over time the least recently used slot will be at the end of the list. Another option is to associate a timestamp with each slot. This could be a logical timestamp like the number of cache references. This value will always increase. Now, each time we access a cache slot, we can set the slot timestamp to the current number of references. To find the least recently used slot, we look for the slot with the lowest (smallest) timestamp, as this will be the slot access least recently compared to all the other slots.

In hardware we can't use a linked list or a full timestamp value. Instead of perfect LRU an approximate form of LRU is used. 

The find_lru_slot() function can first look for an unused slot, then use LRU via timestamps to find the slot to evict.

## Set Associative Caches
Direct-mapped caches are simple, but rigid. A fully-associative cache is flexible, but expensive. A set-associative is a compromise between direct and fully associative. The idea is to think about a cache as an array of sets. Now we map an address to a set using a direct mapped approach, but we allow the address to map to any slot in the set. We refer to the number of slots in a set as the number of **ways**. We refer to a specific set-associative cache as an n-way set associative cache, such as 4-way set associative or 8-way set-associative. 

We can still use the same slot structure for our set-associative cache:
```
 struct cache_slot_st {
    uint64_t tag;
    uint32_t data;
    uint32_t valid;
    uint64_t timestamp;
}

struct cache_slot_st cache[N];
```

The number of sets in the cache will be:
```
num_sets = N / (num_ways)
```

So if we have 4 ways and 32 cache slots, we have 32 / 4 = 8 sets total.

With a set-associative cache, we now need to find the set index given an addr. So our addr now looks like this:
```
tag[63:5] set_index[4:2] byte_offset[1:0]
```

Note that the set_index is 3 bits because we have 8 total sets (2^3 = 8). Here is the pseudo code for handling a set-associative lookup for a 4-way cache with 32 cache slots, which is 8 sets total. So we can reuse the cache array we used for direct mapped and fully associative, we will logically consider the cache slot array to be groups of 4 slots. That is, the first 4 slots will be set 0, then second 4 slots will be set 1, and so on. In this way, once we find the set index, we can use this value to find the first slot in a set.
```
num_refs += 1;
num_ways = 4;
addr_tag = addr >> 5; // remove byte offset bits (2) and set_index bits (3)
set_index = (addr >> 3) & 0b111;

// 4 ways per set, so set_index * 4 gives us the first slot index in a set
set_base = set_index * 4;

for (i = 0; i < num_ways; i++) {
    slot = cache[set_base + i];
    if (slot.valid == 1 && slot.tag == addr_tag) {
        // hit
        slot.timestamp = num_refs;
        return slot.data;
    }
}

// miss
slot = find_lru_slot_in_set(cache, set_base);
slot.data = *((uint32_t *) addr);
slot.tag = addr_tag;
slot.valid = 1;
slot.timestamp = num_refs;
return slot.data;
```

The key point with a set-associative cache is that we only need to do a full lookup on the slots within a set (4-way or 8-way for example) and not all the slots in the cache. This gives some flexibility to choose where an address should go within a set. The find_lru_slot_in_set() function should first look for an unused slot in the set, then pick a slot based on LRU for the slots in the set.

## Block Size

In all the caches described above we assume that each cache slot can hold one word (a 32-bit value). However, to take advantage of spatial locality and to reduce the latency cost for transferring smaller amounts of data from memory to the cache, we can have each slot hold multiple words instead of a single word. For a 4 word block size we can define our cache_slot_st like this:
```
 struct cache_slot_st {
    uint64_t tag;
    uint32_t block[4];
    uint32_t valid;
    uint64_t timestamp;
}

struct cache_slot_st cache[N];
```

Similar to how we treat 4 bytes as a word in memory, we now treat 4 words as a block in memory. That is we can view memory logically as an array of blocks where each block consists of 4 words. Consider a 4 slot cache. Now our addresses can be viewed as follows for the different cache types.

Direct Mapped
```
tag[63:6] slot_index[5:4] block_index[3:2] byte_offset[1:0]
```
Fully Associative
```
tag[63:4] block_index[3:2] byte_offset[1:0]
```

Set Associative (2-way)
```
tag[63:5] set_index[4] block_index[3:2] byte_offset[1:0]
```

So, we associate a tag with a block of words. When we get a miss we need to find the block the addr is pointing to. We can find the base word of a block like this:

```
addr_word = addr / 4;
block_index = addr_word % 4  // where 4 is the block size in words
block_base = addr_word - block_index;
```

Now you can load 4 words from memory starting at the block_base.
Note that block_base is a word address and you will need to convert it to a byte address so you can dereference the address properly. Alternatively, you can perform similar calculations using byte addresses or use bitwise operators to get the same values.
