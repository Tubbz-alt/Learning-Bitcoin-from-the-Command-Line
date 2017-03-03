# 3 - Playing with Bitcoin

> **NOTE:** This is a draft in progress, so that I can get some feedback from early reviewers. It is not yet ready for learning.

This document explains how to begin using Bitcoin from the command line. It presumes that you have a VPS that you installed bitcoin on and that is running bitcoind. It also presumes that you are connected to testnet, allowing for access to bitcoins without using real funds. We explained how to do this either by hand in [2A - Setting up a Bitcoin-Core VPS by Hand](./2A_Setting_Up_a_Bitcoin-Core_VPS_by_Hand.md) or by using a Linode StackScript at Linode.com in [2B - Setting up a Bitcoin-Core VPS with StackScript](./2B_Setting_Up_a_Bitcoin-Core_VPS_with_StackScript.md).

## Part Zero: Getting Started with Bitcoin

Before you start playing with Bitcoin, you should ensure that everything is setup correctly.

### Check Your Aliases

The Bitcoin setup docs suggest creating a set of aliases. In case you didn't run through those docs, you can create them for your main Bitcoin user with the following command:

```
cat >> ~/.bash_profile <<EOF
alias btcdir="cd ~/.bitcoin/" #linux default bitcoind path
        # alias btcdir="cd ~/Library/Application\ Support/Bitcoin/" #mac default bitcoind path
alias bc="bitcoin-cli"
alias bd="bitcoind"
alias btcinfo='bitcoin-cli getinfo | egrep "\"version\"|balance|blocks|connections|errors"'
alias btcblock="echo \\\`bitcoin-cli getblockcount 2>&1\\\`/\\\`wget -O - http://blockexplorer.com/testnet/q/getblockcount 2> /dev/null | cut -d : -f2 | rev | cut -c 2- | rev\\\`"
EOF
```

> **WARNING:** The btcblock alias will not work correctly if you try to place it in your .bash_profile by hand, rather than using the "cat" command as suggested. If you want to enter it by hand, you need to adjust the number of backslashes (usually from three each to one each), so make sure you know what you're doing if you aren't entering the commands exactly as shown.

Note that these aliases includes shortcuts for running `bitcoin-cli`, for running `bitcoind`, and for going to the Bitcoin directory. These aliases are mainly meant to make your life easier. We suggest you create other aliases to ease your use of frequent commands (and arguments) and to minimize errors. Aliases of this sort can even more useful if you have a complex setup where you regularly run commands associated with Mainnet, with Testnet, _and_ with Regtest.

With that said, use of these aliases in _this_ document might accidentally obscure the core lessons being taught about Bitcoin, so the only aliases directly used here are `btcinfo` and `btcblock`, because they encapsulate much longer and more complex commands. Otherwise, we show the full commands; adjust for your own use as appropriate.

> **TESTNET vs MAINNET:** Remember that this tutorial generally assumes that you are using testnet. Notes like this will comment on how things might be different over on Mainnet. In this case, the `btcblock` alias needs to be slightly different. On testnet, you can look up the current block count with the complex command "wget -O - http://blockexplorer.com/testnet/q/getblockcount 2> /dev/null | cut -d : -f2 | rev | cut -c 2- | rev"; on mainnet, you use the much simpler "wget -O - http://blockchain.info/q/getblockcount 2>/dev/null".

### Run Bitcoind

You'll be accessing the Bitcoin network with the `bitcoin-cli` command. However, bitcoind _must_ be running to use bitcoin-cli, as the bitcoin-cli sends JSON-RPC commands to the bitcoind. If you used our standard setup, bitcoind should already be up and running. You can double check by looking at the process table.
```
$ ps auxww | grep bitcoind
user1    29360 11.5 39.6 2676812 1601416 ?     SLsl Feb23 163:42 /usr/local/bin/bitcoind -daemon
```
If it's not running, you'll want to run "/usr/local/bin/bitcoind -daemon" by hand and also place it in your crontab, as explained in [2A - Setting up a Bitcoin-Core VPS by Hand](./2A_Setting_Up_a_Bitcoin-Core_VPS_by_Hand.md).

### Verify Your Blocks

You should have the whole blockchain (or the whole pruned blockchain) ready before you start playing. Just run the `btcblock` alias to see if it's all loaded. You'll see two numbers, which tell you how many blocks have loaded out of how many total.

If the two numbers aren't the same, as shown in this testnet example, you should wait:
```
$ btcblock
973212/1090099
```
Total time can take from several hours to a day, depending on your setup.

If the two numbers are the same, as shown in this testnet example, you're fully loaded:
```
$ btcblock
1090099/1090099
```

### Check Your Directory

If your bitcoind is running and you've downloaded all the blocks, you ready to go. You could move right on to Part 1. However, you may want to be aware of other Bitcoin resources. To start with, take a look at your ~/.bitcoin directory.

The main directory just contains your config file and the testnet directory:
```
$ ls ~/.bitcoin
bitcoin.conf  testnet3
```
The testnet3 directory then contains all of the guts:
```
$ ls ~/.bitcoin/testnet3
banlist.dat   blocks	  database  debug.log  wallet.dat
bitcoind.pid  chainstate  db.log    peers.dat
```
You shouldn't mess with most of these files and directories — particularly not the blocks and chainstate directories, which contain all of the blockchain data. However, do take careful note of the db.log and debug.log file, which you should refer to if you ever have problems with your setup.

> **TESTNET vs MAINNET:** If you're using mainnet, then _everything_ will instead be placed in the main ~/.bitcoin directory. These various setups _do_ elegantly stack, so if you are using mainnet, testnet, and regtest, you'll find that ~/.bitcoin contains your config file and your mainnet data, ~/.bitcoin/testnet3 contains your testnet data, and ~/.bitcoin/regtest contains your regtest data.

### Get Help

Most of your work will be done with the "bitcoin-cli" command. If you ever want more information on its usage, just run the help argument. Without any other arguments, it shows you ever possible command:
```
$ bitcoin-cli help
== Blockchain ==
getbestblockhash
getblock "hash" ( verbose )
getblockchaininfo
getblockcount
getblockhash index
getblockheader "hash" ( verbose )
getchaintips
getdifficulty
getmempoolancestors txid (verbose)
getmempooldescendants txid (verbose)
getmempoolentry txid
getmempoolinfo
getrawmempool ( verbose )
gettxout "txid" n ( includemempool )
gettxoutproof ["txid",...] ( blockhash )
gettxoutsetinfo
verifychain ( checklevel numblocks )
verifytxoutproof "proof"

== Control ==
getinfo
help ( "command" )
stop

== Generating ==
generate numblocks ( maxtries )
generatetoaddress numblocks address (maxtries)

== Mining ==
getblocktemplate ( TemplateRequest )
getmininginfo
getnetworkhashps ( blocks height )
prioritisetransaction <txid> <priority delta> <fee delta>
submitblock "hexdata" ( "jsonparametersobject" )

== Network ==
addnode "node" "add|remove|onetry"
clearbanned
disconnectnode "node" 
getaddednodeinfo dummy ( "node" )
getconnectioncount
getnettotals
getnetworkinfo
getpeerinfo
listbanned
ping
setban "ip(/netmask)" "add|remove" (bantime) (absolute)

== Rawtransactions ==
createrawtransaction [{"txid":"id","vout":n},...] {"address":amount,"data":"hex",...} ( locktime )
decoderawtransaction "hexstring"
decodescript "hex"
fundrawtransaction "hexstring" ( options )
getrawtransaction "txid" ( verbose )
sendrawtransaction "hexstring" ( allowhighfees )
signrawtransaction "hexstring" ( [{"txid":"id","vout":n,"scriptPubKey":"hex","redeemScript":"hex"},...] ["privatekey1",...] sighashtype )

== Util ==
createmultisig nrequired ["key",...]
estimatefee nblocks
estimatepriority nblocks
estimatesmartfee nblocks
estimatesmartpriority nblocks
signmessagewithprivkey "privkey" "message"
validateaddress "bitcoinaddress"
verifymessage "bitcoinaddress" "signature" "message"

== Wallet ==
abandontransaction "txid"
addmultisigaddress nrequired ["key",...] ( "account" )
addwitnessaddress "address"
backupwallet "destination"
dumpprivkey "bitcoinaddress"
dumpwallet "filename"
encryptwallet "passphrase"
getaccount "bitcoinaddress"
getaccountaddress "account"
getaddressesbyaccount "account"
getbalance ( "account" minconf includeWatchonly )
getnewaddress ( "account" )
getrawchangeaddress
getreceivedbyaccount "account" ( minconf )
getreceivedbyaddress "bitcoinaddress" ( minconf )
gettransaction "txid" ( includeWatchonly )
getunconfirmedbalance
getwalletinfo
importaddress "address" ( "label" rescan p2sh )
importprivkey "bitcoinprivkey" ( "label" rescan )
importprunedfunds
importpubkey "pubkey" ( "label" rescan )
importwallet "filename"
keypoolrefill ( newsize )
listaccounts ( minconf includeWatchonly)
listaddressgroupings
listlockunspent
listreceivedbyaccount ( minconf includeempty includeWatchonly)
listreceivedbyaddress ( minconf includeempty includeWatchonly)
listsinceblock ( "blockhash" target-confirmations includeWatchonly)
listtransactions ( "account" count from includeWatchonly)
listunspent ( minconf maxconf  ["address",...] )
lockunspent unlock ([{"txid":"txid","vout":n},...])
move "fromaccount" "toaccount" amount ( minconf "comment" )
removeprunedfunds "txid"
sendfrom "fromaccount" "tobitcoinaddress" amount ( minconf "comment" "comment-to" )
sendmany "fromaccount" {"address":amount,...} ( minconf "comment" ["address",...] )
sendtoaddress "bitcoinaddress" amount ( "comment" "comment-to" subtractfeefromamount )
setaccount "bitcoinaddress" "account"
settxfee amount
signmessage "bitcoinaddress" "message"
```
You can also type "bitcoin help [command]" to get even more extensive info on that command. For example:
```
$ bitcoin-cli help getmininginfo
getmininginfo

Returns a json object containing mining-related information.
Result:
{
  "blocks": nnn,             (numeric) The current block
  "currentblocksize": nnn,   (numeric) The last block size
  "currentblockweight": nnn, (numeric) The last block weight
  "currentblocktx": nnn,     (numeric) The last block transaction
  "difficulty": xxx.xxxxx    (numeric) The current difficulty
  "errors": "..."            (string) Current errors
  "networkhashps": nnn,      (numeric) The network hashes per second
  "pooledtx": n              (numeric) The size of the mempool
  "testnet": true|false      (boolean) If using testnet or not
  "chain": "xxxx",           (string) current network name as defined in BIP70 (main, test, regtest)
}

Examples:
> bitcoin-cli getmininginfo 
> curl --user myusername --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getmininginfo", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:8332/
```

