# Signing and Verifications in Bitcoin

```python
# import everything and define a test runner function
from importlib import reload
from helper import run_test

import ecc
import helper
import tx
```


```python
# Signing Example
from random import randint

from ecc import G, N
from helper import double_sha256

secret = 1800555555518005555555
z = int.from_bytes(double_sha256(b'ECDSA is awesome!'), 'big')
k = randint(0, 2**256)
r = (k*G).x.num
s = (z+r*secret) * pow(k, N-2, N) % N
print(hex(z), hex(r), hex(s))
print(secret*G)
```


```python
# Verification Example
from ecc import G, N, S256Point

z = 0xbc62d4b80d9e36da29c16c5d4d9f11731f36052c72401a76c23c0fb5a9b74423
r = 0x37206a0610995c58074999cb9767b87af4c4978db68c06e8e6e81d282047a7c6
s = 0x8ca63759c1157ebeaec0d03cecca119fc9a75bf8e6d0fa65c841c8e2738cdaec
point = S256Point(0x04519fac3d910ca7e7138f7013706f619fa8f033e6ec6e09370ea38cee6a7574,
                  0x82b51eab8c27c66e26c858a079bcdf4f1ada34cec420cafc7eac1a42216fb6c4)
u = z * pow(s, N-2, N) % N
v = r * pow(s, N-2, N) % N
print((u*G + v*point).x.num == r)
```

### Exercise 1

#### 1.1. Which sigs are valid?

```
P = (887387e452b8eacc4acfde10d9aaf7f6d9a0f975aabb10d006e4da568744d06c,
     61de6d95231cd89026e286df3b6ae4a894a3378e393e93a0f45b666329a0ae34)
z, r, s = ec208baa0fc1c19f708a9ca96fdeff3ac3f230bb4a7ba4aede4942ad003c0f60,
          ac8d1c87e51d0d441be8b3dd5b05c8795b48875dffe00b7ffcfac23010d3a395,
          68342ceff8935ededd102dd876ffd6ba72d6a427a3edb13d26eb0781cb423c4
z, r, s = 7c076ff316692a3d7eb3c3bb0f8b1488cf72e1afcd929e29307032997a838a3d,
          eff69ef2b1bd93a66ed5219add4fb51e11a840f404876325a1e8ffe0529a2c,
          c7207fee197d27c618aea621406f6bf5ef6fca38681d82b2f06fddbdce6feab6
```

#### 1.2. Make [these tests](/edit/session2/ecc.py) pass
```
ecc.py:S256Test:test_verify
ecc.py:PrivateKeyTest:test_sign
```


```python
# Exercise 1.1

from ecc import S256Point, G, N

px = 0x887387e452b8eacc4acfde10d9aaf7f6d9a0f975aabb10d006e4da568744d06c
py = 0x61de6d95231cd89026e286df3b6ae4a894a3378e393e93a0f45b666329a0ae34

signatures = (
    # (z, r, s)
    (0xec208baa0fc1c19f708a9ca96fdeff3ac3f230bb4a7ba4aede4942ad003c0f60,
     0xac8d1c87e51d0d441be8b3dd5b05c8795b48875dffe00b7ffcfac23010d3a395,
     0x68342ceff8935ededd102dd876ffd6ba72d6a427a3edb13d26eb0781cb423c4),
    (0x7c076ff316692a3d7eb3c3bb0f8b1488cf72e1afcd929e29307032997a838a3d,
     0xeff69ef2b1bd93a66ed5219add4fb51e11a840f404876325a1e8ffe0529a2c,
     0xc7207fee197d27c618aea621406f6bf5ef6fca38681d82b2f06fddbdce6feab6),
)

# initialize the public point
# use: S256Point(x-coordinate, y-coordinate)

# iterate over signatures
    # u = z / s, v = r / s
    # finally, uG+vP should have the x-coordinate equal to r
```


```python
# Exercise 1.2

reload(ecc)
run_test(ecc.S256Test('test_verify'))
run_test(ecc.PrivateKeyTest('test_sign'))
```

### Exercise 2

#### 2.1. Verify the DER signature for the hash of "ECDSA is awesome!" for the given SEC pubkey

`z = int.from_bytes(double_sha256('ECDSA is awesome!'), 'big')`

Public Key in SEC Format:
0204519fac3d910ca7e7138f7013706f619fa8f033e6ec6e09370ea38cee6a7574

