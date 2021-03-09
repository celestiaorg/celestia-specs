# Networking

- [Wire Format](#wire-format)
  - [AvailableData](#availabledata)
  - [AvailableDataRow](#availabledatarow)
  - [WireTxPayForMessage](#wiretxpayformessage)

## Wire Format

### AvailableData

| name                | type                                      | description   |
|---------------------|-------------------------------------------|---------------|
| `availableDataRows` | [AvailableDataRow](#availabledatarow)`[]` | List of rows. |

### AvailableDataRow

| name     | type                                    | description      |
|----------|-----------------------------------------|------------------|
| `shares` | [Share](./data_structures.md#share)`[]` | Shares in a row. |

### WireTxPayForMessage

Defined as `WireTxPayForMessage` as a [wire type](./proto/wire.proto).

Accepting a `WireTxPayForMessage` into the mempool requires different logic than other transactions in LazyLedger, since it leverages the paradigm of block proposers being able to malleate transaction data. Unlike [SignedTransactionDataPayForMessage](./data_structures.md#signedtransactiondatapayformessage) (the canonical data type that is included in blocks and committed to with a data root in the block header), each `WireTxPayForMessage` (the over-the-wire representation of the same) has potentially multiple signatures.

Transaction senders who want to pay for a message will create a [SignedTransactionDataPayForMessage](./data_structures.md#signedtransactiondatapayformessage) object, `stx`, filling in the `stx.messageShareCommitment` field [based on the non-interactive default rules](../rationale/message_block_layout.md#non-interactive-default-rules) for `k = AVAILABLE_DATA_ORIGINAL_SQUARE_MAX`, then signing it to get a [transaction](./data_structures.md#transaction) `tx`. This process is repeated with successively smaller `k`s, decreasing by powers of 2 until `k * k <= stx.messageSize`. At that point, there would be insufficient shares to include both the message and transaction. Using the rest of the signed transaction data along with the pairs of `(tx.signedTransactionData.messageShareCommitment, tx.signature)`, a `WireTxPayForMessage` object is constructed.

Receiving a `WireTxPayForMessage` object from the network follows the reverse process: for each `message_commitment_and_signature`, verify using the [based on the non-interactive default rules](../rationale/message_block_layout.md#non-interactive-default-rules) that the signature is valid.