### Optional: Know Your Server Types

> **TESTNET vs MAINNET:** When you set up your node, you choose to create it as either a Mainnet, Testnet, or Regtest node. Though this document presumes a testnet setup, it's worth understanding how you might access and use the other setup types — even all on the same machine!

The type of setup is mainly controlled through the ~/.bitcoin/bitcoin.conf file. If you're running testnet, it probably contains this line:
```
testnet=1
```
While if you're running regtest, it probably contains this line:
```
regtest=1
```
However, if you want to run several different sorts of nodes simultaneously, you should leave the testnet (or regtest) flag out of your configuration file. You can then choose whether you're using the mainnet, the testnet, or your regtest every time you run bitcoind or bitcoin-cli.

Here's a set of aliases that would make that easier by creating a specific alias for starting and stopping the bitcoind, for going to the bitcoin directory, and for running bitcoin-cli, for each of the mainnet (which has no extra flags), the testnet (which is -testnet), or your regtest (which is -regtest).
```
alias bcstart="bitcoind -daemon"
alias btstart="bitcoind -testnet -daemon"
alias brstart="bitcoind -regtest -daemon"

alias bcstop="bitcoin-cli stop"
alias btstop="bitcoin-cli -testnet stop"
alias brstop="bitcoin-cli -regtest -stop"

alias bcdir="cd ~/.bitcoin/" #linux default bitcoin path
alias btdir="cd ~/.bitcoin/testnet" #linux default bitcoin testnet path
alias brdir="cd ~/.bitcoin/regtest" #linux default bitcoin regtest path

alias bc="bitcoin-cli"
alias bt="bitcoin-cli -testnet"
alias br="bitcoin-cli -regtest"
```
For even more complexity, you could have each of your 'start' aliases use the -conf flag to load configuration from a different file. This goes far beyond the scope of this tutorial, but we offer it as a starting point for when your explorations of Bitcoin reach the next level.

### Summary: Getting Started

Before you start playing with bitcoin, you should make sure that the bitcoind is running and that all the blocks have been downloaded. You might get additional help from the `bitcoin-cli help` command or from files in the ~/.bitcoin directory.

## Part One: Setting Up Your Wallet

You're now ready to start working with Bitcoin. To begin with, you'll need to create an address for receiving funds.

### Create an Address

The first thing you need to do is create an address for receiving payments. This is done with the `bitcoin-cli getnewaddress` command. Remember that if you want more information on this command, you should type `bitcoin-cli help getnewaddress`.

Theoretically, you could run it just by typing it on the command line:
```
$ bitcoin-cli getnewaddress
n4cqjJE6fqcmeWpftygwPoKMMDva6BpyHf
```
However, this isn't best practice. Though your address _is_ saved away in your wallet for your future use, you could easily make a mistake if you were retyping or cutting it somewhere else. And then you're telling people to send money to somewhere else (or to nowhere!). So instead we suggest a best practice, which is meant to minimize address mistakes.

#### BEST PRACTICES: Use Variables to Capture Addresses

Instead, use your shell's built-in variables to capture your address.
```
$ unset NEW_ADDRESS_1
$ NEW_ADDRESS_1=$(bitcoin-cli getnewaddress)
```
These commands clear the NEW_ADDRESS_1 variable, then fill it with the results of the `bitcoin-cli getnewaddress` command.

You can then use your shell's `echo` command to look at your (new) address:
```
$ echo $NEW_ADDRESS_1
n4cqjJE6fqcmeWpftygwPoKMMDva6BpyHf
```
Note that this address begins with an "n" (or sometimes an "m"). This signifies that this is a testnet address. 

> **TESTNET vs MAINNET:** The equivalent mainnet address would start with a 1.

We'll use this same technique for other bitcoin-cli results in the future; note that you could do it all by hand, instead of piping stuff in and out of variables ... but we really don't suggest it.

### Capture the Private Key

The address lets you receive bitcoins, but to spend them, you'll need the address' private key. Again, this is all stored in your wallet, and it's not something that you usually need to worry about. But, if you do need it for some purpose (such as proving ownership from some other machine), then you can access the private key with the `bitcoin-cli dumpprivkey` command.
```
$ bitcoin-cli dumpprivkey "$NEW_ADDRESS_1"
cW4s4MdW7BkUmqiKgYzSJdmvnzq8QDrf6gszPMC7eLmfcdoRHtHh
```
We opted not to put this in a variable, because it's not something you want floating around ...

### Sign a Message

You can also sign a message using your address. This verifies that the message for the address was signed by the person who knew the address' private key. You do this with `bitcoin-cli signmessage [address] [message]`. For example:
```
$ NEW_SIG_1=$(bitcoin-cli signmessage $NEW_ADDRESS_1 "Hello, World")
$ echo $NEW_SIG_1
H3yMBZaFeSmG2HgnH38dImzZAwAQADcOiMKTC1fryoV6Y93BelqzDMTCqNcFoik86E8qHa6o3FCmTsxWD7Wa5YY=
```
 A recipient can verify the signature if he inputs the address, the signature, and the message.
```
$ bitcoin-cli verifymessage "n4cqjJE6fqcmeWpftygwPoKMMDva6BpyHf" "H3yMBZaFeSmG2HgnH38dImzZAwAQADcOiMKTC1fryoV6Y93BelqzDMTCqNcFoik86E8qHa6o3FCmTsxWD7Wa5YY=" "Hello, World"
true
```
If some black hat was making up signatures, they'd instead get a negative result:
```
$ bitcoin-cli verifymessage "n4cqjJE6fqcmeWpftygwPoKMMDva6BpyHf" "FAKEBZaFeSmG2HgnH38dImzZAwAQADcOiMKTC1fryoV6Y93BelqzDMTCqNcFoik86E8qHa6o3FCmTsxWD7Wa5YY=" "Hello, World"
false
```

### Summary: Setting Up Your Wallet

You need to create an address to receive funds. We suggest using variables to capture your address, to ensure that you give out the correct address in the future. Based on that address, you can also access a private key and sign messages.

## Part Two: Receiving a Transaction

You're now ready to receive some money at the new address you set up.

### Get Some Money

