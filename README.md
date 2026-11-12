# CDSS HPB — Consulta de Decisão Terapêutica (Grupo Oncológico HPB)

Experimental, offline clinical decision-support tool for HPB multidisciplinary team meetings. Single file: `index.html`. No build step, no dependencies, no remote resources.

> **FERRAMENTA EXPERIMENTAL DE APOIO À DECISÃO.** Not a medical device. All suggestions require validation and sign-off by the MDT.

## Running it
- Open `index.html` directly in a current Chrome or Firefox (`file://`), or serve it from localhost (`python3 -m http.server 8080`).
- Data is stored only in that browser profile (IndexedDB, falling back to localStorage and then to memory). Clearing the browser data deletes the cases, so export backups regularly.

## Modules (deterministic rule engine)
| Module | Classification / calculators | Main sources |
|---|---|---|
| HCC | Child-Pugh, ALBI, MELD/MELD-Na, CSPH (HVPG / Baveno VII), Milan, Up-to-7, UCSF, UNOS-DS, AFP model, BCLC 0–D with B1/B2/B3 | EASL 2025, BCLC 2022 |
| CRLM | Fong CRS, FLR threshold (20/30/40 %), technical resectability, oncological risk, molecular-driven regimen, TRANSMET criteria / Oslo score | ESMO mCRC, EORTC 40983, New EPOC, CAIRO5, COLLISION, TRANSMET |
| BTC | iCCA resectability, pCCA Bismuth-Corlette + Blumgart T, unresectability criteria, biliary drainage, Mayo protocol, gallbladder T-based re-resection | EASL-ILCA 2023, ESMO BTC 2023, BILCAP, TOPAZ-1, KEYNOTE-966 |
| PDAC | NCCN anatomical R/BR/LA (head vs body/tail), IAP biological/conditional BR, restaging logic | NCCN, IAP 2017, ESMO 2023, PRODIGE 24, PREOPANC |
| IPMN/MCN | Kyoto 2024 HRS/WF, European 2018 absolute/relative indications, cyst-fluid interpretation | Kyoto 2024, European 2018 |

Each output shows the primary recommendation, alternative/conditional pathways, alerts, missing data, citations and a full rule trace ("Porquê esta recomendação?"). The ruleset version is `RULESET` in the code.

## MDT workflow
- Pseudonymised case ID only. The app warns when a case ID or pasted text looks like a name, SNS/NIF number, date of birth or address.
- Sign-off: decision status (concordant / modified / overridden), structured final plan, required rationale for divergence, quorum, lead initials. Signing locks the case, snapshots the CDSS output and stores a SHA-256 integrity hash. Reopening a case is logged.
- Outputs: printable *Ata de Reunião Multidisciplinar* (A4), plain text for SClínico, per-case audit trail, CSV audit export (`;`-separated, UTF-8 BOM for Excel PT).
- Backups: plain JSON, or encrypted JSON (AES-256-GCM, PBKDF2-SHA256 310k iterations). Import merges by case UID and keeps the newer copy.

## Optional local AI (Ollama)
- Restricted by a Content-Security-Policy to `http://localhost:11434` / `127.0.0.1:11434`. If Ollama isn't running, the assistant stays inactive.
- It only (1) proposes field values extracted from pasted text, which the clinician must accept one by one (accepted fields are flagged "◆ IA — verificar"), and (2) drafts narrative text for the Ata, which is printed only after it's marked as reviewed. It never produces recommendations.
- For `file://` pages, set the environment variable `OLLAMA_ORIGINS=*` and restart Ollama, or serve the file from localhost. Keep `OLLAMA_HOST` at its default (127.0.0.1) so Ollama is not exposed to the hospital network.

## Known limitations (must be reviewed before any clinical use)
- The rules simplify the guidelines. Thresholds, regimens and drug availability (INFARMED/AUE) must be validated locally by the HPB group.
- Citations were written without access to the source documents during development. Verify each one, especially the 2025/2026 versions named in the specification (EASL HCC 2025, ESMO mCRC "2026", Kyoto 2024 surveillance intervals). The app shows the type of evidence (RCT, cohort, consensus), not formal grades.
- PII detection is a heuristic, not a guarantee.
