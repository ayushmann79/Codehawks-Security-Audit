
### [H-1] Storing the password on-chain makes it visible to anyone and is no longer private

**Description:**  
All data stored on-chain is visible to anyone and can be read directly from the blockchain. The `PasswordStore::s_password` variable is intended to be private and should only be accessed through the `PasswordStore::getPassword` function, which is intended to be called only by the contract owner.

We show one such method of reading on-chain data below.

**Impact:**  
Anyone can read the private password, severely breaking the functionality of the protocol.

**Proof of Concept:**  

The test case below demonstrates how anyone can read the password directly from the blockchain:

1. **Create a locally running chain**

    ```bash
    make anvil
    ```

2. **Deploy the contract to the chain**

    ```bash
    make deploy
    ```

3. **Run the storage tool**

   We use `1` because that's the storage slot of `s_password` in the contract.

    ```bash
    cast storage <ADDRESS_HERE> 1 --rpc-url http://187.0.0.1:8545
    ```

   You will get an output that looks like this:

    ```
    0x6d7950617373776f726400000000000000000000000000000000000000000014
    ```

   Parse that hex to a string with:

    ```bash
    cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
    ```

   You’ll get:

    ```
    myPassword
    ```

**Recommended Mitigation:**  
The contract architecture should be rethought. One approach is to encrypt the password off-chain and store the encrypted value on-chain. This would require the user to remember an off-chain decryption key. Additionally, consider removing the view function to avoid accidental transactions that expose sensitive information.

---



## Likelihood & Impact:
- Impact: HIGH
- Likelihood: High
- Severity: High

# High
- Worst offenders -> Least bad
## Medium
## Low

### [H-2] `PasswordStore::setPassword` has no access controls, allowing a non-owner to change the password

**Description:**  
The `PasswordStore::setPassword` function is marked `external`, but the natspec and the contract’s intended behavior indicate that only the owner should be allowed to call it.

```solidity
function setPassword(string memory newPassword) external {
    //@audit - There are no access controls
    s_password = newPassword;
    emit SetNetPassword();
}
```

**Impact:**  
Anyone can set or change the password, severely breaking the intended functionality.

**Proof of Concept:**  
Add the following to the `PasswordStore.t.sol` test file:

<details>
<summary>Code</summary>

```solidity
function test_anyone_can_set_password(address randomAddress) public {
    vm.assume(randomAddress != owner);
    vm.prank(randomAddress);
    string memory expectedPassword = "myNewPassword";
    passwordStore.setPassword(expectedPassword);

    vm.prank(owner);
    string memory actualPassword = passwordStore.getPassword();
    assertEq(actualPassword, expectedPassword);
}
```

</details>

**Recommended Mitigation:**  
Add access control to the `setPassword` function.

```solidity
if (msg.sender != s_owner) {
    revert PasswordStore_NotOwner();
}
```

---


## Likelihood & Impact:
- Impact: HIGH
- Likelihood: High
- Severity: High

### [I-1] The `PasswordStore::getPassword` natspec references a non-existent parameter

**Description:**

```solidity
/*
 * @notice This allows only the owner to retrieve the password.
 * @param newPassword The new password to set.
 */
function getPassword() external view returns (string memory) {
```

The natspec for `PasswordStore::getPassword` incorrectly references a parameter `newPassword`, even though the function takes no arguments.

**Impact:**  
The documentation is misleading and incorrect.

**Recommended Mitigation:**  
Remove the incorrect `@param` line from the natspec.

```diff
- * @param newPassword The new password to set.
```

## Likelihood & Impact
- Impact: HIGH
- Likelihood: NONE
- Severity: Informational/Gas/Non-crits

