## Clipto Platform
Clipto is a platform for users who want to request video content from their favorite celebrities/content creators. The platform provides a way to onboard creators and consumers who can then request video content by providing fees in crypto, instructions, and deadline for the delivery. The video delivered by a content creator will be delivered to the consumer as an NFT. 

Clipto is a web3 version of [Cameo](https://cameo.com)

Clipto may or may not charge some fee for the platform.

### Ubiquitous Language
- Visitor: A visitor to clipto who is not signed in
- User: A new user to clipto who has not yet registered as a creator
- Creator: A user who has registered as a creator

## Technical Design
### Wallet Connect
  - Website needs to have a `Connect Wallet` button.
  - Clicking on it will give two options a. `Metamask` b. `Mobile Wallet`.
  - Mobile Wallet will then provide a QR Code to connect.
  - The user can switch accounts in their wallets which should be reflected on the site.


### Onboarding
  - A user can connect their respective wallet
  - The user should be able to see an option to `Become a Creator`
  - The user should see the option to tweet which will redirect to Twitter with prepopulated content (including their address).
  - A textbox for verifying the tweet's link in the previous step should be visible
  - On submitting, the tweet should be verified for address, user profile.
  - The final form for onboarding should be prepopulated with Twitter profile details along with min amount, min deadline.
  - When the user submits the form, the creator should be registered on the contract and the profile should be saved in the database via API
  - The contract should also deploy NFT contract per creator.
  - The API is protected with sign auth, so the user will need to Sign with a predefined message string.


### Request Creation
  - A user can explore creators and select one creator they want to request.
  - The user can then fill in days and amount (should be greater than the min set by creator)
  - The user can put some instructions for the creator for the request.
  - On submitting the form, the user will need to sign and make the payment for the amount.
  - This request would be verified on the chain in the backend and once verified, stored in the database.
  - This amount will be transferred to the contract and kept in escrow until request completion.
  - The user should be able to see the made requests.


### Request Completion
  - The creator can see received orders which they can fulfill
  - The website will provide a form with name, description, and video upload to create an NFT.
  - The uploaded video should go to arweave
  - The arweave's permalink will be used to mint the NFT for the creator's NFT contract.
  - After uploading the creator will see a button to send the NFT to the requester.
  - On successful submission, the request should be fulfilled on the contract.
  - The creator will be paid for the request and the status of the request will be updated in the database via API.
  - The payment amount will be depending on the Clipto fee parameter(% of deduction).


### NFT View
  - The requester should be able to view completed requests
  - One clicking on the request, the NFT video, and all relevant details should be visible
  - The NFT data will be fetched from The Graph protocol which will be indexing all the data of the Clipto contract.
  - The user should be able to view the NFT on OpenSea and Polygon Scan.


### Refund
  - The requester should be able to view expired requests.
  - The refund button would be available, on clicking it the contract should initiate a refund.
  - After completion of the refund the status of the request will be updated in the database via API.


## Design Goals
### Constraint
users should have seamless experience when doing the transaction. Even if user closes the tab after doing the transaction the response must be recorded in the database.

### Reason for constraint
There is considerable delay in getting response from the contract when user does a transaction. In that time gap, if user closes the tab or refreshes the page,  tansaction would be completed i.e This transaction would be on chain but the transaction details won't get stored in the database. Since the data did not get stored in the database creator won't get the request from the user even though the transaction was completed. Explained in Drawback section.


## Outcome
### Wallet Connect
Any user coming to the platform should be able to connect their crypto wallet whether that be Metamask or Mobile Wallet and interact with the platform.

### Onboarding Process
Once the user has connected their wallet, they can become a creator.
To onboard a creator, the creator has to tweet which contains their public crypto wallet address, which then can be verified by the platform, and then they can fill up a form containing details of the minimum fee they want to charge and minimum days for delivery. The same Twitter profile/account cannot be used to create a new creator profile on the platform.

### Request Creation
Any user who has their wallet connected on the platform can request any creator of their choice. They can browse the creator's list on the platform and select the one for which they want to request. To make a request the user will need to input the amount (greater than the min set by the creator) deadline days, and include instructions for completing a request. 

### Request Completion
The creator can see all the requests they have received, these requests can be completed by uploading a video along with title and description. The platform should create an NFT for this and make the requester owner of the NFT. After completion of the Request, the creator should be paid the amount.

### NFT View
The requester should be able to browse completed requests and see the NFT video completed by the creator, the user should be able to view it on various sites including OpenSea, and Polygon scan.

### Refund
The requester can request a refund for an expired request, this will happen when the creator fails to upload a video and completion of the request.


## Drawbacks
### DB and Contract Out of Sync
The contract interaction creates a transaction on the blockchain, the transaction usually takes a while to process and mine. The API has to wait until the transaction is completed which also requires the user to be on the website and not close the tab. The transaction is not yet recorded in the database which is the primary source of all the data to look for.


If the user leaves the site before completion of the transaction then only the contract will have the request data and the database fails to track that which results in out-of-sync data.

### API authentication
The APIs even with Recaptcha and SIWE auth can still be abused to some extent, which would create unnecessary records in the database. For example, a malicious user can create multiple requests which could have identical data and can abuse the on-chain verification mechanism or if successful to bypass can create records in the database.


## Solution
### DB and Contract Sync
Instead of relying on the website to confirm the transaction and then inform the database about the change. We propose that the backend/server keeps listening for the contract events (`NewRequest`, `RequestUpdated`, `DeliveredRequest`, `RefundedRequest`) which maps to the respective APIs which are used to update the records in the database.


- Request Creation: When the user submits the request, the request gets stored without on-chain verification as an `unverified request`. When the user completes the transaction, the backend/server will be able to catch the `NewRequest` event and update the request which can be made `verified`. 


- Request Completion / Refund: The transaction initiated for these operations would not require any direct API calls with the backend, the backend just have to listen to their respective events and update the database thereon.

The validity of the data in the database can also be verified by using the Graph which can be called via a Schedule which then can sync the data from Graph and the database (extreme condition only)

The above method can solve API security as well as improve user interaction. If implemented, the user can move on from the website instead of waiting for the transaction on the chain to be completed. The frontend would still be waiting for the confirmation but the backend won't need to rely on the website for data sync and updates.


The above workflow can be visualized as below diagram

![sequence](https://github.com/AP-Atul/clipto-design-doc/raw/main/assets/clipto_proposal.png)