To do anything more, you need to get some money. On testnet this is done through faucets. Since the money is all pretend, you just go to a faucet, request some money, and it's sent over to you. We suggest using the faucet at http://tpfaucet.appspot.com/. If it's not available for some reason, search for "bitcoin testnet faucet", and you should find others. We suggest https://testnet.manu.backend.hamburg/faucet as an excellent alternative.

To use a faucet, you'll usually need to go to a URL and copy and paste in your address. Yes, this violates our Best Practices, but that's how the faucets tend to work.

> **TESTNET vs MAINNET:** Sadly, there are no faucets in real life. If you were playing on the mainnet, you'd need to go and actually buy bitcoins at a bitcoin exchange or ATM, or you'd need to get someone to send them to you. Testnet life is much easier.

### Verify Your Money

After you've requested your money, you should be able to verify it with the 'bitcoin-cli getbalance' command:
```
$ bitcoin-cli getbalance
0.00000000
```
But wait, there's no balance yet!? 

Welcome to the world of Bitcoin latency. Transactions are transmitted across the network and gathered into blocks by miners. If you don't see a balance, your block hasn't been made yet. However, `bitcoin-cli get unconfirmedbalance` should still show it as long as the initial transaction has been created:
```
$ bitcoin-cli getunconfirmedbalance
0.47000000
```
If that's still showing a zero too, you're probably moving through this tutorial too fast. Wait a second. The coins should show up unconfirmed, then rapidly move to confirmed. However, if your "getbalance" and your "getunconfirmedbalance" both still show zero in ten minutes, then there's probably something wrong with the faucet, and you'll need to pick another. (Do note that a coin can move from unconfirmedbalance to confirmedbalance almost immediately, so make sure you check both.)

> **WARNING:** After a block is built and confirmed, another block is built on top of it, and another ... Because this is a stochastic process, there's some chance for reversal when a block is still new. Thus, a block has to be buried several blocks deep in a chain before you can feel total confident in your funds. Each of those blocks tends to be built in an average of 10 minutes ... so it usually takes about an hour for a confirmed transaction to receive full confidence.

### Gain Confidence in Your Money

You can use `bitcoin-cli getbalance "\*" [n]` to see if a confirmed balance is 'n' blocks deep.

The following shows that our transaction has been confirmed one time, but not twice:
```
$ bitcoin-cli getbalance "*" 1
0.47000000
$ bitcoin-cli getbalance "*" 2
0.00000000
```
Obviously, every ten minutes or so this depth will increase.

Of course, on the testnet, no one is that worried about how reliable your funds are. You'll be able to spend your money as soon as it's confirmed.

#### Verify Your Wallet

You can also access all of this information with the `bitcoin-cli getwalletinfo` command:
```
$ bitcoin-cli getwalletinfo
{
  "walletversion": 130000,
  "balance": 0.47000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 0.00000000,
  "txcount": 1,
  "keypoololdest": 1488216266,
  "keypoolsize": 100,
  "paytxfee": 0.00000000,
  "hdmasterkeyid": "b91d5ec57d5ae3e90fff50d12e819429b6496b94"
}
```

### Discover Your Transaction ID

Your money came into you via a transaction. You can discover that transactionid (txid) with the `bitcoin-cli listtransactions` command:
```
$ bitcoin-cli listtransactions
[
  {
    "account": "",
    "address": "n4cqjJE6fqcmeWpftygwPoKMMDva6BpyHf",
    "category": "receive",
    "amount": 0.47000000,
    "label": "",
    "vout": 0,
    "confirmations": 2,
    "blockhash": "00000000fa4fdd22a2c33c6200b68239939ad65af3f1a48ecea25f8200f5d66b",
    "blockindex": 45,
    "blocktime": 1488307692,
    "txid": "88e5d5f3077517d76f5a61491fa52e6aaae078c52bc62d849f09507ef0cfada2",
    "walletconflicts": [
    ],
    "time": 1488307692,
    "timereceived": 1488307696,
    "bip125-replaceable": "no"
  }
]
```
This shows one transaction ("88e5d5f3077517d76f5a61491fa52e6aaae078c52bc62d849f09507ef0cfada2") that was received ("receive") by a specific address in my wallet ("n4cqjJE6fqcmeWpftygwPoKMMDva6BpyHf") for a specific amount ("0.47000000").

You can access similar information with the `bitcoin-cli listunspent` command, but it only shows the transactions for the money that you haven't spent. These are also called UTXOs, and will be vitally important when you're sending money back out into the Bitcoin world:
```
$ bitcoin-cli listunspent
[
  {
    "txid": "88e5d5f3077517d76f5a61491fa52e6aaae078c52bc62d849f09507ef0cfada2",
    "vout": 0,
    "address": "n4cqjJE6fqcmeWpftygwPoKMMDva6BpyHf",
    "account": "",
    "scriptPubKey": "76a914fd67e8a7c7813e7a5c376eb71074f373d924d96888ac",
    "amount": 0.47000000,
    "confirmations": 3,
    "spendable": true,
    "solvable": true
  }
]
```
Note that bitcoins are not just a homogeneous mess of cash jammed into your pocket. Each individual transaction that you receive or that you send is placed in the immutable blockchain ledger, in a block. You can see all of those when you look at your transactions. Note also that this means that bitcoin spending isn't quite as anonymous as you'd think. Though the addresses are fairly private, transactions can be examined as they go in and out of addresses. This makes the funds ultimately fungible and makes the privacy vulnerable to statistical analysis. 

> **TESTNET vs MAINNET:** Why are all of these bitcoin amounts in fractions? Bitcoins are produced slowly, and so there are relatively few in circulation. As a result, each bitcoin over on the mainnet is worth quite a bit (~ $1,200 at the time of this writing). This means that people usually work in fractions. In fact, .47 BTC would be quite a lot in the real-world. You'll often be dealing with even smaller fractions on mainnet. For this reason, names have appeared for smaller amounts of bitcoins, including millibitcoins or mBTCs (one-thousandth of a bitcoin), microbitcoins or bits or μBTCs (one-millionth of a bitcoin), and satoshis (one hundred millionth of a bitcoin).

### Examine Your Transaction

You can get more information on a transaction with the `bitcoin-cli getrawtransaction` command:
```
$ bitcoin-cli getrawtransaction "88e5d5f3077517d76f5a61491fa52e6aaae078c52bc62d849f09507ef0cfada2"
010000000133261a25b44689bab2c6a207381ca21d243de9bbf21f0fa40c3a26ba7282a87b000000006b483045022100a2640761810dfc34dabed599928243afe24e13f520f780ceb382843a530a577c022050b92f5d9843d70ddb60a0aa294938862f2b7372818d6149ffd4f6adec5cf6c80121034dcaa515c2fda0f4a50b90a6d798e01c00a870bef0bd97154066fe202d2b5d75feffffff02c029cd02000000001976a914fd67e8a7c7813e7a5c376eb71074f373d924d96888ac17791703000000001976a914e176ee39c642344df2180863e27e2e936307273c88ac07a41000
```
> **WARNING:** This command will not work in some situation. To be able to view a raw transaction on a standard node, some of the money must be unspent, or the transaction must still be in your mempool — which means that this command will work fine for the money you've just received, but not for old stuff. If you want to be able to view older transactions that have been entirely spent, you can do so by maintaining a set of all transactions with the txindex=1 configuration, which is what our scripts suggest for all non-pruned instances. (You can't maintain a transaction index if your node is pruned.)

