# Agentic Bug Hunter — skill khusus berburu bug bounty (web3)

`~/workspace/custom-skills/agentic-bug-hunter/SKILL.md`
Sumber metodologi: repo `~/workspace/agentic-bug-hunter/` (Awarexone Agentic-Bug-Hunter) —
`SKILL.md` (master workflow web2), `skills/web3-audit/SKILL.md` (10 kelas bug smart contract),
`skills/triage-validation/` (7-Question Gate), `skills/report-writing/` (template laporan).
Baca file-file itu sebelum mulai berburu; skill ini ringkasan operasional + scope target aktif.

## Aturan inti (tidak boleh dilanggar)

1. **Baca FULL SCOPE dulu** sebelum menyentuh aset apa pun.
2. **NO THEORETICAL BUGS** — pertanyaan penentu: "Bisakah attacker melakukan ini SEKARANG ke user nyata
   yang tidak melakukan apa-apa yang aneh — dan apakah ada kerugian nyata (uang hilang, PII bocor,
   account takeover, RCE)?" Kalau TIDAK → STOP, jangan tulis, jangan lanjut.
3. **7-Question Gate sebelum menulis laporan apa pun** (lihat bawah). Satu NO = kill.
4. **PoC wajib** — semua temuan harus punya bukti exploit yang bisa direproduksi (HTTP request /
   Foundry test). Tanpa PoC = tidak disubmit.
5. **Satu kelas bug dalam satu waktu** — dalam, jangan menyebar.
6. **Jangan submit apa pun ke program** tanpa keputusan user — hunter hanya menyiapkan laporan.
   Submission butuh identitas/KYC + alamat payout user.
7. **Hyperliquid: DILARANG testing di mainnet** — testnet / local fork saja.

## 7-Question Gate (semua harus YES)

1. Bisa dieksploitasi SEKARANG dengan PoC nyata? 2. Korban user nyata tanpa aksi aneh?
3. Dampak konkret (uang/PII/ATO/RCE)? 4. Masuk scope program? 5. Sudah cek duplikat
   (disclosed reports / changelog / GitHub issues)? 6. Bukan di daftar "always rejected"?
7. Triager yang baca akan bilang "ini bug beneran"?

Always-rejected: missing CSP/HSTS, SPF/DKIM, GraphQL introspection saja, version disclosure
tanpa exploit, clickjacking halaman non-sensitif, self-XSS, open redirect saja, SSRF DNS-only,
host header saja, no rate limit di form non-kritis, dll. Chain dulu baru valid
(open redirect + OAuth code theft = ATO; SSRF + metadata = RCE).

## 10 kelas bug smart contract (web3-audit)

1. **Accounting desync** (28% Critical) — dua state var harus sinkron, satu path update A lupa B
   → phantom value. Cek early return di claim/redeem yang skip update state.
2. **Access control** (19%) — modifier hilang di sibling function (`vote`/`reset` ada guard,
   `poke` tidak). Cek existence vs ownership (`_requireOwned` ≠ own). Modifier `if` tanpa
   `require` = silent pass. Proxy `initialize` tanpa `initializer`.
3. **Incomplete code path** — fungsi A lakukan X, fungsi B (update/cancel) tidak lakukan kebalikan X.
   `deposit` ada validasi, `mint` tidak.
4. **Off-by-one** — tiap `>` tanya "kalau A == B?". Batas period/epoch/deadline, loop `<= length`.
5. **Oracle** — Chainlink tanpa staleness check (`updatedAt`), Pyth tanpa `conf`, TWAP < 1800s,
   single-source spot price.
6. **ERC4626** — first-depositor attack (tanpa virtual shares/`_decimalsOffset`), transfer shares
   tanpa pindah lock record.
7. **Reentrancy** — CEI dilanggar (call sebelum update state). Termasuk cross-function & read-only.
8. **Flash loan** — spot price / `slot0` / `getReserves` sebagai oracle.
9. **Signature replay** — hash tanpa nonce / chainId / contract address.
10. **Proxy/upgrade** — storage collision, implementation tidak di-initialize, delegatecall
    ke alamat user-controlled.

Aturan emas: "Baca SEMUA sibling function. Kalau `vote()` ada modifier, cek `poke()`/`reset()`/
`harvest()`. Modifier yang hilang di sibling ITULAH bug-nya."

### 4. Metric — Immunefi — $2,500–$500,000 (program baru, live 2026-11-10)
- Critical: 20% dari dana yang langsung terdampak, maks $500,000, minimum $50,000.
- Scope (4 target): `github.com/Metricxyz/oracle-contracts`,
  `github.com/Metricxyz/periphery-contracts`, `github.com/Metricxyz/pool-contracts`,
  `metric.xyz`. Fokus: oracle/price provider (Pyth/Chainlink signature, compressed-feed
  pusher), pool (propAMM), periphery (router/swap).
- Impacts in scope — Critical: oracle terima price update tanpa signature Pyth/Chainlink
  valid atau compressed-feed push dari pusher tidak resmi; pencurian dana user langsung
  (selain unclaimed yield); insolvency protokol. High: bypass admin fee caps atau
  price-provider timelock; swap di anchored price provider pool di luar band / melewati
  staleness limit; penyalahgunaan role (factory owner, oracle ADMIN_ROLE, propAMM adapter
  admin); freezing dana permanen/sementara; pencurian/freezing unclaimed yield/royalties.
- Audit: Zellic (28 Jun 2026), Sherlock (9 Okt 2026) — semua issue di laporan audit +
  hasil audit contest = KNOWN, tidak eligible. Cek dulu
  `github.com/Metricxyz/Known_issues` sebelum mengklaim temuan.
