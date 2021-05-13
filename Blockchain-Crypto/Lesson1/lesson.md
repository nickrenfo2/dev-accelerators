# Blockchain and Bitcoin - The Basics

While I used several sources as inspiration and references, much of this lesson comes from what I learned from reading _
Bitcoin Clarity_, by Kiara Bickers, which I will link down below. I also took much inspiration from 3Blue1Brown's
youtube video, [But how does Bitcoin actually work?](https://www.youtube.com/watch?v=bBC-nXj3Ng4)

Table of Contents

1. Foreword - Things to know
	1. Hashes
	2. Public/Private Keys
2. Why, or How?
3. Inventing Bitcoin
	1. A shared ledger
		1. Ordering history
		2. Securing transactions
	2. Immutability
		1. Securing a block
		2. Securing the chain
	3. Consensus
		1. Survival of the fittest
		2. Work for it (mining)
	4. Analyzing the solution
4. Things we missed
	1. Nodes
	2. Wallets
	3. Transactions / UTXOs
	4. Merkle Trees
10. References and Relevant Links

## Foreword - Things to know

I won't get into the technicals about these things, but here's what you need to know about certain technologies that the
blockchain relies on.

* Hashes
* Public/private key pairs

### Hashes

A hash is a mathematical function that encrypts some given value into a string of seemingly random characters of a fixed
length. There are many hash algorithms, but we'll focus on SHA-256, the algorithm used on the Bitcoin blockchain (and
most of modern encryption).

A hash function has the following properties

* The output is a fixed length, regardless of the input
* A given input will always result in the same output (deterministic)
* A small change to the input results in a massive change to the output (seemingly random)
* The input cannot be derived from the output (irreversible, "one-way")

Here's an example:

```js
SHA256("Hello, world.") // => "f8c3bf62a9aa3e6fc1619c250e48abe7519373d3edf41be62eb5dc45199af2ef"
SHA256("Hello, world!") // => "315f5bdb76d078c43b8ac0064e4a0164612b1fce77c869345bfc94c75894edd3"
SHA256("Hello")         // => "185f8db32271fe25f561a6fc938b2e264306ec304eda518007d1764826381969"
```

### Public/Private key pairs

Naturally, the "Crypto" part of "Cryptocurrency" comes from "Cryptography." The blockchain makes heavy use of
public/private key pairs and digital signatures. If you're interested in the technical details, look up
[Diffie-Helman key pairs](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange). It's how pretty much all
encryption works these days.

Basically think of public/private key encryption like this:

For a given lock, there are two keys. One key is used exclusively to lock the lock. This key is publicly available to
anyone in the world. The second key is used exclusively to unlock the lock, and this is kept private. Think of an
electronic door lock - Anyone can go up and hit the "lock" button to keep the door locked shut, but only the person with
the door code can unlock the door.

Rather than go into the technicals of how DH key pairs work, here's the properties that are relevant to blockchain

* Private keys are always kept private. They are never shared or communicated to anyone.
	* Private keys are used to digitally sign messages - this proves that a given user has actually authenticated a given
	  message/transaction.
		* A digital signature is unique to each message it signs
	* Private keys are used to "unlock" or "spend" bitcoin.
* Public keys are shared public knowledge. Anyone and everyone can know what each other user's public key is.
	* Public keys are used to "lock" bitcoin,
	* Public keys are used to verify the signature (authenticity) of a given message/transaction

## Why, or How?

One of the main questions people have about Bitcoin and the blockchain is "how" - as in, "how does it work?" A better
question to ask is "why" - as in, "why is it necessary," or "why was it created?" To answer this question, I will begin
by quoting the Genesis Block of Bitcoin (the very first block on the blockchain).

> The Times 03/Jan/2009 Chancellor on brink of second bailout for banks

While this is only one example, it is a prime example of what drove Satoshi Nakamoto to develop bitcoin. He wanted a
currency that was under no one person's control, that instead everyone could participate in freely. The goal was a
system that is both a currency, and a payment processor. A system that would allow peers to send money directly to each
other, without a central authority. A system that no one person could control, and would truly belong to the people.

## Inventing bitcoin

To understand how the blockchain works, let us consider how we might build our own system with the same properties as
Bitcoin, the challenges we will face, and how we might overcome them.

