---
title: "A Primite Hashing Function in Go"
date: 2022-01-22T18:30:19+02:00
categories:
- tech
tags:
- security
- golang
---

Cryptographic hash functions are complex mathematical calculations. Therefore understanding them requires a considerable
amount of time and patience. However, they all have things in common: an input, a cryptographic algorithm, and an output.

Recently, I had a chance to study some popular cryptographic hash functions, such as
[MD5](https://en.wikipedia.org/wiki/MD5) and [SHA-1](https://en.wikipedia.org/wiki/SHA-1), and tried to understand how
they really work. Wikipedia pages I linked include a considerable amount of information already, and more can be found
online, but what I want to do was understand similarities between them and write my own primitive hashing
function in Go.

Splitting an input into chunks, dividing chunks into fixed-sized bit blocks, adding bits, padding messages with 0 bit,
shifting bits, running xor on bits, and so on are some common methods that can you come across in many hashing
algorithms. I will not use all these practices in my "primitive" implementation, but I will cover enough to make a
reader get started.

First, let's start by remembering the requirements for any hashing algorithm:
  - Infeasible to produce a given digest (you can't guess possible plaintext messages)
  - Impossible to extract original message (hashing must be one-way)
  - Slight changes in the original message should produce drastic changes in the digest
  - Resulting digest is fixed-width (length)
  - Should be fast, but also not too fast!

Besides these strict requirements, we also want as few [hash collision](https://en.wikipedia.org/wiki/Hash_collision) as
possible. The thing is, collisions can't be avoided in hashing, because it's a byproduct of _fixed width digests_. They
can only be made rare. To make them rarer, we can simply use big block sizes, and as a result, we end up with
big checksums. I won't be concerned too much about collisions or security here, as what I'm trying to write is just a
primitive hashing function for educational purposes, and not something production-ready.

> Here is the full code: https://gist.github.com/msdundar/f88aee1a89d603b3d81f9087e3dc1012

### Step 1: Convert string to binary

We will be using a binary representation to run an XOR machine later. Therefore, any input needs to be converted to
binary first:

```
    h        e        l        l        o        w        o        r       l        d
01101000 01100101 01101100 01101100 01101111 00100000 01110111 01101111 01110010 01101100
```

```go
// stringToBinary converts strings to binary
func stringToBinary(s string) (binaryString string) {
  for _, c := range s {
    binaryString = fmt.Sprintf("%s%.8b", binaryString, c)
  }
  return 
}
```

### Step 2: Split binary into bit-blocks and pad blocks with 0-bit

Here I use blocks of 32bits for simplicity, but in actual implementation it's 128bits:

```
              hell                             owor                             ld
01101000011001010110110001101100 01101111001000000111011101101111 01110010011011000000000000000000
```

For this example, I decided to use blocks of 128bits, but you can configure it depending on your needs. Keep in mind,
a bigger block size will perform badly for short messages but will reduce the chances of collision for long messages.
Think about these tradeoffs when modifying this value.

Here, I'm splitting the binary string into blocks of 128bits, and I pad blocks with 0 bit if they are shorter than
128bits:

```go
// splitToBlocks takes a binary string and splits it into blocks
func splitToBlocks(msg string) []string{
  desiredBlockSize := 128
  numberOfBits := len(msg)

  requiredSliceCap := int(math.Ceil(
    float64(numberOfBits)/float64(desiredBlockSize)),
  )

  fourByteSlice := make([]string, requiredSliceCap)

  byteIndex := 0

  for i := 0; i < len(msg); i++ {
    if i != 0 && i % desiredBlockSize == 0 {
      byteIndex += 1
    }

    fourByteSlice[byteIndex] = fourByteSlice[byteIndex] + string(msg[i])
  }

  // Pad with 0s to ensure all blocks are the same size
  for ; len(fourByteSlice[byteIndex]) < desiredBlockSize; {
    fourByteSlice[byteIndex] = fourByteSlice[byteIndex] + "0"
  }


  return fourByteSlice
}
```

### Step 3: Create a simple XOR machine

Running XOR on bits is a very common technique for many hashing algorithms. When using big block sizes, each XOR
makes it much harder to reverse the hashing process. However, with short messages, there will be fewer blocks to use
with XOR, and the process can be reversed if no additional hardening is implemented. However, we aren't much concerned
about collisions in this _primitive example_.

```go
// xorMachine runs longitudinal parity checks on given blocks
func xorMachine(item1 string, item2 string) string{
  var xorOutput string

  for i, _ := range item1 {
    if item1[i] == item2[i] {
      xorOutput += "0"
    } else {
      xorOutput += "1"
    }
  }

  return xorOutput
}
```

### Step 4: Pass bit-blocks to XOR machine one-by-one

In this step, all we need to do is pass bit-blocks to the XOR machine one by one:

```go
binaryStringSlc := splitToBlocks(
  stringToBinary("Hello world! This is Serhat! This post is all about implementing a primitive hashing function in Go."),
)

for i := 0; i < len(binaryStringSlc); i++{
  if !(i+1 == len(binaryStringSlc)) {
    comparisonStr := xorMachine(binaryStringSlc[i], binaryStringSlc[i+1])

    binaryStringSlc[i+1] = comparisonStr
  }
}

fmt.Println(binaryStringSlc[len(binaryStringSlc)-1])
```

And voila! Here is our checksum in binary:

```go
01001111010111110010000001101001000001110110010101011010000111110001101101011101000010110000100001010110000000010000011100011000
```

Needless to say, you should never use this hashing function on production. Existing hashing functions are 100x more
complicated than this, and probably 1000x more secure. Therefore, you should only use a battle-tested algorithm such as
SHA-3, bcrypt, and so on.
