# v2 clipto architecture
This architecture focuses on having a single source of data for clipto. The older approach involved making use of database and contract storage. The contract and database both needed to work in conjunction for making sure all the data is consistent. Synchronization of the database was an issue for the previous architecture.

The new architecture(v2) uses a subgraph for all its storage and source of data, this will remove the Postgres database from dependency and also prevent the synchronization issue mentioned above. The extra data which was earlier stored in a database will now be indexed by the subgraph via contract events emitting a JSON.

## Outcome
### Request Creation
Any user who has their wallet connected on the platform can request any creator of their choice. They can browse the creator's list on the platform and select the one for which they want to request. To make a request the user will need to input the amount (greater than the min set by the creator) deadline days, and include instructions for completing a request. The request should be persisted even if the user leaves the site when a transaction is waiting for confirmation.

### Constraint
User flow should be seamless meaning even if the user leaves the site after completion of transaction but not waiting for its confirmation, the data should be recorded.

## Technical Design
### Request Creation
A user can explore creators and select one creator they want to request a clip from. The user can then fill in days and the amount (should be greater than the min set by the creator). The user will also put some instructions for the creator for the request. On submitting the form, the user will need to sign and make the payment, which will be triggered when the function `newRequest` will be called. Once the transaction is completed, the data from the form along with the transaction hash should be sent to the backend /request/create via API which will verify the transaction on-chain using the Alchemy API and validate the amount paid. The amount paid by the user will be transferred to the contract and kept in escrow until request completion. The user will see a notification of request creation. The request data will be indexed.

