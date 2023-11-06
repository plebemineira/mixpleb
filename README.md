# Mixpleb: A Bitcoin sidechain for incentivized mixnets

Authors: 
`afm <afm2@duck.com>`
`plebhash <plebhash@proton.me>`

**Abstract.** Bitcoin can be used to create incentives for privacy providers of message-based applications and services. Mixpleb creates a network of mix-nodes that obfuscates traffic to protect users from metadata-based de-anonymization techniques. Mixpleb is also a federated sidechain with inspiration from Blockstream's Liquid, providing a 2-way peg to Bitcoin and carries the State Transition Function for staking, credential issuance and reward distribution. Mixpleb design is heavily inspired by NYM, while explicitly avoiding the introduction of a new token.

## Introduction

Chaum introduced the concept of mixnet in 1981 [1]. Similarly to Tor, a mixnet consists of an overlay network of mixnodes that routes messages anonymously. Each Sphinx packet is encrypted multiple times and sent through a sequence of mixnodes. Each mixnode removes a layer of encryption, until the last mixnode routes the message to its final destination. Each individual packet is routed independently. Differently to Tor, mixnets are designed to protect traffic from metadata-based de-anonymization attacks from global network adversaries. Aside from reordering packets on the temporal dimension, mixnodes also transform them cryptographically. Resulting traffic is untraceable in terms of metadata and timing. 

On one hand, Mixpleb is inspired by Nym [4], in that it tries to be a general-purpose incentivized mixnet architecture. On the other hand, Mixpleb avoids NYM's drawback (an independent blockchain with its own native token) by using Blockstream's Liquid concepts of federated 2-way-peg (2WP) sidechain.

## Federated Mixpleb Sidechain

As a sidechain of Bitcoin, Mixpleb provides a 2WP with the Bitcoin blockchain. The amount of mBTC circulating on Mixpleb is always verifiably equal to the amount of BTC locked on the Bitcoin blockchain. 

mBTC is created when BTC is moved to the Mixpleb sidechain, and destroyed when BTC is moved back the original blockchain.

The Federated Mixpleb Sidechain design is heavily inspired by Liquid Network [1].

### Federation

The Mixpleb federation is composed of different entities that have some vested interest in privacy preserving infrastructure for Bitcoin.

The federation performs tasks that are essential to the Mixpleb sidechain operation, and are responsible for running the nodes that execute the Byzantine Fault Tolerant consensus along with the State Transition Function (STF).

### Functionary

Functionaries are a subset of the Mixpleb federation. They perform two roles:
- Take turns proposing and signing new Mixpleb blocks.
- Manage the 2WP while securing the BTC held on the federation's multisig wallet.

### Peg-Out Services

A peg-out requires two confirmations on the Mixpleb sidechain, after which the BTC can be sent from the Mixpleb Federation's mainchain multisig wallet to a whitelisted address held by the federation member performing the peg-out.

Mixpleb federation members can also provide peg-out services.

### Mixpleb Infrastructure

Mixpleb enables privacy-preserving applications and services. Users have cryptographic guarantees that their communications will not be de-anonymized by proving the "right to use" these services. mBTC is used to fund operational costs, dynamically scale the mixnet to meet bandwidth demand, and reward mixnodes for their work.

The Mixpleb infrastructure is composed of three types of nodes: 
- Mixnodes: enable communication privacy by relaying data packets anonymously. 
- Gateways: make the mixnet accessible while checking for bandwidth crendials (avoid free-riding).
- Sidechain nodes: act as clients for the Mixpleb sidechain. They allow for submission of Mixpleb transactions, issue bandwidth credentials to users, and keep track of the sidechain state.

The information included in the sidechain **State** consists of:
- participant's stake on mixnodes (reputation system)
- the set of active mixnodes
- the mixpool

The sidechain's **State Transition Function** (STF) allows for:
- users to add or remove stake on mixnodes
- users to swap tokens for bandwidth credentials
- mixnodes to swap "proof of mixing" for token rewards from the mixpool (as proposed by NYM [3])
- miners to rotate the set of active mixnodes

