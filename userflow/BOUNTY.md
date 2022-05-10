## Bounty Feature

### Outcome

The requester should be able to book a request to a creator who wants to join the platform and
who is not a part of the platform yet. The creator should be able to onboard on the platform
with the invite the requester will send.

### Approach

---

#### Sol A: Get the creator's address from the requester

The requester visits the Clipto's bounty page. In the bounty form, they enter the "to be a creator"'s
`Ethereum wallet address` or a `link` which points to an NFT preferably on OpenSea, of which the
owner he wants to request. The `link` will involve the NFT contract address and the token id which
will help to get the owner of the NFT. After having the creator's address and the requester's address,
the requester can proceed with normal request creation. And finally, the requester tweets out the
link of the invitation and will tag the creator. The creator joins the website with the link, this link
doesn't have to be unique since we know exactly which address to point the request to. Once the creator
signs up, they should see the order on the Orders page.

We could also fetch `ENS` if the address is associated with the one which will help the requester to
validate.

Advantages

- Extra work to only fetch the owner from the nft
- Requires minor changes to the contracts
- No centralized party is involved

Disadvantages

- Requires creator's address beforehand to make a request

---

#### Sol B: Storing the bounty on the contracts

The requester visits the Clipto's bounty page. In the bounty form, the requester enters the "to be a creator"'s
Twitter handle and completes the form. The requested amount is paid on the contract, the request will be then
stored on the contract with a unique identifier as the Twitter handle. Finally, the requester will tweet out
the onboarding link with the creator's Twitter handle tagged. The creator can then follow the onboarding process
using the link. The bounty request will be visible on the Orders page which will be fetched based on the
creator's Twitter handle. The further request flow will remain the same.

Advantages

- Doesn't require the creator's address to request creation

Disadvantages

- Need to make multiple changes to contracts, and add new events, states, and functions.
- Need centralized service to verify if the creator is using the same Twitter handle as the bounty request.
