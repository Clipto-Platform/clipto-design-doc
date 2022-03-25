## Multi-Currency
Multi-currency refers to using different currencies to make requests to a content creator.
Currenly, clipto only supports native (matic) crypto on the polygon chain, which limits 
some user from taking interest and interacting with the platform.

### Approach
A user visiting the site will go through list of creators and select the creator they want 
to request from. The user will then select the type of the crypto they want to use to make
the request with. 


`User selects ERC20 token type from selectable`


When the user clicks on `Book Now` button the `ERC20` contract of the 
selected currency will be initialized.

```js
const ERC20_CONTRACT_ADDRESS = "0x....";
const erc20Contract = new ethers.Contract(ERC20_CONTRACT_ADDRESS, ERC_ABI, SIGNER);
```

The `approve` function with the amount will be called on `ERC20` contract. Which approves the 
transfer of the `amount` from user's account to `CliptoExchange` Contract

```js
const CLIPTOEXCHANGE = "0x....";
const transaction = await erc20Contract.approve(CLIPTOEXCHANGE, amount);
```


Once the user approves the call, the `newRequest` function which is now a `non payable` function
will be called.

```js
const transaction = await cliptoExchangeContract.newRequest(ERC20_CONTRACT_ADDRESS, ...data);
```

The contract call when approved, the clipto contract will check for allowance of the amount specified.
And call the `transfer` of the ERC20 and transfer the allowed amount to the Clipto Exchange contracts.

```s
ERC20 token = ERC20(ERC20_CONTRACT_ADDRESS);
uint256 allowance = token.allowance(msg.sender, address(this));

# check whether the allowance matches the amount
require(allowance >= amount, "Check the token allowance");

# transfer the allowance amount to the contract
# ERC.transferFrom(from, to, amount)
token.transferFrom(msg.sender, address(this), amount);

# ...go to complete the request
```

To keep track of which token was used to make the request, one way is to store the ERC20 contract 
address in the `Request` struct and use in `deliverRequest` and `refundRequest` and transfer
exactly the type of crypto which was initially used


```s
struct Request {
    # Address of the requester
    address requester;

    # Amount of L1 token set for the request
    uint256 amount;
    
    # state of request fulfillment
    bool fulfilled;

    # erc 20 token used while create a request
    address token;
}
```


For completing a request the flow of contract functions would be

```s
# getting request from state
Request storage request = requests[msg.sender][index];

# nft token minting stuffs
...

# get the fee
uint256 feeAmount = (request.amount * feeRate) / scale;

# transfer the fee to the owner
bool sent = ERC20(ERC20_CONTRACT_ADDRESS).transfer(owner, feeAmount);
require(sent, "Fee delivery failed");

# transfer the creator their share
uint256 paymentAmount = = request.amount - feeAmount;
sent = ERC20(ERC20_CONTRACT_ADDRESS).transfer(msg.sender, feeAmount);
require(sent, "request delivery failed");
```

### Notes
- This flow ensures the same crypto type is used for refund, completion of request, and 
payment of the fee which was used for making a request.