To build a system similar to Bitcoin, let us start with the following rules/properties:

* The ledger must be publicly available
* Anyone can add transactions to the ledger
* Everyone must agree on the ledger's contents
* No one is allowed to spend more money than they have

For the sake of this example, let us assume that Alice, Bob, Charlie, and Donna all put some money in the pot, and the
ledger starts off with each user having 10 BTC. We will discuss later where bitcoins come from, but for now, this will
do.

### Step 1 - A shared ledger

Since the goal is a public ledger, lets start with keeping a history of transactions. That way, we can ensure that each
user actually has the money they say they have, and that they can't double-spend it (they can't hand the same dollar
bill to two different people). Let's take a lump of transactions, and put them together in a **block**. Great, now we've
got a starting point. Whenever we have some new transactions to add to our ledger, we'll create a new block, and add it
on to the end.

With that in mind, here's our initial ledger:

| Transactions |
|---|
|Alice receives 10 BTC|
|Bob receives 10 BTC|
|Charlie receives 10 BTC|
|Donna receives 10 BTC|

### Step 1.1 - Ordering history

But, we've got a problem - these blocks need to be in order. Bob can't send Charlie 12 BTC to purchase a widget until
after Alice pays Bob the 3 BTC she owes him for lunch. So, let's give each block an Index (and each transaction within
the block, for the same reason). Great, now we've got the beginnings of a shared ledger. To fill it out a little more,
lets have Charlie purchase a widget from Donna.

|   | Block 0                 |
|---|-------------------------|
| 0 | Alice receives 10 BTC   |
| 1 | Bob receives 10 BTC     |
| 2 | Charlie receives 10 BTC |
| 3 | Donna receives 10 BTC   |

|   | Block 1                  |
|---|--------------------------|
| 0 | Alice sends Bob 3 BTC    |
| 1 | Bob sends Charlie 12 BTC |
| 2 | Charlie sends Donna 3 BTC|

It is worth mentioning, rather than timestamp each individual transaction, the block as a whole is timestamped, and all
transactions within that block are considered to be happening at that time (in top-to-bottom order).

### Step 1.2 - Securing transactions

Because anybody can add transactions to the ledger, we should make sure that Bob isn't allowed to send money to his own
account on behalf of Alice. That is to say, Bob shouldn't be able to add "Alice sends Bob 10 BTC" to the ledger - only
Alice can add such a message. To that end, we will add a Digital Signature to each transaction, and we will update our
ruleset with the following rule: "Only signed transactions are allowed on the ledger."

* The ledger must be publicly available
* Anyone can add transactions to the ledger
* Everyone must agree on the ledger's contents
* No one is allowed to spend more money than they have
* **Only signed transactions are allowed on the ledger\*.**

(* coinbase transactions do not need a signature)

Here's our updated ledger:

|   | Block 0                 | Signature |
|---|-------------------------|-----------|
| 0 | Alice receives 10 BTC   | N/A       |
| 1 | Bob receives 10 BTC     | N/A       |
| 2 | Charlie receives 10 BTC | N/A       |
| 3 | Donna receives 10 BTC   | N/A       |

|   | Block 1                  | Signature |
|---|--------------------------|-----------|
| 0 | Alice sends Bob 3 BTC    | ALICE     |
| 1 | Charlie sends Donna 3 BTC| CHARLIE   |

|   | Block 2                  | Signature |
|---|--------------------------|-----------|
| 1 | Bob sends Charlie 12 BTC | BOB       |

### Step 2 - Immutability

In order to ensure consistency across time, we need to make sure that no one can go back and change history. Right now,
within the given system, there's nothing stopping Charlie from simply removing his transaction in Block 1 from the
chain, and keeping the BTC he was supposed to send Donna. Right now, we must _trust_ that he won't do that. So let's
find a way to remove that trust requirement - we want to deal with strangers, after all.

### Step 2.1 - Securing a block

We need a way to verify that the contents of a block were not changed. This is where hashing comes in handy. Rather than
give each block an index, lets give it an actual name. Let's calculate that name by putting the contents of the block
into our Secure Hash Algorithm, SHA-256. As we discussed before, any change, even a small one, to the contents of our
block will cause its hash to change. For the purpose of brevity, I'll be using a truncated hash.

