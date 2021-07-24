# Blockchain Commons `mori-cli`

<!--Guidelines: https://github.com/BlockchainCommons/secure-template/wiki -->

### _by [Francisco Calderón](https://github.com/grunch)_

Death is something uncomfortable, usually our death is not something we want to talk about, much less in our youth, but as Bitcoiners we have to prepare our funds for that day that will inevitably come.

This project proposes a way that a btc owner can leave his bitcoins to his heirs in the event of death in a decentralized way and with the least possible complication.

The main characteristic of this proposal is that the heirs can recover the funds in the event of death but CANNOT access the funds while the btc owner lives.

Not knowing when we are going to die is what adds complexity to the matter.

As a Bitcoiner and custodians of our privacy, we need to remove the trusted third party whenever possible.

Bitcoin script allows us to carry out a method so that the heirs cannot access while the btc owner is alive, but they will after his death.

## How does this work?
We will call the btc owner **Alice** and we will call the heir **Bob**:

1. User generates two descriptors, the first one is for Alice, with this descriptor Alice will be able to generate new addresses and spend those UTXO at any moment she need. The second one is for Bob, with this descriptor he will be able to spend every UTXO from Alice after 25920 blocks being mined, this is 6 months approximately.
2. Alicia gives to Bob his descriptor.
3. Alice can use this wallet like any other wallet, but she has to be sure to spend every UTXO before 6 months, if she doesn't we can assume she's dead and Bob can inherit the money.

