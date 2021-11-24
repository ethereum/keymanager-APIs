# Process Flows

## Import

__POST /eth/v1/keystores__

1. Verify keystores / passwords list length is the same
 - If this process precondition fails, then `400` error

2. Import slashing data, if present
 - No requirement for slashing protection data to contain any of the keys present in the keystores list, but any imports should be merged with any current records to ensure slashing-protection data is not being corrupted.
 - An empty keystores list with slashing protection data should still import slashing protection data.
 - Failure to import / read slashing protection data (if present) results in a `400` error, and _NO KEYS_ will be stored to disk or loaded in memory.
 - Success (either no-op when not provided, or successful import) will allow batch processing to start.

 __NOTE__ after this point, a `200` code _WILL_ be the response, unless a `500`, which should be considered a _BUG_.

3. Batch process keystores and passwords.
 - The keystores and passwords lists should be a matched set, otherwise
 - Each keystore is loaded separately, and can be fairly client dependent. An example may be:
 ```
  verify key is not currently active (status: duplicate)
  decode and verify keystore / password combination (status: error if fails)
  store to disk (status: error if fails)
  load into active validators in client (status: error if fails)
 ```
  - any individual failure should not stop the batch process, but at worst cause a
  failure to load an individual keystore.
4. Collate the response objects in the same order as the request keystores, and return a `200` status with the batch results.