| 9a8007 | Block 0                 | Signature |
|---|-------------------------|-----------|
| 0 | Alice receives 10 BTC   | N/A       |
| 1 | Bob receives 10 BTC     | N/A       |
| 2 | Charlie receives 10 BTC | N/A       |
| 3 | Donna receives 10 BTC   | N/A       |

| 8eb412 | Block 1                  | Signature |
|---|--------------------------|-----------|
| 0 | Alice sends Bob 3 BTC    | ALICE     |
| 1 | Charlie sends Donna 3 BTC| CHARLIE   |

| 3098ea | Block 2                  | Signature |
|---|--------------------------|-----------|
| 1 | Bob sends Charlie 12 BTC | BOB       |

Now, we can be sure that if someone tries to retroactively modify a block, that we can compare its hash against the one
we know in order to be sure of its contents. However, this has two significant vulnerabilities. First, it relies on me
having an existing copy of the blockchain to compare against. Second, it doesn't stop anyone from removing a block
entirely.

### Step 2.2 - Securing the chain

Fortunately, these two issues can both be resolved by the same simple solution. Rather than referring to blocks by
index, let's refer to them by their hash. In order to maintain the order of blocks, each block will begin with the name
of the previous block.

| prev &nbsp;&nbsp;&nbsp; | current &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; |
|---|---|

| 000000 | 9a8007                 | Signature |
|---|-------------------------|-----------|
| 0 | Alice receives 10 BTC   | N/A       |
| 1 | Bob receives 10 BTC     | N/A       |
| 2 | Charlie receives 10 BTC | N/A       |
| 3 | Donna receives 10 BTC   | N/A       |

| 9a8007 | 8eb412                  | Signature |
|---|--------------------------|-----------|
| 0 | Alice sends Bob 3 BTC    | ALICE     |
| 1 | Charlie sends Donna 3 BTC| CHARLIE   |

| 8eb412 | 3098ea                  | Signature |
|---|--------------------------|-----------|
| 1 | Bob sends Charlie 12 BTC | BOB       |

Now, if someone tries to manipulate or remove a block, the entire chain will break.

### Step 3 - Consensus

We've come a long way from the start, but we still have one important question - how do we get everyone to agree on the
same chain? In order for this to work, we need to have everyone agree on what transactions belong in which block and in
what order, and which blocks come before or after which other blocks. We've established a way to verify the
_validity_ of block and the _validity_ of the order, but not on how to _agree_ on such blocks or such order.

### Step 3.1 - Survival of the Fittest

At present, if Alice adds a couple blocks containing some set of transactions (6899be, and 37135d), and at the same
time, Donna adds a block containing some other set of transactions (20a6d6), we'll have a fork - two separate chains,
each with different endings:

|   | Alice's Chain | Donna's Chain |
|---|---------------|---------------|
| 0 | 9a8007        | 9a8007        |
| 1 | 8eb412        | 8eb412        |
| 2 | 3098ea        | 3098ea        |
| 3 | **6899be**    | **20a6d6**    |
| 3 | **37135d**    |               |

Let's add one more rule to our list: Only the longest chain survives.

* The ledger must be publicly available
* Anyone can add transactions to the ledger
* Everyone must agree on the ledger's contents
* No one is allowed to spend more money than they have
* Only signed transactions are allowed on the ledger\*.
* **Only the longest chain survives**

This will help us to agree on which fork to use, but only a little.

### Step 3.2 - Work for it (mining)

If someone wanted to control the blockchain at this point, all it would take is endlessly adding blocks faster than
anyone else. The person who could do this would always be in control of which transactions did and did not make it onto
the public ledger. They could also go back in time and remove existing transactions from blocks, and then recalculate
the rest of the blockchain so as to once again become the longest chain.

So, let's make it difficult to add blocks onto the ledger. The concept of Proof of Work has actually been around for
quite some time, dating back to 1993, originally conceptualized as a way to fight email spam. The concept is this - we
need to design a puzzle that is computationally difficult to solve, but very easy to check. For example, a Rubik's Cube
with 10,000 faces on each side is difficult to solve, but once it has been solved, the solution is easy to check (all
faces have the same color). In cryptographic terms, we call this a "Proof of Work." In order to solve the puzzle, you
must expend some amount of computational resources, in order to impose a cost.

