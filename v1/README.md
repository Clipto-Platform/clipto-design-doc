## Clipto Platform
Clipto is a platform for users who want to request video content from their favorite celebrities/content creators. The platform provides a way to onboard creators and consumers who can then request video content by providing fees in crypto, instructions, and deadline for the delivery. The video delivered by a content creator will be delivered to the consumer as an NFT. 

Clipto is a web3 version of [Cameo](https://cameo.com)

Clipto may or may not charge some fee for the platform.

### Ubiquitous Language
- Visitor: A visitor to clipto who is not signed in
- User: A new user to clipto who has not yet registered as a creator
- Creator: A user who has registered as a creator


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


## Design Goals
### Constraint
users should have seamless experience when doing the transaction. Even if user closes the tab after doing the transaction the response must be recorded in the database.

### Reason for constraint
There is considerable delay in getting response from the contract when user does a transaction. In that time gap, if user closes the tab or refreshes the page,  tansaction would be completed i.e This transaction would be on chain but the transaction details won't get stored in the database. Since the data did not get stored in the database creator won't get the request from the user even though the transaction was completed. Explained in Drawback section.


## Technical Design
### Wallet Connect
The website needs to have a `Connect Wallet` button. Wallets are software that stores the private/public address of the user. This wallet is used to sign all the transactions/messages. Wallets are basically mediators which connect our application with the blockchain.
 	Clicking on the connect wallet button will give two options a- To connect browser wallets b- To connect mobile wallets. We are using the `web3-react` library which basically connects our application with the wallet. There are two types of connectors `injected` for browser wallets and `wallet connect` for mobile wallets which can be used to connect specific wallets.  Choosing the Mobile Wallet will prompt the user with the QR code on the app. The user needs to scan the QR code from the mobile wallet. This QR code contains all the information required to initiate the connection between the mobile wallet and the application
 	Users can have multiple accounts on the same wallet. The user can switch accounts in their wallets which should be reflected on the site.


### Onboarding Creators
A user who wants to become a creator, will visit the website and connect their respective wallet. The user should be able to see an option to `Become a Creator`. There are two steps to get onboarded, first the user tweets using their Twitter account and then copy the link of the tweet and paste it into the form. The tweet should be prepopulated with a message (including their address). On submitting, the tweet link and creator's address will be sent to the backend via REST API `user/verify`, which will verify the tweet via Twitter APIs, which will check for a wallet address. The API should return all the Twitter profile details in the response. The response data will then populate the form containing their name, Twitter id, profile picture, and wallet address. Then the user will enter the minimum amount(crypto), the minimum deadline for delivery of the clip. When the user submits the form, the `registerCreator` function of the contract will be called, which will initiate a transaction, once the transaction is completed the frontend will send all the respective data to the backend via API `user/create`, which should add the creator to the database. This will complete the creator onboarding. 


### Request Creation
A user can explore creators and select one creator they want to request a clip from. The user can then fill in days and the amount (should be greater than the min set by the creator). The user will also put some instructions for the creator for the request. On submitting the form, the user will need to sign and make the payment, which will be triggered when the function `newRequest` will be called. Once the transaction is completed, the data from the form along with the transaction hash should be sent to the backend `/request/create` via API which will verify the transaction on-chain using the Alchemy API and validate the amount paid. After successful verification, the request should be stored in the database. The amount paid by the user will be transferred to the contract and kept in escrow until request completion. The user will see a notification of request creation.


### Request Completion
The creator can see received orders which they can fulfil. The website will provide a form with a name, description, and video upload to create an NFT. The creator will select a video to upload which will be uploaded using the Glass API(third party service) which on completion will return us a permalink, which stores the metadata for the NFT. This permalink will be used to mint the NFT, by setting the tokenUri of the NFT contract. Once successful uploading the creator will call the contract function `deliverRequest` via a button. Once the transaction is successful the frontend will update the request status to `delivered` in the database via API `request/finish`. Once the API responds, the website will show a notification denoting the successful completion of the request. The creator will be paid for the request via the contract. The payment amount will be dependent on the Clipto fee parameter(% of deduction).


### NFT View
  The requester will be able to view completed requests. Once clicking on a completed request, the NFT video, and all relevant details should be visible. The NFT details like token address and index will be fetched from the Graph API which will be indexing all the events from the contract. The user will be able to see NFT video, links to OpenSea, and Polygon Scan. There will be an NFT History card that will show the owner history for the current NFT which will be fetched via Alchemy API by reading the `Transfer` event of the ERC721 contract.


### Refund
 The requester will be able to view expired requests for which they can initiate refunds. The requester will simply initiate the refund via clicking the `Claim Refund` button which will call `refundRequest` function of the contract. Once, the transaction is completed, the website will update the status of the request via REST API `request/refund`. That will update the request in the database as refunded. After getting the response from the API the website will show notification of refund completion and update the request UI denoting that the request was refunded. The contract should credit the requester’s account with the amount earlier paid for the request.

## Drawbacks
### DB and Contract Out of Sync
The contract interaction creates a transaction on the blockchain, the transaction usually takes a while to process and mine. The API which updates the record in the database for certain entity need to wait until the transaction is completed which also requires the user to be on the website and not close the tab. If the user closes the window/tab before the transaction completes the data is not recorded in the database. So in this case, the contract is up to date but the database is not which results in out-of-sync data.

### API authentication
The APIs even with Recaptcha and SIWE auth can still be abused to some extent, which would create unnecessary records in the database. For example, a malicious user can create multiple requests which could have identical data and can abuse the on-chain verification mechanism or if successful to bypass can create records in the database.


## Solution
### DB and Contract Sync
Instead of relying on the website to confirm the transaction and then inform the database about the change. We propose that the backend/server keeps listening for the contract events (`NewRequest`, `RequestUpdated`, `DeliveredRequest`, `RefundedRequest`) which maps to the respective APIs which are used to update the records in the database.


- Request Creation: When the user submits the request, the request gets stored without on-chain verification as an `unverified request`. When the user completes the transaction, the backend/server will be able to catch the `NewRequest` event and update the request which can be made `verified`. 


- Request Completion / Refund: The transaction initiated for these operations would not require any direct API calls with the backend, the backend just has to listen to their respective events and update the database thereon.

The validity of the data in the database can also be verified by using the Graph which can be called via a Scheduler which then can sync the data from Graph and the database (extreme condition only)

The above method can solve API security as well as improve user interaction. If implemented, the user can move on from the website instead of waiting for the transaction on the chain to be completed. The frontend would still be waiting for the confirmation but the backend won't need to rely on the website for data sync and updates.



The above workflow can be visualized as below diagram

![sequence](https://github.com/AP-Atul/clipto-design-doc/raw/main/assets/clipto_proposal.png)










