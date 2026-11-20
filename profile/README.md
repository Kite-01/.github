<a href="https://usekite.site"><img src="./header.webp" alt="Kite: agents pay too. Every payment link is an x402 resource." width="100%" /></a>

<div align="center">

### Open-source crypto payments that people and AI agents can both pay.

[![Website](https://img.shields.io/badge/usekite.site-000000?style=for-the-badge)](https://usekite.site)
[![Start on testnet](https://img.shields.io/badge/Start_on_testnet-FBC400?style=for-the-badge&labelColor=000000)](https://testnet.usekite.site)
[![x402](https://img.shields.io/badge/x402-ready-000000?style=for-the-badge&labelColor=FBC400)](https://www.x402.org)
[![Sponsor](https://img.shields.io/badge/Sponsor-Kite-000000?style=for-the-badge&logo=githubsponsors&logoColor=EA4AAA)](https://usekite.site/sponsor)

</div>

Kite is a crypto payment processor a business runs itself. The business creates a payment link, the customer pays it in any token on any of the business's chains, a Kite contract settles the payment on-chain, and a signed webhook tells the business's systems.

- **A person** opens the link and sees a normal checkout: connect a wallet, or send tokens to an address shown as a QR code.
- **An AI agent** requests the same link and gets an HTTP `402 Payment Required` [x402](https://www.x402.org) resource. It pays with a signed HTTP request: no browser, and no gas, because Kite is its own facilitator.
- **Any token in, one token out.** Payments are swapped on-chain, and with [NEAR Intents](https://docs.near-intents.org/) every payment can be delivered as one token at one address, whichever chain the customer paid on.

<details>
<summary><b>🤖 Pay a sample link with your agent</b></summary>

<br />

```sh
LINK=https://testnet.usekite.site/pay/cc4eaa8e-2673-4b5c-af09-c151b1736ffd

# 1. ask for the link: Kite answers 402 with an x402 offer per network
curl -i -H "Accept: application/json" $LINK

# 2. the agent signs one offer and asks again
curl -i $LINK -H "X-PAYMENT: <signed payload>"
# HTTP/1.1 200 OK
```

| Network | Price | Link                                                                    |
| ------- | ----- | ----------------------------------------------------------------------- |
| Testnet | 1 USD | <https://testnet.usekite.site/pay/cc4eaa8e-2673-4b5c-af09-c151b1736ffd> |
| Testnet | 1 GBP | <https://testnet.usekite.site/pay/6c103330-4adc-43a1-9335-084027a3f284> |
| Mainnet | 1 USD | <https://mainnet.usekite.site/pay/a1b264be-d168-42a9-9a2e-d2b0b0aa0467> |

</details>

## One contract per payment

Each payment gets its own contract, at an address known before anyone pays. Kite never holds the money: the contract receives the customer's tokens, then settles or refunds them on-chain.

```mermaid
sequenceDiagram
    autonumber
    participant C as Customer or agent
    participant K as Kite server
    participant F as KiteFactory
    participant P as Payment contract
    participant M as Merchant

    K->>F: predictAddress(transfer)
    F-->>K: deposit address (nothing deployed yet)
    K-->>C: payment link / 402 offer
    C->>P: pay any token to the address
    K->>F: settle(transfer)
    F->>P: deploy clone and settle in one transaction
    P->>P: swap if needed
    P->>M: pay fees, send the rest
    P-->>C: return any overpayment
    Note over C,P: can't settle? refund(transfer). After expiry anyone can call it.
```

<details>
<summary><b>📜 How the contracts keep funds safe</b></summary>

<br />

- **Predict.** The factory derives the deposit address from the payment's terms: merchant, token, minimum, fees, refund address and expiry.
- **Settle.** The operator deploys the contract and settles in one transaction: swap through Uniswap V3 or V2 (Rhea on NEAR), pay each fee, send the rest to the merchant and return any overpayment.
- **Refund.** Before expiry only the operator can refund. After expiry _anyone_ can, so a customer is never stuck waiting on a server.
- **Recover.** Tokens sent after settlement, or a refund the refund address couldn't accept, stay recoverable by the operator.
- **Upgrades can't touch live payments.** Each payment is bound to the contract code it was created with, so no upgrade can change the code of a payment that already holds funds.

</details>

## Repositories

| Repository                                                  | What it is                                                                                                                                                 | Stack                                    |
| ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| [server](https://github.com/Kite-01/server)                 | The API, the merchant dashboard, the checkout page and the x402 facilitator                                                                                | TypeScript, Bun, Elysia, React, Postgres |
| [contracts](https://github.com/Kite-01/contracts)           | `KiteFactory` and `Kite` for EVM chains: predictable deposit addresses, Uniswap V2/V3 swaps, fees, refunds and recovery. Published as `@kite-01/contracts` | Solidity, Hardhat                        |
| [near-contracts](https://github.com/Kite-01/near-contracts) | The same model on NEAR: a deposit account per payment, swaps through Rhea Finance                                                                          | Rust, near-sdk                           |
| [monorepo](https://github.com/Kite-01/monorepo)             | [`@kite-01/sdk`](https://github.com/Kite-01/monorepo/tree/main/packages/sdk): create payment links, sign API requests and verify webhooks                  | TypeScript                               |

## Supported networks

<p align="center">
  <a href="https://ethereum.org"><img src="https://cdn.jsdelivr.net/gh/trustwallet/assets@master/blockchains/ethereum/info/logo.png" alt="Ethereum" title="Ethereum" width="44" height="44" /></a>
  <a href="https://optimism.io"><img src="https://cdn.jsdelivr.net/gh/trustwallet/assets@master/blockchains/optimism/info/logo.png" alt="OP Mainnet" title="OP Mainnet" width="44" height="44" /></a>
  <a href="https://www.bnbchain.org"><img src="https://cdn.jsdelivr.net/gh/trustwallet/assets@master/blockchains/binance/info/logo.png" alt="BNB Smart Chain" title="BNB Smart Chain" width="44" height="44" /></a>
  <a href="https://polygon.technology"><img src="https://cdn.jsdelivr.net/gh/trustwallet/assets@master/blockchains/polygon/info/logo.png" alt="Polygon" title="Polygon" width="44" height="44" /></a>
  <a href="https://base.org"><img src="https://cdn.jsdelivr.net/gh/trustwallet/assets@master/blockchains/base/info/logo.png" alt="Base" title="Base" width="44" height="44" /></a>
  <a href="https://arbitrum.io"><img src="https://cdn.jsdelivr.net/gh/trustwallet/assets@master/blockchains/arbitrum/info/logo.png" alt="Arbitrum One" title="Arbitrum One" width="44" height="44" /></a>
  <a href="https://www.avax.network"><img src="https://cdn.jsdelivr.net/gh/trustwallet/assets@master/blockchains/avalanchec/info/logo.png" alt="Avalanche" title="Avalanche" width="44" height="44" /></a>
  <a href="https://www.monad.xyz"><img src="https://cdn.jsdelivr.net/gh/trustwallet/assets@master/blockchains/monad/info/logo.png" alt="Monad" title="Monad" width="44" height="44" /></a>
  <br />
  <a href="https://www.unichain.org"><img src="https://icons.llamao.fi/icons/chains/rsz_unichain.jpg" alt="Unichain" title="Unichain" width="44" height="44" /></a>
  <a href="https://zora.co"><img src="https://icons.llamao.fi/icons/chains/rsz_zora.jpg" alt="Zora" title="Zora" width="44" height="44" /></a>
  <a href="https://robinhood.com"><img src="https://cdn.jsdelivr.net/gh/trustwallet/assets@master/blockchains/robinhoodchain/info/logo.png" alt="Robinhood Chain" title="Robinhood Chain" width="44" height="44" /></a>
  <a href="https://www.arc.network"><img src="https://cdn.jsdelivr.net/gh/trustwallet/assets@master/blockchains/arc/info/logo.png" alt="Arc" title="Arc" width="44" height="44" /></a>
  <a href="https://aurora.dev"><img src="https://cdn.jsdelivr.net/gh/trustwallet/assets@master/blockchains/aurora/info/logo.png" alt="Aurora" title="Aurora" width="44" height="44" /></a>
  <a href="https://qday.io"><img src="https://qday.io/apple-icon.png" alt="QDay Aevum" title="QDay Aevum (beta)" width="44" height="44" /></a>
  <a href="https://near.org"><img src="https://cdn.jsdelivr.net/gh/trustwallet/assets@master/blockchains/near/info/logo.png" alt="NEAR" title="NEAR (beta, testnet)" width="44" height="44" /></a>
</p>

<details>
<summary><b>⛓️ Chains and testnets</b></summary>

<br />

| Chain             | Mainnet |      Testnet       |
| ----------------- | :-----: | :----------------: |
| Ethereum          |   ✅    |      Sepolia       |
| OP Mainnet        |   ✅    |     OP Sepolia     |
| BNB Smart Chain   |   ✅    |    BNB Testnet     |
| Polygon           |   ✅    |        Amoy        |
| Base              |   ✅    |    Base Sepolia    |
| Arbitrum One      |   ✅    |  Arbitrum Sepolia  |
| Avalanche         |   ✅    |        Fuji        |
| Monad             |   ✅    |   Monad Testnet    |
| Unichain          |   ✅    |  Unichain Sepolia  |
| Zora              |   ✅    |    Zora Sepolia    |
| Robinhood Chain   |   ✅    | Robinhood Testnet  |
| Arc               |   ✅    |    Arc Testnet     |
| Aurora            |   ✅    |   Aurora Testnet   |
| QDay Aevum (beta) |   ✅    | QDay Aevum Testnet |
| NEAR (beta)       |    —    |    NEAR Testnet    |

What each provider (deposit notifiers, gas sponsors, swaps, NEAR Intents) supports on each chain is in [`registry.ts`](https://github.com/Kite-01/server/blob/main/src/shared/chains/registry.ts).

</details>

## Get started

- **Try it:** open a payment link above, or point your x402 agent at one.
- **Run your own:** follow [Getting started](https://github.com/Kite-01/server#getting-started), then [deploy your own factory](https://github.com/Kite-01/contracts#deploying) on each chain you'll accept.
- **Integrate:** create payment links from your backend with `@kite-01/sdk` and fulfil orders from signed webhooks.

## Security

Kite's contracts hold and move funds. If you find a vulnerability, please don't open a public issue: email [security@usekite.site](mailto:security@usekite.site) instead.

## License

The contracts and SDK are MIT. The server is AGPL-3.0.