So, let's put a new rule in place: Blocks must be "solved" before being added to the chain.

* The ledger must be publicly available
* Anyone can add transactions to the ledger
* Everyone must agree on the ledger's contents
* No one is allowed to spend more money than they have
* Only signed transactions are allowed on the ledger\*.
* Only the longest chain survives
* **Only "solved" blocks are allowed**

Before proceeding, we need to define what a "solved" block looks like. This is where our friend SHA-256 comes in handy
once again. A single hash is fairly easy to compute, but millions (or billions, trillions, etc.) of them are much more
difficult to compute. Since a block's name is a hash of all its contents, lets add a new field to our block called
"nonce" (number used only once), and say that a block is considered "solved" when its hash begins with 10 0's. This
field is important, because without it, there would be no way to solve the block. This field allows us to essentially
make the block's name anything we want. One small change to this nonce gives the block a completely new name, and this
field's only purpose is to give us a way to change the name of a block without really changing its content. The process
of solving a block is called "mining."

With this new restriction, we can be sure that anyone adding a block to the blockchain has gone through some real-world
expense to do so, and we can verify that expense easily. While this doesn't ensure that some specific cost has been
incurred on every block, it is a good average. Attempting to manipulate the blockchain becomes a very expensive venture,
as you would have to consistently outperform all other participants.

Furthermore, we can easily adjust the difficulty of adding a new block by requiring more or fewer 0's at the beginning
of a block's name. It may be worth noting that choosing "0" as a signal is entirely arbitrary, we could have chosen any
value.

The Bitcoin protocol is designed to target an average of adding 1 block every 10 minutes. The difficulty of how many 0's
are required is adjusted every 2016 blocks (about 2 weeks) based on the current overall hash rate of the network.

Here's what our block chain looks like now:

|Prev block | Current Block | Nonce |
|---|---|---|

| 000000 | 000...234be2             | 1a2a3c    |
|---|-------------------------|-----------|
| 0 | Alice receives 10 BTC   | N/A       |
| 1 | Bob receives 10 BTC     | N/A       |
| 2 | Charlie receives 10 BTC | N/A       |
| 3 | Donna receives 10 BTC   | N/A       |

| 234be2 | 000...bb608c              | 4d5e6f    |
|---|--------------------------|-----------|
| 0 | Alice sends Bob 3 BTC    | ALICE     |
| 1 | Charlie sends Donna 3 BTC| CHARLIE   |

| bb608c | 000...2934df              | 7g8h9i    |
|---|--------------------------|-----------|
| 1 | Bob sends Charlie 12 BTC | BOB       |

### Analyzing the Solution

And that's basically it. We've invented Bitcoin. There's a lot more that goes on, technically speaking, but that's the
big picture of how "The Blockchain" works in the context of Bitcoin. We've created a system where

* Each person can keep their own copy of the ledger
* Each person can broadcast their own transactions
* Everyone can agree on the order of transactions
* Each person can individually verify the validity and authenticity of each transaction
* Each person can individually verify the validity and authenticity of each block
* Each person can individually verify the validity of the chain as a whole
* No trust of any centralized authority is required
* No centralized authority (or individual participant) has the ability to control or censor the network

### Things we missed

While that was most of the big picture stuff, I glossed over a lot of details.

* Nodes
* Wallets
* Transactions / UTXOs
* Confirmations
* Merkle Trees

---


## Nodes

Originally, all parts of Bitcoin were done by the same software, running on CPUs, and these computers were called Nodes.

* Wallet application
* Consensus Rules - verifying block validity, create (not solve) new blocks
* P2P Network - connect to other bitcoin nodes, listen/send transactions
* Mining - solve new blocks


Over time, people realized that Graphics Processing Units (GPUs, or Graphics Cards) were significantly more 
effective at mining than CPUs, so miners began to write software meant only mine, and to utilize the maximum 
capabilities of GPU processing for hashing. So mining was split off as a separate task from the rest of the node.

Nodes (CPU):
* Wallet
* Consensus
* P2P

Miners (GPU)
* Mining

Eventually, even GPUs were not good enough for the miners. Now, they use something called an ASIC - Application 
Specific Integrated Circuit. Basically, took all the benefits of a GPU (tons of tiny processors), stripped out the 
general-purpose compute power, and maximized efficiency towards hashing. This hardware is better than GPUs by about 
a millionfold. They are also more energy-efficient for hashing than GPUs.