Granted, this isn't super useful, because it's the hex-encoded transaction data. Fortunately, you can get a more verbose description just by adding a '1' to your command:
```
$ bitcoin-cli getrawtransaction "88e5d5f3077517d76f5a61491fa52e6aaae078c52bc62d849f09507ef0cfada2" 1
{
  "hex": "010000000133261a25b44689bab2c6a207381ca21d243de9bbf21f0fa40c3a26ba7282a87b000000006b483045022100a2640761810dfc34dabed599928243afe24e13f520f780ceb382843a530a577c022050b92f5d9843d70ddb60a0aa294938862f2b7372818d6149ffd4f6adec5cf6c80121034dcaa515c2fda0f4a50b90a6d798e01c00a870bef0bd97154066fe202d2b5d75feffffff02c029cd02000000001976a914fd67e8a7c7813e7a5c376eb71074f373d924d96888ac17791703000000001976a914e176ee39c642344df2180863e27e2e936307273c88ac07a41000",
  "txid": "88e5d5f3077517d76f5a61491fa52e6aaae078c52bc62d849f09507ef0cfada2",
  "hash": "88e5d5f3077517d76f5a61491fa52e6aaae078c52bc62d849f09507ef0cfada2",
  "size": 226,
  "vsize": 226,
  "version": 1,
  "locktime": 1090567,
  "vin": [
    {
      "txid": "7ba88272ba263a0ca40f1ff2bbe93d241da21c3807a2c6b2ba8946b4251a2633",
      "vout": 0,
      "scriptSig": {
        "asm": "3045022100a2640761810dfc34dabed599928243afe24e13f520f780ceb382843a530a577c022050b92f5d9843d70ddb60a0aa294938862f2b7372818d6149ffd4f6adec5cf6c8[ALL] 034dcaa515c2fda0f4a50b90a6d798e01c00a870bef0bd97154066fe202d2b5d75",
        "hex": "483045022100a2640761810dfc34dabed599928243afe24e13f520f780ceb382843a530a577c022050b92f5d9843d70ddb60a0aa294938862f2b7372818d6149ffd4f6adec5cf6c80121034dcaa515c2fda0f4a50b90a6d798e01c00a870bef0bd97154066fe202d2b5d75"
      },
      "sequence": 4294967294
    }
  ],
  "vout": [
    {
      "value": 0.47000000,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 fd67e8a7c7813e7a5c376eb71074f373d924d968 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914fd67e8a7c7813e7a5c376eb71074f373d924d96888ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "n4cqjJE6fqcmeWpftygwPoKMMDva6BpyHf"
        ]
      }
    }, 
    {
      "value": 0.51869975,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 e176ee39c642344df2180863e27e2e936307273c OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914e176ee39c642344df2180863e27e2e936307273c88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "n256of3JH1A6X8AQUU7LYkcaRcmrfGjGKC"
        ]
      }
    }
  ],
  "blockhash": "00000000fa4fdd22a2c33c6200b68239939ad65af3f1a48ecea25f8200f5d66b",
  "confirmations": 3,
  "time": 1488307692,
  "blocktime": 1488307692
}
```
Now you can see the full information on the transaction, including all of the inputs ("vin") and all the outputs ("vout). One of the interesting things to note is that though we received .47 BTC in the transaction, another .51869975 was sent to another address. That was probably a change address, a concept that is explored more in the next section. It is quite typical for a transaction to have multiple inputs and/or multiple outputs.

### Optional: Use a Block Explorer

Even looking at the verbose information for a transaction can be a little intimidating. The main goal of this tutorial is to teach about dealing with raw transactions from the command line, but we're happy to talk about other tools when they're applicable. One of those tools is a block explorer, which you can use to look at transactions from a web browser in a much friendlier format.

Currently, our preferred block explorer is [https://live.blockcypher.com/](https://live.blockcypher.com/). 

You can use it to look up transactions for an address:
```
[https://live.blockcypher.com/btc-testnet/address/n4cqjJE6fqcmeWpftygwPoKMMDva6BpyHf/](https://live.blockcypher.com/btc-testnet/address/n4cqjJE6fqcmeWpftygwPoKMMDva6BpyHf/)
```
You can also use it to look at individual transactions:
```
[https://live.blockcypher.com/btc-testnet/tx/88e5d5f3077517d76f5a61491fa52e6aaae078c52bc62d849f09507ef0cfada2/](https://live.blockcypher.com/btc-testnet/tx/88e5d5f3077517d76f5a61491fa52e6aaae078c52bc62d849f09507ef0cfada2/)
```
A block explorer doesn't generally provide any more information than a command line look at a raw transaction; it just does a good job of highlighting the important information and putting together the puzzle pieces, including the transaction fees behind a transaction — another concept that we'll be covering in future sections.

### Summary: Receiving a Transactions

Faucets will give you money on the testnet. They come in as raw transactions, which can be examined with `getrawtransaction` or a block explorer. Once you've receive a transaction, you can see it in your balance and your wallet.

### Interlude: Sending Coins the Easy Way

You're now ready to send some coins, and that's actually quite simple via the command line. You just type `bitcoin-cli sendtoaddress [address] [amount]`. So, to send a little coinage to the address `msoix3SHNr6eRDUJsRSqQwZRhxZnLXhNef` just requires:
```
$ bitcoin-cli sendtoaddress msoix3SHNr6eRDUJsRSqQwZRhxZnLXhNef 0.1
6ad295c280798e9746dcdf7e5a60dfb6219d93bf31aab9b540ce892537c41e0c
```
Make sure the address you write in is where you want the money to go. Make _double_ sure. If you make mistakes in Bitcoin, there's no going back. 

You'll receive a txid back when you issue this command.

> **WARNING:** As noted previously, the bitcoin-cli command generates JSON-RPC commands to talk to the bitcoind. They can be really picky. This is an example: if you list the bitcoin amount without the leading zero (i.e. ".1" instead of "0.1"), then bitcoin-cli will fail with a mysterious message.

You'll be able to see the transaction in your list immediately. 
```
$ bitcoin-cli listtransactions
[
  {
    "account": "",
    "address": "n4cqjJE6fqcmeWpftygwPoKMMDva6BpyHf",
    "category": "receive",
    "amount": 0.47000000,
    "label": "",
    "vout": 0,
    "confirmations": 15,
    "blockhash": "00000000fa4fdd22a2c33c6200b68239939ad65af3f1a48ecea25f8200f5d66b",
    "blockindex": 45,
    "blocktime": 1488307692,
    "txid": "88e5d5f3077517d76f5a61491fa52e6aaae078c52bc62d849f09507ef0cfada2",
    "walletconflicts": [
    ],
    "time": 1488307692,
    "timereceived": 1488307696,
    "bip125-replaceable": "no"
  }, 
 
  {
    "account": "",
    "address": "msoix3SHNr6eRDUJsRSqQwZRhxZnLXhNef",
    "category": "send",
    "amount": -0.10000000,
    "label": "",
    "vout": 0,
    "fee": -0.00004520,
    "confirmations": 0,
    "trusted": false,
    "txid": "6ad295c280798e9746dcdf7e5a60dfb6219d93bf31aab9b540ce892537c41e0c",
    "walletconflicts": [
    ],
    "time": 1488321652,
    "timereceived": 1488321652,
    "bip125-replaceable": "unknown",
    "abandoned": false
  }
]
```
However, note that as always it'll take a while for your balances to settle. Be aware that the default transaction fee for `sendtoaddress` is quite low and should probably be increased, as we note in later sections. This means that your transaction might not placed into the first several blocks, until it's reached enough priority. This is usually fine, if you're sending money to someone. It's less fine if you're working through a tutorial, wanting to get to the next step. 

Using the `sendtoaddress` command isn't necessarily that interesting, if you're planning to write your own raw transactions. However, it's a great test so that you can successfully see a transaction leave your machine, taking some of your money with it.

## Part Three: Sending a Raw Transaction to a P2PKH

You're now ready to create Bitcoin raw transactions. This allows you to send money (as in the interlude above) but to craft the transactions precisely as you want. 

This section focuses on paying to addresses that are P2PKH: pay to public-key hashes. In other words: just a normal Bitcoin address, representing some person who wants your money.

## Understand the Bitcoin Transaction

Before you dive into actually creating raw transactions, you should make sure you understand how a Bitcoin transaction works. 

When you receive cash in your Bitcoin wallet, it appears as an individual transaction. Each of these transactions is called a Unspent Transaction Output (UTXO). It doesn't matter if multiple blobs of money came into the same address or to multiple addresses: each UTXO (incoming transaction) remains distinct in your wallet.

When you create a new outgoing transaction, you gather together one or more UTXOs, each of which represents a blob of money that you received. Together their amount must equal what you want to spend _or more_. You use these as inputs into the new transaction. Then, you generate one or more outputs, which give the money represented by the inputs to one or more people. This creates new UTXOs for the recipients, which may then use _those_ to fund future transactions.

Here's the trick: _all of the UTXOs that you gather are spent in full!_ That means that if you want to send just part of the money in a UTXO to someone else, then you also have to generate an additional output that sends the rest back to you!

## List Your Unspent Transactions

In order to create a new raw transaction, you must know what UTXOs you have on-hand to spent. You can determine this information with the `bitcoin-cli listunspent` command:
```
$ bitcoin-cli listunspent
[
  {
    "txid": "ee9805676271f6244eba94c3d1a48b303a8f8359bf711c630eb6f2ea339d0e72",
    "vout": 0,
    "address": "mrS1ypy2pCweh2nBpkMD7r2T3Zj344wxaY",
    "account": "",
    "scriptPubKey": "76a91477ba616a2778b05a5fd73c7449964050fd1a6fd288ac",
    "amount": 0.08000000,
    "confirmations": 2,
    "spendable": true,
    "solvable": true
  }, 
  {
    "txid": "c1abb6951e6a9aae7e384412b69b69e59c10daac9397d01d0c52b7bc6278d589",
    "vout": 1,
    "address": "mygxipnJUsBgFvscKAaoxDdE8aCmHhRfTZ",
    "account": "",
    "scriptPubKey": "76a914c756c7bd67bf83d83c04e3dc6fd1ff0c6fe8ea9888ac",
    "amount": 0.07800000,
    "confirmations": 1,
    "spendable": true,
    "solvable": true
  }, 
  {
    "txid": "ab7ca727055b812df882298f4e6e10ec699fb6250d843c813623171781f896d8",
    "vout": 0,
    "address": "mygxipnJUsBgFvscKAaoxDdE8aCmHhRfTZ",
    "account": "",
    "scriptPubKey": "76a914c756c7bd67bf83d83c04e3dc6fd1ff0c6fe8ea9888ac",
    "amount": 0.07800000,
    "confirmations": 1,
    "spendable": true,
    "solvable": true
  }
]
```
This listing shows three different UTXOs, worth .08, .078, and .078 BTC. Note that each has its own distinct txid and remains distinct in the wallet, even though two of them were sent to the same address.

However, when you spend a UTXO you need more than just the transaction id. That's because each transaction can have multiple outputs! Remember the example of that first bit of money we got from the faucet: some of it went to us, but some went so someone else (probably a change address). The `txid` just refers to the overall transaction, while a `vout` says which of multiple outputs you've received. In this list, two of our UTXOs are the 0th vout of a transaction, and the other is the 1st. This makes a difference!

So txid+vout=UTXO. This will be the foundation of any raw transaction.

### Write a Raw Transaction with One Output

You're now ready to write a simple raw transaction where you're be sending the entirety of a UTXO to another party; that is, you'll be acting like a money launderer, taking all of the money you received in one blob and just passing it on.

#### Prepare the Raw Transaction

For best practices, we'll start out each transaction by carefully recording the txids and vouts that we'll be spending.

In this case, we're going to spend the oldest transaction, because that's the one that's been validated the most:
```
$ utxo_txid="ee9805676271f6244eba94c3d1a48b303a8f8359bf711c630eb6f2ea339d0e72"
$ utxo_vout="0"
```

> **TESTNET vs MAINNET:** Obviously the "validated the most" criteria would matter a lot more on mainnet, where real money is being used.

You should similarly record your recipient address, to make sure there are no problems. We're sending some money back to the TP faucet:
```
$ recipient="n2eMqTT929pb1RDNuqEnxdaLau1rxy3efi"
```
As always, check your variables carefully, to make sure they match!
```
$ echo $utxo_txid
ee9805676271f6244eba94c3d1a48b303a8f8359bf711c630eb6f2ea339d0e72
$ echo $utxo_vout
0
$ echo $recipient
n2eMqTT929pb1RDNuqEnxdaLau1rxy3efi
```
That recipient is particularly important, because if you mess it up, your money is _gone_! So triple check it. 

#### Write the Raw Transaction

You're now ready to create the raw transaction. This uses the `createrawtransaction` command, which might look a little intimidating. That's because the `createrawtransaction` command doesn't entirely shield you from the JSON RPC that the bitcoin-cli uses. Instead, you are going to input a JSON array to list the UTXOs that you're spending and a JSON object to list the outputs.

Here's the standard format:
```
$ bitcoin-cli createrawtransaction 
'''[
     {
       "txid": "'$your_txid'",
       "vout": "'$your_vout'"
      }
]'''
'''{
   "'$your_recipient'": bitcoin_amount
 }'''
 ```
 Yeah, there are all kinds of crazy quotes there, but trust that they'll do the right thing. Use `'''` to mark the start and end of the JSON array and the JSON object. Protect normal words like `"this"` and normal numbers like `0`. If they're variables, insert single quotes, like `"'$this_word'"` and `'$this_num'`. (Whew. You'll get used to it.)
 
 Here's a command that creates a raw transaction to send your $utxo to your $recipient
 ```
$ rawtxhex=$(bitcoin-cli createrawtransaction '''[ { "txid": "'$utxo_txid'", "vout": '$utxo_vout' } ]''' '''{ "'$recipient'": 0.0795 }''')
```
Note that we followed our best practices to immediately save that to variable, because this is another complex result that we don't want to mess up. The result is hex, and here's what it looks like:
```
$ echo $rawtxhex
0100000001720e9d33eaf2b60e631c71bf59838f3a308ba4d1c394ba4e24f67162670598ee0000000000ffffffff01b04e7900000000001976a914e7c1345fc8f87c68170b3aa798a956c2fe6a9eff88ac00000000
```

#### Verify Your Raw Transaction

You should next verify your rawtransaction with `decoderawtransaction` to make sure that it will do the right thing.
```
$ bitcoin-cli decoderawtransaction $rawtxhex
{
  "txid": "6d39d612fe382eaab9897d3ea4d5e44233d8acef40e63fe0f06df785fd7a0c45",
  "hash": "6d39d612fe382eaab9897d3ea4d5e44233d8acef40e63fe0f06df785fd7a0c45",
  "size": 85,
  "vsize": 85,
  "version": 1,
  "locktime": 0,
  "vin": [
    {
      "txid": "ee9805676271f6244eba94c3d1a48b303a8f8359bf711c630eb6f2ea339d0e72",
      "vout": 0,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.07950000,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 e7c1345fc8f87c68170b3aa798a956c2fe6a9eff OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914e7c1345fc8f87c68170b3aa798a956c2fe6a9eff88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "n2eMqTT929pb1RDNuqEnxdaLau1rxy3efi"
        ]
      }
    }
  ]
}
```
Check the vin. Are you spending the right transaction? Does it contain the expected amount of money? Check your vout. Are you sending the right amount? Is it going to the right address? Finally, do the math to make sure the money balances. Does the value of the UTXO minus the amount being spent equal the expected transaction fee?

#### Understand the Transaction Fee

You'll note that we didn't send the whole .08 BTC to our recipient. That's because you have to pay minor fees to use the Bitcoin network. The amount that you will pay as a fee is always equal to the amount of your input minus the amount of your output. So, you have to decrease your output a little bit from your input to make sure that your transaction goes out.

> **WARNING:** This is very dangerous!! Because you automatically expend all of the amount in the UTXOs that you use, it's critically important to make sure that you know: (1) precisely what UTXOs you're using; (2) exactly how much money they contain; (3) exactly how much money you're sending out; and (4) what the difference is. If you mess up and you use the wrong UTXO (with more money than you thought) or if you send out too little money, the excess is lost. Forever. Don't make that mistake! Know your inputs and outputs _precisely_.

How much should you spend? [Bitcoin Fees](https://bitcoinfees.21.co/) has a nice live assessment. It says that the "fastest and cheapest transaction fee is currently 220 satoshis/byte" and that "For the median transaction size of 226 bytes, this results in a fee of 49,720 satoshis". Since this transaction will have just one input and one output, we decided that was more than enough. So, we subtracted 50,000 satoshis, which is .0005 BTC. .0800 BTC - .0005 BC= .0795, which is what we sent. (Often transactions don't need to be the "fastest" and can get away with much lower transaction fees; we opted not to because we don't want to delay working through this tutorial.)

> **WARNING:** The lower that you set your transaction fee, the longer before your transaction is built into a block. The Bitcoin Fees sites lists expected times, from an expected 0 blocks, to 22. Since blocks are built on average every 10 minutes, that's the difference between a few minutes and a few hours! So, choose a transaction fee that's appropriate for what you're sending. Note that you should never drop below the minimum relay fee, which is .0001 BTC.

#### Sign the Raw Transaction

To date, your raw transaction is just something theoretical: you _could_ send it, but nothing has been promised. You have to do a few things to get it out onto the network.

First, you need to sign your raw transaction:
```
$ bitcoin-cli signrawtransaction $rawtxhex
{
  "hex": "0100000001720e9d33eaf2b60e631c71bf59838f3a308ba4d1c394ba4e24f67162670598ee000000006b483045022100d8f17dadc2501596f75f2c90b8279130e588638d4f7a4f7d5ebb10fea15252f702200ceb164e81335c430893780d06cfe194c36acec26886f180408e3ac4a7d2292f0121035de6239e70523c8f392e32f98e65f6ef704c4b6b0df994e407212b839bf51048ffffffff01b04e7900000000001976a914e7c1345fc8f87c68170b3aa798a956c2fe6a9eff88ac00000000",
  "complete": true
}
$ signedtx="0100000001720e9d33eaf2b60e631c71bf59838f3a308ba4d1c394ba4e24f67162670598ee000000006b483045022100d8f17dadc2501596f75f2c90b8279130e588638d4f7a4f7d5ebb10fea15252f702200ceb164e81335c430893780d06cfe194c36acec26886f180408e3ac4a7d2292f0121035de6239e70523c8f392e32f98e65f6ef704c4b6b0df994e407212b839bf51048ffffffff01b04e7900000000001976a914e7c1345fc8f87c68170b3aa798a956c2fe6a9eff88ac00000000"
```
Note that we captured the signed hex by hand, rather than trying to parse it out of the JSON object. A software package called "JQ" could do better, but for the moment we're keeping things simple and clear.

#### Send the Raw Transaction

You've now got a ready-to-go raw transaction, but it doesn't count until you actually put it on the network, which you do with the `sendrawtransaction` command. You'll get back a txid:
```
$ bitcoin-cli sendrawtransaction $signedtx
bf02068e3d2a99542c6a9ad07d915f1664374e911d809afe156dbacc46a5183e
```
You'll immediately be able to see that the UTXO and its money have been removed from your account:
```
$ bitcoin-cli listunspent
[
  {
    "txid": "c1abb6951e6a9aae7e384412b69b69e59c10daac9397d01d0c52b7bc6278d589",
    "vout": 1,
    "address": "mygxipnJUsBgFvscKAaoxDdE8aCmHhRfTZ",
    "account": "",
    "scriptPubKey": "76a914c756c7bd67bf83d83c04e3dc6fd1ff0c6fe8ea9888ac",
    "amount": 0.07800000,
    "confirmations": 12,
    "spendable": true,
    "solvable": true
  }, 
  {
    "txid": "ab7ca727055b812df882298f4e6e10ec699fb6250d843c813623171781f896d8",
    "vout": 0,
    "address": "mygxipnJUsBgFvscKAaoxDdE8aCmHhRfTZ",
    "account": "",
    "scriptPubKey": "76a914c756c7bd67bf83d83c04e3dc6fd1ff0c6fe8ea9888ac",
    "amount": 0.07800000,
    "confirmations": 12,
    "spendable": true,
    "solvable": true
  }
]
$ bitcoin-cli getbalance
0.15600000
```
Finally, `listtransactions` should soon show a confirmed transaction of category 'send".
```
 {
    "account": "",
    "address": "n2eMqTT929pb1RDNuqEnxdaLau1rxy3efi",
    "category": "send",
    "amount": -0.07950000,
    "vout": 0,
    "fee": -0.00050000,
    "confirmations": 1,
    "blockhash": "0000000000000dd6f6f455be7eecaf8055bb61d5d18d142d75bcdf8aa6d81456",
    "blockindex": 3,
    "blocktime": 1488410944,
    "txid": "bf02068e3d2a99542c6a9ad07d915f1664374e911d809afe156dbacc46a5183e",
    "walletconflicts": [
    ],
    "time": 1488410872,
    "timereceived": 1488410872,
    "bip125-replaceable": "no",
    "abandoned": false
  }
