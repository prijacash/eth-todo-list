# Ethereum Todo List

A 2023 learning project: a Solidity todo-list contract, a Truffle migration onto local Ganache, and a static page that starts a MetaMask connection.

Prija Cash. Commits run from 14 January 2023 through 5 February 2023.

## What it does

`TodoList` stores tasks on a local Ethereum chain. Each task has an id, a text field, and a completed flag. The constructor stores one task, `"Check out dappuniversity.com"`. `createTask` appends another task with `completed` set to `false`. No function in the contract changes that flag.

The page in `src/` is a Bootstrap layout: a navbar slot for an account, a “Loading…” message, a form, an open-task list, and a completed-task list. On load, `src/app.js` only tries to connect a wallet:

- With no injected `web3` object, it alerts `Please connect to Metamask.`
- With `window.ethereum`, it builds a `Web3` instance, requests accounts with `ethereum.enable()`, then calls `web3.eth.sendTransaction` with an empty transaction object. That call sits inside the MetaMask connection snippet and runs during page load.
- It saves the first account on `App.account` (`web3.eth.accounts[0]`, the web3.js 0.x API). The navbar element `#account` is never filled in.

The script stops there. It does not read `build/contracts/TodoList.json`, and `App.createTask` is undefined even though the form submits with `App.createTask()`. `#content` is `display: none`, and nothing hides the loader, so the task list stays on “Loading…”. `index.html` also has no web3.js script tag. `new Web3(...)` runs only when the page already has a `Web3` global. MetaMask today injects `window.ethereum` and does not provide that global, so the connection code throws before an account is stored.

`contracts/` also holds short language exercises (values, arrays, mappings, structs, constants, ether units, gas, and a get/set counter). `migrations/2_deploy_contracts.js` deploys `TodoList` only. Those exercises stay on disk.

## Smart contract

`contracts/TodoList.sol`, pragma `solidity^0.5.0`. The committed artifact was compiled with solc `0.5.0+commit.1d4f565a`.

| Name | Kind | Behavior |
| --- | --- | --- |
| `taskCount` | public `uint` | Starts at `0`. `createTask` increments it. Solidity generates the getter. |
| `Task` | struct | `id` (`uint`), `content` (`string`), `completed` (`bool`). |
| `tasks` | public `mapping(uint => Task)` | Getter returns `id`, `content`, and `completed` for a task id. |
| constructor | | Calls `createTask("Check out dappuniversity.com")`, so task `1` exists after deploy. |
| `createTask(string _content)` | public | Increments `taskCount` and stores `Task(taskCount, _content, false)`. |

There is no event, no access-control modifier, and no check for an empty string. Any account can call `createTask`.

`migrations/1_initial_migration.js` deploys Truffle’s `Migrations` contract (`owner`, `last_completed_migration`, and `setCompleted`, which only `owner` may call). `migrations/2_deploy_contracts.js` deploys `TodoList`.

`build/contracts/TodoList.json` records one previous deploy on network id `5777`. A fresh Ganache chain gets a new address when you migrate again.

### Practice contracts