While this is generally still the model, there also now exist Light Nodes, which are the same thing as Full Nodes,
but they only perform partial-consensus (verify _your_ transactions, rather than _all_ transactions). These are more 
performant, but also since they do not verify the entire blockchain, still require some level of trust. They are 
typically used for mobile wallet applications.

---
---

## Wallets

What is a Bitcoin wallet, and does it work the same way as a regular billfold?

The term wallet is a good analogy, but there are some important differences between physical wallets and bitcoin
wallets. With a physical wallet, when you put dollar bill in there, you physically hold that dollar. With a bitcoin
wallet, you really only ever hold _access_ to any bitcoin you may have. It's a little closer to a bank account than a
wallet - you don't physically posses the dollars in your account, but you do retain access to them, namely, the ability
to withdraw or spend those dollars. Consider this - if you cross a border with a cash wallet, have you brought money
across that border? Yes, definitively. On the other hand, if you cross a border with a bitcoin wallet, has the bitcoin
actually crossed that border? Not really. In some sense, it's like asking if your Facebook status has crossed the
border. The question doesn't really make sense, because it exists digitally, everywhere. If you cross a border, does all
the money in your bank account move with you? No, not really.

### What does wallet software do?

Wallet software is what allows you to generate a bitcoin wallet, receive funds, sign and broadcast transactions, and
depending on the software you use, it may also download the whole blockchain and verify its validity. Wallet software
typically comes in the form of an app on the app store for mobile devices, or as a downloadable application for laptop
or desktop computers.

### Creating a wallet

Moving beyond the philosophical implications of where data exists, it is generally appropriate to consider a bitcoin
wallet a key. In fact, that's exactly what it is. When you generate a bitcoin wallet, what you're really doing is
generating a private key. Here's how it works

1. You click "Generate new wallet" within your wallet software
2. A backup Seed Phrase is generated (generally a dozen words)
	1. This seed phrase is used to re-create your private key if you ever lose it. KEEP THIS SAFE AND SECURE.
3. The Seed Phrase is used to generate a cryptographic Private Key
	1. This key is how you access your bitcoin. Controlling this is analagous to possessing a cash wallet, and the money
	   it contains.
4. The Private Key is then used to generate a Public Key
5. The Public Key is then used to generate a Wallet Address
	1. This is how you receive funds, addresses are public
	2. A public key can generate many wallet addresses

### Securing your wallet

As mentioned above, possession of your private keys is tantamount to ownership of your bitcoin. If someone else can
access your private keys, they can sign transactions on your behalf. For this reason, securing your private keys is
particularly important. Usually, this comes in the form of storing them as a file on a secure flash drive, or using a
hardware wallet (explained below).

If your keys become compromised, you may be able to secure your bitcoin by quickly generating a new bitcoin wallet
(and thus a new set of private keys), and sending all of your balance from your old wallet to your new wallet. As long
as you do this before an attacker does, you can safely secure your bitcoin at only the cost of a miner's fee.

### Types of wallets

There are 3 types of wallets to manage your bitcoin - custodial wallets, software wallets, and hardware wallets.

#### Custodial wallets

Custodial wallets are the kind used on trading exchanges like Coinbase, Binance, etc. It is where your private keys are
actually managed not by you, but by some trusted third party (i.e. Coinbase, Binance, etc). You access your account
balances and manage transactions not by unlocking/signing them with your private keys, but by a username and password,
and sending a signal to the custodian of your wallet to sign a transaction on your behalf.

The trade-offs of custodial wallets:

|pros|cons|
|---|---|
|Easy to use | Requires trust |
|Good for high trade volume when exchanging between currencies | No privacy |
| | Security is determined by custodian|
| | _**You don't control your own bitcoin**_ |

### Software wallets

The next type of wallet is a software wallet. While technically speaking all wallets are software wallets, the term is
generally applied to a wallet that you control, rather than a custodial wallet. These wallets are typically apps from
the app store, or downloaded applications from a website. There are many different wallet softwares out there, each with
various features. Examples include Electrum, Bitcoin Core, Wasabi, and Samourai.

Trade-offs of software wallets:

|pros|cons|
|---|---|
|Fairly easy to use| Keys are only as secure as you can keep them|
|Free | |
|Control your own bitcoin| |
|Wide array of features | |
|More privacy| |

### Hardware wallets

The third type of wallet is a Hardware wallet. These take the form of a dedicated hardware device (not just a flash
drive) where the private keys are generated and stored, with the purpose of keeping them air-gapped from any computer.
To sign a transaction, you use some wallet software to generate the transaction (send X amount of BTC to Y address),
communicate the unsigned transaction to the hardware wallet (USB, MicroSD, etc), and the device will sign the
transaction, and then communicate the signed transaction back to the wallet software in order to verify and broadcast.
In this way, even a computer infected with malicious software would not be able to collect your private keys - they
never leave the device. These are the most secure, and often require a PIN code before signing transactions, and have a
wide array of security features to ensure that even if someone acquires your hardware wallet, only you can sign a
transaction. Examples include ColdCard, OpenDime, Trezor, Ledger, and KeepKey.

Trade-offs of hardware wallets

* More difficult to use
* Control your own bitcoin
* Expensive
* Keys are super secure
	* Make sure not to lose it

|pros|cons|
|---|---|
|Control your own bitcoin| More difficult to use|
|Keys are super secure | Expensive |

---
---

## Transactions, and UTXOs

So how is a person's bitcoin "balance" calculated? We can determine this by calculating the value of any Unspent
Transaction Outputs (UTXOs) that point to them. This works in essentially the same way as your bank will determine what
your account balance is, with some minor technical differences. Let's compare the two. For the purpose of this lesson,
let us assume that 1 BTC = $100

---

When Alice opens an account, she deposits a certain amount of cash, say $1000. Alice also decides to invest in Bitcoin,
starting off with 10 BTC.

#### Classical bank:

After alice hands $1000 to a clerk, the bank will open an account in Alice's name, and add a record describing $1000
being added to Alice's account.

|Account Name | Amount | Notes | Available Balance |
|---|---|---|---|
|Alice|1000|Initial Deposit| 1000 |

Alice now has one "Unspent Transaction Output" (UTXO) in the amount of $1000 at her disposal.

#### Blockchain:

After Alice hands him some cash, Bob sends 10 BTC to Alice

|Sender|Recipient|Amount|
|---|---|---|
|0xb0345 (Bob)|0xa1234 (Alice)|10|

Again, Alice now has one UTXO in the amount of 10 BTC at her disposal.

---


Alice then spends $50 at Target. This is where things start to differ between classical banking and the blockchain.
While the conclusion is the same, there are some very important differences in the nature of how these records are kept.

#### Classical bank:

Here, the bank will describe a record in the amount of "-$50," and calculate her Available Balance by adding up the
"+1000" and the "-50."

|Account Name | Amount | Notes | Available Balance |
|---|---|---|---|
|Alice|1000|Initial Deposit| 1000 |
|Alice|-50|Purchase at Target| 950 |

Target will then independently record a transaction of "+$50." For now, we'll focus only on the bank's records, though
consider now how significant it is that both Target and the Bank are keeping accurate transaction records, regarding the
total Money Supply of USD. While there are built in validation and correction measures in this form of accounting, these
records are still ultimately susceptible to both human error and intentional corruption. Does anyone even know how many
USD there are, or where they are?

#### Blockchain:

On the blockchain, such a transaction would take the form of Alice "spending" the entire output of the previous
transaction, and sending part of the output to Target, and the rest of the output to herself as a "change output."

|Sender|Recipient|Amount|
|---|---|---|
|0xb0345 (Bob)|0xa1234 (Alice)|10|
|0xa1234 (Alice)|0xdb902 (Target), 0xa4321 (Alice)|0.5, 9.5|

Notice how there are multiple recipients for the single transaction. This means that there are now two Unspent
Transaction Outputs (UTXO) on this ledger. The entirety of Alice's first transaction (10BTC) has been spent, but she now
has a new UTXO of 9.5 BTC (a 'change output'), representing the "unspent" portion of her original 10BTC, or the
'change'. The second UTXO belongs to Target.

This is similar to handing the cashier $1000, and getting back $950 - the original $1000 has been 'spent', but you
receive back the 'unspent' portion in the form of new (to you) dollar bills.

