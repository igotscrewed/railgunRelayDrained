# railgunRelayDrained

## Incident Regarding ~ $730 of Ethereum drained from my railgun relay. 

#### Synopsis
Starting at Apr-07-2025 at 10:12:47 AM UTC, my railgun relay suffered a series of attacks. Twelve transactions over the course of an hour reverted, costing me about $730, or about .48 Ethereum. The attacks contiunued until all of my relay's available ethereum was depleted. Upon further analysis, I discovered that the target contract for these transactions was this contract: [https://etherscan.io/address/0x87c7f7d6c8e4a358eb798e92574ae129f001d7f5/advanced#internaltx](0x87c7f7d6c8e4a358eb798e92574ae129f001d7f5)


#### Reverting contract: 
<p>
I decompiled the bytecode as the source is not available:
</p>
<pre>
  function _SafeAdd(uint256 varg0, uint256 varg1) private { 
    require(varg0 <= varg0 + varg1, Panic(17)); // arithmetic overflow or underflow
    return varg0 + varg1;
}

function _SafeMul(uint256 varg0, uint256 varg1) private { 
    require(!varg0 | (varg1 == varg0 * varg1 / varg0), Panic(17)); // arithmetic overflow or underflow
    return varg0 * varg1;
}

function fallback() public payable { 
    revert();
}

function 0x67658be1(uint256 varg0, uint256 varg1) public payable { 
    require(4 + (msg.data.length - 4) - 4 >= 64);
    v0 = v1 = 0;
    while (v0 >= varg0) {
        v2 = _SafeAdd(v0, v0);
        v3 = _SafeMul(v2, 2);
        require(2, Panic(18)); // division by zero
        v0 = v3 >> 1;
        v0 += 1;
    }
    require(varg1 >= block.number);
}

// Note: The function selector is not present in the original solidity code.
// However, we display it for the sake of completeness.

function __function_selector__( function_selector) public payable { 
    MEM[64] = 128;
    require(!msg.value);
    if (msg.data.length >= 4) {
        if (0x67658be1 == function_selector >> 224) {
            0x67658be1();
        }
    }
    fallback();
}
</pre>



<p>
Obviously this contract is designed to burn gas and then revert. I am surprised that railgun processed this transaction, as I was under the impression that railgun would 
not proccess a tranasction that would revert. It goes without saying why that must be the case for us relayers. I am not operating a relay until this is fixed or I can 
  figure out how to prevent it from happening again, that's for sure. 
</p>

#### Contract call trace 

<pre>
https://pastebin.com/YKpJU5aN
</pre>

#### Prevelance of this problem

<p>
  Take a look at the contract's internal message calls and you'll see I am, at least to my knowledge, the third victim that the operator of this contract has hit. This 
  is a serious problem that must be patched immediately. It is extreemely risky to run a relay under these circumstances. Contract calls that burn a bunch of gas and 
  revert will sabotage relayers. My client just lost almost half an ETH. I am annoyed. Not going to lie. Had I known that all of my client's money could be ciphoned, I 
  would not have recommended running this relay. And I don't know how often this has happened, but I imagine more than just 3 times with other contracts. I will analyze the blockchain and find out and update this repo.
</p>

#### Interesting twist
<p>
I think the deployer of this contract is in fact a relay operator. I haven't verified that for sure, but it sure looks that way.
[https://etherscan.io/address/0xfacfc25c10ff9e44cf2439dbbec9b20e887d4530](take a look ... )
</p>

<p>
 I am assuming the culprit is trying to take out the competition. If this type of attack is not fixed, the railgun project will fail. If it's not fixable, there 
  needs to be some kind of insurance fund to reimburse rekt operators. But I think it should be fixable. I thought transactions that would fail were rejected. How is that not the case?
</p>
