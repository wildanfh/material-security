### [M-2] Smart contract wallets without `receive`/`fallback` as winners can permanently block `selectWinner`

**Description:** `PuppyRaffle::selectWinner` pushes ETH directly to the winner using a low-level call. If the winner is a smart contract that does not implement a `receive` or `fallback` function, the call will revert, preventing the raffle from being reset.

```solidity
(bool success,) = winner.call{value: prizePool}("");
require(success, "PuppyRaffle: Failed to send prize pool to winner");
```

Since `selectWinner` also resets `players` and advances `raffleStartTime`, a revert here blocks all future raffles.

**Impact:** If a smart contract wallet wins the raffle and rejects ETH, `selectWinner` will revert on every call, permanently locking the protocol in a broken state where no new raffle can begin.

**Proof of Concept:**
1. 10 smart contract wallets without `receive`/`fallback` enter the raffle.
2. The raffle period ends.
3. `selectWinner` is called — it selects one of the rejecting contracts.
4. The `call{value: prizePool}` reverts.
5. No new raffle can ever start.

**Recommended Mitigation:** Use a **pull-over-push** payment pattern. Instead of pushing ETH to the winner, let the winner claim it:

```solidity
mapping(address => uint256) public winnings;

function selectWinner() external {
    // ... compute winner ...
    winnings[winner] += prizePool;
    // reset raffle state
}

function claimPrize() external {
    uint256 amount = winnings[msg.sender];
    require(amount > 0, "No winnings to claim");
    winnings[msg.sender] = 0;
    (bool success,) = msg.sender.call{value: amount}("");
    require(success, "Transfer failed");
}
```
