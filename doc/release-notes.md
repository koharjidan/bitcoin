<<<<<<< HEAD
(note: this is a temporary file, to be added-to by anybody, and moved to
release-notes at release time)

Block file backwards-compatibility warning
===========================================

Because release 0.10.0 makes use of headers-first synchronization and parallel
block download, the block files and databases are not backwards-compatible
with older versions of Bitcoin Core:

* Blocks will be stored on disk out of order (in the order they are
received, really), which makes it incompatible with some tools or
other programs. Reindexing using earlier versions will also not work
anymore as a result of this.

* The block index database will now hold headers for which no block is
stored on disk, which earlier versions won't support.

If you want to be able to downgrade smoothly, make a backup of your entire data
directory. Without this your node will need start syncing (or importing from
bootstrap.dat) anew afterwards.

This does not affect wallet forward or backward compatibility.

Transaction fee changes
=======================

This release automatically estimates how high a transaction fee (or how
high a priority) transactions require to be confirmed quickly. The default
settings will create transactions that confirm quickly; see the new
'txconfirmtarget' setting to control the tradeoff between fees and
confirmation times.

Prior releases used hard-coded fees (and priorities), and would
sometimes create transactions that took a very long time to confirm.

Statistics used to estimate fees and priorities are saved in the
data directory in the `fee_estimates.dat` file just before
program shutdown, and are read in at startup.

New Command Line Options
---------------------------

- `-txconfirmtarget=n` : create transactions that have enough fees (or priority)
so they are likely to confirm within n blocks (default: 1). This setting
is over-ridden by the -paytxfee option.

New RPC methods
----------------

- `estimatefee nblocks` : Returns approximate fee-per-1,000-bytes needed for
a transaction to be confirmed within nblocks. Returns -1 if not enough
transactions have been observed to compute a good estimate.

- `estimatepriority nblocks` : Returns approximate priority needed for
a zero-fee transaction to confirm within nblocks. Returns -1 if not
enough free transactions have been observed to compute a good
estimate.

RPC access control changes
==========================================

Subnet matching for the purpose of access control is now done
by matching the binary network address, instead of with string wildcard matching.
For the user this means that `-rpcallowip` takes a subnet specification, which can be

- a single IP address (e.g. `1.2.3.4` or `fe80::0012:3456:789a:bcde`)
- a network/CIDR (e.g. `1.2.3.0/24` or `fe80::0000/64`)
- a network/netmask (e.g. `1.2.3.4/255.255.255.0` or `fe80::0012:3456:789a:bcde/ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff`)

An arbitrary number of `-rpcallow` arguments can be given. An incoming connection will be accepted if its origin address
matches one of them.

For example:

| 0.9.x and before                           | 0.10.x                                |
|--------------------------------------------|---------------------------------------|
| `-rpcallowip=192.168.1.1`                  | `-rpcallowip=192.168.1.1` (unchanged) |
| `-rpcallowip=192.168.1.*`                  | `-rpcallowip=192.168.1.0/24`          |
| `-rpcallowip=192.168.*`                    | `-rpcallowip=192.168.0.0/16`          |
| `-rpcallowip=*` (dangerous!)               | `-rpcallowip=::/0`                    |

Using wildcards will result in the rule being rejected with the following error in debug.log:

    Error: Invalid -rpcallowip subnet specification: *. Valid are a single IP (e.g. 1.2.3.4), a network/netmask (e.g. 1.2.3.4/255.255.255.0) or a network/CIDR (e.g. 1.2.3.4/24).

RPC Server "Warm-Up" Mode
=========================

The RPC server is started earlier now, before most of the expensive
intialisations like loading the block index.  It is available now almost
immediately after starting the process.  However, until all initialisations
are done, it always returns an immediate error with code -28 to all calls.

This new behaviour can be useful for clients to know that a server is already
started and will be available soon (for instance, so that they do not
have to start it themselves).
=======
Bitcoin Core version 0.9.3 is now available from:

  https://bitcoin.org/bin/0.9.3/

This is a new minor version release, bringing only bug fixes and updated
translations. Upgrading to this release is recommended.

Please report bugs using the issue tracker at github:

  https://github.com/bitcoin/bitcoin/issues

Upgrading and downgrading
==========================

How to Upgrade
--------------

If you are running an older version, shut it down. Wait until it has completely
shut down (which might take a few minutes for older versions), then run the
installer (on Windows) or just copy over /Applications/Bitcoin-Qt (on Mac) or
bitcoind/bitcoin-qt (on Linux).

If you are upgrading from version 0.7.2 or earlier, the first time you run
0.9.3 your blockchain files will be re-indexed, which will take anywhere from 
30 minutes to several hours, depending on the speed of your machine.

Downgrading warnings
--------------------

The 'chainstate' for this release is not always compatible with previous
releases, so if you run 0.9.x and then decide to switch back to a
0.8.x release you might get a blockchain validation error when starting the
old release (due to 'pruned outputs' being omitted from the index of
unspent transaction outputs).

Running the old release with the -reindex option will rebuild the chainstate
data structures and correct the problem.

Also, the first time you run a 0.8.x release on a 0.9 wallet it will rescan
the blockchain for missing spent coins, which will take a long time (tens
of minutes on a typical machine).

0.9.3 Release notes
=======================

RPC:
- Avoid a segfault on getblock if it can't read a block from disk
- Add paranoid return value checks in base58

Protocol and network code:
- Don't poll showmyip.com, it doesn't exist anymore
- Add a way to limit deserialized string lengths and use it
- Add a new checkpoint at block 295,000
- Increase IsStandard() scriptSig length
- Avoid querying DNS seeds, if we have open connections
- Remove a useless millisleep in socket handler
- Stricter memory limits on CNode
- Better orphan transaction handling
- Add `-maxorphantx=<n>` and `-maxorphanblocks=<n>` options for control over the maximum orphan transactions and blocks

Wallet:
- Check redeemScript size does not exceed 520 byte limit
- Ignore (and warn about) too-long redeemScripts while loading wallet

GUI:
- fix 'opens in testnet mode when presented with a BIP-72 link with no fallback'
- AvailableCoins: acquire cs_main mutex
- Fix unicode character display on MacOSX

Miscellaneous:
- key.cpp: fail with a friendlier message on missing ssl EC support
- Remove bignum dependency for scripts
- Upgrade OpenSSL to 1.0.1i (see https://www.openssl.org/news/secadv_20140806.txt - just to be sure, no critical issues for Bitcoin Core)
- Upgrade miniupnpc to 1.9.20140701
- Fix boost detection in build system on some platforms

Credits
--------

Thanks to everyone who contributed to this release:

- Andrew Poelstra
- Cory Fields
- Gavin Andresen
- Jeff Garzik
- Johnathan Corgan
- Julian Haight
- Michael Ford
- Pavel Vasin
- Peter Todd
- phantomcircuit
- Pieter Wuille
- Rose Toomey
- Ruben Dario Ponticelli
- shshshsh
- Trevin Hofmann
- Warren Togami
- Wladimir J. van der Laan
- Zak Wilcox

As well as everyone that helped translating on [Transifex](https://www.transifex.com/projects/p/bitcoin/).
>>>>>>> 5b9f78d69ccf189bebe894b1921e34417103a046
