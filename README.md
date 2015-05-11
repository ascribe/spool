# SPOOL

<pre>
  Title: SPOOL Protocol (SPOOL/1.0)
  Author: Dimitri de Jonghe (dimi@ascribe.io) and Trent McConaghy (trent@ascribe.io)
  First public release: 2015-03-14
  Current public release: 2015-05-06
</pre>

## Introduction
SPOOL is:
* a protocol for documenting transactions relating to ownership of digital property, i.e. copyright rights
* an overlay of the bitcoin blockchain
* SPOOL = Secure Public Online Ownership Ledger

SPOOL is not:
* The law. SPOOL must be used with a contract concerning transfer of copyright rights that participants have agreed to. E.g. the ascribe Terms of Service (TOS).

SPOOL transactions include
* Register ownership. Supports >1 unique editions, e.g. edition 1/10, 2/10, …, 10/10.
* Transfer ownership
* Consign, for someone else to sell on your behalf
* And more

SPOOL + TOS = copyright on the blockchain.

Useful for digital art, photography, and other media.

## Visual Description

**[Here](https://s3-us-west-2.amazonaws.com/ascribe0/public/2015-05-07-copyright-on-the-blockchain--spool-protocol.pdf)** is a full slide-based description. It also shows this protocol's place in an overall software stack.

The key points are:

![alt tag](https://s3-us-west-2.amazonaws.com/ascribe0/public/ascribe_spool_register_editions.png)
![alt tag](https://s3-us-west-2.amazonaws.com/ascribe0/public/ascribe_spool_transfer.PNG)

## Protocol Description
Summary: Via OP_RETURN we attach verbs to the blockchain tx, such as Register, Consign, Transfer, Loan, ....

Declarations
- ```FEE```: mining fee based on transaction size (minimum ```10000 satoshi```)
- ```DUST```: minimum size of an output to be accepted by the network (currently ```600 satoshi```)
- ```FEDERATION```: A trusted wallet that acts as a federated agency, e.g. ```1AScRhqdXMrJyxNmjEapMZi1PLFsqmLquG```
- ```FUEL```: A wallet that is used to provide enough bitcoin for subsequent interations
- ```META```: Header for SPOOL protocol, e.g. ```<protocol><version> = ASCRIBESP00L01```

## Federated SPOOL actions
A federated action (e.g. "register") associates a user with digital content. Hereto, a trusted federation wallet sends a transaction to both an address owned by the user and an address that represents a base58 hash of the digital content. Note that the wallet with the digital content address is not required to be owned by the user.

### Associate ###
General association action (e.g. "change password") to attach content to a wallet
- Mapping: ```1-to-1```
- PySPOOL: ```pyspool.Association(FEDERATION, address, hash, reason=<string>)```
- Bitcoin:
```
TX = [(FEDERATION : FEE+DUST] 
     -> 
     [(hash : DUST), 
      (address : DUST), 
      (OP_RETURN=<META>ASSOCIATE<reason> : 0), 
      (FEE)]
> balance of hash = DUST
> balance of address = DUST
```

### Register Editions ###
Associate N editions of the same content to a wallet with N addresses
- Mapping: ```1-to-many ```
- PySPOOL: ```pyspool.Registration(FEDERATION, editions=[edition1_address, ..., editionN_address], hash)```
- Bitcoin:
```
TX = [(FEDERATION : FEE+(1+<num_editions>)*DUST] 
     -> 
     [(hash : DUST), 
      (edition1_address : DUST), 
      ..., 
      (editionN_address : DUST), 
      (OP_RETURN=<META>REGISTER<num_editions> : 0), 
      (FEE)]
> num_editions = len(editions)
> balance of piece_hash = DUST
> balance of editionX_address = DUST
```

### Migrate Content ###
Associate two wallets, where the address does not represent hashed content. Usefull for password reset and other user/content updates.
- Mapping: ```1-to-1 ```
- PySPOOL: ```pyspool.Migration(FEDERATION, from_address, to_address, reason=<string>)```
- Bitcoin:
```
TX = [(FEDERATION : FEE+2*DUST] 
     -> 
     [(from_address : DUST), 
      (to_address : DUST), 
      (OP_RETURN=<META>MIGRATE<reason> : 0), 
      (FEE)]
> balance of from_address = DUST + previous_balance
> balance of to_address = DUST
```

### Fuel wallet ###
Fuel a wallet with enough bitcoin to 
- Mapping: ```1-to-1 ```
- PySPOOL: ```pyspool.Fuel(FUEL, address)```
- Bitcoin:
```
TX = [(FUEL : 2*FEE] 
     -> 
     [(address : FEE), 
      (OP_RETURN=<META>FUEL : 0), 
      (FEE)]
> balance of address = FEE + previous_balance
```

## User-to-user interactions
An interaction transfers rights, ownership or smart contracts between users.

### Transfer ownership ###
Transfer the ownership of registered content
- Mapping: ```1-to-1 ```
- SPOOL: ```pyspool.Transfer(from_address, to_address)```
- Dependencies: ```Fuel, Register, optional: Migrate, Consign```
- Bitcoin:
```
TX = [(from_address : DUST+FEE)] 
     -> 
     [(to_address : DUST), 
      (OP_RETURN=<META>TRANSFER : 0), 
      (FEE)]
> balance of from_address = 0
> balance of to_address = DUST
```

### Consign ###
Assign the rights to transfer content to another user
- Mapping: ```1-to-1 ```
- SPOOL: ```pyspool.Consign(from_address, to_address)```
- Dependencies: ```Fuel, Register, optional: Migrate, Transfer```
- Bitcoin:
```
TX = [(from_address : DUST+FEE)] 
     -> 
     [(to_address : DUST), 
      (OP_RETURN=<META>CONSIGN : 0), 
      (FEE)]
> balance of from_address = 0
> balance of to_address = DUST
```

### Unconsign ###
Return the rights to transfer the content back to the owner
- Mapping: ```1-to-1 ```
- SPOOL: ```pyspool.Unconsign(from_address, to_address)```
- Dependencies: ```Fuel, Consign, optional: Migrate```
- Bitcoin:
```
TX = [(from_address : DUST+FEE)] 
     -> 
     [(to_address : DUST), 
      (OP_RETURN=<META>UNCONSIGN : 0), 
      (FEE)]
> balance of from_address = 0
> balance of to_address = DUST
```

### Loan ###
Assign the right to display or use the content for a period of time
- Mapping: ```1-to-1 ```
- SPOOL: ```pyspool.Loan(from_address, to_address, start_date, end_date)```
- Dependencies: ```Fuel, Register, optional: Migrate```
- Bitcoin:
```
TX = [(from_address : DUST+FEE)] 
     -> 
     [(to_address : DUST), 
      (OP_RETURN=<META>LOAN<start_date><end_date> : 0), 
      (FEE)]
> balance of from_address = 0
> balance of to_address = DUST
```

### Share, unshare ###
Don't put on blockchain.

## Background
SPOOL was developed by ascribe GmbH as part of the overall ascribe technology stack. http://www.ascribe.io

## Copyright

This document has been placed in the public domain.
