## Bounty 

Sol A: Trying to get creator's address

    Step 1: User submit a link 
    https://opensea.io/assets/matic/0xf74c1b1e1b59fc574c36117e54bcd1ab6d5e19b7/0
    Step 2: Fetch the owner of token
    Step 3: Check if existing creator
    Step 4: Find available ENS
    Step 5: Confirm from requester
    Step 6: Proceed with transaction

Sol B: Trying to only invite the creator

    Step 1: User creates an invite
    Step 2: Invite is sent
    Step 3: Creator accepts the invite and onboards
    Step 4: Notify requester to make a booking
    Step 5: Usual userflow

### Suggestion

1. To add this feature, we should not consider changing the contracts, it should not
be required.

### Issues

1. While creating new request the contract checks if the creator exists
2. Request requires payment  of value greater than 0
