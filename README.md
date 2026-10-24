# Alipay Fund Monitor

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md) · [日本語](README.ja.md)

**Android fund-position monitoring and strategy automation with [AutoX.js](https://github.com/aiselp/AutoX).**

> **Real-money capability:** live mode can buy and sell funds in Alipay and cause real charges. The dedicated CNY 1 test is also a real purchase. Start with the default dry-run mode and inspect the plan before enabling any live action.

This personal-use project reads holdings through Android accessibility, displays fund cards and runs configured strategies. It is not investment advice. Automation may conflict with platform terms or break after UI changes; the MIT license provides no warranty against financial loss.

## Features

- Collect fund names, holding amounts, daily/holding returns and rates from Alipay's fund-holdings screen; reconcile collected totals with the page.
- Phone UI with search/sort, red-for-gain/green-for-loss cards and floating collection/strategy progress.
- Per-run history (the latest 100 runs), order details, pause after the current operation and resume from persisted batch progress.
- Buy/sell flows with amount checks, large-amount confirmation, local audit records and AES-GCM payment-password storage backed by Android Keystore.
- Four configurable strategy templates, executed as a sell phase followed by a buy phase.

### Strategy semantics

| Template | Planning behavior |
| --- | --- |
| `base` | If holding value is below the configured target, propose the configured buy amount |
| `dca` | Propose a buy for each fund in enabled groups, using the highest matching group amount |
| `costReduce` | For a loss, use the first matching enabled loss tier or the enabled fallback |
| `takeProfit` | Use the highest matching enabled profit tier and sell its configured share fraction |

For buys, `base` and `dca` compete by amount, with `base` winning a tie. A matching `costReduce` amount is **added** to that winner and combined into one buy for the fund. The loss-reduction component is not discarded by a universal “take the largest amount” rule.

## Execution safeguards and limits

- `dryRun` defaults to `true`; the dry-run flow stops before password submission. Live operation requires a configured payment password.
- Guards validate amount type, decimal precision, minimum/maximum and confirmation threshold.
- A failed fund operation is handled separately; failures and navigation recovery are recorded.
- Persisted batch state tracks completed and uncertain operations. Interrupted in-flight operations are marked for reconciliation and skipped on resume to avoid blindly retrying a purchase.
- The dedicated real-test path fixes the requested amount at CNY 1. Eligibility, platform rejection and the actual result still depend on Alipay; it is not a no-risk sandbox.
- Audit file: `/sdcard/Download/alipay_trade.log`. Treat logs, holdings and payment settings as private data.

## Run on Android

Requires **AutoX.js v6**, accessibility permission and floating-window permission. Existing project notes describe a Redmi HyperOS / Android 15 environment; this is not a claim of universal device compatibility.

1. Install AutoX.js v6 and grant the required permissions on your own phone.
2. Use the Auto.js–Autox.js VSCode extension to connect over Wi-Fi, open `main.js` and run it with F5.
3. Use **↻** to collect, **⚙** to review settings and strategies, and **▶** to open the strategy run flow.
4. Keep dry-run enabled while validating navigation, collection and the proposed plan.

The application's interface remains Chinese. These README translations do not change application behavior or UI language.

## Packaging

`project.json` declares `main.js` as the entry point and `res/logo.png` as the icon. The historical workflow uses a locally prepared `release/` directory, which is ignored by Git and is not supplied in this checkout. Prepare the project files/resources locally, copy them to the phone, and choose **UI application** in AutoX.js's packaging menu. Use `adb install` for a generated APK. See the [original navigation and payment-flow notes](verified-flows/README.md) before changing device flows.

## Development and source map

| Path | Purpose |
| --- | --- |
| `main.js` | Self-contained Android UI, collection, trade and strategy flows |
| `src/` | Extracted logic, configuration and diagnostics; corresponding logic is inlined in `main.js` |
| `test/` | Node-based pure-logic tests |
| `verified-flows/` | Recorded navigation/payment-flow scripts and explanations |
| `monitor/` | ADB collection and an HTML monitoring panel |
| `probe*.js`, `test_pwd_flow.js` | Device diagnostics; inspect before running because some reach payment screens |

Available pure-logic checks:

```bash
node test/strategy-logic.test.js
node test/trade-logic.test.js
node test/apply-query-sort.test.js
node test/run-history.test.js
```

These do not verify the current Alipay UI or a live transaction. Documentation maintenance does not run payment flows.

## License

[MIT](LICENSE). Copyright and attribution remain as recorded in the license.
