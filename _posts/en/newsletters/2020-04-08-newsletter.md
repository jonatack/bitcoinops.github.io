---
title: 'Bitcoin Optech Newsletter #92'
permalink: /en/newsletters/2020/04/08/
name: 2020-04-08-newsletter
slug: 2020-04-08-newsletter
type: newsletter
layout: newsletter
lang: en
---
This week's newsletter contains a list of notable changes to popular
Bitcoin infrastructure projects.  FIXME: mention_PR_club_section_if_added

## Action items

*None this week.*

## News

*No significant development news this week.*

## Bitcoin Core PR Review Club

_In this section, we summarize a recent Bitcoin Core PR Review Club meeting,
highlighting some of the important questions and answers.  Click on a
question below to see a summary of the answer from the meeting._

**[Retry notfounds with more urgency][review club 18238]** is a PR
([#18238][Bitcoin Core #18238]) by Anthony Towns that would change peer-to-peer
behavior so that when nodes receive a NOTFOUND in response to a request for a
transaction, they would skip the current time-out period and instead:

1. look through their peers for any others that announced the transaction;

2. select all that haven't exceeded the max in-flight transaction limit;

3. of those, have the peer that would look at it first do so as soon as
   possible.

Discussion began with fundamental reasons for the PR:

<div class="review-club-questions"></div>
- <details><summary>Why could retrying NOTFOUNDs more quickly be helpful?</summary>
  DoS prevention, transaction propagation speed, privacy, and future `mapRelay`
  removal.</details>
- <details><summary>What is a potential DoS attack concern?</summary>
  Nodes with small mempools could force peers to wait a long time before
  receiving a transaction.</details>
- <details><summary>Why is transaction propagation speed important?</summary>
  Short delays in seconds aren't an issue (and can even be desirable for
  privacy), but larger delays in minutes can hurt propagation of transactions,
  BIP152 relay, and even that of blocks.</details>
- <details><summary>When and why was `mapRelay` originally added?</summary>
  `mapRelay` was present in the first Bitcoin git commit to ensure a
  transaction could be downloaded even if a block had already confirmed it.</details>
- <details><summary>Why could removing `mapRelay` be desirable?</summary>
  `mapRelay` has poor memory bounding, which leads to memory exhaustion issues
   that have blocked progress.</details>
- <details><summary>Describe one issue with removing `mapRelay`?</summary>
  It could cause requested transactions in honest situations to more often be
  NOTFOUND with delays of up to 2 minutes, hurting propagation.</details>

Later in the meeting, the `TxDownloadState` data structure was discussed:

<div class="review-club-questions"></div>
- <details><summary>Describe the role of the `TxDownloadState` struct?</summary>
    A per-peer state machine, with timers, to coordinate requesting transactions
    from peers.</details>
- <details><summary>What are 3 `TxDownloadState` states a transaction can be in?</summary>
  (1) announced, but not requested; (2) announced, requested, and waiting for
  delivery; (3) announced, requested, but timed out.</details>

Discussion then delved deeply into the PR implementation, potential issues, and
future improvements and their timing with respect to the upcoming [wtxid
transaction relay](https://bitcoincore.reviews/18044). Don't hesitate to read
the study notes and meeting log at https://bitcoincore.reviews/18238 for more.

## Notable code and documentation changes

*Notable changes this week in [Bitcoin Core][bitcoin core repo],
[C-Lightning][c-lightning repo], [Eclair][eclair repo], [LND][lnd repo],
[libsecp256k1][libsecp256k1 repo], [Bitcoin Improvement Proposals
(BIPs)][bips repo], and [Lightning BOLTs][bolts repo].*

- [C-Lightning #3612][] adds startup parameters `--large-channels` and
  `--wumbo` (which are equivalent).  If used, the node will advertise
  support for `option_support_large_channel` in its `init` message,
  meaning it will accept channel opens with a value higher than the
  previous limit of about 0.168 BTC.  If the remote peer also supports
  this option, C-Lightning's `fundchannel` RPC will allow the user to
  create channels over the previous limit.  See also Eclair's support
  for this option described in [Newsletter #88][news88 eclair1323].

- [C-Lightning #3600][] adds experimental support for *onion messages*
  using *blinded paths*:

    - *Onion messages* (called "LN direct messages" in
      [Newsletter #86][news86 ln dm]) allow a node to send an encrypted
      message across the network without using the LN payments
      mechanism.  This can replace the mechanism of
      messages-over-payments used by apps such as [Whatsat][].  Compared
      to messages-over-payments, onion messages have several advantages:

        1. They have a [draft specification][onion messages draft spec]
           which, if adopted, will make it easier for multiple
           implementations to support them.

        2. They don't need the security of an onchain-enforceable payment
           channel so onion messages can be routed even between peers
           that don't share an established payment channel.  <!-- BOLT7
           draft PR: "SHOULD accept onion messages from peers without an
           established channel." -->

        3. They don't require the bidirectional transmission of
           information like HTLCs or error messages, so once a node has
           forwarded a message, it doesn't need to keep any information
           related to that message.  This minimizes the memory
           requirements for nodes.  If the sending node wants to receive
           a reply, the draft specification allows it to include a
           blinded `reply_path` field that the receiving node can use to
           send a reply in a new message.

    - *Blinded paths* (called "lightweight rendez-vous routing" in
      [Newsletter #85][news85 lw rv]) make it [possible][blinded path
      gist] to route a payment or a message without the originator
      learning the destination's network identity or the full path used.
      This is accomplished through several steps:

        1. The destination node chooses a path from an intermediate node
           to itself and then onion-encrypts that path information so
           that each hop in the path will only be able to decrypt the
           identifier for the next node that should receive the message.
           The destination node gives this encrypted ("blinded") path
           information to the sending node (e.g. via a field in a
           [BOLT11][] invoice or using the previously-mentioned onion
           message `reply_path` field).

        2. The sending node relays its message using normal onion
           routing to the intermediate node.

        3. The intermediate node decrypts the next hop to use from the
           blinded path and sends the message to it.  The next node
           decrypts its own next hop field and further relays the
           message; this process continues until the message reaches the
           destination node.

      Just as with normal onion routing, no routing node should learn
      more about the blinded path than which node sent them the message
      and which node should receive the message after them.  Unlike with
      normal routing, neither the origination nor destination node needs
      to learn the identity of the other node or what exact path it
      used.  This improves not just the privacy of those endpoints but
      also the privacy of any unannounced nodes along the blinded paths.

    As the PR notes, "there are no contents defined yet [for the
    messages], except for those required for routing and replies, but
    the intent is to use this mechanism for offers."  Offers would allow
    nodes to request and send invoices through the LN; see [Newsletter
    #72][news72 offers] for details.

- [LND #4087][] Merge pull request #4087 from Crypt-iQ/wt_hs_0310 FIXME:moneyball
  FIXME: suggested link to https://github.com/lightningnetwork/lnd/blob/master/docs/watchtower.md#tor-hidden-services

- [LND #4079][] Merge pull request #4079 from guggero/psbt-chanfunding FIXME:dongcarl
  FIXME: link to https://github.com/lightningnetwork/lnd/blob/master/docs/psbt.md

- [LND #3970][] adds support for [multipath payments][topic multipath
  payments] to the LND's payment lifecycle system, which is the part of
  LND that tracks "all information about the current state of a payment
  needed to resume it from any point". <!-- routing/payment_lifecycle.go
  -->  This brings LND much closer to its goal for version 0.10 of being
  able to fully support multipath payments. <!-- Alex Bosworth email,
  "In 0.10.0 I think the main new feature will be the ability to
  multipath." -->

{% include references.md %}
{% include linkers/issues.md issues="3612,3600,4087,4079,3970" %}
[news86 ln dm]: /en/newsletters/2020/02/26/#ln-direct-messages
[news85 lw rv]: /en/newsletters/2020/02/19/#decoy-nodes-and-lightweight-rendez-vous-routing
[news88 eclair1323]: /en/newsletters/2020/03/11/#eclair-1323
[news72 offers]: /en/newsletters/2019/11/13/#proposed-bolt-for-ln-offers
[whatsat]: https://github.com/joostjager/whatsat
[onion messages draft spec]: https://github.com/lightningnetwork/lightning-rfc/pull/759
[blinded path gist]: https://github.com/lightningnetwork/lightning-rfc/blob/route-blinding/proposals/route-blinding.md
