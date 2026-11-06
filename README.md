# Connect Incentiv to kwant

kwant does not need a chain adapter to reach Incentiv. Incentiv ships an official MCP server that runs on your machine, and kwant spaces already take MCP servers over stdio. Connect the two and your kwant bots can read the Incentiv network, quote swaps and move funds, while kwant keeps asking the person before every payment and the Incentiv wallet enforces the spending policy you set during setup.

## What this connection gives you

Once the server is registered in a space and assigned to bots, your kwant bots can call Incentiv's tools through kwant: network and wallet reads, swap quotes, and transfers inside the policy you configured. Nothing here turns kwant into an autonomous spender. Both products are built against that.

| | |
|---|---|
| Chain | Incentiv, EVM L1, chain ID 24101 |
| Server | incentiv-mcp, official, local stdio |
| Client | kwant, self hosted, stdio MCP in spaces |
| Payments | always behind a kwant approval card |

## What you need

- kwant installed and running, on a Mac, on Linux or on a server
- Node.js 22.18 or newer with npm, on the same machine kwant runs on
- Assets to fund the agent wallet: gas can be paid in whatever asset the wallet holds, USDC, WETH, CENT and others, through the unified token paymaster, so holding stable is enough
- About five minutes

## The setup, five steps

### 1. Install the server

On the machine kwant runs on:

```
npm install --global --ignore-scripts incentiv-mcp
```

### 2. Run incentiv-mcp setup

```
incentiv-mcp setup
```

It sets spending permissions, asks you to save two passwords, one for administration and one for backup recovery, each at least 12 bytes, and verifies your backup. The wallet stays disabled until you review the permissions and type ENABLE. Setup never sends funds. Keys stay on your computer and never enter MCP inputs or outputs.

### 3. Register it in kwant

Open kwant's tool servers screen, Give your bots new abilities, and add a stdio server. If setup could not update another app automatically, it prints the exact configuration to copy: paste that command into kwant, it points at the absolute Node and package paths. From a source checkout instead of the npm install, the command is `node --use-system-ca dist/index.js`.

Click Connect: kwant starts the server, lists its tools and shuts it down again, so you confirm it works before any bot touches it.

### 4. Assign it to the right bots

Under Tools this bot can use, pick which bots may call it. A bot only ever sees the servers assigned to it, never your own MCP configuration.

### 5. Run the first test

In the space, ask:

> Show the Incentiv network status, then my wallet balance.

Expected: the network is Incentiv Mainnet and the wallet address matches the one setup showed. Both calls are read only, so they run without a card under kwant's default approval mode, and every call shows on the bot's reply.

## Two addresses, one agent

Your agent wallet carries two addresses. The owner EOA is the signing address behind the smart account: plain EVM data shows up there, balances and plain transaction history, and it is the address for plain interactions outside the smart account flow. The smart account is the ERC 4337 wallet the agent works through: transfers, swaps and bundled operations run there, gas settling in whatever asset the wallet holds through the paymaster, signed by the owner key, which never leaves your machine. Ask the bot for its wallet details and it can read both.

## Fund the wallet

The official docs point funding at the smart account address shown by setup, and it can receive before deployment. Confirm you are on Incentiv Mainnet and fund it within the holdings limits you chose. If you fund through the owner EOA first, have the agent move the funds into the smart account before it starts working with them. Keep enough balance for gas in whatever asset pays it. The first transaction also deploys the account. A quote is never permission to spend.

## Two safety nets, stacked

**kwant side.** Every payment asks the person first, in every mode. A payment command in the shell or a checkout in the bot's browser shows an approval card, even behind a plain Continue. MCP tool calls follow the bot's approval mode: tools marked read only run without a card, everything else shows one.

**Incentiv side.** Local key custody: keys never leave your computer. Defined permissions: permitted assets, recipients, spending limits and gas allowances. A swap quote never signs anything. To stop new signing at any time:

```
incentiv-mcp wallet disable
```

## The tools your bots get

| Tool | What it does | kwant card |
|---|---|---|
| get_network_status | chain status | none, read only |
| get_agent_wallet, get_portfolio | local wallet and balances | none, read only |
| get_swap_quote | a swap quote, signs nothing | none, read only |
| get_transactions, get_account_activity | history | none, read only |
| send_transfer, send_swap | moves funds inside your policy | approval card |
| build_transfer, build_swap | an unsigned operation for your own signer | no key moves |
| get_bridge_routes | bridge routes | none, read only |

## If something does not connect

- Connect fails: kwant shows what the server printed. Check the Node version first, then rerun `incentiv-mcp setup`.
- Setup was interrupted: rerun `incentiv-mcp setup` to finish it or reconnect an existing wallet.
- You want a second wallet: `incentiv-mcp --wallet-profile second` creates a separate one.
- You want everything off: `incentiv-mcp wallet disable` stops new signing without reversing what already went out.

## Credit

Written by Adeniyi (0xleetv), Incentiv community moderator, after the discussion in [alexsergivan/kwant discussion #328](https://github.com/alexsergivan/kwant/discussions/328). Sources: [kwant MCP docs](https://kwant.si/docs/mcp), [incentiv-mcp](https://github.com/IncentivNetwork/incentiv-mcp), [incentiv.io/mcp](https://incentiv.io/mcp).
