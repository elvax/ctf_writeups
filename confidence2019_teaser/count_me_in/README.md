## Count me in!
 Points: 59,
 Solves: 115,
 crypto
 
 #### Description
 I've been lately using this cool AES-CTR, but it was super slow, so I made a parallel version. Now it's blazing fast, but for some reason I have trouble decrypting the data...
 
 Files: [count.py](count.py) [output.txt](output.txt)
 
 #### Solution
 First of all, processess don't share variable "counter". Each process starts with its own copy of this variable.
 Here are couple of results after printing variable "counter":
 ```
 0
0
1
0
0
0
1
1
0
0
2
1
0
2
...
```

Encryption is as follow:

```python
def worker_function(block):
    global counter
    key_stream = aes.encrypt(pad(str(counter)))
    result = xor_string(block, key_stream)
    counter += 1
    return result
 ```
Plaintext chunk is xored with key_stream. Key stream is produced by encryptig padded variable counter with AES ECB mode. 
As we can see in first listing there are a lot of repetitions of the same value. This leads to plaintext chunks being 
xored with the same key stream.

First we retrieve all possible key streams xoring ciphertext with plaintext and then xor them with last chunks of ciphertext
(where the flag is).

```python
def chunk(input_data, size):
    return [input_data[i:i + size] for i in range(0, len(input_data), size)]

def xor(*t):
    from functools import reduce
    from operator import xor
    return [reduce(xor, x, 0) for x in zip(*t)]


def xor_string(t1, t2):
    t1 = map(ord, t1)
    t2 = map(ord, t2)
    return "".join(map(chr, xor(t1, t2)))

plaintext = """The Song of the Count

You know that I am called the Count
Because I really love to count
I could sit and count all day
Sometimes I get carried away
I count slowly, slowly, slowly getting faster
Once I've started counting it's really hard to stop
Faster, faster. It is so exciting!
I could count forever, count until I drop
1! 2! 3! 4!
1-2-3-4, 1-2-3-4,
1-2, i love couning whatever the ammount haha!
1-2-3-4, heyyayayay heyayayay that's the sound of the count
I count the spiders on the wall...
I count the cobwebs in the hall...
I count the candles on the shelf...
When I'm alone, I count myself!
I count slowly, slowly, slowly getting faster
Once I've started counting it's really hard to stop
Faster, faster. It is so exciting!
I could count forever, count until I drop
1! 2! 3! 4!
1-2-3-4, 1-2-3-4, 1,
2 I love counting whatever the
ammount! 1-2-3-4 heyayayay heayayay 1-2-3-4
That's the song of the Count!
"""

with open('output.txt', 'r') as file:
    cipher = file.read().decode('hex')

plaintext_chunks = chunk(plaintext, 16)
cipher_chunks = chunk(cipher, 16)

keys = set()
for p, c in zip(plaintext_chunks, cipher_chunks):
    key = xor_string(p, c)
    keys.add(key)

for key in keys:
    for flag_part in cipher_chunks[57:]:
        print( xor_string(flag_part, key))
```
flag p4{at_the_end_of_the_day_you_can_only_count_on_yourself}
