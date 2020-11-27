Networking
===

- [Wire Format](#wire-format)
    - [AvailableData](#availabledata)
    - [AvailableDataRow](#availabledatarow)

## Wire Format

### AvailableData

| name                | type                                      | description   |
| ------------------- | ----------------------------------------- | ------------- |
| `availableDataRows` | [AvailableDataRow](#availabledatarow)`[]` | List of rows. |

### AvailableDataRow

| name     | type                                    | description      |
| -------- | --------------------------------------- | ---------------- |
| `shares` | [Share](./data_structures.md#share)`[]` | Shares in a row. |
