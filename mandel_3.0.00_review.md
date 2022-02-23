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



Important notes
---------------

WABT virtual machine is no longer supported. `eos-vm-jit` is set as
default VM.

Boost 1.67 is no longer a requirement. The software should work with
newer versions of Boost.

A Mandel nodeos binary is not a drop-in for EOSIO, because of a newer
chainbase version and changes in `global_property_object`. A node
needs to start from a snapshot, because the Mandel state is
incompatible with EOSIO-2.0. The blocks log is compatible with
EOSIO-2.0.

PR #25 and #26 are changing many files globally across the whole
codebase, so they make it unpractical to cherry-pick any new updates
in `release/2.0.x` branch in B1 repository. They are, however,
compatible with 2.1 code, so new features can be taken from there.


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

Commit `8748f596`: PR#7, backport 2.1 ACTION_RETURN_VALUE. This is a
*consensus upgrade feature*. The following changes become active even
before the feature is activated: snapshot format version 5;
`action_trace` object gets a new field: `return_value`; state history
plugin starts writing new types: `action_trace_v1`, `chain_config_v1`.

Commit `3e102d52`: PR#8, Backport CONFIGURABLE_WASM_LIMITS from
2.1. `kv_database_config` structure is pulled from the upstream, but
not used. This allows starting the node from V5 snapshots, and can be
removed in later releases.

Commit `3dbbf953`: PR#11, Backport BLOCKCHAIN_PARAMETERS from 2.1. It
introduces a new *consensus upgrade feature* that allows smart
contracts read the contents of `chain_config_v*`. It also introduces
`chain_config_v1` with `max_action_return_value_size` parameter.

Commit `f602dfa5`: PR #12, get_code_hash. *Consensus upgrade feature*,
new feature: GET_CODE_HASH bringing a new intrinsic that returns a
contract hash.

Commit `0bc4b188`: PR#14, backport 2.1 missing wasm tests. Eos-vm 2.1
introduces a bugfix in memory access, and this PR is adding unit tests
for this bug.

Commit `c3e89c65`: PR#15, ship support for wasm_config. *Change in
state history format*: `global_property_v1` has an additional field
wasm_config as a binary extension. Also a new type `wasm_config_v0` is
defined.

Commit `fb5d7b59`: PR#16, backport cbbab053a eos-vm (Switch to
develop-boxed's eos-vm). The feature allows collecting profiling data
for a selected number of accounts, and that will help debugging and
optimizing the contracts. In its current state, it writes the traces
in current directory, and a correspoding [issue is
created](https://github.com/eosnetworkfoundation/mandel/issues/41).

Commit `81433708`: PR#17, enhancements for clsdk. Adds
`replace_producer_keys()` and `replace_account_keys()` which will
allow building and integrating debugger utilities.

Commit `b5b2cea0`: PR#18, Log onblock errors. Up to now it was
difficult to troubleshoot errors in `onblock` actions, and this patch
solves that.

Commit `979a4493`: PR#19, Backport 2.1 enhanced snapshot
reporting. More informative fields in `/v1/producer/create_snapshot`
output.

Commit `0d125b55`: PR#24, SHiP compressed deltas now exceeds
4GB. This patch resolves the problem with starting a WAX node from a
modern snapshot, because compressed table deltas exceed 4GB.

Commit `748131b9`: PR#25, Replace N macro with operator ""_n. This is
a big cleanup in the code, removing the use of old macro for
names. *It might make it difficult to cherry-pick patches from
release/2.0.x branch in B1 repository*.

Commit `878b756c`: PR#26, Remove fc::optional and
fc::static_variant. Another big change which makes it difficult to
follow new patches in release/2.0.x branch.

Commit `368045ba`: PR#28, Increment codegen_version. It prevents the
OC code cache from picking up the cache of 2.1.

Commit `1a8407b3`: Mandel version `v3.0.0-rc1`


Further development
-------------------

Kevin Heifner has provided a list of recent changes in EOSIO
repository that are worth evaluating for backport into Mandel:

  1. Remove `history_plugin`.  The plugin is long deprecated,
  unstable, and unusable.

  2. SHiP enhancements. Many performance improvements were made in the
  `develop-boxed` branch.

  3. `rodeos` bugfixes in develop-boxed branch. Also related,
  [PR#10625](https://github.com/EOSIO/eos/pull/10625). It is yet to be
  evaluated if Mandel needs it.

  4. `net_plugin` bugfixes and improvements (develop branch of
  net_plugin). 2.0 branch:
  [PR#10651](https://github.com/EOSIO/eos/pull/10651),
  [PR#10764](https://github.com/EOSIO/eos/pull/10764),
  [PR#10961](https://github.com/EOSIO/eos/pull/10961),
  [PR#11066](https://github.com/EOSIO/eos/pull/11066).

  5. `net_plugin` ssl support & security group (in 2.2 branch). Needs
  evaluation.

  6. Transaction trace logging (in 2.1). There are additional loggers
  in `producer_plugin` for logging full traces of transactions.

  7. [New command line option
  --snapshot-to-json](https://github.com/EOSIO/eos/pull/11058). This
  allows exporting the contents of a snapshot into a JSON file which
  can be used for data analysis and testing.

  8. [Fix boost:beast
  vulnerability](https://github.com/EOSIO/eos/pull/10981). Boost::beast
  uses hardcoded zlib which was vulnerable to CVE-2016-9840. This
  updates unpinned builds, and build scripts to use newer version of
  Boost, and for pinned builds to apply patch to address the
  vulnerability.

  9. [audit tool](https://github.com/EOSIO/eos/pull/10772). The PR
  adds a new HTTP RPC endpoint, `get_all_accounts`. The need for this
  tool is doubtdul, as there are more efficient ways to retrieve all
  accounts.

  10. [amqp_trx_plugin](https://github.com/EOSIO/eos/tree/develop/plugins/amqp_trx_plugin). The
  plugin is exporting raw transaction traces (in the same form as
  state history plugin does) into an AMQP socket. It needs a filter by
  accounts or contracts before being usable for production.

  11. [Add map mode options for EOS VM OC code
  cache](https://github.com/EOSIO/eos/pull/10683). The change is
  improving the performance in EOSVM optimized compiler.

  12. [Add new loggers for dumping transaction
  trace](https://github.com/EOSIO/eos/pull/10051). The feature is
  adding the ability to log full details of accepted or failed
  transactions.

  13. [Handle special case in
  trim_blocklog_front](https://github.com/EOSIO/eos/pull/10370). This
  is a bugfix for block trimming utility.

  14. [bug in https](https://github.com/EOSIO/eos/pull/10767).

  15. [Add total CPU and NET to
  get_info](https://github.com/EOSIO/eos/pull/10932).

