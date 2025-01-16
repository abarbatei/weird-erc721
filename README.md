# Weird ERC721 Tokens

This repository contains examples, in Solidity, of ERC721 (NFTs) tokens with behavior that may be surprising or unexpected. 

The behavior mentioned in this repo is based, as much as possible, on real-world tokens and implementations. However, some theoretical corner cases are also mentioned where no concrete examples have been identified (to the author's knowledge).

The goal of this repository is to serve as a valuable resource for developers and auditors, similar to how it's predecessor [weird-erc20](https://github.com/d-xo/weird-erc20) has been.

While the ERC721 specification is well-defined, over the years, certain non-standard behaviors have appeared. These often arise from combining ERC721 tokens with other standards in novel or unintended ways.

If you are developing an ERC721 token, consider this repository a guide to potential pitfalls and behaviors to approach with caution. If you are building a protocol that interacts with ERC721 tokens, you are **strongly** advised to account for these cases or, at the very least, define which scenarios your product intends to support.

ERC721 resources:
- Official ERC721 documentation: https://eips.ethereum.org/EIPS/eip-721
- RareSkills ERC721 guid: https://www.rareskills.io/post/erc721 

# ERC721 Tokens

## 1️⃣ Simultaneous ERC721 and ERC1155 Standards

Some NFT collections are both ERC721 and ERC1155 at the same time. Protocols that automate transfers by attempting to detect if an asset is an ERC721 or ERC1155 may result in an incorrect number of assets being transferred.

_examples_:
- [`Sandbox Game Asset`](https://etherscan.io/address/0x7fbf5c9af42a6d146dcc18762f515692cd5f853b#code#F2#L1047) — an [example integration issue](https://solodit.cyfrin.io/issues/h-06-some-real-world-nft-tokens-may-support-both-erc721-and-erc1155-standards-which-may-break-infinityexchange_transfernfts-code4rena-infinity-nft-marketplace-infinity-nft-marketplace-contest-git)
- [`F1® Delta Time`](https://etherscan.io/token/0x2af75676692817d85121353f0d6e8e9ae6ad5576#code#L973) — old [Animoca collection](https://www.animocabrands.com/f1-delta-time-to-cease-operations-announces-rewards-for-supporters)

## 2️⃣ Wrapped ERC721 and Old NFT Collections

Before the ERC721 standard was fully established, many NFT collections were created without adhering to a standardized framework. As demand grew for trading these legacy collections within the current ERC721 ecosystem, wrapper solutions emerged. Holders would deposit the original non-compliant NFT and receive an ERC721 wrapper NFT which can be redeemed for the original NFT.

_examples_: 

- [`CryptoPunks`](https://cryptopunks.app/) — the wrapped versions can be traded here [Wrapped CryptoPunks](https://opensea.io/collection/wrapped-cryptopunks)
- [`CryptoKitty`](https://opensea.io/collection/cryptokitties) — some collections, such as `CryptoKitty`, were wrapped to `ERC20` tokens
- [`CryptoFighter`](https://opensea.io/collection/cryptofighters) — collections like `CryptoFighters`, which are still in circulation but are not fully ERC721 compliant and are not wrapped, create issues for any integrating protocol

_More information on old NFTs can be found by joining the [NFT Relics community](https://x.com/nft_relics)_

## 3️⃣ Multiple NFT Collections On the Same ERC721 Contract

An NFT collection is usually tied to a single instances of an ERC721 contract. There are cases, however, where multiple collections are on the same ERC721 contract.

Of special importance for users is that, giving approval for one collection via `ERC721::setApprovalForAll` will actually give allowance over all NFTs of the underlying ERC721 contract.

_examples_:

- both the [CyberKongz (Babies)](https://opensea.io/collection/cyberkongz-babies) and OG [CyberKongz](https://opensea.io/collection/cyberkongz) are [on the same contract](https://etherscan.io/address/0x57a204aa1042f6e66dd7730813f4024114d74f37).
- [@punk6529](https://x.com/punk6529)'s [NextGen contract](https://etherscan.deth.net/address/0x45882f9bc325E14FBb298a1Df930C43a874B83ae), which is specifically [designed for multiple collections on the same contract](https://seize-io.gitbook.io/nextgen/nextgen-smart-contracts/features). It currently has only [one collection mapped](https://6529.io/nextgen). Adding another collection may break ERC compatibility. For example, the `ERC721Enumerable` compatibility will be broken (due to `totalSupply` returning an incorrect value) by adding another collection


## 4️⃣ Large or Custom ERC721 Token IDs

The [ERC721 standard](https://eips.ethereum.org/EIPS/eip-721) specifically does not limit how mining and burning is to be done:

> Creation of NFTs (“minting”) and destruction of NFTs (“burning”) is not included in the specification. Your contract may implement these by other means.

The only constraint is that that the ID of a collection NFT must be representable as a number that fis in a `unit256` variable.

While the majority of tokens opt for incremental minting, or start the mining process from 0/1 and up until the number of NFTs, there are collections that create custom schemas for the tokenId.

_examples_:

- having information within the token ID. The [`Sandbox Game Asset`](https://opensea.io/collection/the-sandbox-assets?tab=items) collection encodes [asset creator, NFT type and other data in the token ID](https://etherscan.io/address/0x7fbf5c9af42a6d146dcc18762f515692cd5f853b#code#F2#L174). Example Token ID of a [Wooden Chair](https://opensea.io/assets/ethereum/0xa342f5d851e866e18ff98f351f2c6637f4478db5/93539746106169265185984122850742147987924040810729814653747710597909864011777): `93539746106169265185984122850742147987924040810729814653747710597909864011777`
- [`Wrapped Number Board`](https://opensea.io/collection/numberboard73) collection with the [max `uint256` value as token ID](https://opensea.io/assets/ethereum/0x4cd48854f5099e671081fc14f74e51da897c6b41/115792089237316195423570985008687907853269984665640564039457584007913129639935): `115792089237316195423570985008687907853269984665640564039457584007913129639935`

## 5️⃣ Multiplying or Breeding ERC721s

Depending on the project behind the ERC721 contract, other ERC721 contracts may be airdropped (or _"breed"_) to the same holding address. 
The airdrop paradigm has slowly changed over time to a _claim airdrop_ logic (implemented pull versus push pattern) instead, thus this type of behavior is not that common, but it may lead to issues [if an airdrop is expected](https://solodit.cyfrin.io/issues/m-07-ntokenmoonbirds-reserve-pool-cannot-receive-airdrops-code4rena-paraspace-paraspace-contest-git).

Holder contracts of NFTs that might receive airdrops for holding them, are encouraged to implement their [ERC721TokenReceiver](https://eips.ethereum.org/EIPS/eip-721) interface in such a way to allow arbitrary NFTs to be received (if protocol logic allows it).

## 6️⃣ Fractionalized ERC721

Fractionalized NFTs is when ownership of an ERC721 is split into several other components, usually ERC20 tokens.

Initial implementations would have the ERC721 deposited into a smart contract to hold, then mint a corresponding amount of ERC20 tokens. Under certain conditions, usually holding all the tokens, the ERC721 was retrieved.

Fractionalizing ERC721 can lead to scenarios where the underlying NFT changes ownership in debatable circumstances, although it does not introduce integration issues after the ERC721 was made whole.

_example_: Ape `CryptoPunk #2386` (worth millions at that time) was acquired with only 10 ETH by [gaming the fractionalization mechanism](https://x.com/nftnow/status/1833957726097125687) 

## 7️⃣ Mixed ERC20 and ERC721 Implementation

Some contract merge features of fungible (ERC20) and non-fungible (ERC721) tokens to create what was coined as semi-fungible tokens. Users could own fractions of an NFT by holding the underlying ERC20 base tokens.

The first semi-fungible variation to gain significant popularity was named [ERC404](https://github.com/Pandora-Labs-Org/erc404) (the name is not an ERC/EIP standard). ERC404-tokens combines both ERC20 and ERC721 standards into one contract but is not fully compatible with either standard. 

Supporting these types of ERC721 tokens requires special and dedicated attention, the readme on the [ERC404](https://github.com/Pandora-Labs-Org/erc404) repository needs to be rigorously inspected while making a protocol.

_examples_: [`Pandora`](https://etherscan.io/token/0x9E9FbDE7C7a83c43913BddC8779158F1368F0413#code), [`DeFrogs`](https://etherscan.io/token/0xd555498a524612c67f286df0e0a9a64a73a7cdc7#code), [`Palette`](https://etherscan.io/token/0x553afe6468949e0685959022217336717df5fbe8#code)


## 8️⃣ ERC721 bound to ERC20

Extending on the idea of pairing ERC20 with ERC721, a new standard appeared, namely [ERC-7631: Dual Nature Token Pair](https://eips.ethereum.org/EIPS/eip-7631) which, quoting the documentation represents a way of having:

> A fungible ERC-20 token contract and non-fungible ERC-721 token contract can be interlinked, allowing actions performed on one contract to be reflected on the other. 

The most popular implementation of ERC-7631 is the [DN404 project](https://github.com/Vectorized/dn404). DN404 uses two separate contracts, one for each token type, interlinked and each fully compatible with their respective standard.

Protocols that integrate with `ERC7631` type tokens must pay attention to subtleties, such as determining which ERC721 token is moved or minted when the underlying ERC20 tokens are transferred. 

[DN404 - Vulnerabilities Reported And Resolved](https://guardianaudits.com/blog/dn404-vulnerabilities-reported-and-resolved) is article written by the author of this document, [ABA](https://x.com/abarbatei) (shameless marketing), which details issues and corner-cases that can appear when integrating the DN404 implementation of `ERC-7631`.

_examples_: [`Asterix Labs`](https://etherscan.io/address/0x0000000000ca73A6df4C58b84C5B4b847FE8Ff39#code), [`Velocity Pass 2.0 by Oracle Red Bull Racing`](https://etherscan.io/address/0x1c72523EEcADe307C9bFfd03953Ff91C5B80be09), [`Sheboshis`](https://etherscan.io/address/0xe5bc6c59fecf634a2d1c09b4f2e780497c713e4f#code)

## 9️⃣ Self Destructing or Self Burning ERC721

There are some ERC721 collections that will burn tokens automatically due to predefined triggers.

Integrating protocols need to account for token IDs randomly disappearing (being burned).

_examples_: 
- the [`Two Degrees`](https://opensea.io/collection/two-degrees) collection will burn its only token ID when global warming reaches 2 degrees above average.
- another example is the [`Complex Death`](https://magiceden.io/collections/apechain/0x566a75b60ddcddc8981e896c8e65ee75036700ae) collection, where there is a 30% chance of burning your NFT on each transfer.

## 1️⃣0️⃣ Upgradable ERC721

There are ERC721 collections that can be upgradable.

While not fitting the weird classification, it is important to note that protocols interacting with upgradable ERC721 tokens may need to update their code in response to it changing. 

Example, if a legitimate collection was compromised and the attacker would revert all transfers, any protocol using the ERC721 as collateral, for example, would have issues.

_examples_: [`DeGods`](https://etherscan.io/address/0x8821bee2ba0df28761afff119d66390d594cd280#code), [`Mocaverse`](https://etherscan.io/address/0x59325733eb952a92e069c87f0a6168b29e80627f#code), [`Sproto Gremlins`](https://etherscan.io/address/0xeeca64ea9fcf99a22806cd99b3d29cf6e8d54925#code), [`Neo Tokyo Citizens`](https://etherscan.io/address/0xb9951b43802dcf3ef5b14567cb17adf367ed1c0f#code)

## 1️⃣1️⃣ Pausable ERC721

Some ERC721 collections can be paused by a privileged role, usually the owner.
In case of a contract ownership compromise, any integrating protocol may have issues with blocked or trapped NFTs.

_examples_: [`Pudgy Rods`](https://opensea.io/collection/pudgyrods), [`PudgyPresent`](https://etherscan.io/address/0x062e691c2054de82f28008a8ccc6d7a1c8ce060d#code)


## 1️⃣2️⃣ ERC721 With Blacklists

There are ERC721 collections that can block addresses from transferring tokens (to and from). This is done at contract level and often employ a registry for managing the blocked addresses.

The same type of issues as when transfers are paused can appear, by blacklisting say, marketplaces, some collections become untradable.

Registry-based blacklists targeting marketplaces were popular at one point when the trend was to not allow NFTs to be traded if creator royalties are not enforced.

Projects like [ClosedSea](https://github.com/Vectorized/closedsea) and [operator-filter-registry](https://github.com/ProjectOpenSea/operator-filter-registry) gained popularity and were integrated in several known and important collections.

_examples (with registry-based blacklists)_: [`Azuki Elementals`](https://etherscan.io/address/0xb6a37b5d14d502c3ab0ae6f3a0e058bc9517786e#code), [`Sproto Gremlins`](https://etherscan.io/address/0xeeca64ea9fcf99a22806cd99b3d29cf6e8d54925#code), [`goblintown`](https://etherscan.io/address/0x8c6def540b83471664edc6d5cf75883986932674#code)

## 1️⃣3️⃣ ERC721 That Mint During Contract Creation

The [ERC721](https://eips.ethereum.org/EIPS/eip-721) standard includes an exception regarding the `Transfer` event:

> This event emits when NFTs are created (`from` == 0) and destroyed (`to` == 0). Exception: during contract creation, any number of NFTs may be created and assigned without emitting Transfer.

The `Transfer` event is essential for off-chain systems to determine and track token ownership. `ERC721` contracts that mint in the constructor without emitting the `Transfer` event, although technically ERC721 compliant, may break off-chain token trackers.

It’s important to note that the phrase _during contract creation_ is somewhat ambiguous and could also be interpreted to include the `initialize` function call in upgradable contract patterns. 

To ensure compatibility with off-chain systems, ERC721 developers are strongly encouraged to emit `Transfer` events whenever associating a token ID with an address.

## 1️⃣4️⃣ ERC721 With Permit 

[ERC-4494: Permit for ERC-721 NFTs](https://eips.ethereum.org/EIPS/eip-4494) was a proposed standard aimed at bringing ERC20-permit (EIP-2612) functionality to ERC721 tokens. This standard allowed users to sign an ERC721 approve transaction off-chain, generating a signature that could then be submitted to the permit function by anyone.

However, the standard is now stagnant and is not recommended for use. [It can cause issues](https://solodit.cyfrin.io/issues/m-02-a-malicious-borrower-can-hijack-any-nft-with-permit-function-he-rents-code4rena-renft-renft-git) with protocols that assume ERC721 to not have any alternative approval mechanism.

While few NFT collections have implemented this standard, it is still important for protocols interacting with ERC721 tokens to account for them. Most notably, the [Uniswap V3 Position NFT](https://etherscan.io/address/0xc36442b4a4522e871399cd717abdd847ab11fe88#code) adheres to this standard.

_examples_: [`Uniswap V3: Positions NFT`](https://opensea.io/collection/uniswap-v3-positions), [`Wallkanda Curated Artists`](https://opensea.io/collection/wallkanda-curated-artists), [`Bleeps`](https://opensea.io/collection/bleeps)