Signature in DER Format: 304402201f62993ee03fca342fcb45929993fa6ee885e00ddad8de154f268d98f083991402201e1ca12ad140c04e0e022c38f7ce31da426b8009d02832f0b44f39a6b178b7a1



```python
# Exercise 2.1

from ecc import S256Point, Signature
from helper import double_sha256

der = bytes.fromhex('304402201f62993ee03fca342fcb45929993fa6ee885e00ddad8de154f268d98f083991402201e1ca12ad140c04e0e022c38f7ce31da426b8009d02832f0b44f39a6b178b7a1')
sec = bytes.fromhex('0204519fac3d910ca7e7138f7013706f619fa8f033e6ec6e09370ea38cee6a7574')

# message is the double_sha256 of the message "ECDSA is awesome!"
z = int.from_bytes(double_sha256(b'ECDSA is awesome!'), 'big')

# parse the der format to get the signature
# parse the sec format to get the public key

# use the verify method on S256Point to validate the signature
```

### Exercise 3

WIF is the serialization of a Private Key.

#### 3.1. Find the WIF Format of the following:

* \\(2^{256}-2^{199}\\), mainnet, compressed
* \\(2^{256}-2^{201}\\), testnet, uncompressed
* 0dba685b4511dbd3d368e5c4358a1277de9486447af7b3604a69b8d9d8b7889d, mainnet, uncompressed
* 1cca23de92fd1862fb5b76e5f4f50eb082165e5191e116c18ed1a6b24be6a53f, testnet, compressed

#### 3.2. Make [this test](/edit/session3/ecc.py) pass
```
ecc.py:PrivateKeyTest:test_wif
```


```python
# WIF Example
from helper import encode_base58_checksum

secret = 2**256 - 2**200
s = secret.to_bytes(32, 'big')
print(encode_base58_checksum(b'\x80'+s))
print(encode_base58_checksum(b'\x80'+s+b'\x01'))
print(encode_base58_checksum(b'\xef'+s))
print(encode_base58_checksum(b'\xef'+s+b'\x01'))
```


```python
# Exercise 3.1
from helper import encode_base58_checksum

components = (
    # (secret, testnet, compressed)
    (2**256-2**199, False, True),
    (2**256-2**201, True, False),
    (0x0dba685b4511dbd3d368e5c4358a1277de9486447af7b3604a69b8d9d8b7889d, False, False),
    (0x1cca23de92fd1862fb5b76e5f4f50eb082165e5191e116c18ed1a6b24be6a53f, True, True),
)

# iterate through components
    # get the private key in 32-byte big-endian: num.to_bytes(32, 'big')
    # prepend b'\x80' for mainnet, b'\xef' for testnet
    # append b'\x01' for compressed
    # base58 the whole thing with checksum
    # print the wif
```


```python
# Exercise 3.2

reload(ecc)
run_test(ecc.PrivateKeyTest('test_wif'))
```

### Exercise 4

#### 4.1. Make [this test](/edit/session3/tx.py) pass
```
tx.py:TxTest:test_parse_version
```


```python
# Exercise 4.1

reload(tx)
run_test(tx.TxTest('test_parse_version'))
```

### Exercise 5

#### 5.1. Make [this test](/edit/session3/tx.py) pass
```
tx.py:TxTest:test_parse_inputs
```


```python
# Exercise 5.1

reload(tx)
run_test(tx.TxTest('test_parse_inputs'))
```

### Exercise 6

#### 6.1. Make [this test](/edit/session3/tx.py) pass
```
tx.py:TxTest:test_parse_outputs
```


```python
# Exercise 6.1

reload(tx)
run_test(tx.TxTest('test_parse_outputs'))
```

### Exercise 7

#### 7.1. Make [this test](/edit/session3/tx.py) pass
```
tx.py:TxTest:test_parse_locktime
```


```python
# Exercise 7.1

reload(tx)
run_test(tx.TxTest('test_parse_locktime'))
```

### Exercise 8

#### 8.1. What is the scriptSig from the second input in this tx?
#### 8.2. What is the scriptPubKey and amount of the first output in this tx?
#### 8.3. What is the amount for the second output?

