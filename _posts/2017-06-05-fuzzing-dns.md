---
layout: post
title: Improve DNS decoding robustness through fuzzing
---

## Forced unwrapping results in crashes (d0h!)
In order to implement Bonjour in Swift (for use on Linux), I needed a way to marshall / unmarshall DNS queries. I have written the [DNS](https://github.com/Bouke/DNS) library to do this. However sometimes the code would crash the whole application if the input was either incorrect or unsupported. I'd written the library mostly focused on the success path, not taking into account the possibilities of error. This meant that my Swift code was riddled with forced unwrapping of `Optional`s and  `precondition` that would segfault whenever the input didn't match expectations.

After making most of the methods throwable;

```diff
-    public init(unpack bytes: Data) {
+    public init(unpack bytes: Data) throws {
```

and replacing the `preconditions`;

```diff
-        precondition(position == expectedPosition, "Unexpected length")
+        guard position == expectedPosition else {
+            throw DecodeError.invalidDataSize
+        }
```

the code would still fail on assumptions that weren't explicitly written down. However sufficient the test data required to resolve those issues proved difficult. 

## Fuzzing

Having read about fuzzing previously, I kept it on my list of things to play with, and this would be a great time to get some air time with it. A short summary from Wikipedia about fuzzing:

> **Fuzzing** or **fuzz testing** is an automated [software testing](https://en.wikipedia.org/wiki/Software_testing) technique that involves providing invalid, unexpected, or [random data](https://en.wikipedia.org/wiki/Random_data) as inputs to a [computer program](https://en.wikipedia.org/wiki/Computer_program). The program is then monitored for exceptions such as [crashes](https://en.wikipedia.org/wiki/Crash_(computing)), or failing built-in code [assertions](https://en.wikipedia.org/wiki/Assertion_(computing)) or for finding potential [memory leaks](https://en.wikipedia.org/wiki/Memory_leak). Typically, fuzzers are used to test programs that take structured inputs. This structure is specified, e.g., in a [file format](https://en.wikipedia.org/wiki/File_format) or [protocol](https://en.wikipedia.org/wiki/Communications_protocol) and distinguishes valid from invalid input. An effective fuzzer generates semi-valid inputs that are "valid enough" so that they are not directly rejected from the parser but able to exercise interesting behaviors deeper in the program and "invalid enough" so that they might stress different corner cases and expose errors in the parser.
>
> From: [Fuzzing](https://en.wikipedia.org/wiki/Fuzzing).

My first attempt was to feed completely randomized data to the parser. However this meant that 0.0000001% (made-up statistic) of the input would be valid, and as a result fuzzing would require too much time. So on my second attempt I started with a valid message, on which I'd perform minor changes. This increased effictiveness significantly and I was getting a lot of crashes. In my fuzzing, I performed a random combination of any of the following operations: 

* replace a byte at a random position
* remove a byte at a random position
* insert a byte at a random position

```swift
let message = Message(...)
let original: Data = try! message.pack()
while true {
    var corrupted = original
    for _ in 1..<(2 + arc4random_uniform(4)) {
        let index = Data.Index(arc4random_uniform(UInt32(corrupted.endIndex)))
        switch arc4random_uniform(3) {
        case 0:
            corrupted[index] = corrupted[index] ^ UInt8(truncatingBitPattern: arc4random_uniform(256))
        case 1:
            corrupted.remove(at: index)
        case 2:
            corrupted.insert(UInt8(truncatingBitPattern: arc4random_uniform(256)), at: index)
        default:
            abort()
        }
    }
    do {
        _ = try Message(unpack: corrupted)
    } catch {
    }
}
```

In the code above, line 19 would segfault on incorrect input. Every time that happened, I would record the `corrupted` value and improve the code so it correctly handled that input.

## Profit!

Overall fuzzing is a great way to generate a large variety in test data and is a good tool to add to your toolbelt. You can see all the fuzzing related changes in the [DNS repository](https://github.com/Bouke/DNS/compare/0.3.0...0.3.1).
