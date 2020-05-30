---
title: 'Bitcoin Optech Newsletter #100'
permalink: /en/newsletters/2020/06/03/
name: 2020-06-03-newsletter
slug: 2020-06-03-newsletter
type: newsletter
layout: newsletter
lang: en
---
This week's newsletter summarizes a proposed design for a coinswap
implementation and links to two transaction size calculators.  Also
included are our regular sections with descriptions of several recently
transcribed talks, new releases and release candidates, and notable
changes to popular Bitcoin infrastructure software.  A special final
section celebrates the publication of Newsletter #100.

## Action items

*None this week.*

## News

- **Design for a coinswap implementation:** Chris Belcher
  [posted][belcher coinswap] a design for a full-featured coinswap
  implementation.  Coinswap is a protocol that allows two users to
  create a pair of transaction that look like regular payments but which
  actually swap their coins with each other.  This improves the privacy
  of not just the coinswap users but all Bitcoin users, as anything that
  looks like a payment could have instead been a coinswap.

    Belcher's post summarizes the history of the coinswap idea, suggests
    ways the multisig conditions needed for coinswap could be disguised
    as more common transaction types, proposes using a market for
    liquidity (like JoinMarket already does), describes splitting and
    routing techniques to reduce privacy losses from amount correlation
    or spying participants, mentions alternative coinswap protocols such
    as succinct atomic swaps (see [Newsletter #98][news98 sas]), suggests
    combining coinswap with [payjoin][topic payjoin], and discusses some
    of the backend requirements for the system.  Additionally,
    comparisons are made to other privacy techniques such as using LN,
    [coinjoin][topic coinjoin], payjoin, and [payswap][zmn payswap].
    Near the end of his email, Belcher summarizes the requirements for
    the system he describes:

    > * [Two-party ECDSA]
    > * Liquidity market
    > * Routed CoinSwaps
    > * Multi-transaction CoinSwaps
    > * Breaking change output heuristics
    > * Fidelity bonds
    > * PayJoin with CoinSwap
    > * Federated message boards protected from spam with fidelity bonds

    Belcher has a history of creating and maintaining privacy enhancing
    open source software for Bitcoin, such as [JoinMarket][] and
    [Electrum Personal Server][eps], which gives particular weight to
    the conclusion of his email: "I intend to create this CoinSwap
    software.  It will be almost completely decentralized and available
    for all to use for free."

- **Transaction size calculators:** Jameson Lopp [posted][lopp size] to
  the Bitcoin-Dev mailing list with links to a [transaction size
  calculator][lopp calc] he'd developed as well as a similar [calculator
  developed by Optech][optech calc].  Neither tool claims to be complete
  or bug-free, but both should be useful to developers who want to
  quickly compare the sizes of different types of transactions.

## Recently transcribed talks and conversations

*[Bitcoin Transcripts][] is the home for transcripts of technical
Bitcoin presentations and discussions. In this monthly feature, we
highlight a selection of the transcripts from the previous month.*

FIXME:michaelfolkson

## Releases and release candidates

*New releases and release candidates for popular Bitcoin infrastructure
projects.  Please consider upgrading to new releases or helping to test
release candidates.*

- [Bitcoin Core 0.20.0rc2][bitcoin core 0.20.0] is the most recent
  release candidate for the next major version of Bitcoin Core.

- [LND 0.10.1-beta.rc3][] is the latest release candidate for the next
  maintenance release of LND.

## Notable code and documentation changes

*Notable changes this week in [Bitcoin Core][bitcoin core repo],
[C-Lightning][c-lightning repo], [Eclair][eclair repo], [LND][lnd repo],
[Rust-Lightning][rust-lightning repo], [libsecp256k1][libsecp256k1 repo],
[Bitcoin Improvement Proposals (BIPs)][bips repo], and [Lightning
BOLTs][bolts repo].*

*Note: the commits to Bitcoin Core mentioned below apply to its master
development branch and so those changes will likely not be released
until version 0.21, about six months after the release of the upcoming
version 0.20.*

- [Bitcoin Core #19010][] net processing: Add support for getcfheaders FIXME:jonatack

- [Bitcoin Core #16030][] changes how long Bitcoin Core waits until
  it queries DNS seeds for the IP addresses of potential peers.
  Previously, if the node had peer IP addresses in its database, it
  would try opening several connections and wait 11 seconds for
  successful connections before requesting new addresses.  Now, if it
  has more than 1,000 IP addresses in its database---which is common for
  nodes that have been online more than a few hours---it'll wait up to 5
  minutes before querying.  This improves the chance that a restarted
  node will entirely use P2P address discovery without relying on the
  centralized DNS seeds.

- [LND #4228][] walletrpc: add LabelTransaction endpoint to retrospectively label txns FIXME:dongcarl

## On the occasion of Optech Newsletter #100

> "I am somewhat surprised that nobody has taken to [writing] weekly
> summaries of research and development activity. Summarizing recent
> work is a valuable task that others can engage in just by reading the
> mailing list and aggregating multiple thoughts together."
>
> {:.right}
> ---Bryan Bishop, [19 August 2015][bishop summaries]

{% comment %} <!--
  $ find _posts/en/newsletters/ _includes/specials/bech32/  -type f | xargs cat | wc -w
  155217

  443 pages = 155217 words / 350 words per printed page
-->{% endcomment %}

Almost five years after Bishop made the above comment, we remain
convinced that writing weekly summaries of research and development
activity is a task that's valuable to both the open source Bitcoin
development community and to the many businesses that depend upon the
community's work.  But in the two years we've been producing this
newsleter, we've also discovered that summarizing isn't quite as quick
and simple as we initially expected it to be.  Accordingly, we'd like to
take this chance to thank the people who make this newsletter possible
by generously contributing a significant amount of their valuable time
week after week: [Adam Jonas][], [Carl Dong][], [David A. Harding][],
[John Newbery][], [Jon Atack][], [Mike Schmidt][], and [Steve Lee][].

We additionally thank the experienced Bitcoin and LN contributors who
kindly provided us with special help on certain complex topics or who
contributed field reports and other special content to the newsletter
over the past two years.

Publishing a high-quality weekly newsletter and working to fulfill other
aspects of Optech's [mission][about page] wouldn't be possible without
the financial support of our member companies.  We thank them for their
continuing commitment to improving communication between Bitcoin users,
developers, and businesses.

    FIXME:harding:screenshot of members from optech homepage after #406
    is merged

We also remain eternally thankful to our founding sponsors [Wences
Casares][], [John Pfeffer][], and [Alex Morcos][] <!-- same order as on
About page --> as well as to organizations such as [Chaincode Labs][]
and [SquareCrypto][] who both allow and encourage their staff to use
their work hours to contribute to Optech.

{% include references.md %}
{% include linkers/issues.md issues="19010,16030,4228" %}
[bitcoin core 0.20.0]: https://bitcoincore.org/bin/bitcoin-core-0.20.0
[lnd 0.10.1-beta.rc3]: https://github.com/lightningnetwork/lnd/releases/tag/v0.10.1-beta.rc3
[bishop summaries]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2015-August/010488.html
[news98 sas]: /en/newsletters/2020/05/20/#two-transaction-cross-chain-atomic-swap-or-same-chain-coinswap
[belcher coinswap]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-May/017898.html
[zmn payswap]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-January/017595.html
[joinmarket]: https://github.com/JoinMarket-Org/joinmarket-clientserver
[eps]: https://github.com/chris-belcher/electrum-personal-server
[lopp size]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-May/017905.html
[lopp calc]: https://jlopp.github.io/bitcoin-transaction-size-calculator/
[optech calc]: https://bitcoinops.org/en/tools/calc-size
[about page]: /about/
[Adam Jonas]: https://github.com/adamjonas
[Carl Dong]: https://github.com/dongcarl
[David A.  Harding]: https://github.com/harding
[John Newbery]: https://github.com/jnewbery
[Jon Atack]: https://github.com/jonatack
[Mike Schmidt]: https://github.com/bitschmidty
[Steve Lee]: https://github.com/moneyball
[Wences Casares]: https://en.wikipedia.org/wiki/Wences_Casares
[John Pfeffer]: https://twitter.com/jlppfeffer
[Alex Morcos]: https://twitter.com/morcosa
[Chaincode Labs]: https://chaincode.com/
[SquareCrypto]: https://twitter.com/sqcrypto
