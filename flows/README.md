# Process Flows

## Import

__POST /eth/v1/keystores__

1. Verify keystores / passwords list length is the same
 - If this process precondition fails, then `400` error

2. If the keystores / passwords lists are blank, then `200`. Could optionally read slashing data and respond error if it is invalid, but there is nothing to do with this import, as slashing data will only be loaded for keys that can be imported.

3. Load the slashing data from the request, if present. The slashing data may either be loaded in its entirety, or imported on a per-key basis during the batch processing, which may be cleaner as it would ensure only keys that are being loaded have anything changed in slashing protection data.
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

## Delete

__DELETE /eth/v1/keystores__

1. Deserialize the JSON request from the caller. On failure return an HTTP `400` status. Let `request` be
   the deserialized request.

   _NOTE_: after this point the response will be a 200 OK. Any 500 response is to be considered a
   bug.

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

