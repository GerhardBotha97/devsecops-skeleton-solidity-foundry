## DevSecOps Skeleton – Solidity + Foundry

This repository is a minimal Solidity project wired with a DevSecOps-focused GitHub Actions pipeline.  
It shows how to combine Foundry with multiple security scanners and keep security checks visible but non-blocking.

Key components:

- **Foundry** (forge/cast/anvil) for build, test, coverage, and formatting.
- **Slither** for static analysis of Solidity contracts.
- **Aderyn** for advanced security analysis and markdown reports.
- **OpenGrep** with Trail of Bits rules for SAST over `.sol` files.

---

## CI / DevSecOps Pipeline

The workflow is defined in `.github/workflows/devsecops.yml` and runs on:

- `push` to `main` / `master`
- `pull_request` targeting `main` / `master`
- Manual trigger via **Run workflow** (workflow_dispatch)

### 1. `foundry-ci` – Build, Test, Coverage, Formatting

Steps:

- Install Foundry (stable toolchain).
- `forge fmt --check` – formatting check (non-blocking via `continue-on-error`).
- `forge build` – compile contracts.
- `forge test -vvv` – run unit tests with verbose output.
- `forge coverage` – generate coverage information.

This is the core quality gate: if build/tests fail, the pipeline fails.

### 2. `slither-analysis` – Static Analysis (Advisory)

Steps:

- Install Foundry and Slither.
- If `src/*.sol` exists, run:

  ```bash
  slither ./src --solc-remaps "$(forge remappings 2>/dev/null | tr '\n' ',')"
  ```

- Step uses `continue-on-error: true`, so:
  - All findings (including version warnings) appear in the logs.
  - The overall job and pipeline remain green.

Use this job to monitor common Solidity anti-patterns without blocking merges.

### 3. `aderyn-analysis` – Aderyn Security Report (Advisory)

Steps:

- Install Foundry and build contracts with `forge build`.
- Install Rust toolchain and `cargo install aderyn`.
- Run `aderyn .` (if `src/*.sol` exists).
- If `report.md` is produced, it is printed into the GitHub Actions log.
- The step always exits `0`, even if Aderyn returns a non-zero code or panics.

This gives you a readable Aderyn report on every run, without failing CI.

### 4. `opengrep-analysis` – OpenGrep + Trail of Bits (Advisory)

Steps:

- Install OpenGrep via the official script:

  ```bash
  curl -fsSL https://raw.githubusercontent.com/opengrep/opengrep/main/install.sh | bash
  ```

- Add OpenGrep to `PATH` and, if `src/*.sol` exists, run:

  ```bash
  opengrep scan --config "p/trailofbits" src
  ```

- The step prints the exit code but always exits `0`.

This integrates the community-maintained Trail of Bits ruleset as SAST for Solidity, while staying non-blocking.

---

## Local Development

You can work with the project locally using Foundry.

### Prerequisites

- Rust toolchain (for Foundry installer).
- Foundry installed (`foundryup`) – see: <https://book.getfoundry.sh/>

### Common Commands

- Build:

  ```bash
  forge build
  ```

- Test:

  ```bash
  forge test
  ```

- Format:

  ```bash
  forge fmt
  ```

- Gas snapshots:

  ```bash
  forge snapshot
  ```

- Local node:

  ```bash
  anvil
  ```

- Example deploy script:

  ```bash
  forge script script/Counter.s.sol:CounterScript --rpc-url <your_rpc_url> --private-key <your_private_key>
  ```

---

## How to Evolve This Skeleton

Some ideas to extend this DevSecOps setup:

- Make specific Slither/Aderyn/OpenGrep findings blocking (e.g., by severity).
- Export reports as SARIF and upload them as GitHub code scanning alerts.
- Add secret scanning (e.g., Gitleaks) and dependency scanning.
- Add environment- or branch-specific policies (e.g., stricter checks on `main`).

This repository is meant as a starting point: you can tighten or relax the gates as needed for your team’s risk tolerance.