These files sit beside the app. `migrations/2_deploy_contracts.js` deploys `TodoList` only. `Array`, `Mapping`, `Structs`, `Constants`, `EtherUnits`, `Gas`, and `Primitives` match the public [Solidity by Example](https://solidity-by-example.org/) lessons from this period: same contract names, comments, and sample values, with the `^0.8.3` / `^0.8.17` pragmas those lessons used then. `SimpleContract.sol` is a separate get/set counter named `ReadWrite` (`pragma solidity ^0.8.4`).

The committed artifacts were compiled with solc 0.5.0 and contain only `Migrations` and `TodoList`. The `^0.8` files were added after that compile. Truffle compiles every `.sol` file in `contracts/`, and solc 0.5.0 stops on a `^0.8` pragma, so a fresh `truffle migrate` fails until those practice files are outside `contracts/`.

| File | Contract | What the code shows |
| --- | --- | --- |
| `Primitives.sol` | `Primitives` | `bool`, `uint` / `int` widths, `address`, `bytes1`, and default values. The address is the lesson sample `0xCA35b7d915458EF540aDe6068dFe2F44E8fa733c`. |
| `Arrays.sol` | `Array` | A dynamic `uint[]` and a fixed `uint[3]`: push, read, update, delete, pop, length, an in-memory array, and returning the dynamic array. |
| `Mapping.sol` | `Mapping` | `mapping(address => uint)` and a nested `mapping(address => mapping(address => bool))`: set, read, add, and delete. |
| `Structs.sol` | `Structs` | A `Car` struct (`model`, `year`, `owner`), an array of cars, a mapping from owner to cars, and a storage update versus a memory value. |
| `Constants.sol` | `Constants` | Constant `MY_ADDRESS` and `MY_UINT` (`123`). The address is the lesson sample `0x777788889999AaAAbBbbCcccddDdeeeEfFFfCcCc`. |
| `Ether.sol` | `EtherUnits` | `1 wei == 1` and `1 ether == 1e18`. |
| `Gas.sol` | `Gas` | `forever()` increments a counter in `while (true)` until the call runs out of gas. The comment in the file notes that the state change is discarded and the gas spent is not refunded. |
| `SimpleContract.sol` | `ReadWrite` | `get` and `set` for a single `uint`. |

The two sample addresses are public constants from those lessons. This repo does not include a private key or a mnemonic.

## Tech stack

- **Solidity** `^0.5.0` for `TodoList`. The lockfile resolves **solc** `0.5.0`. The practice contracts use `^0.8.x`.
- **Truffle** `5.0.2`, with the optimizer enabled (`runs: 200`) in `truffle-config.js`.
- **Ganache** at `127.0.0.1:7545`, the host and port in `truffle-config.js` (the Ganache GUI port this tutorial used). Ganache is an external app, not an npm dependency. `network_id` is `"*"`, so Truffle accepts whatever id the node reports. The committed artifact was deployed to network id `5777`.
- **Node.js** and npm. `npm run dev` starts **lite-server** (dependency `^2.3.0`, lockfile `2.6.1`) from `bs-config.json`.
- **Page:** Bootstrap `4.1.3`, jQuery `1.12.4` from the Google CDN, and `truffle-contract` `3.0.6`. The HTML loads `truffle-contract`; `app.js` never calls it.
- **Declared and unused:** `chai`, `chai-as-promised`, `chai-bignumber`, and `nodemon`. There is no `test/` directory. `npm test` is the stub `echo "Error: no test specified" && sexit 1`, so the command fails looking for `sexit`.

`package.json` still has the tutorial scaffold’s `"author": "gregory@dappuniversity.com"` and `"description": "Blockchain Todo List Powered By Ethereum"`. The license field is `ISC`.

## Run it locally

The contract is what you can use end to end. The dev server will start the page, and the page will stop at the wallet script.

1. Use Node.js and npm. Truffle is pinned to `5.0.2` (2019). On a current Node release, `npm install` can fail while building old native dependencies. The committed artifact was produced with that pin in early 2023.
2. Install from the repo root:

   ```bash
   npm install
   ```

3. Start Ganache on `127.0.0.1:7545`. If Ganache CLI binds to `8545`, start it on port `7545` so it matches `truffle-config.js`.
4. Deploy `Migrations` and `TodoList`. Truffle compiles every file in `contracts/` with solc 0.5.0, and the practice contracts are Solidity 0.8, so move those eight files aside for the compile. Leave `TodoList.sol` and `Migrations.sol` where they are:

   ```bash
   mkdir -p /tmp/eth-todo-practice
   mv contracts/Arrays.sol contracts/Constants.sol contracts/Ether.sol \
     contracts/Gas.sol contracts/Mapping.sol contracts/Primitives.sol \
     contracts/SimpleContract.sol contracts/Structs.sol /tmp/eth-todo-practice
   npx truffle migrate --network development
   ```

   Move the practice files back afterward if you want the working tree unchanged. Nothing in the contracts themselves needs an edit.

5. Serve the page. lite-server’s default port is `3000`:

   ```bash
   npm run dev
   ```

   `bs-config.json` serves `src/` and `build/contracts/`, and maps `/vendor` to `node_modules`, which is how the Bootstrap and truffle-contract tags resolve.

6. Open the site. With MetaMask pointed at `http://127.0.0.1:7545`, expect the MetaMask alert and a `Web3 is not defined` error. The task form stays hidden.

### Read and write tasks from the Truffle console

After migrate, with Ganache still running:

```bash
npx truffle console --network development
```

```javascript
const list = await TodoList.deployed()
const count = await list.taskCount()
const first = await list.tasks(1)
await list.createTask("Buy milk")
```

`tasks(1)` is the seed task. The new task is stored at the next id with `completed` false. `count` above is the value from before `createTask`.

## Built while learning

I built this while learning Solidity and Ethereum, by following a tutorial.

The todo app follows Dapp University’s [Build Your First Blockchain App](https://www.youtube.com/watch?v=rzvk2kdjr2I). The first README in this repo linked that video, the seed task names the tutorial site, and `package.json` still lists the tutorial author. I got as far as a contract that stores tasks and a page that starts a wallet connection. Wiring the page to the contract was still ahead of me.

The practice contracts came in the following commits, from the Solidity by Example lessons linked above, plus the small `ReadWrite` contract.

## What I learned

- How a Truffle project is laid out: contracts, numbered migrations, and a JSON artifact that holds the ABI, bytecode, and the address from a local deploy.
- A first contract that stores data: a struct, a mapping, a counter, a constructor that writes the first row, and a public function that appends storage.
- Language basics, each in its own file: integer widths, arrays, mappings, structs, constants, `wei` and `ether`, and what an unbounded loop does to gas. In `Gas.sol`, `forever()` runs until the transaction fails and the `i` update is discarded.
- The start of a browser dapp: detect an injected provider and request the account. I also left the MetaMask sample’s empty `sendTransaction` on the load path, and I used `ethereum.enable()` with `web3.eth.accounts[0]`, which is the older connection style.
- The HTML for listing and completing tasks landed before either the contract or `app.js` could support it. The checkbox and the completed list are markup only.

## What I would improve

- Load `TodoList.json` on the page, read tasks `1` through `taskCount`, and define `App.createTask` so the form writes to the contract. Hide `#loader` once that load finishes, and show `#content`.
- Add a function that sets `Task.completed`, then connect the checkbox markup to it. The flag is stored today and stays `false`.
- Remove the empty `web3.eth.sendTransaction` call from page load. Ship a web3 build with the page, or call `window.ethereum.request({ method: "eth_requestAccounts" })`, and write the account into `#account`.
- Reject an empty task string in `createTask`.
- Add tests for `taskCount` and `createTask`. The chai packages are already dependencies. Fix the `npm test` stub (`sexit` needs to be `exit`).
- Keep one compiler story. Either leave the `^0.8` exercises outside the Truffle 0.5 compile, or move `TodoList` forward and set the compiler version in `truffle-config.js`.
- Keep local Ganache artifacts and `.DS_Store` out of git. `.gitignore` currently lists `node_modules/` only. Put my own name on the `package.json` author field.

## Project layout

```
contracts/          TodoList, Migrations, and the practice contracts
migrations/         Deploys Migrations, then TodoList
build/contracts/    Committed Truffle artifacts for those two contracts
src/index.html      Bootstrap page
src/app.js          Wallet-connection script
truffle-config.js   Ganache at 127.0.0.1:7545
bs-config.json      lite-server paths
```
