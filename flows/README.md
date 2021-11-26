# Process Flows

## Import

__POST /eth/v1/keystores__

1. Verify keystores / passwords list length is the same
 - If this process precondition fails, then `400` error

2. If they keystores / passwords lists are blank, then `200`. Could optionally read slashing data and respond error if it is invalid, but there is nothing to do with this import, as slashing data will only be loaded for keys that can be imported.

3. Load the slashing data from the request, if present. The slashing data may either be loaded in its entirety, or imported on a per-key basis during the batch processing, which may be cleaner as it would ensure only keys that are being loaded have anything changed in slashing protection data...
 - failure to read slashing protection data if it is present is a `400` error.


 __NOTE__ after this point, a `200` code _WILL_ be the response, unless a `500`, which should be considered a _BUG_.

4. Batch processing for each keystore:
```
  Decode and verify keystore / password combination (status: error if fails)
  Verify key is not currently active (status: duplicate if it is)
  Load the slashing protection data for this key, if supplied (status: error if the import of slashing data here fails for any reason)
  Write keystore and password to persistent storage (status: error if either fails, cleanup any keystore or password data)
  Load into active validators in client (status: error if fails, cleanup any keystore or password data)
```
  - any individual failure should not stop the batch process, but at worst cause a failure to load an individual keystore.
  - any individual failure should attempt to ensure that the key being imported can't possibly be started later. This may be via some kind of disabled mechanism, or removing files - the details here would be implementation specific.
  - presumes that storage of keystore/password combinations is separate to activating validator keystores. The key idea is if a validator 'fails' to be imported, it can't possibly later be started by other processes, so if it's written to disk and loads on restart, this would be bad.
  - Slashing protection data will only be loaded if there is a keystore that matches, and passes other validations (password validation, not active).
5. Collate the response objects in the same order as the request keystores, and return a `200` status with the batch results.
