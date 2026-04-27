# 523a8.openbankruptcyproject.org

**§ 523(a)(8) Adversary Proceedings — Empirical Research**

Open dataset and research site documenting student-loan dischargeability adversary proceedings filed in U.S. bankruptcy courts. Maintained by the [Open Bankruptcy Project](https://openbankruptcyproject.org/), a 501(c)(3) public charity (EIN 41-5159631).

Live site: <https://523a8.openbankruptcyproject.org/>

## Headline finding

The November 17, 2022 DOJ/Department of Education guidance reframed the attestation procedure for § 523(a)(8) APs. Annualized filing rate jumped from a recent (2015–2022) baseline of ~17 APs/year nationally to ~1,070 APs/year post-guidance — **~63× annualized rate increase**.

## Dataset

- **Records:** 1,771 adversary proceedings
- **Districts:** 87 of 94 U.S. bankruptcy districts represented
- **Date range:** 1991-09-27 → 2026-04-27
- **Pre-DOJ-guidance** (before 2022-11-17): 254
- **Post-DOJ-guidance** (2022-11-17 onward): 1,516

### File: `data.csv`

| Column | Description |
|---|---|
| `court_id` | CourtListener court identifier (e.g., `ksb`, `mowb`, `paeb`) |
| `case_name` | Adversary proceeding case name |
| `date_filed` | AP filing date (ISO 8601) |
| `date_terminated` | AP termination date (where set) |
| `judge` | Assigned judge (where populated by enrichment) |
| `docket_number` | AP docket number (e.g., `25-04001`) |
| `defendant_type` | Inferred defendant category: `doe`, `nelnet`, `mohela`, `navient`, `sallie_mae`, `ecmc`, `great_lakes`, `pheaa`, `edfinancial`, `aidvantage`, `other`, `unknown` |
| `outcome_class` | Heuristic outcome class — see Limitations below |
| `duration_days` | Days from filing to termination (where terminated) |
| `pre_or_post_doj` | `pre` or `post` (cutoff: 2022-11-17) |

## Methodology

### Source
RECAP archive (Free Law Project) via the CourtListener API v4. Public search and authenticated docket detail endpoints.

### Identification of § 523(a)(8) APs
Filings are identified by case-name match against student-loan creditor patterns:

> Department of Education, NELNET, Mohela, Navient, Sallie Mae, Great Lakes Higher Education, ECMC, Pennsylvania Higher Education Assistance Agency, Edfinancial, Aidvantage

Combined with bankruptcy-court filter (court IDs ending in `b` or containing `bk`) and case-name styling (`X v. Y` adversary-proceeding form).

### Sampling strategy
Two complementary pulls:

1. **Per-court deep pagination** — each of the 89 covered bankruptcy courts paginated to exhaustion (up to 25 pages of 20 results) ordered by date filed descending.
2. **Wide date-window historical pull** — three additional date windows (2008–2014, 2015–2018, 2019–2021) ordered by date filed ascending to capture older filings beyond the recent-results cap.

### Outcome classification (HEURISTIC)
Outcome classes are assigned based on temporal signals only — they do **not** reflect document-level analysis of dispositive orders.

| Class | Definition |
|---|---|
| `pending` | No termination date; last filing within 180 days |
| `stale_abandoned` | No termination date; last filing more than 180 days ago |
| `quick_dismiss` | Terminated within 30 days of filing |
| `settled_or_dismissed` | Terminated 30–180 days after filing |
| `litigated` | Terminated 180+ days after filing |

**A case classified as `litigated` may have ended in either discharge OR denial.** Distinguishing those requires document-level review of the dispositive order, which is the next research step.

## Limitations

1. **No document-level outcome data.** This is the primary limitation of v1. Whether each `litigated` AP ended in discharge, denial, partial discharge, settlement, or conversion is not yet classified.
2. **CourtListener premium gap.** The free CourtListener docket-entries endpoint requires premium access. The dataset captures docket-level metadata but not filing-level text.
3. **Defendant-pattern coverage.** The defendant-name search captures the major federal student loan servicers and the Department of Education. Some private student loan APs may use lender names not in the search list and would be missed.
4. **Pre-DOJ historical depth varies by court.** Some courts have RECAP coverage going back to the early 1990s; others have shorter histories. The `1991-09-27` earliest filing represents the earliest record we found, not necessarily the earliest filing nationally.

## License

Dataset released under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

## Citation

> Open Bankruptcy Project. *§ 523(a)(8) Adversary Proceedings Dataset.* 2026-04-27. <https://523a8.openbankruptcyproject.org/>

## Reproducing this dataset

The pull and classification scripts are in the `pacer/scripts/` tree of the OBP research repository:

- `student_loan_ap_pull.py` — initial CourtListener API pull
- `student_loan_ap_deep_pull.py` — per-court deep pagination
- `student_loan_ap_classify.py` — enrichment + heuristic outcome classification
- `_build_523a8_research_page.py` — generates the published HTML and CSV from the SQLite DB

Requires a CourtListener API token (free at <https://www.courtlistener.com/sign-up/>).

## Related Open Bankruptcy Project resources

- <https://studentloans.openbankruptcyproject.org/> — Consumer guide
- <https://undue-hardship.openbankruptcyproject.org/> — Brunner test reference
- <https://screener.openbankruptcyproject.org/> — Self-screening tool
- <https://tpd-discharge.openbankruptcyproject.org/> — Total & Permanent Disability Discharge
- <https://closed-school.openbankruptcyproject.org/> — Closed School Discharge
- <https://false-certification.openbankruptcyproject.org/> — False Certification Discharge

## Disclaimer

The information in this repository and on the live site is general legal information for educational and research purposes. It is not legal advice and does not create an attorney-client relationship. For advice on a specific situation, consult a licensed bankruptcy attorney in the relevant jurisdiction.
