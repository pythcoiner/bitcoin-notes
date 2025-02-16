
# Mempool replacement

## Rule 2

From [bitcoin core docs](https://github.com/bitcoin/bitcoin/blob/master/doc/policy/mempool-replacements.md):

```
The replacement transaction only include an unconfirmed input 
if that input was included in one of the directly conflicting 
transactions. An unconfirmed input spends an output from a 
currently-unconfirmed transaction.

Rationale: When RBF was originally implemented, the mempool 
did not keep track of ancestor feerates yet. This rule was 
suggested as a temporary restriction.
```

### Allowed
```
   ┌────────────┐   ┌────────────┐   ┌────────────┐
   │            │   │            │   │            │
   │    TxA     │   │    TxB     │   │    TxC     │
   │            │   │            │   │            │
   │   ┌───┐    │   │   ┌───┐    │   │   ┌───┐    │
   └───└─┬─┘────┘   └───└─┬─┘────┘   └───└───┘────┘
         │                │                           Confirmed
─────────│────────────────┼─────────────────────────── 
         │                │                           Unconfirmed
   ┌─────┴──────┐         │
   │            │         │
   │    TxD     │         └──┐
   │            │            │
   │   ┌───┐    │            │
   └───└─┬┬┘────┘            │
         │└────────────┐     │
         │             │     │
         │             │     │
   ┌─────┴──────┐   ┌──┴─────┴───┐
   │            │   │            │
   │    TxE     │   │    TxE'    │
   │            │   │            │
   │   ┌───┐    │   │   ┌───┐    │
   └───└───┘────┘   └───└───┘────┘
```

### Not allowed
```
   ┌────────────┐   ┌────────────┐   ┌────────────┐
   │            │   │            │   │            │
   │    TxA     │   │    TxB     │   │    TxC     │
   │            │   │            │   │            │
   │   ┌───┐    │   │   ┌───┐    │   │   ┌───┐    │
   └───└─┬─┘────┘   └───└─┬─┘────┘   └───└─┬─┘────┘
         │                │                │          Confirmed
─────────│────────────────│────────────────│────────── 
         │                └─┐   ┌──────────┘          Unconfirmed
   ┌─────┴──────┐       ┌───┴───┴────┐
   │            │       │            │
   │    TxD     │       │    TxF     │
   │            │  Ok   │            │                                       
   │   ┌───┐    │   │   │   ┌───┐    │    I1 unconfirmed input is allowed    
   └───└─┬┬┘────┘   ▼   └───└─┬─┘────┘    as the output was already existing 
         │└────────────┐      │  Not Ok                                      
         │           I1│   I2┌┘     │     I2 unconfirmed input is not allowed
         │             │     │<─────┘     as I2 funding input was not used   
   ┌─────┴──────┐   ┌──┴─────┴───┐        prior the TxE -> TxE' replacement  
   │            │   │            │
   │    TxE     │   │    TxE'    │
   │            │   │            │
   │   ┌───┐    │   │   ┌───┐    │
   └───└───┘────┘   └───└───┘────┘
```



