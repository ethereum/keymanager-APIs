# Consensus Validator-client key management API

## Scope
The PR currently stipulates basically 3 endpoints for the validator client (VC) to provide:

 - `GET /eth/v1/keystores` - list the current keys of a VC
 - `DELETE /eth/v1/keystores` - delete keys from VC 
 - `POST /eth/v1/keystores` - add keys to a VC

# USE-Cases

## Add a key
As a home staker, 
I want to use a GUI to add a new validator key to my running validator-client.
__NEED__ validator-key file and password.

### API
 - Call `POST` to add key, supplying password, validator keystore, and (optionally) slashing protection data.

### GUI 
 - Get validator keystores and associated passwords from the user.
 - If the user has slashing-protection data, accept that also.
 - Get confirmation from the user that none of the supplied keys are currently in use.
 - Call `POST` API.

### Other behavior & Errors
 - `POST` API is not required to be transactional on bulk import.
 - `POST` API responds in the same order as request
 - `POST` API will not return an error response on failed key imports but instead return the `error` status.
 - `POST` API will fail when the length of passwords does not match the length of keystores. it is assumed the keystores will be one to one with the positions of the password.
 - Duplicate Keystores on import will not be imported and return with  `duplicate` status. ex.  first key is new, it gets success; second key is the same, then it gets duplicate (the first import in the batch added it)
 - Slashing Protection on keys that are not imported will still be imported. You could import slashing protection thats completely unrelated to the entire import keystore set. Keys that are covered in slashing protection but are not imported will count as `not_active`
    example:
    ```
    Import Request:
    keystores: a, b
    slashing protection: a, b, c
    Response: a, b keystores imported, and a, b, c slashing protection imported
    ```
- flows will cover errors in detail.


## Delete a key
As a home staker,
I want to use a GUI to remove a validator key from my running validator-client, 
because it has been exited and is no longer in use.
__NEED__ validator public key

As a home staker,
I just want to remove my validator key and ensure the validator is idle,
so that I can then add it to a new validator-client for hosting.

### API
 - call `DELETE` to remove the key

### GUI
 - Get the public key or validator id of the validator key to remove.
 - Get confirmation from the user the key will no longer be used by the validator-client.
 - Call `DELETE` API.
 - Allow user to save slashing protection data, or potentially cache it for future add operations associated with that key.

### Other behavior & Errors
 - `DELETE` API is not required to be transactional on bulk delete.
 - `DELETE` API will only return slashing protection for keys in the request
 - `DELETE` API will not return an error response on failed key deletions but instead return the public key with `error` status
 - Sending duplicate public keys in the request will result for the first instance of a key to return `deleted` status and the remaining instances of the same public key return `not_found` or `not_active` status.
 - Subsequent `DELETE` API calls can re-retrieve Slashing Protection data without need to delete a keystore.
 - Slashing Protection will only export for keys with `deleted` or `not_active` status.
 example:
    ```
    Request:
    a, a, b (not found), c
    Response:
    DELETED, NOT_ACTIVE, NOT_FOUND, DELETED
    Should give slashing protection for: a, c
    ```
- flows will cover errors in detail.


## Move a key
As a home staker,
I want to move my active validator key to another validator-client.
__NEED__ validator-key file and password.

### API
 - call `DELETE` to remove key from the old validator-client, get slashing protection data.
 - call `POST` to add key to new validator-client, including slashing-protection data, password, public key.

### GUI
 - Get the validator key file and password of the validator to move.
 - Get confirmation from the user the user is sure they want to move the validator key to the new client.
 - Call API's.

## Moving a key carefully
As a home staker,
I want to move my active validator key, but I want to be certain that the validator is seen as idle before it gets added to its new home.

### API
 - call `DELETE` to remove key from the old validator-client, get slashing protection data.
 - at this point, some kind of wait and check liveness would be required...
 - call `POST` to add key to new validator-client, including slashing-protection data, password, public key.

### GUI
 - Get the validator key file and password of the validator to move.
 - Get confirmation from the user the user is sure they want to move the validator key to the new client.
 - Call `DELETE`
 - Check validator liveness during next epoch(s), once it's seen idle prompt the user that the validator now appears idle, and it may be safe to finish the move operation now.
 - Call `POST` to add to new validator-client, including slashing-protection data, password, public key.


## List keys
As a home staker,
I want to list the keys on my validator-client, 
so that i can clearly identify where they are when I'm using my management GUI.

### API
 - Call `GET` to get a listing of the keys in use on a validator-client

### GUI
 - User is able to retrieve a listing of the public keys (and validator id's?) in use on a validator client.

## Get slashing-protection data for a removed key
As a home staker,
I want to be able to get a listing of a removed key,
so that I can retrieve slashing protection information about that key

### API
 - Call `DELETE` to get the slashing-protection data of a key that has already been removed.

### GUI
 - Get the public key of the validator to download slashing protection data for.
 - Call `DELETE` API.
 - Allow user to save slashing protection data.
