### [H-2] Weak randomness in `PuppyRaffle::selectWinner` allows users to predict or manipulate the winner and NFT rarity

**Description:** `PuppyRaffle::selectWinner` uses on-chain values (`msg.sender`, `block.timestamp`, `block.difficulty`) as the seed for randomness. All of these values are public and can be known or influenced before a transaction is mined.

```solidity
uint256 winnerIndex =
    uint256(keccak256(abi.encodePacked(msg.sender, block.timestamp, block.difficulty))) % players.length;

uint256 rarity = uint256(keccak256(abi.encodePacked(msg.sender, block.difficulty))) % 100;
```

**Impact:** A validator or motivated user can:
1. Know `block.timestamp` and `block.difficulty` in advance.
2. Mine different `msg.sender` addresses until they find one that produces a winning index.
3. Deploy a contract that calls `selectWinner` and reverts if the result isn't them — repeating until they win.

This effectively breaks the core raffle mechanic and allows stealing the entire prize pool plus the rarest NFT.

**Proof of Concept:**
1. An attacker writes a smart contract that calls `selectWinner`.
2. Inside the call, it computes the same `keccak256` with known on-chain values.
3. If the resulting `winnerIndex` is not the attacker's index, the transaction reverts.
4. The attacker repeats until the block produces a winning result.

**Recommended Mitigation:** Use **Chainlink VRF (Verifiable Random Function)** for cryptographically secure, on-chain verifiable randomness that cannot be manipulated by validators or callers.

```
https://docs.chain.link/vrf/v2/introduction
```
