# AGV Contracts — Build-Ready Checklists (.md)

## GVT.sol — Governance/Utility Token

- [ ] **ERC Standards**
  - [ ] ERC-20 compliant
  - [ ] EIP-2612 Permit (gasless approvals)
  - [ ] (Optional) ERC20Votes (snapshot-based voting)
  - [ ] (Optional) ERC20Burnable

- [ ] **Supply & Minting**
  - [ ] Immutable **max supply = 1,000,000,000 GVT**
  - [ ] No public mint; only role-gated mints
  - [ ] Roles:
    - [ ] `ONCHAIN_VERIFIER_MINTER` (emissions)
    - [ ] `AIRDROP_VAULT_MINTER` (vesting/airdrop)
    - [ ] `DAO_TREASURY_MINTER` (if policy requires)
  - [ ] All minter roles grantable/revocable by **DAO multisig + timelock**

- [ ] **Burn / Buyback**
  - [ ] `burn` and `burnFrom` enabled
  - [ ] (Optional) `treasuryBurn(address,uint256)` for Buyback/Treasury

- [ ] **Controls & Safety**
  - [ ] `Pausable` (pause transfers/mints)
  - [ ] RBAC: `DEFAULT_ADMIN_ROLE` (DAO timelock), `MINTER_ROLE`, `PAUSER_ROLE`, `BURNER_ROLE`
  - [ ] No owner-only backdoors; parameter changes timelocked
  - [ ] (Optional) Compliance hook toggles (blocklist/KYC), default off

- [ ] **Cross-Chain**
  - [ ] Canonical supply accounting if bridged
  - [ ] Split L1/L2 minter roles to keep **total ≤ cap**

- [ ] **Events**
  - [ ] `Minted(source,to,amount,refId)`
  - [ ] `Burned(by,amount)`
  - [ ] Role grant/revoke events surfaced

---

## rGGP.sol — Reward Token

- [ ] **ERC Standards**
  - [ ] ERC-20 compliant
  - [ ] EIP-2612 Permit
  - [ ] (Optional) ERC20Burnable

- [ ] **Issuance / Emissions**
  - [ ] Only **EmissionManager** (verifier/distributor contract) can mint
  - [ ] DAO-timelocked **emission coefficients** (e.g., per kWh/kg/compute-hour)
  - [ ] (Optional) Global/moving cap for the “Yield Incentives” envelope

- [ ] **Utility / Conversion**
  - [ ] **Converter** (BondingCurve/Converter) may burn rGGP → release/mint GVT
  - [ ] (Optional) Partner dApps allowed to **burn rGGP for discounts**
  - [ ] (Optional) Cooldowns/daily caps on conversion (rate-limit abuse)

- [ ] **Controls & Safety**
  - [ ] `Pausable`
  - [ ] Roles: `EMITTER_ROLE`, `CONVERTER_ROLE`, `PAUSER_ROLE` (DAO + timelock)
  - [ ] (Optional) Non-transferable mode toggle (default: transferable)

- [ ] **Events**
  - [ ] `Emission(source,to,amount,metricType,periodRef)`
  - [ ] `Converted(to,rggpIn,gvtOut,rate,refId)`
  - [ ] `DiscountBurn(user,amount,refId)`

---

## BondingCurve.sol — Pricing / Conversion / Buyback

- [ ] **Core Mechanics**
  - [ ] **Buy** GVT with stablecoin via curve or oracle-guarded price
  - [ ] **Sell** GVT back:
    - [ ] Spread/fee configurable
    - [ ] **Floor price** / minimum redemption enforcement (reserve-backed)
    - [ ] Optional partial **burn** on buyback
  - [x] **rGGP → GVT** conversion:
    - [x] Burns rGGP, releases/mints GVT at DAO-set base rate ± curve
    - [x] Per-epoch **caps/cooldowns**

- [ ] **Pricing & Reserves**
  - [x] Select curve (linear/exponential/logistic) **or** oracle-anchored with guardrails
  - [ ] Maintain stablecoin/GVT reserves with min reserve ratio
  - [ ] Slippage checks; target slippage policy (e.g., ≤ ~1.5% at ref size)

- [x] **Treasury / LP Integration**
  - [x] Route fees to Treasury and/or LP recycling
  - [x] Periodic surplus sweep to Treasury (DAO/timelock)
  - [ ] (Optional) LP top-up hooks

- [ ] **Controls & Safety**
  - [x] `Pausable` (independent switches for buy/sell/convert)
  - [ ] **Circuit breakers** (price band breach, low reserves)
  - [x] **Rate limiters** per tx/epoch for buy, sell, convert
  - [x] **Timelocked** updates for fees, floors, curve params, whitelists
  - [ ] **Whitelist window** for early phases
  - [x] `ReentrancyGuard`; pull-over-push; input validation

- [ ] **Events**
  - [ ] `Bought(buyer,stableIn,gvtOut,price,fee)`
  - [ ] `Sold(seller,gvtIn,stableOut,price,fee,burned)`
  - [ ] `ConvertedRggp(user,rggpIn,gvtOut,rate)`
  - [x] `ParamsUpdated(...)`, `Paused(op)`, `Unpaused(op)`

---

## Cross-Contract & Governance (All)

- [ ] **DAO Multisig + Timelock** controls all sensitive actions/roles
- [ ] **Emergency Pause** available on each module
- [ ] **Power-to-Mint Path**: only Verifier/EmissionManager mints GVT/rGGP
- [ ] **Separation of Concerns**:
  - [ ] Vesting/Airdrop in dedicated vaults
  - [ ] Staking/claiming in staking contracts
  - [ ] Pricing/Conversion strictly in BondingCurve/Converter
- [ ] **Accounting Transparency** with rich, referenceable events (`refId`, metric, period)
- [ ] **Security Baseline**:
  - [ ] Solidity ≥ 0.8 (checked arithmetic)
  - [ ] Reentrancy guards, access control, input validation
  - [ ] No privileged backdoors; upgrade policy clearly defined (non-upgradeable or proxy with guardian + timelock)
- [ ] **Multi-Chain Supply Invariant** enforced if bridged (global cap not exceeded)

--- 

*Paste this file into your repo as `CHECKLISTS.md` and tick items during implementation/audit.*
