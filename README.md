# Celestia Specifications

[![Community](https://img.shields.io/badge/chat%20on-discord-orange?&logo=discord&logoColor=ffffff&color=7389D8&labelColor=6A7EC2)](https://discord.gg/YsnTPcSfWQ)

## Notice

**THIS REPOSITORY IS NOT CURRENTLY ACTIVELY MAINTAINED, AND DOES NOT REFLECT THE CELESTIA PROTOCOL. THE REFERENCE IMPLEMENTATION OF THE CELESTIA PROTOCOL SHOULD BE CONSULTED AS THE COMPLETE SPECIFICATION.**

1. <https://github.com/celestiaorg/celestia-core>: Tendermint Core full node
1. <https://github.com/celestiaorg/celestia-node>: Celestia-specific logic, attached to Celestia Core node
1. <https://github.com/celestiaorg/lazyledger-app>: Celestia state machine (staking and fee payments) logic

The following core ideas in this repository inform the implementation:

1. [Erasure coding](./src/specs/data_structures.md#erasure-coding)
2. [Namespace IDs](./src/specs/consensus.md#reserved-namespace-ids)
3. [PayForMessage transaction](./src/specs/data_structures.md#signedtransactiondatapayformessage)
4. [State transition](./src/specs/consensus.md#blockavailabledatatransactiondata)

## Building From Source

To build book:

```sh
mdbook build
```

To serve locally:

```sh
mdbook serve
```

## Contributing

Markdown files must conform to [GitHub Flavored Markdown](https://github.github.com/gfm/). Markdown must be formatted with:

- [markdownlint](https://github.com/DavidAnson/markdownlint)
- [Markdown Table Prettifier](https://github.com/darkriszty/MarkdownTablePrettify-VSCodeExt)
