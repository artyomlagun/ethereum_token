# How to create Ethereum Token (ERC20 Ethereum Standard For Tokens)

## ERC20 Standard
The ERC20 standard is basically a specific set of functions which developers
must use in their tokens to make them ERC20 compliant. While this is not an
enforced rule, most DAPP developers are encouraged to follow the standards to
ensure that their tokens can undergo interactions with various wallets, exchanges
and smart contracts without any issues. This was great news for everyone because
now they at least had an idea of how future tokens are expected to behave. ERC20
tokens have gotten widespread approval and most of the DAPPS sold on ICO’s have
tokens based on the ERC20 standard.

So, what does a token need to have to be ERC20 compliant? It is basically a set
of 6 functions that can be recognized and identified by other smart contracts,
which in turn leads to seamless interactions. When executed, the following 4
basic activities are what all the ERC20 tokens required to do:

* Get the total token supply.
* Get the account balance.
* Transfer the token from one party to another.
* Approve the use of token as a monetary asset.

## Create token
In order to create an ERC20 token, you need the following:

* The Token’s Name
* The Token’s Symbol
* The Token’s Decimal Places
* The Number of Tokens in Circulation
### Code the Contract

```
  pragma solidity ^0.4.4;

  contract Token {

    /// @return total amount of tokens
    function totalSupply() constant returns (uint256 supply) {}

    /// @param _owner The address from which the balance will be retrieved
    /// @return The balance
    function balanceOf(address _owner) constant returns (uint256 balance) {}

    /// @notice send `_value` token to `_to` from `msg.sender`
    /// @param _to The address of the recipient
    /// @param _value The amount of token to be transferred
    /// @return Whether the transfer was successful or not
    function transfer(address _to, uint256 _value) returns (bool success) {}

    /// @notice send `_value` token to `_to` from `_from` on the condition it is approved by `_from`
    /// @param _from The address of the sender
    /// @param _to The address of the recipient
    /// @param _value The amount of token to be transferred
    /// @return Whether the transfer was successful or not
    function transferFrom(address _from, address _to, uint256 _value) returns (bool success) {}

    /// @notice `msg.sender` approves `_addr` to spend `_value` tokens
    /// @param _spender The address of the account able to transfer the tokens
    /// @param _value The amount of wei to be approved for transfer
    /// @return Whether the approval was successful or not
    function approve(address _spender, uint256 _value) returns (bool success) {}

    /// @param _owner The address of the account owning tokens
    /// @param _spender The address of the account able to transfer the tokens
    /// @return Amount of remaining tokens allowed to spent
    function allowance(address _owner, address _spender) constant returns (uint256 remaining) {}

    event Transfer(address indexed _from, address indexed _to, uint256 _value);
    event Approval(address indexed _owner, address indexed _spender, uint256 _value);
  }



  contract StandardToken is Token {

    function transfer(address _to, uint256 _value) returns (bool success) {
      //Default assumes totalSupply can't be over max (2^256 - 1).
      //If your token leaves out totalSupply and can issue more tokens as time goes on, you need to check if it doesn't wrap.
      //Replace the if with this one instead.
      //if (balances[msg.sender] >= _value && balances[_to] + _value > balances[_to]) {
      if (balances[msg.sender] >= _value && _value > 0) {
        balances[msg.sender] -= _value;
        balances[_to] += _value;
        Transfer(msg.sender, _to, _value);
        return true;
      } else { return false; }
    }

    function transferFrom(address _from, address _to, uint256 _value) returns (bool success) {
      //same as above. Replace this line with the following if you want to protect against wrapping uints.
      //if (balances[_from] >= _value && allowed[_from][msg.sender] >= _value && balances[_to] + _value > balances[_to]) {
      if (balances[_from] >= _value && allowed[_from][msg.sender] >= _value && _value > 0) {
        balances[_to] += _value;
        balances[_from] -= _value;
        allowed[_from][msg.sender] -= _value;
        Transfer(_from, _to, _value);
        return true;
      } else { return false; }
    }

    function balanceOf(address _owner) constant returns (uint256 balance) {
      return balances[_owner];
    }

    function approve(address _spender, uint256 _value) returns (bool success) {
      allowed[msg.sender][_spender] = _value;
      Approval(msg.sender, _spender, _value);
      return true;
    }

    function allowance(address _owner, address _spender) constant returns (uint256 remaining) {
      return allowed[_owner][_spender];
    }

    mapping (address => uint256) balances;
    mapping (address => mapping (address => uint256)) allowed;
    uint256 public totalSupply;
  }


  //name this contract whatever you'd like
  contract ERC20Token is StandardToken {

    function () {
      //if ether is sent to this address, send it back.
      throw;
    }

    /* Public variables of the token */

    /*
    NOTE:
    The following variables are OPTIONAL vanities. One does not have to include them.
    They allow one to customise the token contract & in no way influences the core functionality.
    Some wallets/interfaces might not even bother to look at this information.
    */
    string public name;                   //fancy name: eg Simon Bucks
    uint8 public decimals;                //How many decimals to show. ie. There could 1000 base units with 3 decimals. Meaning 0.980 SBX = 980 base units. It's like comparing 1 wei to 1 ether.
    string public symbol;                 //An identifier: eg SBX
    string public version = 'H1.0';       //human 0.1 standard. Just an arbitrary versioning scheme.

  //
  // CHANGE THESE VALUES FOR YOUR TOKEN
  //

  //make sure this function name matches the contract name above. So if you're token is called TutorialToken, make sure the
  //contract name above is also TutorialToken instead of ERC20Token

    function ERC20Token() {
      balances[msg.sender] = NUMBER_OF_TOKENS_HERE;               // Give the creator all initial tokens (100000 for example)
      totalSupply = NUMBER_OF_TOKENS_HERE;                        // Update total supply (100000 for example)
      name = "NAME OF YOUR TOKEN HERE";                                   // Set the name for display purposes
      decimals = 0;                            // Amount of decimals for display purposes
      symbol = "SYM";                               // Set the symbol for display purposes
    }

    /* Approves and then calls the receiving contract */
    function approveAndCall(address _spender, uint256 _value, bytes _extraData) returns (bool success) {
      allowed[msg.sender][_spender] = _value;
      Approval(msg.sender, _spender, _value);

      //call the receiveApproval function on the contract you want to be notified. This crafts the function signature manually so one doesn't have to include a contract in here just for this.
      //receiveApproval(address _from, uint256 _value, address _tokenContract, bytes _extraData)
      //it is assumed that when does this that the call *should* succeed, otherwise one would use vanilla approve instead.
      if(!_spender.call(bytes4(bytes32(sha3("receiveApproval(address,uint256,address,bytes)"))), msg.sender, _value, this, _extraData)) { throw; }
      return true;
    }
  }
```

