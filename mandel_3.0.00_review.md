Mandel 3.0.0 code review
========================

Fractally team started working on a new EOSIO distribution in the end
of 2021, as there was a demand for action: The EOSIO 2.1 version
delivered by Block One had a number of compatibility breaking changes
and features that did not perform as planned.

Development of a new branch of EOSIO was sponsored by EOS Network
Foundation, with the code name "mandel", aiming to boost the
development and improve the quality of software. The first release
candidate [Mandel
v3.0.0-rc1](https://github.com/eosnetworkfoundation/mandel/releases/tag/v3.0.0-rc1)
was published on February 1st 2022.

The purpose of this document is to give an independent view on new
development, and to provide an argumentation for public blockchains to
adopt one or the other software branches.



Changes worth noting
--------------------

WABT virtual machine is no longer supported. `eos-vm-jit` is set as
default VM.

Boost 1.67 is no longer a requirement. The software should work with
newer versions of Boost.



Git history review
------------------

The [Mandel repository on
github](https://github.com/eosnetworkfoundation/mandel) is a clone of
[EOSIO repository](https://github.com/EOSIO/eos) with the `main`
branch stemming from `v2.0.13` tag. Original tags were not preserved,
and the branching point is tagged as `historical-v2.0.13-base`.

Backports from EOSIO repository were squashed, because /Todd Fleming/
in many cases, code we needed had history that was heavily tangled
with code we needed to avoid. e.g. transaction pruning.

Commit `285335af`: all dependency submodules are relocated into ENF
organization repositories.

Commit `7ecc83d7`: PR#1 merged. This upgrades the code to the latest
FC library from 2.1 branch.

Commit `c9e94e92`: PR#2, backport 2.1 eos-vm. This brings the EOS-VM
from EOSIO 2.1, and *removes support for wabt*.

Commit `cd6c83bc`: PR#3, backport 29233b8 ship. This adds changes in
state history from 2.0.x EOSIO branch.

Commit `e287d52a`: PR#4, backport 2.1 chainbase. Chainbase is the
database engine that is storing the whole blockchain state, tracks its
changes in reversible blocks, and rolls back on microforks. This PR is
merging the updated version of chainbase from EOSIO 2.1.

Commit `18d80f22`: PR#5, cicd system for automated testing.

Commit `1b701a20`: PR#6, backport 2.1 tester cmake. Among other
changes, it removes the strict dependency on Boost 1.67.