If you look carefilly, you will also see that the recipient address for Alice's change output is different than her
initial address. This is because when sending bitcoin to a wallet address, a single-use transaction address is
generated. In this way, you can trace each specific transaction output, and determine which have been spent, and which
haven't. If a transaction output is used as an input to another transaction, then you know that it has been spent.

Additionally, notice now how we are no longer relying on Target to keep accurate records of their accounting - everyone
participating on the bitcoin network can verify the amount of coin that Target has received and spent. It also means
that in order to calculate the balance of your account/wallet (or any entity's balance), you need to look through the
entire blockchain.

---

Using the materials she purchased at Target, Alice produced a widget, and sold it to Bob for $100. Let's compare ledgers
again:

#### Classical bank:

|Account Name | Amount | Notes | Available Balance |
|---|---|---|---|
|Alice|1000|Initial Deposit| 1000 |
|Alice|-50|Purchase at Target| 950 |
|Alice|100|Deposit from Bob| 1050 |

A single new entry has been added representing a $100 increase, and Alice's new balance is calculated.

#### Blockchain:

For clarity, I will rename the "sender" and "recipient" fields to "Input" and "Output," respectively. This is a more 
technical description of the fields.

|Input|Output|Amount|
|---|---|---|
|0xb0345 (Bob)|0xa1234 (Alice)|10|
|0xa1234 (Alice)|0xdb902 (Target), 0xa4321 (Alice)|0.5, 9.5|
|0xb80c4 (Bob)|0xa6c54 (Alice)|1|


Again, a new recipient address is visible from Alice (and a new sender address from Bob, as well). Now Alice has two 
UTXOs - one from Bob for 1 BTC, and 1 from herself (change output) for 9.5 BTC. Her total available balance is the 
sum of all UTXOs - so 10.5 BTC.

---

Finally, the new TV that Alice wanted is on sale for $1000, so she goes and buys it. Let's compare ledgers once more:


#### Classical bank:

|Account Name | Amount | Notes | Available Balance |
|---|---|---|---|
|Alice|1000|Initial Deposit| 1000 |
|Alice|-50|Purchase at Target| 950 |
|Alice|100|Deposit from Bob| 1050 |
|Alice|-1000|Purchase at Best Buy| 50 |

A single new entry has been added representing a $1000 decrease, and Alice's new balance is calculated.

#### Blockchain:

|Input|Output|Amount|
|---|---|---|
|0xb0345 (Bob)|0xa1234 (Alice)|10|
|0xa1234 (Alice)|0xdb902 (Target), 0xa4321 (Alice)|0.5, 9.5|
|0xb80c4 (Bob)|0xa6c54 (Alice)|1|
|0xa6c54 (Alice) 0xa4321 (Alice)| 0xbb901 (Best Buy) 0xac103 (Alice)|10, .5|

Just as there can be multiple recipients (outputs) in a transaction, there can be senders (inputs) as well. And in 
the same way, the total sum of the inputs become spent, and new outputs are generated. Now Alice has only one UTXO 
in the amount of .5 BTC


This system of tracking transactions allows the entire ledger to be publicly accessible and verifiable. You can 
always know what someone's balance is before they purchase something from you, and you can be certain that their 
"checks" won't bounce.

---
---

## Confirmations

After a user broadcasts their transaction to the bitcoin network, it still hasn't been "confirmed" yet - that is to 
say, it doesn't appear on the ledger (the blockchain), so in some sense, it hasn't really "happened" yet. The way 
for a transaction to be confirmed is to be included in a block that is then solved and added to the blockchain. Each 
new block that is then considered another "confirmation" - you typically want to wait for about 4-6 confirmations 
before trusting a transaction, possibly longer for larger transactions. This is because in the short term, there may 
be competing forks of the blockchain - two different miners solving blocks at about the same time, potentially with 
different transactions. If you don't wait for additional confirmations, then its possible you may lose your 
transaction if another miner solves multiple blocks very quickly, and those blocks contain different transactions.

---
---

## References and Relevant Links

* _Bitcoin Clarity_, by Kiara Bickers -
  on [Amazon](https://www.amazon.com/Bitcoin-Clarity-Complete-Beginners-Understanding/dp/1733871209/)
* [But how does Bitcoin actually work?](https://www.youtube.com/watch?v=bBC-nXj3Ng4), 3Blue1Brown
