+++
title = "A Year In POKT"
date = 2023-12-14
categories = [ "articles" ]
tags = [ "pokt", "development" ]
draft = false 
+++

<!-- toc -->

- [Introduction](#introduction)
- [Context](#context)
- [The Good](#the-good)
  * [SMT](#smt)
  * [Compute Units](#compute-units)
  * [Gateways And Rings](#gateways-and-rings)
- [The Bad](#the-bad)
- [The Ugly](#the-ugly)
- [Summary](#summary)

<!-- tocstop -->

## Introduction

Hey 👋, I'm [h5law](https://github.com/h5law). I'm 23, in my first year of a
Computer Science degree at university 👨🏼‍💻 (starting a little late but that's a story
for another day). This last year has been a wild ride for me, and as "memoir"
of sorts I thought I'd write down some of the highlights.

So in addition to being a student I am a protocol developer at
[Pocket Network](https://pokt.network) working on building out the new version
of the protocol - referred to as "Shannon". We are building a permissionless,
decentralised RPC network that incentivises API services in general. The main
focus at the moment is for Web3 projects, Blockchains and dApp developers but
the longer term vision is for Pocket to be the base layer for RPC access to any
and all API services (LLMs, and whatever else you can run we could service your
requests).

This article is about my journey over the last year, after submitting my
[first PR](https://github.com/pokt-network/pocket/pull/397) a lot has changed.
I've learned so much; grown as a developer; made/contributed to some **really
cool** stuff, in my opinion at least; and been challenged in so many ways. I
hope to capture some of these in this article

## Context

For those who don't know, in the past year Pocket has transitioned from
building out its own L1 to a rollup utilising [Rollkit] and [Celestia] for our
data-availability layer.

When I started I was building lots of new things, below is a list of things that
stood out as of the time of writing:

1. [Crypto and Key primitives.][crypto]
1. [Keybases.][keybase]
1. [Hierarchical Key Derivation Systems.][slip0010]
   - See: [SLIP-0010](https://github.com/satoshilabs/slips/blob/master/slip-0010.md)
     for more information on this specifically.
1. The [CLI] and [RPC] to interact with the L1 itself.
1. A non-cosmos dependent implementation of [ibc-go](https://github.com/cosmos/ibc-go)
   - This was very much still a WIP

But it was when I began to work on the [SMT] that I found a true passion of
mine. We will dive into this in a lot more detail later, but it was in the old
[Morse (v1) repo][v1] this all began.

After much deliberation, many late night calls, debates and most importantly
**research**; we decided as a team to move away from building out our own L1 and
focus on our core offerings. Decentralised RPC. [Olshansky] wrote a
great [blog post](https://olshansky.substack.com/p/why-pocket-is-rolling-with-rollkit)
detailing everything so I won't go into it all now. But as a result it meant
starting over with: a new stack, a new codebase and unfortunately a lot of the
work I had previously done was scrapped in favour of the multitude of things
that come for free out of the box with using [`cosmos-sdk`][cosmos] (rollups
using [rollkit] on [celestia] can still use the sdk as if
they were building a regular app-chain) and benefit from all the work done in the
[cosmos] ecosystem.

So lets get into this last year: the [**good**](#the-good), the
[**bad**](#the-bad) and the [**ugly**](#the-ugly).

![Good Bad Ugly](/media/good-bad-ugly.gif#center)

## The Good

I have honestly learned so much, its unbelievable how many challenges you face
when building something completely new, from the ground up. It's true you
shouldn't re-invent the wheel but what if the wheel never existed in the first
place? We have had to make so many cool things that honestly make me so proud to
work with such a great team and build such an **amazing** and **inovative**
protocol.

From the guidance of [Olshansky] and the other amazing members of the
protocol team [Bryan], [Redouane], [Dmitry] - all who have a lot more industry
experience than I do; I have managed to build my intuition on problem solving
which is the hardest thing, in my opinion, when it comes to programming. Writing
code is easy; debugging is hard; testing is a ball-ache; but the way you approach
a problem, especially one you haven't encountered before is by far the most
challenging aspect of development.

As part of building out the [Shannon][poktroll] upgrade to the protocol, I am
still involved heavily in the cryptographic side of things. Integrating ring
signatures for our App to Gateway delegation process and request signing. I've
been able to contribute back to the [library][ring-go] we use, which I find
so cool.

So lets break down some of the highlights of this year. Of course there will be
lots of exciting things I am going to miss out, otherwise this blog would be
incredibly long.

### SMT

Over this past year I have learned a lot. I have become somewhat of a specialist
in the field of trees/tries 🌴 with the [SMT repo][smt] that I have taken over,
since [celestia] archived [their repo](https://github.com/celestiaorg/smt)
which we were using previously.

Since taking over I have merged in numerous features:

1. Lazy Loading (developed by celestia but never finished)
1. [Merkle Sum Trie][mstdocs]
   - A _possibly_ world first implementation of a (Sparse) Merkle Sum Tree/Trie
     implementation based on the [plasma specification][plasmaspec]
1. The [`ClosestProof`][clpdocs] proof mechanism
   - This is a novel proof mechanic based off the [Relay Mining Paper][rmpaper]

We had our first [external PR](https://github.com/pokt-network/smt/pull/28) and
as such discovered [IOTA Ledger](https://github.com/iotaledger/hive.go) are using
**our SMT**! It really felt like my baby 👨‍🍼 was taking his first steps.

I've got so many plans for this repo and its future, that I will save for
another post but all I should say is:

> Expect big things 😉

### Compute Units

If you are familiar with RPC providers in general you would know providers such
as Lava, Ankr and Alchemy utilise `Compute Units` 💾 these allow for each RPC call
to be paid for proportional to the cost required to execute it. This is beneficial
as it rewards suppliers for providing more demanding services and thus gives
more access to those services.

I am leading the work on integrating `Compute Units` into Shannon, this is one
of the main reasons I made the [SMST][mstdocs] as we can attach a `weight` or
`sum` to each relay/response a supplier inserts into their tree and easily get
an accounting for how much work they did.

It is super exciting to be working on getting this feature into the protocol,
with out-of-the-box ideas instead of the typical verbose approach one would
immediately think of. Again big shout out to Olshansky and [Ramiro] from
[PoktScan](https://poktscan.com) for helping me flesh out these ideas before we
begin the implementation.

### Gateways And Rings

I implemented the on-chain gateway actor and in conjunction with Redouane built
our MVP off-chain `appgateserver` (which is essentially a gateway like Portal
or NodiesDLB). We are currently working on building out an SDK to allow for
Gateways and applications to interact with the chain. All of this is under
active development but seeing some of the issues with Morse (v1) being
eliminated is actually amazing and I am so proud to be a part of it.

My main contribution to the Application\<->Gateway interaction was (un)delegation.
This allows for an application to delegate trust to a gateway such that the
gateway can sign incoming requests from the application on its behalf. All of
this works through the usage of [ring signatures][ring-go] 💍✍🏻 and the
[`RingCache`][ring-cache] which handles the creation and naturally caching of
rings on behalf of anyone who requires them, (applications and gateways for
signing request and suppliers for verifying requests).

Cryptography has definitely stood out to me as being another area of interest,
and I hope that *soon*™️ , I will be on the same level of knowledge and
intuition with cryptographic primitives as I am with trees. But there are lots
of abstract algebra textbooks 📚 to read before I am fluent here.

![Absorbing Books](/media/absorb-book.gif#center)

## The Bad

Naturally as part of pivoting: a lot of work I was very proud of; ongoing work
related to a native go implementation of IBC (without the `cosmos-sdk`
dependency) and other parts of the old repo I was invested in suddenly vanished.

It's never nice to see the things you've worked on and eagerly waited for people
to use not see the light of day. But that's life and we move on. As the
[section above](#the-good) details there is still so much good that this doesn't
really matter. What we are doing now is the right thing and I stand behind that
with my whole self.

Personally, I find it hard to find any downsides to my work here at Pocket.
Going from a community contributor to working full time on the protocol, I live
and breath POKT. I love this project, I have ideas 💡 ranging from pre-testnet,
to testnet and after mainnet. And I want to see all these implemented. It's been
great to have my say and shape the protocol - or at least influence its shape.

That being said I find coming from a different background to the others on the
team to be a real challenge. I am young, have less "experience" (but this
doesn't matter in Web3 - if you can deliver it doesn't matter who you are), and
this shows. Often its an opportunity for me to learn, other times its an
opportunity for me to teach them something. We are building out the protocol
rapidly and everyone is committing their all. Working weekends, late nights etc.
But occasionally there are miscommunications and this is expected working in a
team. Maybe its because I haven't worked in the "traditional" corporate setting,
but it seems my style of work doesn't fit well with others. I think the new
generation hussles, grinds and works differently from the generations that came
before us. The rest of the team work hard; they grind; they hussle; just
differently.

![Hustling My Whole Life](/media/hustling-whole-life.gif#center)

This could be the single downside I have had working full time at Pocket. Going
from a community contributor to a full time core protocol dev, it has become a
job, there are more rules; people to please; feelings to take into account. In
my world all that matters is:

> Can you deliver?

## The Ugly

As I said previously, I am at university 🎓 full time as well as doing all this
work with Pocket. Surprisingly this isn't the easiest thing to do.

Doing both university full time (with all the: exams, coursework, lectures,
practicals etc.), on top of the full-time work I do at Pocket, has been a lot.
It has definitely been ugly 😖. But I've achieved time management skills
like no one would believe. I surprise myself sometimes with how I manage it all.
It's far from easy but its most definitely the ugliest part of the POKT+UNI
combination.

I think I've come close to burning out a couple times, I'm definitely someone
who gives 100% to whatever I do and this level of intensity is tough. But I love
it. The title of this section may be a little misleading in that regard. It's
tough, it's hard, but there's nothing I'd rather be doing than what I am doing
right now.

## Summary

In summary, I live and breath POKT. I love working on and building out such an
innovative protocol. Being part of something that I **know** will be **huge**
and used by so many people, devs, projects, companies, etc. excites me.
It drives me and motivates me to continue working on this amazing project.

The interests I have developed working on POKT: Trees 🌴, Databases 💾,
Cryptography 🔑 will stay with me and I will continue working on building the
best solutions not only for our use case but for the entire industry. I truly
believe that we will soon enter the `PoktVerse` 🪐 phase:

- We underpin so many projects as their infrastructure and data provider
- Our innovative technology is used across the entire industry
- We moon? 🌝

In Web3 (and Crypto in general) it doesn't matter **who** you are. All that
matters is **what** you can do. Your past isn't important, neither where you
have come from. I think this is the best part of the industry. And I'm really
looking forward to what comes next, so pay attention 😉.

![Things Are Changing](/media/things-are-changing.gif#center)

[bryan]: https://x.com/bryanchriswhite
[celestia]: https://celestia.org
[cli]: https://github.com/pokt-network/pocket/tree/main/app/client/cli
[clpdocs]: https://github.com/pokt-network/smt/blob/main/docs/SMT.md#closest-proof
[cosmos]: https://github.com/cosmos/cosmos-sdk
[crypto]: https://github.com/pokt-network/pocket/tree/main/shared/crypto
[dmitry]: https://x.com/kdasme
[keybase]: https://github.com/pokt-network/pocket/tree/main/app/client/keybase
[mstdocs]: https://github.com/pokt-network/smt/blob/main/docs/MerkleSumTrie.md
[olshansky]: https://olshansky.info
[plasmaspec]: https://plasma-core.readthedocs.io/en/latest/specs/sum-tree.html
[poktroll]: https://github.com/pokt-network/poktroll
[redouane]: https://x.com/redzeratul
[ramiro]: https://twitter.com/Rama_stdout
[ring-cache]: https://github.com/pokt-network/poktroll/tree/main/pkg/crypto/rings
[ring-go]: https://github.com/noot/ring-go
[rmpaper]: https://arxiv.org/abs/2305.10672
[rollkit]: https://rollkit.dev
[rpc]: https://github.com/pokt-network/pocket/tree/main/rpc
[slip0010]: https://github.com/pokt-network/pocket/tree/main/shared/crypto/slip
[smt]: https://github.com/pokt-network/smt
[v1]: https://github.com/pokt-network/pocket
