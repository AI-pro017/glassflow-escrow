# GlassFlow Escrow

A CosmWasm escrow contract that can hold many escrows at once, each with its own ID, arbiter and recipient. It accepts both native tokens and CW20 tokens.

Someone creates an escrow and funds it. A trusted arbiter then decides when to release the funds to the recipient. Escrows can have a deadline, after which they can no longer be released.

## Messages

`create` opens a new escrow and funds it with the native tokens sent along:

```json
{
  "create": {
    "id": "deal-42",
    "arbiter": "juno1arbiter...",
    "recipient": "juno1seller...",
    "end_time": 1767225600,
    "cw20_whitelist": ["juno1token..."]
  }
}
```

`end_height` or `end_time` set the deadline (both are optional). `cw20_whitelist` lists which CW20 tokens the escrow will accept.

To fund an escrow with CW20 tokens, send them to this contract from the token contract with a `send` message whose `msg` is either `{"create": {...}}` or `{"top_up": {"id": "deal-42"}}`.

Other messages:

- `top_up { id }` adds more native tokens to an existing escrow.
- `approve { id }` lets the arbiter release everything to the recipient. It fails once the escrow has expired.
- `refund { id }` closes the escrow. Only the arbiter can call it.

Query:

- `details { id }` returns the arbiter, recipient, funder, deadline, native and CW20 balances, and the token whitelist.

## Known issues

- `refund` currently sends the funds to the recipient instead of back to the funder, and anyone other than the arbiter is rejected even after the deadline. That's the opposite of what a refund should do, so it needs fixing before this holds real funds.
- There's no query for listing all escrows yet.

## Building

You'll need Rust with the `wasm32-unknown-unknown` target.

```bash
git clone https://github.com/AI-pro017/glassflow-escrow.git
cd glassflow-escrow
cargo test
cargo wasm
```

`cargo schema` regenerates the JSON schemas in `schema/`. There are more notes on building and deploying in `Developing.md` and `Publishing.md`.

## Credits

Based on the escrow example from [cosmwasm-examples](https://github.com/CosmWasm/cosmwasm-examples) by Ethan Frey. Apache 2.0, see [LICENSE](LICENSE) and [NOTICE](NOTICE).
