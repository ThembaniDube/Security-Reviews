### [S-#] TITLE (Root Cause + Impact)
Storing the password on-chain makes it visible to anyone and no longer private

**Description:** 
All data stored on-chain is visible to anyone, and can be read directly on the blockchain.
The `PasswordStore::s_password` is intended to be private variable and only accessed 
through the `PasswordStore::getPassword` function, which is intended to be only called
by the owner of the contract.


**Impact:** 

**Proof of Concept:**
`make deploy`

`cast storage <ADDRESS_HERE> 1 --rpc-url http://127.0.0.1:8545`

the below test case shows how anyone can read the password directly from the blockchain
`deployed to 0x5FbDB2315678afecb367f032d93F642f64180aa3`
`cast storage 0x5FbDB2315678afecb367f032d93F642f64180aa3 1 --rpc-url 127.0.0.1:8545`
 and we get the bytes representation of the password
`cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014`
we get `myPassword`


**Recommended Mitigation:** 
:br
**Recommended Mitigation:** Due to this, the overall architecture of the contract should be rethought. One could encrypt the password off-chain, and then store the encrypted password on-chain. This would require the user to remember another password off-chain to decrypt the stored password. However, you're also likely want to remove the view function as you wouldn't want the user to accidentally send a transaction with this decryption key.



>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

### [S-#] TITLE (Root Cause + Impact)
`PasswordStore::setPassword` has no access controls, meaning a non owner can change password

**Description:** 
`PasswordStore::setPassword` is an external function, however, the natspec of the function
and overall purpose of the smart contract is that only the owner sets new password

```javascript

 function setPassword(string memory newPassword) external {
        s_password = newPassword;
        emit SetNetPassword();
    }


```

**Impact:** 
anyone can set/change the password, severly breaking the contract

**Proof of Concept:**

### [S-#] `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password

**Description:** The `PasswordStore::setPassword` function is set to be an `external` function, however the purpose of the smart contract and function's natspec indicate that `This function allows only the owner to set a new password.`

'''js
function setPassword(string memory newPassword) external {
    // @Audit - There are no Access Controls.
    s_password = newPassword;
    emit SetNewPassword();
}
'''

**Impact:** Anyone can set/change the stored password, severly breaking the contract's intended functionality

**Proof of Concept:** Add the following to the PasswordStore.t.sol test file:

'''js
function test_anyone_can_set_password(address randomAddress) public {
        vm.assume(randomAddress != owner);
        vm.startPrank(randomAddress);
        string memory expectedPassword = "myNewPassword";
        passwordStore.setPassword(expectedPassword);

        vm.startPrank(owner);
        string memory actualPassword = passwordStore.getPassword();
        assertEq(actualPassword, expectedPassword);
    }
'''

**Recommended Mitigation:** Add an access control conditional to `PasswordStore::setPassword`.

'''js
if(msg.sender != s_owner){
    revert PasswordStore__NotOwner();
}
'''

**Recommended Mitigation:** 
Add an access control conditional to `PasswordStore::setPassword`.

'''js
if(msg.sender != s_owner){
    revert PasswordStore__NotOwner();
}
'''
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
**Title:** [S-#] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect.

**Description:**
'''
/*
 * @notice This allows only the owner to retrieve the password.
@> * @param newPassword The new password to set.
 */
function getPassword() external view returns (string memory) {}
'''

The `PasswordStore::getPassword` function signature is `getPassword()` while the natspec says it should be `getPassword(string)`.

**Impact** The natspec is incorrect

**Recommended Mitigation:** Remove the incorrect natspec line
    /*
     * @notice This allows only the owner to retrieve the password.
-     * @param newPassword The new password to set.
     */

     >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
     ### [H-1] Storing the password on-chain makes it visible to anyone and no longer private

### [H-2] `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password

### [S-#] The 'PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect


