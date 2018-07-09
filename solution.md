
# Parsing Transactions

Transactions are at the heart of Bitcoin. Transactions, simply put, are value transfers from one entity to another. We'll see in Chapter 7 how "entities" in this case are really smart contracts. But we're getting ahead of ourselves. Lets first look at what transactions in Bitcoin are, what they look like and how they are parsed.

### Transaction Components

At a high level, a transaction really only has 4 components. They are:

1. Version
2. Inputs
3. Outputs
4. Locktime

At this point a general overview of these fields might be helpful. Version defines what the format of the transaction is supposed to be, Inputs define what bitcoins are being spent, Outputs define where the bitcoins are going and Locktime defines what time this transaction starts being valid. We'll go through each component in depth.

### Version

When you see a version number in something, it's meant to give the receiver information about what the versioned thing is supposed to represent. If, for example, you are running Windows 3.1, that's a version number that's very differnt than Windows 8 or Windows 10. You could specify just "Windows", but specifying the version number after the operating system helps you know what features it has and what API's you can program against.

In a similar way, Bitcoin transactions have version numbers. In Bitcoin's case, the transaction version has never been updated since version 1.

You may notice here that the actual value in hexadecimal is 01000000, which doesn't look like 1. It actually is when the interpreted as a little-endian integer (recall the discussion from chapter 4).


### Test Driven Exercise

Parse the transaction version.


```python
from tx import Tx
from helper import (
    little_endian_to_int,
    read_varint,
)

class Tx(Tx):

    @classmethod
    def parse(cls, s):
        '''Takes a byte stream and parses the transaction at the start
        return a Tx object
        '''
        # s.read(n) will return n bytes
        # version has 4 bytes, little-endian, interpret as int
        version = little_endian_to_int(s.read(4))
        # num_inputs is a varint, use read_varint(s)
        num_inputs = read_varint(s)
        # each input needs parsing
        inputs = []
        for _ in range(num_inputs):
            inputs.append(TxIn.parse(s))
        # num_outputs is a varint, use read_varint(s)
        num_outputs = read_varint(s)
        # each output needs parsing
        outputs = []
        for _ in range(num_outputs):
            outputs.append(TxOut.parse(s))
        # return an instance of the class (cls(...))
        return cls(version, inputs, outputs, locktime)
```

### Inputs

Each input is an output of a previous transaction. This last sentence requires more explanation as it's not intuitively obvious at first.

TODO: explain how all bitcoin inputs are previous transaction outputs

The inputs field can contain more than one input. This makes sense as you could use a single 100 bill to pay for a 70 dollar meal or a 50 and a 20. There are situations where there could be lots of inputs. In our analogy, we can pay for a 70 dollar meal with 14 5-dollar bills or even 7000 pennies. Each input contains 4 fields, the first two which point ot the previous transaction output and two more that define how it can be spent. These are as follows:

* Previous transaction id
* Previous transaction index
* ScriptSig
* Sequence

As explained above, each input is actually a previous transaction's output. The previous transaction id is the double_sha256 of the previous transaction's contents. This uniquely defines the previous transaction as the probability of a hash collision is very, very small. As we'll see, each transaction has to have at least 1 output, but may have many. Thus, we need to define exactly which output we're spending.

ScriptSig has to do with Bitcoin's smart contract language SCRIPT, and will be discussed more thoroughly in chapter 6. For now, think of ScriptSig as opening a lock box. Something that can only be done by the owner of the transaction output.

Sequence was originally intended as a way to do payment channels (see sidebar), but is currently used with Replace-By-Fee and Check Sequence Verify.

A couple things to note here. The amount of each input is actually not specified here. We have no idea how much is being spent unless we actually look up the transaction. Furthermore, we don't even know if we're signing the right check, so to speak, without knowing something about the previous transaction. Every node must verify that this transaction is actually signing the right check and that they're not overspending.


### Test Driven Exercise

Parse the transaction inputs


```python
from tx import TxIn

class TxIn(TxIn):

    @classmethod
    def parse(cls, s):
        '''Takes a byte stream and parses the tx_input at the start
        return a TxIn object
        '''
        # s.read(n) will return n bytes
        # prev_tx is 32 bytes, little endian
        prev_tx = s.read(32)[::-1]
        # prev_index is 4 bytes, little endian, interpret as int
        prev_index = little_endian_to_int(s.read(4))
        # script_sig is a variable field (length followed by the data)
        # get the length by using read_varint(s)
        script_sig_length = read_varint(s)
        script_sig = s.read(script_sig_length)
        # sequence is 4 bytes, little-endian, interpret as int
        sequence = little_endian_to_int(s.read(4))
        # return an instance of the class (cls(...))
        return cls(prev_tx, prev_index, script_sig, sequence)
```

### Outputs

As hinted in the previous section, outputs define where the bitcoins are actually going. We must have at least one output and can have lots of outputs. An exchange may batch transactions, for example, and pay out a lot of people at once instead of generating a single transaction for every single person that requests Bitcoins.

Outputs each have two fields: amount and ScriptPubKey. Amount is the amount of bitcoin being assigned and is specified in satoshis, or 1/100,000,000th of a Bitcoin. This allows us to divide Bitcoin very finely, down to 1/100th of a penny in USD terms as of this writing.

ScriptPubKey is much like ScriptSig in that it has to do with Bitcoin's smart contract language SCRIPT. Think of ScriptPubKey as the lock box that can only be opened by the holder of the key. A one-way safe that can receive deposits from anyone, but can only be opened by the owner of the safe.

### Test Driven Exercise

Parse the transaction outputs.


```python
from tx import TxOut

class TxOut(TxOut):
    
    @classmethod
    def parse(cls, s):
        '''Takes a byte stream and parses the tx_output at the start
        return a TxOut object
        '''
        # s.read(n) will return n bytes
        # amount is 8 bytes, little endian, interpret as int
        amount = little_endian_to_int(s.read(8))
        # script_pubkey is a variable field (length followed by the data)
        # get the length by using read_varint(s)
        script_pubkey_length = read_varint(s)
        script_pubkey = s.read(script_pubkey_length)
        # return an instance of the class (cls(...))
        return cls(amount, script_pubkey)
```