```
You can see that it matches the txid and the recipient address. Not only does it show the amount sent, but it also shows the transaction fee. And, it's already received a confirmation, because we offered a fee that would get it swept up into a block quickly.

Congratulations! You're now a few satoshis poorer!

### Create a Change Address

Our first raw transaction was very simplistic: we sent the entirety of a UTXO to a new address. More frequently, you'll want to send someone an amount of money that doesn't match a UTXO. But you'll recall that the excess money from a UTXO that's not sent to your recipient just becomes a transaction fee. So, how do you send someone just part of a UTXO, while keeping the rest for yourself?

The solution is to create a change address:
```
$ changeaddress=$(bitcoin-cli getnewaddress)
$ echo $changeaddress
mxU9cmhJfkKWDtBspHaA36LkeafEDeaogJ
```
You now have an additional address inside your wallet that you can use to receive change from a UTXO!

### Write a Raw Transaction with Two Outputs

You're now ready to write a more complex raw transaction. It will be based on the two unspent UTXOs still in our wallet:
```
$ bitcoin-cli listunspent
[
  {
    "txid": "c1abb6951e6a9aae7e384412b69b69e59c10daac9397d01d0c52b7bc6278d589",
    "vout": 1,
    "address": "mygxipnJUsBgFvscKAaoxDdE8aCmHhRfTZ",
    "account": "",
    "scriptPubKey": "76a914c756c7bd67bf83d83c04e3dc6fd1ff0c6fe8ea9888ac",
    "amount": 0.07800000,
    "confirmations": 237,
    "spendable": true,
    "solvable": true
  }, 
  {
    "txid": "ab7ca727055b812df882298f4e6e10ec699fb6250d843c813623171781f896d8",
    "vout": 0,
    "address": "mygxipnJUsBgFvscKAaoxDdE8aCmHhRfTZ",
    "account": "",
    "scriptPubKey": "76a914c756c7bd67bf83d83c04e3dc6fd1ff0c6fe8ea9888ac",
    "amount": 0.07800000,
    "confirmations": 237,
    "spendable": true,
    "solvable": true
  }
]
```
We want to send 0.1 BTC back to the TP faucet. This requires combining our two UTXOs, because neither contains quite that amount of bitcoin. It also requires sending to two addresses: one for the TP faucet (n2eMqTT929pb1RDNuqEnxdaLau1rxy3efi) and one as a change address (mxU9cmhJfkKWDtBspHaA36LkeafEDeaogJ). In other words, we're going to have two inputs and two outputs in our raw transaction

#### Set Up Your Variables

We've already have `$changeaddress` and `$recipient` variables from our previous examples:
```
$ echo $changeaddress
mxU9cmhJfkKWDtBspHaA36LkeafEDeaogJ
$ echo $recipient
n2eMqTT929pb1RDNuqEnxdaLau1rxy3efi
```
We also need to record the txid and vout for each of our two UTXOs:
```
$ utxo_txid_1="c1abb6951e6a9aae7e384412b69b69e59c10daac9397d01d0c52b7bc6278d589"
$ utxo_vout_1="1"
$ utxo_txid_2="ab7ca727055b812df882298f4e6e10ec699fb6250d843c813623171781f896d8"
$ utxo_vout_2="0"

