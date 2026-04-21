## [H-2] Storing the password on-chain makes it visible to anyone and no longer private

**Description:** All data stored on chain is public and visible to anyone. The `PasswordStore::s_password` variable is intended to be hidden and only accessible by the owner through the `PasswordStore::getPassword` function. I show one such method of reading any data off chain below.
**Impact:** Anyone is able to read the private password, severely breaking the functionality of the protocol.
**Proof of Concept:** The below test case shows how anyone could read the password directly form the blockchain. We use foundry's cast tool to read directly from the storage of the contract, without being the owner.
Create a locally running chain
```bash
make anvil
```
Deploy the contract to the chain
```bash
make deploy
```
Run the storage tool
```bash
cast storage <ADDRESS_HERE> 1 --rpc-url http://127.0.0.1:8545
```
We use 1 because that's the storage slot of `PasswordStore::s_password`.
You'll get an output that looks like this:
```
0x6d7950617373776f726400000000000000000000000000000000000000000014
```
You can then parse that hex to a string with:
```bash
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```
And get an output of:
```bash
myPassword
```
**Recommended Mitigation:** De to this, the overall architecture of the contract should be rethought. One could encrypt the password off-chain, and then store the encrypted password on-chain. This would require the user to remember another password off-chain to decrypt the stored password. However, you're also likely want to remove the view function as you wouldn't want the user to accidentally send a transaction with this decryption key.
