## Migrations of current exising v2 contract
1. The current storage structure for creator
```solidity
creators {
    // mapping
    address,  // creator's address
    address,  // clipto nft token address
}

Event CreatorRegistered {
    address,   // creator's address
    address,   // clipto nft token address
    data,      // extra json for other details
}
```

Migration function can be created with params as
```solidity
function migrateCreator(
    address [] creators_address,    // all addresses of creator
    address [] tokens_address,      // all nft tokens of creator 
    string  [] json_data            // all extra json of creator
)
public
onlyOwner                           // allows only owner to call     
{
   // loop through the array
   for(uint i = 0; i < creators_address.length; i++) {
       // create nft contract instance
       CliptoToken token = CliptoToken(tokens_address[i]);
       
       // update creator mappings
       creators[creators_address[i]] = token;

       // could create an event for each creator
       emit CreatorRegistered(creators_address[i], token, json_data[i]);
   }

   // or use a common new event to emit all and index in subgraph 
   emit MigrationCreator(creators_address, tokens_address, json_data);
}
```


2. Storage structure of request
```solidity
requests {
    // mapping
    address,    // creator's address
    Request[],  // array of all request struct
}

// Request struct
struct Request
{
    address,   // requester's address
    amount,    // amount of the request
    fulfilled, // status of the request
}
```

Migration function can be created with params as
```solidity
function migrateRequest(
    address [] creators_address,     // all addresses of creator
    address [] requester_address,    // all addresses of the requester
    uint256 [] amounts,              // all amounts of the requests
    bool    [] fulfilleds,           // all statuses of the requests
    string  [] json_data,            // extra json data of the requests
)
public
onlyOwner       // only owner has the access to call this function
{
    for(uint i = 0; i < creators_address, i++) {
        Request request = Request({
            requester_address[i],
            amounts[i],
            fulfilleds[i]
        });

        requests[creators_address[i]].push(request);

        // we could emit a single event per requests
        emit NewRequest(
            creators_address[i],
            requester_address[i],
            amounts[i],
            requests[creators_address[i]].length - 1, 
            fulfilleds[i], 
            json_data[i]
        );
    }

    // or could emit all data at once and index and parse in the subgraph
    emit MigrationRequests(creators_address, requester_address, amounts, fulfilleds, json_data);
}
```