```
010000000456919960ac691763688d3d3bcea9ad6ecaf875df5339e148a1fc61c6ed7a069e010000006a47304402204585bcdef85e6b1c6af5c2669d4830ff86e42dd205c0e089bc2a821657e951c002201024a10366077f87d6bce1f7100ad8cfa8a064b39d4e8fe4ea13a7b71aa8180f012102f0da57e85eec2934a82a585ea337ce2f4998b50ae699dd79f5880e253dafafb7feffffffeb8f51f4038dc17e6313cf831d4f02281c2a468bde0fafd37f1bf882729e7fd3000000006a47304402207899531a52d59a6de200179928ca900254a36b8dff8bb75f5f5d71b1cdc26125022008b422690b8461cb52c3cc30330b23d574351872b7c361e9aae3649071c1a7160121035d5c93d9ac96881f19ba1f686f15f009ded7c62efe85a872e6a19b43c15a2937feffffff567bf40595119d1bb8a3037c356efd56170b64cbcc160fb028fa10704b45d775000000006a47304402204c7c7818424c7f7911da6cddc59655a70af1cb5eaf17c69dadbfc74ffa0b662f02207599e08bc8023693ad4e9527dc42c34210f7a7d1d1ddfc8492b654a11e7620a0012102158b46fbdff65d0172b7989aec8850aa0dae49abfb84c81ae6e5b251a58ace5cfeffffffd63a5e6c16e620f86f375925b21cabaf736c779f88fd04dcad51d26690f7f345010000006a47304402200633ea0d3314bea0d95b3cd8dadb2ef79ea8331ffe1e61f762c0f6daea0fabde022029f23b3e9c30f080446150b23852028751635dcee2be669c2a1686a4b5edf304012103ffd6f4a67e94aba353a00882e563ff2722eb4cff0ad6006e86ee20dfe7520d55feffffff0251430f00000000001976a914ab0c0b2e98b1ab6dbf67d4750b0a56244948a87988ac005a6202000000001976a9143c82d7df364eb6c75be8c80df2b3eda8db57397088ac46430600
```


```python
# Exercise 8.1/8.2/8.3

from io import BytesIO
from tx import Tx

hex_transaction = '010000000456919960ac691763688d3d3bcea9ad6ecaf875df5339e148a1fc61c6ed7a069e010000006a47304402204585bcdef85e6b1c6af5c2669d4830ff86e42dd205c0e089bc2a821657e951c002201024a10366077f87d6bce1f7100ad8cfa8a064b39d4e8fe4ea13a7b71aa8180f012102f0da57e85eec2934a82a585ea337ce2f4998b50ae699dd79f5880e253dafafb7feffffffeb8f51f4038dc17e6313cf831d4f02281c2a468bde0fafd37f1bf882729e7fd3000000006a47304402207899531a52d59a6de200179928ca900254a36b8dff8bb75f5f5d71b1cdc26125022008b422690b8461cb52c3cc30330b23d574351872b7c361e9aae3649071c1a7160121035d5c93d9ac96881f19ba1f686f15f009ded7c62efe85a872e6a19b43c15a2937feffffff567bf40595119d1bb8a3037c356efd56170b64cbcc160fb028fa10704b45d775000000006a47304402204c7c7818424c7f7911da6cddc59655a70af1cb5eaf17c69dadbfc74ffa0b662f02207599e08bc8023693ad4e9527dc42c34210f7a7d1d1ddfc8492b654a11e7620a0012102158b46fbdff65d0172b7989aec8850aa0dae49abfb84c81ae6e5b251a58ace5cfeffffffd63a5e6c16e620f86f375925b21cabaf736c779f88fd04dcad51d26690f7f345010000006a47304402200633ea0d3314bea0d95b3cd8dadb2ef79ea8331ffe1e61f762c0f6daea0fabde022029f23b3e9c30f080446150b23852028751635dcee2be669c2a1686a4b5edf304012103ffd6f4a67e94aba353a00882e563ff2722eb4cff0ad6006e86ee20dfe7520d55feffffff0251430f00000000001976a914ab0c0b2e98b1ab6dbf67d4750b0a56244948a87988ac005a6202000000001976a9143c82d7df364eb6c75be8c80df2b3eda8db57397088ac46430600'

# bytes.fromhex to get the binary representation
# create a stream using BytesIO()
# parse from tx
# print tx's second input's scriptSig
# print tx's first output's scriptPubKey
# print tx's first output's amount
```