$ echo $utxo_txid_1
c1abb6951e6a9aae7e384412b69b69e59c10daac9397d01d0c52b7bc6278d589
$ echo $utxo_vout_1
1
$ echo $utxo_txid_2
ab7ca727055b812df882298f4e6e10ec699fb6250d843c813623171781f896d8
$ echo $utxo_vout_2
0
```

#### Write the Transaction

Writing the new raw transaction is surprisingly simple. All you need to do is include an additional, comma-separated JSON object in the JSON array of inputs and an additional, comma-separated key-value pair in the JSON object of outputs.

Here's the example. Take a look at it carefully to understand where the inputs end the outputs begin:
```
rawtxhex2=$(bitcoin-cli createrawtransaction '''[ { "txid": "'$utxo_txid_1'", "vout": '$utxo_vout_1' }, { "txid": "'$utxo_txid_2'", "vout": '$utxo_vout_2' } ]''' '''{ "'$recipient'": 0.1, "'$changeaddress'": 0.0555 }''')
```
We were _very_ careful figuring out our money math. These two UTXOs contain .156 BTC. After sending .1 BTC, we'll have .056 BTC left. We again chose .0005 BTC for the transaction fee: this transaction about twice as big, but it's still under the average transaction size quoted by Bitcoin Fees, so that should be more than enough to make this transaction go through quickly. To accomodate that fee, we set our change to .0555 BTC. If we'd messed up our math and instead set our change to .051 BTC, that additional .0045 BTC would be lost. That's $5 or $6 gone to the miners when it didn't need to be. If we'd forgot to make change at all, then the whole .056 ($70!) would have disappeared. So, again, _be careful_. 

Back to the raw transaction itself: we'll be honest, it took us a few times to get all the commas, quotes, and brackets in the right places, writing from the command line. We got JSON errors when we didn't format things quite right; when we fixed the errors, our variable (correctly) got filled with the hex for the raw transaction. So, as always, be careful here too, and check your work:
```
$ bitcoin-cli decoderawtransaction $rawtxhex2
{
  "txid": "99b7454c20313806454f9fd4f3e0454ce93e6edd5e84dfc731bcc0ad352b2260",
  "hash": "99b7454c20313806454f9fd4f3e0454ce93e6edd5e84dfc731bcc0ad352b2260",
  "size": 160,
  "vsize": 160,
  "version": 1,
  "locktime": 0,
  "vin": [
    {
      "txid": "c1abb6951e6a9aae7e384412b69b69e59c10daac9397d01d0c52b7bc6278d589",
      "vout": 1,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "sequence": 4294967295
    }, 
    {
      "txid": "ab7ca727055b812df882298f4e6e10ec699fb6250d843c813623171781f896d8",
      "vout": 0,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.10000000,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 e7c1345fc8f87c68170b3aa798a956c2fe6a9eff OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914e7c1345fc8f87c68170b3aa798a956c2fe6a9eff88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "n2eMqTT929pb1RDNuqEnxdaLau1rxy3efi"
        ]
      }
    }, 
    {
      "value": 0.05550000,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 b9f2579814712f82a3dadfd73e0356339c6705b4 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914b9f2579814712f82a3dadfd73e0356339c6705b488ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "mxU9cmhJfkKWDtBspHaA36LkeafEDeaogJ"
        ]
      }
    }
  ]
}
```
#### Finish It Up

You can now sign, seal, and deliver your transaction, and it's yours (and the faucet's):
```
$ bitcoin-cli signrawtransaction $rawtxhex2
{
  "hex": "010000000289d57862bcb7520c1dd09793acda109ce5699bb61244387eae9a6a1e95b6abc1010000006a4730440220194cf452c76630419b90ef16bc9c0f31bd3187b97357eff6b610a3054968579b02200bcdb125ac182bda338c1a64d7887efd3e4eaf1a8e7f517eb7630d4dc258bf88012102f9006968b057f5212752964546d4c259f359601d00de32ec027fd84760bb4a46ffffffffd896f88117172336813c840d25b69f69ec106e4e8f2982f82d815b0527a77cab000000006a47304402201e65fe69dd231f73a9f493ca6c49503d51a766cf27be1281c9a4eefd04b4ac1c022003750d1ec93ef7be552aa55656c1860c986ef04a8733587f4f7be38707cccaf2012102f9006968b057f5212752964546d4c259f359601d00de32ec027fd84760bb4a46ffffffff0280969800000000001976a914e7c1345fc8f87c68170b3aa798a956c2fe6a9eff88acb0af5400000000001976a914b9f2579814712f82a3dadfd73e0356339c6705b488ac00000000",
  "complete": true
}
$ signedtx2="010000000289d57862bcb7520c1dd09793acda109ce5699bb61244387eae9a6a1e95b6abc1010000006a4730440220194cf452c76630419b90ef16bc9c0f31bd3187b97357eff6b610a3054968579b02200bcdb125ac182bda338c1a64d7887efd3e4eaf1a8e7f517eb7630d4dc258bf88012102f9006968b057f5212752964546d4c259f359601d00de32ec027fd84760bb4a46ffffffffd896f88117172336813c840d25b69f69ec106e4e8f2982f82d815b0527a77cab000000006a47304402201e65fe69dd231f73a9f493ca6c49503d51a766cf27be1281c9a4eefd04b4ac1c022003750d1ec93ef7be552aa55656c1860c986ef04a8733587f4f7be38707cccaf2012102f9006968b057f5212752964546d4c259f359601d00de32ec027fd84760bb4a46ffffffff0280969800000000001976a914e7c1345fc8f87c68170b3aa798a956c2fe6a9eff88acb0af5400000000001976a914b9f2579814712f82a3dadfd73e0356339c6705b488ac00000000"
$ bitcoin-cli sendrawtransaction $signedtx2
18f1a791e3f7735dbe1c92b86645ba9b1f62b8044f61efd76d082a084982c9ae
```
#### Wait

As usual, your money will be in flux for a while. You used all of your UTXOs, so your `getbalance` may show that you have cash, but the transaction must be confirmed before you see a UTXO with your change.

But, in 10 minutes or less (probably), you'll have your remaining money back and fully spendable again:
```
$ bitcoin-cli listunspent
[
  {
    "txid": "18f1a791e3f7735dbe1c92b86645ba9b1f62b8044f61efd76d082a084982c9ae",
    "vout": 1,
    "address": "mxU9cmhJfkKWDtBspHaA36LkeafEDeaogJ",
    "account": "",
    "scriptPubKey": "76a914b9f2579814712f82a3dadfd73e0356339c6705b488ac",
    "amount": 0.05550000,
    "confirmations": 2,
    "spendable": true,
    "solvable": true
  }
]
```

This also might be a good time to revisit a blockchain explorer, so that you can see more intuitively how the inputs, outputs, and transaction fee are all laid out: [https://live.blockcypher.com/btc-testnet/tx/18f1a791e3f7735dbe1c92b86645ba9b1f62b8044f61efd76d082a084982c9ae/](https://live.blockcypher.com/btc-testnet/tx/18f1a791e3f7735dbe1c92b86645ba9b1f62b8044f61efd76d082a084982c9ae/).

### Optional: Write a Raw Transaction with Automatic Funding

The purpose of this tutorial is to show you the very basics of Bitcoin raw transactions, so that you can work at things at a fundamental level. If you were writing a wallet or something other Bitcoin software, you'd probably want to do things exactly as described here. However, if you were (satoshi forfend!) regularly sending bitcoins about through raw transactions created by hand, then you'd want to have a little better insurance that you weren't making mistakes.

The bitcoin-cli accomodates this with a `fundrawtransaction` command. First, you have to make sure that your ~/.bitcoin/bitcoin.conf file has some rational variables for calculating transaction fees. We've been quite aggressive with fees in this tutorial, to make sure the examples finish quickly, but the following would allow for cheaper transmissions that might take a few hours:
```
 paytxfee=0.0001
 mintxfee=0.0001
 txconfirmtarget=25
 ```
 With that in hand (and in bitcoin.conf) you can use `createrawtransaction` with just your output(s), then run `fundrawtransaction` on the resulting hex:
 ```