**Morí** is based on [Bitcoin Dev Kit](https://github.com/bitcoindevkit/bdk) and creates descriptors from a miniscript policy, this policy have two spending conditions, **this descriptors ONLY works on BDK at this moment** but hopefully it will be working on [Bitcoin core soon](https://github.com/bitcoin/bitcoin/pull/16800) and others will follow.

Condition 1: Alice can spend the funds at any time.
Condition 2: Bob can spend the funds after N blocks have been mined.

miniscript policy:
```
or(
    pk(A),
    and(
        pk(B),
        older(N)
    )
)

```
## Status - Late Alpha

`Morí`  is currently under active development and in the late alpha testing phase. It should not be used for production tasks until it has had further testing and auditing.

This first version is a stateless wallet, this means that we are regenerating it every time we run the command line.

### Roadmap
- [x] Basic stateless wallet (connects to a electrum server)
- [x] Add miniscripts support with CSV time-locks
- [x] Generate main descriptors (A can spend anytime, B can spend after N blocks mined)
- [x] Generate account addresses from descriptor main descriptor
- [x] Add PSBT generation, signing PSBT and broadcast tx
- [] Generate descriptors for B (can spend after N blocks mined)
- [] Error handling
- [] Change from stateless to stateful wallet

## Prerequisites
Make sure you have [Rust and Cargo](https://doc.rust-lang.org/cargo/getting-started/installation.html) installed.

## Installation Instructions
```
$ git clone https://github.com/BlockchainCommons/mori-cli.git
$ cd mori-cli
$ cargo build
```
The resulting executable is `target/debug/mori-cli`

You can alternatively run and build automatically with a single command: `cargo run`.

## Usage Instructions
We generate two descriptors, they only have a little difference, one will generate the addresses to receive and the other one for the change addresses.
```
$ cargo run -- descriptor
```
Everything before `--` are arguments we are passing to Rust's cargo tool, what is after to `--` are arguments to our program.

If we want to execute the binary we can run it like this:
```
$ target/debug/mori-cli descriptor
```
We will see our receiving descriptor and our change descriptor, as we are using a stateless wallet we need to pass as arguments those descriptors as parameters everytime, so let's create two variables in a .env file.
```
DESC="wsh(or_d(pk([f7e6924a/87h/1h/0h]tprv8ZgxMBicQKsPdoksgNYN6yy6JxCNTGnHGEcoKwiEM6z8i4v8kZEUzp3UC5LLypQT2mrRTW4Zo4jPTsPmzjTH8MPBTNsHQvbamfxmQRrfoDk/0/*),and_v(v:pk([aab88436/87h/1h/0h]tprv8ZgxMBicQKsPeoE3PXG3hRGDVnSV7fWgnUZ8yaG9JaSQBGqzGEXUyyxj5Dkp4xxbPUZzedjBSghLsoqfuUYukbit47dbkLT3PY3oRXViJGr/0/*),older(25920))))"
CHANGE="wsh(or_d(pk([f7e6924a/87h/1h/0h]tprv8ZgxMBicQKsPdoksgNYN6yy6JxCNTGnHGEcoKwiEM6z8i4v8kZEUzp3UC5LLypQT2mrRTW4Zo4jPTsPmzjTH8MPBTNsHQvbamfxmQRrfoDk/1/*),and_v(v:pk([aab88436/87h/1h/0h]tprv8ZgxMBicQKsPeoE3PXG3hRGDVnSV7fWgnUZ8yaG9JaSQBGqzGEXUyyxj5Dkp4xxbPUZzedjBSghLsoqfuUYukbit47dbkLT3PY3oRXViJGr/1/*),older(25920))))"
```
Now we bring those vars to our shell running `source .env`

To receiving you need to generate a new address, for this we need to pass the descriptor and a index, it starts with 0.
```
cargo run -- receive --index 0 --desc $DESC
```
Send some btc to that address, if you don't have tBTC you can get some from this [faucet](https://bitcoinfaucet.uo1.net/).

To see your balance just run
```
cargo run -- balance --desc $DESC --change $CHANGE
```
Now you want to send some coins to a other user address, for this you need to `build` a transaction.
```
cargo run -- build --desc $DESC --change $CHANGE --amount <amount in satoshsi> --destination <tBTC address>
```
We will see a [PSBT](https://github.com/bitcoin/bitcoin/blob/master/doc/psbt.md) that we need to sign and broadcast with the `send` command.
```
cargo run -- send --desc $DESC --psbt <psbt transaction>
```
That should show you a transaction Id, that means that it works 😀 Congratulations! you can die now! ⚰️🪦⚱️💀

## Origin, Authors, Copyright & Licenses

Unless otherwise noted (either in this [/README.md](./README.md) or in the file's header comments) the contents of this repository are Copyright © 2020 by Blockchain Commons, LLC, and are [licensed](./LICENSE) under the [spdx:BSD-2-Clause Plus Patent License](https://spdx.org/licenses/BSD-2-Clause-Patent.html).

In most cases, the authors, copyright, and license for each file reside in header comments in the source code. When it does not, we have attempted to attribute it accurately in the table below.

This table below also establishes provenance (repository of origin, permalink, and commit id) for files included from repositories that are outside of this repo. Contributors to these files are listed in the commit history for each repository, first with changes found in the commit history of this repo, then in changes in the commit history of their repo of their origin.

| File      | From                                                         | Commit                                                       | Authors & Copyright (c)                                | License                                                     |
| --------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ | ----------------------------------------------------------- |
| exception-to-the-rule.c or exception-folder | [https://github.com/community/repo-name/PERMALINK](https://github.com/community/repo-name/PERMALINK) | [https://github.com/community/repo-name/commit/COMMITHASH]() | 2020 Exception Author  | [MIT](https://spdx.org/licenses/MIT)                        |

### Libraries

External libraries are listed in [Cargo.toml](./Cargo.toml).

### Derived from BDK tutorials

This project is derived from many tutorials but specially:

- [Building a CLI Bitcoin wallet with BDK and Rust](https://github.com/futurepaul/bdk-cli-tutorial) — Repo which is a rust descriptor wallet tutorial, by [Paul Miller](https://github.com/futurepaul) and [BDK by example](https://www.bitcoindevkit-by-example.com/).


## Financial Support

`Morí` is a project of [Blockchain Commons](https://www.blockchaincommons.com/). We are proudly a "not-for-profit" social benefit corporation committed to open source & open development. Our work is funded entirely by donations and collaborative partnerships with people like you. Every contribution will be spent on building open tools, technologies, and techniques that sustain and advance blockchain and internet security infrastructure and promote an open web.

To financially support further development of `mori-cli` and other projects, please consider becoming a Patron of Blockchain Commons through ongoing monthly patronage as a [GitHub Sponsor](https://github.com/sponsors/BlockchainCommons). You can also support Blockchain Commons with bitcoins at our [BTCPay Server](https://btcpay.blockchaincommons.com/).

## Contributing

We encourage public contributions through issues and pull requests! Please review [CONTRIBUTING.md](./CONTRIBUTING.md) for details on our development process. All contributions to this repository require a GPG signed [Contributor License Agreement](./CLA.md).

### Discussions

The best place to talk about Blockchain Commons and its projects is in our GitHub Discussions areas.

[**Gordian System Discussions**](https://github.com/BlockchainCommons/Gordian/discussions). For users and developers of the Gordian system, including the Gordian Server, Bitcoin Standup technology, QuickConnect, and the Gordian Wallet. If you want to talk about our linked full-node and wallet technology, suggest new additions to our Bitcoin Standup standards, or discuss the implementation our standalone wallet, the Discussions area of the [main Gordian repo](https://github.com/BlockchainCommons/Gordian) is the place.

[**Wallet Standard Discussions**](https://github.com/BlockchainCommons/AirgappedSigning/discussions). For standards and open-source developers who want to talk about wallet standards, please use the Discussions area of the [Airgapped Signing repo](https://github.com/BlockchainCommons/AirgappedSigning). This is where you can talk about projects like our [LetheKit](https://github.com/BlockchainCommons/bc-lethekit) and command line tools such as [seedtool](https://github.com/BlockchainCommons/bc-seedtool-cli), both of which are intended to testbed wallet technologies, plus the libraries that we've built to support your own deployment of wallet technology such as [bc-bip39](https://github.com/BlockchainCommons/bc-bip39), [bc-slip39](https://github.com/BlockchainCommons/bc-slip39), [bc-shamir](https://github.com/BlockchainCommons/bc-shamir), [Sharded Secret Key Reconstruction](https://github.com/BlockchainCommons/bc-sskr), [bc-ur](https://github.com/BlockchainCommons/bc-ur), and the [bc-crypto-base](https://github.com/BlockchainCommons/bc-crypto-base). If it's a wallet-focused technology or a more general discussion of wallet standards,discuss it here.

[**Blockchain Commons Discussions**](https://github.com/BlockchainCommons/Community/discussions). For developers, interns, and patrons of Blockchain Commons, please use the discussions area of the [Community repo](https://github.com/BlockchainCommons/Community) to talk about general Blockchain Commons issues, the intern program, or topics other than the [Gordian System](https://github.com/BlockchainCommons/Gordian/discussions) or the [wallet standards](https://github.com/BlockchainCommons/AirgappedSigning/discussions), each of which have their own discussion areas.

### Other Questions & Problems

As an open-source, open-development community, Blockchain Commons does not have the resources to provide direct support of our projects. Please consider the discussions area as a locale where you might get answers to questions. Alternatively, please use this repository's [issues](./issues) feature. Unfortunately, we can not make any promises on response time.

If your company requires support to use our projects, please feel free to contact us directly about options. We may be able to offer you a contract for support from one of our contributors, or we might be able to point you to another entity who can offer the contractual support that you need.

### Credits

The following people directly contributed to this repository. You can add your name here by getting involved. The first step is learning how to contribute from our [CONTRIBUTING.md](./CONTRIBUTING.md) documentation.

| Name              | Role                | Github                                            | Email                                 | GPG Fingerprint                                    |
| ----------------- | ------------------- | ------------------------------------------------- | ------------------------------------- | -------------------------------------------------- |
| Francisco Calderón | Software engineer | [@grunch](https://github.com/grunch) | \<fjcalderon@gmail.com\> | 7178 F2F4 6986 871C F584 884C 151F 521C 8D3E 66D9 |

## Responsible Disclosure

We want to keep all of our software safe for everyone. If you have discovered a security vulnerability, we appreciate your help in disclosing it to us in a responsible manner. We are unfortunately not able to offer bug bounties at this time.

We do ask that you offer us good faith and use best efforts not to leak information or harm any user, their data, or our developer community. Please give us a reasonable amount of time to fix the issue before you publish it. Do not defraud our users or us in the process of discovery. We promise not to bring legal action against researchers who point out a problem provided they do their best to follow the these guidelines.

### Reporting a Vulnerability

Please report suspected security vulnerabilities in private via email to ChristopherA@BlockchainCommons.com (do not use this email for support). Please do NOT create publicly viewable issues for suspected security vulnerabilities.

The following keys may be used to communicate sensitive information to developers:

| Name              | Fingerprint                                        |
| ----------------- | -------------------------------------------------- |
| Christopher Allen | FDFE 14A5 4ECB 30FC 5D22  74EF F8D3 6C91 3574 05ED |

You can import a key by running the following command with that individual’s fingerprint: `gpg --recv-keys "<fingerprint>"` Ensure that you put quotes around fingerprints that contain spaces.
