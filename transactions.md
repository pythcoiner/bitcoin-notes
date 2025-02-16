# Transactions

## Transaction structure

`[version][marker][flag][inputs][outputs][witness_data][locktime]`

## Inputs

the `inputs` block is composed several sub blocks:

`[input_count][input_0][..][input_n]`

### General

Each transaction input contains 4 fields:

`[previous_txid(32 bytes)][previous_vout(4 bytes)][script_sig(VarInt)][sequence(4 bytes)]`

   little endian              little endian               ?             little endian

`previous_txid`: txid of the previous transaction where the output is spend from

`previous_vout`: ouput index of spent output in the previous transaction

`script_sig`: spending part of the script

`sequence`: sequence

Note: post segwit context witnesses are stored in the `witness_data` field of the transaction

`witness`: spending part of the script in post segwit context

## Ouputs

the `outputs` block is composed several sub blocks:

`[outputs_count][output_0][..][output_n]`

### General

Each transaction output contains 2 fields:

`[amount(8 bytes)][script_pubkey(VarInt)]`

   little endian           ?

`amount`: amount of the output in sats

`script_pubkey`: the "contract" part of the script (aka spending condition)

## Script types

### Note about pubkeys

Uncompressed: `[0x04][X (32 bytes)][Y (32 bytes)]`

Compressed(even): `[0x02][X (32 bytes)]` (lowest bit of Y == 0)

Compressed(odd): `[0x03][X (32 bytes)]` (lowest bit of Y == 1)

XOnly: `[X (32 bytes)]` (Y is always even)

### Note about signatures

Ecdsa signature(bytes size): 

`[0x30(1)][total_length(1)][0x02(1)][length(1)][R(31-32)][0x02(1)][S(31-32)][SIGHASH(1)]`

Taproot (schnorr) signature: 

`[R(32 bytes)][S(32 bytes)][SIGHASH (optionnal, 1 byte)]`

### P2PK

`script_pubkey`: 
```
[OP_PUSHBYTE_XX][public_key]
[OP_CHECKSIG]
```

`script_sig`: `[OP_PUSHBYTE_XX][signature]`

`witness`: `[]`

Note: pubkey can be compressed or uncompressed

### P2PKH

`script_pubkey`: 
```
[OP_DUP]
[OP_HASH160]
[OP_PUSHBYTE_20][pubkey_hash(20 bytes)]
[OP_EQUALVERIFY]
[OP_CHECKSIG]
```

`script_sig`: 
```
[OP_PUSHBYTE_XX][signature]
[OP_PUSHBYTE_XX][public_key]
```

`witness`: `[]`

Note: pubkey can be compressed or uncompressed

### P2MS

`script_pubkey`: 
```
[OP_0] (off by 1 bug)
[threshold]
[OP_PUSHBYTE_XX][public_key_0]
[..]
[OP_PUSHBYTE_XX][public_key_n]
[count]
[OP_CHECKMULTISIG]
```

Note: pubkey can be compressed or uncompressed

`script_sig`: 
```
[OP_PUSHBYTE_XX][signature_0]
[..]
[OP_PUSHBYTE_XX][signature_n]
```

Note: signatures must be placed in same order than pubkeys

`witness`: `[]`


### P2SH

`script_pubkey`: `[OP_HASH160][script_hash(20 bytes)][OP_EQUAL]`

Note: pubkey can be compressed or uncompressed

`script_sig`: `[script_input][redeem_script]`

`witness`: `[]`


### P2WPKH

`script_pubkey`: 
```
[0x00(segwit version 0)]
[OP_PUSHBYTE20][pubkey_hash(20 bytes)]
```

placeholder script that will  be replaced by:

```
[OP_DUP]
[OP_HASH160]
[OP_PUSHBYTE_20][pubkey_hash(20 bytes)]
[OP_EQUALVERIFY]
[OP_CHECKSIG]
```

`script_sig`: `[]`

`witness`:
```
[OP_PUSHBYTE_XX][signature]
[OP_PUSHBYTE_XX][public_key]
```

Note: pubkey will be of compressed format


### P2WSH

`script_pubkey`: 
```
[0x00(segwit version 0)]
[OP_PUSHBYTE_32][pubkey_hash(32 bytes)]
```

`script_sig`: `[]`

`witness`:
```
[script_input]
[witness_script]
```

Note: pubkey will be of compressed format

### P2TR

`script_pubkey`: 
```
[0x01(segwit version 1)]
[OP_PUSHBYTE_32][tweaked_pubkey(32 bytes)]
```

`script_sig`: `[]`

Internal key spend:

`witness`: `[signature (64-65 bytes)]`

Taptree spend

`witness`:
```
[script_input]
[leaf_script]
[control_block]
```

where control_block can be defined as:

```
[LEAF_VERSION (1 byte)]
[INTERNAL_KEY (32 bytes)]
[BRANCH_HASH]
[..]
[BRANCH_HASH]
[LEAF_HASH]
```