$ unfinishedtx=$(bitcoin-cli createrawtransaction '''[]''' '''{ "'$recipient'": 0.04 }''')
$ bitcoin-cli fundrawtransaction $unfinishedtx
{
  "hex": "0100000001aec98249082a086dd7ef614f04b8621f9bba4566b8921cbe5d73f7e391a7f1180100000000feffffff0208951700000000001976a914f26e11dc0fcc79fe76fca1d24d7588798922ca7488ac00093d00000000001976a914e7c1345fc8f87c68170b3aa798a956c2fe6a9eff88ac00000000",
  "changepos": 0,
  "fee": 0.00004520
}
$ rawtxhex3="0100000001aec98249082a086dd7ef614f04b8621f9bba4566b8921cbe5d73f7e391a7f1180100000000feffffff0208951700000000001976a914f26e11dc0fcc79fe76fca1d24d7588798922ca7488ac00093d00000000001976a914e7c1345fc8f87c68170b3aa798a956c2fe6a9eff88ac00000000"
```
It seems like magic, but when you examine the raw transaction, you'll see it works:
```
$ bitcoin-cli decoderawtransaction $rawtxhex3
{
  "txid": "3099716446e0761e823077fc2f33c158b1dbaa7c157cb03c25a7cf6b7ac4921d",
  "hash": "3099716446e0761e823077fc2f33c158b1dbaa7c157cb03c25a7cf6b7ac4921d",
  "size": 119,
  "vsize": 119,
  "version": 1,
  "locktime": 0,
  "vin": [
    {
      "txid": "18f1a791e3f7735dbe1c92b86645ba9b1f62b8044f61efd76d082a084982c9ae",
      "vout": 1,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "sequence": 4294967294
    }
  ],
  "vout": [
    {
      "value": 0.01545480,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 f26e11dc0fcc79fe76fca1d24d7588798922ca74 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914f26e11dc0fcc79fe76fca1d24d7588798922ca7488ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "n3cogFw4y3A1LJgRz2G97otEGT7BK8ca3D"
        ]
      }
    }, 
    {
      "value": 0.04000000,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 e7c1345fc8f87c68170b3aa798a956c2fe6a9eff OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914e7c1345fc8f87c68170b3aa798a956c2fe6a9eff88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "n2eMqTT929pb1RDNuqEnxdaLau1rxy3efi"
        ]
      }
    }
  ]
}
```
As you can see, `fundrawtransaction` chose UTXOs for the inputs, it created a change address, it calculated a (low) transaction fee based on the variables in the bitcoin.conf file, and it sent the amount of the UTXO minus the tx fee back to the change address.

You can even verify that the new address belongs to you:
```
$ bitcoin-cli validateaddress n3cogFw4y3A1LJgRz2G97otEGT7BK8ca3D
{
  "isvalid": true,
  "address": "n3cogFw4y3A1LJgRz2G97otEGT7BK8ca3D",
  "scriptPubKey": "76a914f26e11dc0fcc79fe76fca1d24d7588798922ca7488ac",
  "ismine": true,
  "iswatchonly": false,
  "isscript": false,
  "pubkey": "0347251fdd4fe8f9c66c0c137cc641e864dd27fc5dc0c8d0c85ff884d2a3fa1574",
  "iscompressed": true,
  "hdkeypath": "m/0'/0'/4'",
  "hdmasterkeyid": "75807dcb1226537ceb54c38c4a75a61407fdd02d"
}
```
At this point you could sign and send the transaction as usual ... then wait much longer for it to come back due to the lower transaction fees!
```
$ signedtx3="0100000001aec98249082a086dd7ef614f04b8621f9bba4566b8921cbe5d73f7e391a7f118010000006b483045022100a9b1454114bb2c04b51619eb5a00ad391605920ae801405b6191a64d1fb1e6e8022054def9ccbd75cb7929279cfef73ac573cdac7a325a1e3c8f43e139a1340b5d4b012103f7c794378db1c070b07d74f427f394f8a5d53f1abe1d2dab100d5a7a49db8785feffffff0208951700000000001976a914f26e11dc0fcc79fe76fca1d24d7588798922ca7488ac00093d00000000001976a914e7c1345fc8f87c68170b3aa798a956c2fe6a9eff88ac00000000"
$ bitcoin-cli sendrawtransaction $signedtx3
ecfc625fff594683e90d21618b64f44f7046c55bcda6468c1c37c1abe8b83913
```
And voila!
```
$ bitcoin-cli getbalance
0.01545480
```
We will _not_ be using this technique in the rest of the tutorial.

### Summary: Sending a Raw Transaction to a P2PKH

When money comes into your Bitcoin wallet, it remains as distinct amounts, called UTXOs. When you create a raw transaction to send that money back out, you use one or more UTXOs to fund it. If you want change back from the UTXOs you're spending, you _must_ create a change address to receive the excess funds. You must also define how much money to spend as the transaction fee. Once you've created a raw transaction, you can check it, sign it, and then actually send it to the Bitcoin network.

## Part Four: Sending a Raw Transaction to an OP_RETURN

P2PKHs are the simplest sort of Bitcoin recipient ... except perhaps the OP_RETURN. That's because an OP_RETURN is basically a null: an invalid output. Why would you use one? To store data on the blockchain: up to 80 bytes. 

This opens up whole new possibilities for the blockchain, because you can embed data that proves that certain things happened at certain times. Though there is some controversy over using the Bitcoin blockchain in this way, various organizations have used OP_RETURNs for proof of existence, for copyright, to color coins, and [for other purposes](https://en.bitcoin.it/wiki/OP_RETURN). Though 80 bytes might not seem a lot, it can be quite effective if OP_RETURNs are used to store hashes of the actual data.

### Create Your OP_RETURN Data

The first thing you need to do is create the 80 bytes (or less) of data that you'll be recording in your OP_RETURN. This might be as simple as preparing a message or you might be hasing existing data. For example, `md5sum` produces 128 bits of data, which is 16 bytes, well under the limits:
```
$ md5sum mycontract.jpg 
3b110a164aa18d3a5ab064ba93fdce62  mycontract.jpg
$ op_return_data="3b110a164aa18d3a5ab064ba93fdce62"
```

### Prepare Some Money

Your purpose in creating an OP_RETURN isn't to send money to anyone, it's to put data into the blockchain. However, you _must_ send money to do so. You just need to use a change address as your _only_ recipient. Then you can identify a UTXO and send that to your change address, minus a transaction, while also using the same transaction to create an OP_RETURN.

Here's that process:
```
$ bitcoin-cli listunspent
[
  {
    "txid": "0c198125f5a0e2e598ce3b7e4253a95dce780dec12601ed0a44c8544606782b2",
    "vout": 0,
    "address": "n3cogFw4y3A1LJgRz2G97otEGT7BK8ca3D",
    "account": "",
    "scriptPubKey": "76a914f26e11dc0fcc79fe76fca1d24d7588798922ca7488ac",
    "amount": 0.06600000,
    "confirmations": 11,
    "spendable": true,
    "solvable": true
  }
]