After these steps we should test our token.

## Test the token

If you don’t have it already, download [MetaMask](https://metamask.io/). They have an easy-to-use interface to test this.

Once you’ve installed MetaMask, make sure that you’re logged in and setup on the Ropsten test network. If you click in the top left where it says ‘Main Ethereum Network’ you can change it to Ropsten.

To confirm, the top of your MetaMask window should look like this:
![screen1](https://steemitimages.com/0x0/https://steemitimages.com/DQmRR7avYAnDebACb1CCEBLvDeEo1cckHkXTViJKPc336KG/Screen%20Shot%202017-07-10%20at%201.25.35%20PM.png)

Now head to the [Solidity Remix Compiler](https://ethereum.github.io/browser-solidity/) - it’s an online compiler that allows you to publish the Smart Contract straight to the blockchain.

Copy/Paste the source of the contract you just modified into the main window. It'll look something like this:
![screen2](https://steemitimages.com/DQmbSygUYivCNyTiMLqrKcYXrsP2WWJ6qJLTacDM3ygJVGJ/Screen%20Shot%202017-07-10%20at%202.34.16%20PM.png)

Now, go to settings on the right and select the latest release compiler version (NOT nightly), as well as unchecking ‘Enable Optimization’.

So it should look something like this:
![screen3](https://steemitimages.com/0x0/https://steemitimages.com/DQmUqM2jbnebusoywDez8qN7hQ9qBWp9HVhPmwRDDeSbvwp/Screen%20Shot%202017-07-10%20at%201.47.33%20PM.png)

Keep note of the current Solidity version in the compiler. We’ll need that later to verify the contract source.

Now go back to the Contract tab and hit ‘Create’ under the name of the Token function that you created.

So you’d hit ‘Create’ under ‘TutorialToken’.
![screen4](https://steemitimages.com/DQmfFhHPUkGTM9nBUSrQGBPGiZB6X8GEgmUPge43ECWv2Ca/Screen%20Shot%202017-07-10%20at%201.31.03%20PM.png)

What will happen is MetaMask will pop up asking you to hit ‘Submit’ to pay for the transaction.

Remember, this is the Ropsten test net, so it’s not real Ether. You can double check to make sure you’re on the test network in MetaMask before you hit submit just to be sure.

When you hit Submit, it’ll say ‘Contract Pending’ in MetaMask. When it’s ready, click the ‘Date’ and it’ll bring up the transaction in EtherScan. Like this:
![screen5](https://steemitimages.com/0x0/https://steemitimages.com/DQmfDsvRuu5JPcP8udz2iaXFhauHGs5nVB53PjMep9P3ni9/Screen%20Shot%202017-07-10%20at%201.34.45%20PM.png)

If you click the date, it’ll bring up the following screen:

![screen6](https://steemitimages.com/DQmWJT5jURdy5no1NuFPZFrBJw1KPufqWDmJ5MxTLudJhzX/txscreen.jpg)

If this process worked, it’s now time to verify the source code.

If not, you’ll want to go back to the code and modify the source to get it to work.

I can’t say exactly what that’d look like, but this is where having a “programmers mindset” comes in. A lot of time there’s unexpected bugs that can’t be predicted until you just do it.

[More information you can find here](https://steemit.com/ethereum/@maxnachamkin/how-to-create-your-own-ethereum-token-in-an-hour-erc20-verified)