- KYC TIDAK diperlukan. PoC selalu wajib semua severity. Responsible publication Category 3
  (butuh approval). Submit via Immunefi.
- Workspace: `~/workspace/bug-hunt/metric/`.

## Target aktif

### 1. Revert v4lend — Cantina — $50,000 USDC (HANYA High & Medium)
- Repo: `github.com/revert-finance/v4lend`, commit `f6d0fb41b7487f60ce64e011312a956e184e2a75`
- Scope: `src/vault/V4Vault.sol`, `src/oracle/V4Oracle.sol`, `src/RevertHook.sol`,
  `src/hook/**`, `src/automators/**`, `src/vault/transformers/**`, `src/shared/**`
- Out of scope: `test/`, `script/`, `docs/`; perilaku yang didokumentasikan sebagai intentional
  (whole-balance sweep, AutoLend pegang vault shares, hook swap tanpa amountOutMin floor,
  non-vault remint tanpa automation config); trusted roles (owner/operator/transformer);
  MEV, gas griefing, frontrunning; frontend.
- **Mandatory coded PoC** untuk semua High/Medium sebelum kompetisi berakhir.
- First external review — belum pernah diaudit. Target paling segar, prioritaskan.
- Submit: via Cantina (akun user). Workspace: `~/workspace/bug-hunt/revert-v4lend/`.

### 2. Hyperliquid — sampai $1,000,000 USDC
- Scope: mainnet — bug yang menyebabkan outage / logical error di node atau API server.
  Testnet experimental features TIDAK in scope (kecuali diumumkan di Discord/Telegram).
- Klasifikasi: Critical <1M USDC (loss of user funds / pelanggaran invariansi eksekusi L1),
  High <50k USDC (network downtime tanpa incorrect state), Medium <10k USDC (API performance).
- Submit: email `bugbounty@hyperfoundation.org` + repro steps + PoC. First report wins.
- Payout: USDC di Hyperliquid. Wajib KYC/KYB + bisa terima USDC di Hyperliquid.
- **DILARANG testing di mainnet** — testnet / local fork saja. Dilarang: phishing, DDoS besar,
  testing third-party, public disclosure sebelum fix + dibayar, exploit untuk keuntungan
  di luar reward.
- Workspace: `~/workspace/bug-hunt/hyperliquid/`.

### 3. Ethereum Foundation — sampai $1,000,000 USD
- Scope: spec (Execution + Consensus Layer), EL clients (Besu, Erigon, Ethrex, Geth,
  Nethermind, Reth), CL clients (Grandine, Lighthouse, Lodestar, Nimbus, Prysm, Teku),
  compiler Solidity & Vyper, Deposit Contract, dependensi C-KZG-4844 & Go-ETH-KZG.
- Glamsterdam upgrade in scope dengan multiplier: 0.5x sekarang (spec/EIP), 2.0x setelah
  Sepolia upgrade, 1.5x setelah Hoodi sampai 1 minggu sebelum mainnet upgrade.
- Severity: Low >0.01% validator kena slash / network split; Medium >1% / >5%;
  High >33%; Critical >50% / infinite ETH / steal/burn ETH semua EOA / crash semua client
  dengan 1 tx.
- Out of scope: infra (web/DNS/email), ERC-20, ENS, API yang diekspos publik, EngineAPI,
  typo, tests, single-peer DoS berat, issue yang sudah publik.
- Submit: form `bbp-form.ethereum.org`. Wajib PoC. Dilarang public disclosure sebelum disetujui.
  Identitas wajib (KYC), pajak tanggung researcher.
- Workspace: `~/workspace/bug-hunt/ethereum/`.

## Workflow

Recon → Learn (baca disclosed reports + changelog + "what changed") → Hunt (satu kelas,
dalam) → Validate (7-Question Gate + 4 gates: reality, impact, dedup, report quality)
→ Report (draft, JANGAN submit tanpa user).

Catat leads di `~/workspace/bug-hunt/<target>/leads.md` — jangan sampai hilang.
Satu laporan per chain (A→B→C), bukan per bug — chain bayar 3-10x lebih besar.

## Foundry PoC (wajib untuk Revert; pakai pola yang sama untuk web3 lain)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
import "forge-std/Test.sol";
contract ExploitTest is Test {
    function setUp() public { vm.createSelectFork("mainnet", BLOCK_NUMBER); /* deploy/load target, deal dana */ }
    function test_exploit() public {
        vm.startPrank(attacker);
        // Step 1 setup, Step 2 exploit, Step 3 verifikasi dampak
        vm.stopPrank();
        assertGt(token.balanceOf(attacker), INITIAL_BALANCE, "Exploit failed");
    }
}
```

Jalankan: `forge test --match-test test_exploit -vvvv` (fork: `--fork-url $MAINNET_RPC`).
Cheatcode kunci: `vm.prank/startPrank`, `vm.deal`, `deal(token,addr,amt)`, `vm.warp`,
`vm.roll`, `vm.createSelectFork`, `vm.expectRevert`, `vm.assume`.

## Tooling

- Static: `semgrep --config=p/security-audit ./` (+ `p/owasp-top-ten`, per-bahasa).
- **Slither (Solidity, wajib sebelum manual audit):** `~/.venvs/slither/bin/slither <dir-repo> --exclude-dependencies` (Slither 0.11.6, venv). Triage hasilnya satu-satu (banyak false positive / informational); jangan klaim mentah tanpa 7-Question Gate + PoC.
- Grep pola Solidity: `tx.origin`, `delegatecall`, `selfdestruct`, `block.timestamp`,
  `latestRoundData` tanpa `updatedAt`, `getPriceUnsafe` tanpa `conf`, `ecrecover` tanpa nonce.
- Rust: `.unwrap()`/`.expect()` di path decode input jaringan saja yang exploitable.
