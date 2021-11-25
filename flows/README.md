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

## Delete

__DELETE /eth/v1/keystores__

1. Deserialize the JSON request from the caller. On failure return an HTTP `400` status. Let `request` be
   the deserialized request.

2. Remove each key from `request.pubkeys` one-by-one, recording its provisional status:
   1. Remove the key from the in-memory registry of keys so that it is no
      longer actively signing messages. If no matching key is found, provisionally
      set the key's status to `not_found` and continue to the next key.

   2. (Optional) If the keymanager keeps a database of known keys on disk (i.e. metadata),
      remove or disable the key in this database. Disabling the key first may
      be useful for certain storage schemes, to provide crash safety.
      This is implementation dependent.

      If the delete/disable fails, set the key's status to `error` with the
      reason for the failure and continue to the next key.

   3. Delete the keystore file or keystore material from disk. If this fails, set the provisional
      status to `error` with an appropriate message and continue to the next key.

   4. (Optional) If the key was disabled rather than deleted in (ii),
      permanently remove the key metadata from the on-disk database. If this fails, set the
      provisional status to `error` with an appropriate message and continue to the next key.

   5. Set the key's provisional status to `deleted` (all previous steps must have succeeded).

3. For all keys from `request.pubkeys` with provisional status equal to `deleted` or `not_found`,
   export slashing protection data in EIP-3076 format.

   - If the export fails, set the final status for all provisionally `deleted` or `not_found`
     keys to `error`, with the `error` message describing the slashing protection failure.
     The response should include a slashing protection object with valid `metadata` and
     `data == []`.

   - On success, for all keys with provisional status `not_found`, set their final status
     accordingly:
     * If slashing protection data was found, set the final status to `not_active`.
     * Otherwise, set the final status to `not_found`.

   - On success set the final status for all `deleted` keys to `deleted` regardless of whether
     slashing protection data was found. It could be that a key lacks slashing protection data
     because it was never active. The caller may warn the user in such a case if it wishes.

4. Collate the final statuses for each key in the same order as the request and send them in a
   correctly-structured response along with the slashing protection data from (3).

Notes:

- The flow above is only one example of a conformant implementation, implementations are free to
  implement the API in any way they wish while remaining compatible. E.g. if a keymanager stores all
  key data in a single database and is able to perform an atomic delete for all keys, then it may do
  this. Likewise if an implementation would like to export slashing protection data for each key one
  at a time and then merge these into the final response, then it may do this.
- The keymanager must never delete the slashing protection data for a key.
- The keymanager must never return slashing protection data for keys that are not part of the
  request, and should never return slashing protection data for a key with an `error` status.
