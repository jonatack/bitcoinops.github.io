---
title: 'Bitcoin Optech Newsletter #106'
permalink: /en/newsletters/2020/07/15/
name: 2020-07-15-newsletter
slug: 2020-07-15-newsletter
type: newsletter
layout: newsletter
lang: en
---
This week's newsletter describes a proposed update to the draft BIP118
`SIGHASH_NOINPUT` and summarizes notable changes to popular Bitcoin
infrastructure projects.

## Action items

*None this week.*

## News

- **BIP118 update:** Anthony Towns [posted][towns post] to the
  Bitcoin-Dev mailing list a link to a PR that proposes to replace the
  existing text of the [BIP118][] draft of [SIGHASH_NOINPUT][topic
  sighash_noinput] with the [draft specification][anyprevout spec] for
  `SIGHASH_ANYPREVOUT` and `SIGHASH_ANYPREVOUTANYSCRIPT`.  Both
  proposals describe optional signature hash (sighash) flags that do not
  commit to the particular UTXOs (inputs/previous outputs) being spent
  in a transaction, making it possible to create a signature for a
  transaction without knowing which UTXO it will spend.  That feature can
  be used by the proposed [eltoo][topic eltoo] settlement layer to allow
  creating a series of transactions where any later transaction can
  spend the value from any earlier transaction, allowing an offchain
  contract to be settled in its latest state even if earlier states are
  confirmed onchain.

    The main difference between the noinput and anyprevout proposals is
    that noinput would require its own new version of segwit but
    anyprevout uses one of the upgrade features from the proposed [BIP342][]
    specification of [tapscript][topic tapscript].  Additional
    differences between the proposals are described in the [revised
    text][anyprevout revisions] itself.

## Notable code and documentation changes

*Notable changes this week in [Bitcoin Core][bitcoin core repo],
[C-Lightning][c-lightning repo], [Eclair][eclair repo], [LND][lnd repo],
[Rust-Lightning][rust-lightning repo], [libsecp256k1][libsecp256k1 repo],
[Hardware Wallet Interface (HWI)][hwi], [Bitcoin Improvement Proposals
(BIPs)][bips repo], and [Lightning BOLTs][bolts repo].*

- [Bitcoin Core #19219][] FIXME:jonatack, also [Bitcoin Core #19469][]

- [Bitcoin Core #19328][] updates the `gettxoutsetinfo` RPC with a new
  `hash_type` parameter that allows specifying how to generate a
  checksum of the current UTXO set.  Currently the only two options are
  `hash_serialized_2`, which produces the checksum that has been the
  default since Bitcoin Core 0.15 (September 2017), or `none`, which
  returns no checksum.  It's [planned][Bitcoin Core #18000] to later
  allow a [muhash][]-based checksum along with an index that will allow
  returning the checksum much more quickly than is now possible (in less
  than two seconds, per early testing by an Optech contributor).  For
  now, requesting the `none` result allows the `gettxoutsetinfo` RPC to
  run much more quickly, which is useful for anyone running it after
  each block (e.g. to audit the number of spendable bitcoins).  For
  historical context on fast UTXO set checksums, see this [2017
  post][wuille rolling] by Pieter Wuille.

- [Bitcoin Core #19191][] updates the `-whitebind` and `-whitelist`
  configuration settings with a new `download` permission.  When this
  permission is applied to a peer, it will be allowed to continue
  downloading from the local node even if the node has reached its
  `-maxuploadtarget` maximum upload limit.  This makes it easy for a
  node to restrict how much data it offers to the public network
  without restricting how much it offers to local peers on the same
  private network.  The existing `noban` permission also gives peers
  with that permission an unlimited download capability, but that may be
  changed in a future release.

- [LND #971][] FIXME:dongcarl

- [LND #4281][] adds an `--external-hosts` command line flag that accepts
  a list of one or more domain names.  LND will periodically poll DNS
  for each domain's IP address and advertise that LND is listening for
  connections on that address.  This makes it easy for users of [dynamic
  DNS][] services to automatically update their node's advertised IP
  address.

{% include references.md %}
{% include linkers/issues.md issues="19219,19469,19328,19191,971,4281,18000" %}
[dynamic dns]: https://en.wikipedia.org/wiki/Dynamic_DNS
[towns post]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-July/018038.html
[anyprevout spec]: https://github.com/ajtowns/bips/blob/bip-anyprevout/bip-0118.mediawiki
[anyprevout revisions]: https://github.com/ajtowns/bips/blob/bip-anyprevout/bip-0118.mediawiki#revisions
[wuille rolling]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2017-May/014337.html
[muhash]: https://cseweb.ucsd.edu/~mihir/papers/inchash.pdf
[bitcoin core 0.15]: https://bitcoincore.org/en/releases/0.15.0/#low-level-rpc-changes