$ utxo_txid_4="0c198125f5a0e2e598ce3b7e4253a95dce780dec12601ed0a44c8544606782b2"
$ utxo_vout_4="0"
$ changeaddress_4=$(bitcoin-cli getnewaddress)
```

### Write A Rawtransaction

You can now write a new rawtransaction with two outputs: one is your change address to get back (most of) your money, the other is a data address, which is bitcoin-cli's term for an OP_RETURN.
```
$ rawtxhex4=$(bitcoin-cli createrawtransaction '''[ { "txid": "'$utxo_txid_4'", "vout": '$utxo_vout_4' } ]''' '''{ "data": "'$op_return_data'", "'$changeaddress_4'": 0.0655 }''')
```

Here's what that transaction actually looks like:
```
$ bitcoin-cli decoderawtransaction $rawtxhex4
{
  "txid": "58a75a85833c396082e6603a0a2537c7e8c207d188656791679df7da7fe0879a",
  "hash": "58a75a85833c396082e6603a0a2537c7e8c207d188656791679df7da7fe0879a",
  "size": 112,
  "vsize": 112,
  "version": 1,
  "locktime": 0,
  "vin": [
    {
      "txid": "0c198125f5a0e2e598ce3b7e4253a95dce780dec12601ed0a44c8544606782b2",
      "vout": 0,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.00000000,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_RETURN 3b110a164aa18d3a5ab064ba93fdce62",
        "hex": "6a103b110a164aa18d3a5ab064ba93fdce62",
        "type": "nulldata"
      }
    }, 
    {
      "value": 0.06550000,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 effa6f1be9433e051ddbc33d7021259f5e1333c7 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914effa6f1be9433e051ddbc33d7021259f5e1333c788ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "n3PqoMxnQQmafdup2hAmV6SzanJGo4FG8f"
        ]
      }
    }
  ]
}
```
As you can see, this sends the majority of the money straight back to the change address (n3PqoMxnQQmafdup2hAmV6SzanJGo4FG8f) minus that standard transaction fee we've been using of 0.0005 BTC. _Do_ note that these transactions get larger as they add extra inputs and outputs — and now 16 bytes of data. More importantly, the first output shows an OP_RETURN with the data (3b110a164aa18d3a5ab064ba93fdce62) right after it.

Sign it and send it, and soon that OP_RETURN will be embedded in the blockchain!

### Check Your OP_RETURN

Again, remember that you can look at this transaction using a blockchain explorer: [https://live.blockcypher.com/btc-testnet/tx/ed445dc970bb40b17c207109e19a37b2be301acb474ccd30680c431cb681bce2/](https://live.blockcypher.com/btc-testnet/tx/ed445dc970bb40b17c207109e19a37b2be301acb474ccd30680c431cb681bce2/)

You may note a warning about the data being in an "unknown protocol". If you were designing some regular use of OP_RETURN data, you'd probably mark it with a special prefix, to mark that protocol. Then, the actual OP_RETURN data might be something like "CONTRACTS3b110a164aa18d3a5ab064ba93fdce62". This example didn't use a prefix to avoid muddying the data space.

[Coinsecrets](http://coinsecrets.org/) offers another interesting way to look at OP_RETURN data. It does its best to keep abreast of protocols, so that it can tell you who is doing what in the blockchain. Here's thisour transaction there: [https://www.blocktrail.com/tBTC/tx/ed445dc970bb40b17c207109e19a37b2be301acb474ccd30680c431cb681bce2](https://www.blocktrail.com/tBTC/tx/ed445dc970bb40b17c207109e19a37b2be301acb474ccd30680c431cb681bce2)

### Summary: Sending a Raw Transaction to an OP_RETURN

You can use an OP_RETURN output to store up to 80 bytes of data on the blockchain. You do this with the 'data' codeword for a 'vout'. You still have to send money along too, but you just send it back to a change address, minus a transaction fee.

### Interlude: Using JQ