In practice, Bandwidth credentials can be implemented as Coconut Credentials (as proposed by NYM [3]).

### Mixnode Reputation

What if a mixnode goes malicious? Or if they are providing low quality service? Mixpleb establishes a reputation system for mixnodes. The set of active mixnodes is finite, and its size is determined by market demand for bandwidth. Mixnode operators and general users stake mBTC on mixnodes as an attestation of their trust in such mixnode.

The more mBTC is staked in a single mixnode, the higher is their reputation, and they're more likely to be (randomly) picked for the active mixnode set.

A Mixnode's quality of service (QoS) is objectively measured in a fair and decentralized manner by leveraging the "proof of mixing" scheme proposed by NYM [4].

Mixnodes that misbehave or provide low QoS will eventually lose their reputation because users will move their stake to other mixnodes.

Every epoch (`X` sidechain blocks), the set of active mixnodes is renewed. Mixnodes above a reputation threshold are eligible, and the miner determines the new active set based on a Verifiably Random Function (VRF).


### Mixpleb Usage

1. User deposits mBTC to mixpool on the sidechain, getting a bandwidth credential in return.
2. User provides bandwidth credential to gateway.
3. Gateway publishes a credential commitment to the sidechain (avoid double spending of bandwidth credential). 
4. User and the gateway establish a shared secret and a temporary pseudonym associated to the bandwidth allowance and validity period represented by the credential.
5. User encrypts their data with the shared secret and sends it to the gateway. The pseudonym is used by the gateway to keep track of bandwidth consumption.
6. Each mixnode mixes the data and moves it to the next layer.
7. The last mix node sends the message to the recipient’s gateway.
8. Mixpool rewards are distributed on every sidechain block. Rewards are destined to mixnodes and gateways.

## Implementations

Mixpleb took initial inspiration from `NoTrustVerify`'s Oheka's work on routing Nostr traffic over NYM [4]. A Proof of Concept translation of the original Golang codebase [5] to Rust [6] was performed by the authors to gain familiarity with the topic.

## Conclusion

In conclusion, Mixpleb integrates the robust privacy-preserving capabilities of mixnets with the financial incentives and security of Bitcoin. The federated sidechain approach, inspired by Blockstream's Liquid, offers a seamless 2-way peg to Bitcoin, fostering a decentralized environment where mBTC serves as both a utility and a reward token within the ecosystem.

Our design addresses the limitations of previous systems by avoiding the need for a native token, and the federated sidechain plays a role in maintaining the network's integrity.
The reputation system for mixnodes ensures that only trustworthy and high-performing nodes are selected, fostering a self-regulating network where quality of service is paramount.
As we move towards an era where privacy is increasingly under threat, Mixpleb expectes to offer a scalable and incentivized solution in alignment with Bitcoin's principles.

## References

1. [David Chaum. Untraceable electronic mail, return addresses, and digital pseudonyms. Commun. ACM, 24(2):84–
88, 1981.](https://chaum.com/wp-content/uploads/2022/09/UNTRACEABLE-ELECTRONIC-MAIL-RETURN-ADDRESSES-AND-DIGITAL-PSEUDONYMS-tech-report.pdf)
2. [Liquid: A Bitcoin Sidechain](https://blockstream.com/assets/downloads/pdf/liquid-whitepaper.pdf)
3. [Ania M Piotrowska, Jamie Hayes, Tariq Elahi, Sebastian Meiser, and George Danezis. The Loopix anonymity
system. In 26th USENIX Security Symposium (USENIX Security 17), pages 1199–1216, 2017.](https://www.usenix.org/conference/usenixsecurity17/technical-sessions/presentation/piotrowska)
4. [Claudia Diaz, Harry Halpin, and Aggelos Kiayias. Nym Whitepaper. 2021.](https://nymtech.net/nym-whitepaper.pdf)
5. [https://github.com/notrustverify/nostr-nym](https://github.com/notrustverify/nostr-nym)
6. [https://github.com/a-moreira/nymstr](https://github.com/a-moreira/nymstr)
