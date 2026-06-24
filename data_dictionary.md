# Data Dictionary — ATO Legal Database

Data Target [ato.gov.au/law](https://www.ato.gov.au/law/view/), write UTF-8 CSV and NDJSON (JSON Lines).

---

## Common conventions

| Convention | Detail |
|---|---|
| **Delimiter** | ` \| ` (space-pipe-space) separates multiple values or sub-sections within a single cell |
| **Empty values** | Empty string `""`, never `NULL` or `N/A` |
| **Boolean columns** | Python `True` / `False` (written as-is by `csv.DictWriter`; JSON booleans `true`/`false` in NDJSON) |
| **Date strings** | Stored verbatim as scraped — no normalisation applied (see per-scraper notes) |
| **Multi-value columns** | Values joined with ` \| ` — e.g. `ITAA 1936 s 6 \| ITAA 1997 s 995-1` |
| **Encoding** | UTF-8, no BOM |
| **Dual output** | Every scraper writes a `.csv` and a parallel `.jsonl` (NDJSON). JSON keys match CSV column names exactly; one JSON object per line. The NDJSON is the recommended format for `datasets.load_dataset()`. |
| **Zip artefacts** | Each dataset produces four files: `<base>.csv`, `<base>.jsonl`, `<base>.zip` (CSV compressed), `<base>_jsonl.zip` (NDJSON compressed). Zips are refreshed only when new rows were written or the zip is older than its source. |

---

- **`Unmatched_Content` (all datasets):** safety-net column appended to every schema. The scraper sweeps the article for text blocks that no structured extractor captured and stores them here, so format changes or unusual documents never silently drop content. Existing CSV/JSONL files are migrated in place (old rows get an empty value) on the next scraper run.

## 1. Edited Private Advice

**Output files:**
- `EV_Data/edited_private_advice_all.csv` — all years combined into one file
- `EV_Data/edited_private_advice_all.jsonl` — parallel NDJSON (one JSON object per row)
- `EV_Data/edited_private_advice_all.zip` — CSV compressed
- `EV_Data/edited_private_advice_all_jsonl.zip` — NDJSON compressed

**Source URL pattern:** `https://www.ato.gov.au/law/view/document?docid=EV/{Authorisation_Number}`

| Column | Type | Nullable | Description / Format |
|---|---|---|---|
| `Authorisation_Number` | string | No | Primary key. ATO-assigned identifier, e.g. `1-2ABCDEF`. Used to construct `Source_URL`. |
| `Date_of_Advice` | string | Yes | Date the private advice was issued. Scraped verbatim from the HTML label `Date of advice:`. Format is typically `D Month YYYY` (e.g. `1 July 2024`). Blank for older documents that omit this field. |
| `Subject` | string | Yes | Single-sentence topic description of the advice, e.g. `Capital gains tax — main residence exemption`. |
| `Ruling` | string | Yes | All question-and-answer pairs concatenated. Format: `Q1: [text] \| A1: [text] \| Q2: [text] \| A2: [text]`. |
| `Ruling_Period` | string | Yes | Income years to which the ruling applies, pipe-separated. Format: `Year ending 30 June 20XX \| Year ending 30 June 20XX`. |
| `Date_Scheme_Commenced` | string | Yes | Date the relevant arrangement or scheme commenced, if stated. Verbatim from HTML. Blank when not present. |
| `Relevant_Facts_and_Circumstances` | string | Yes | Full text of the facts and circumstances section, collapsed to single-space-separated prose. |
| `Relevant_Legislative_Provisions` | string | Yes | All act/section references concatenated with ` \| `, e.g. `Income Tax Assessment Act 1997 section 104-10 \| Income Tax Assessment Act 1997 section 116-20`. |
| `Reasons_for_Decision` | string | Yes | Summary paragraph plus all detailed reasoning sub-headings. Format: `Summary: [text] \| Detailed Reasoning - [Sub-heading]: [text]`. Sub-heading labels are preserved for parseability. |
| `Is_Archived` | boolean | No | `True` if the document carries an archival disclaimer in its HTML body; `False` otherwise. |
| `Source_URL` | string | No | Full URL of the scraped document, e.g. `https://www.ato.gov.au/law/view/document?docid=EV/1-2ABCDEF`. Stable resume key. |
| `Unmatched_Content` | string | Yes | Fallback capture: article text blocks that none of the structured field extractors claimed (new/changed page formats, unusual sections), joined with ` \| `. Orphan blocks are prefixed with their nearest uncaptured heading. Empty when the structured fields captured everything. |

---

## 2. ATO Interpretative Decisions

**Output files:**
- `EV_Data/ato_interpretative_decisions.csv`
- `EV_Data/ato_interpretative_decisions.jsonl`
- `EV_Data/ato_interpretative_decisions.zip`
- `EV_Data/ato_interpretative_decisions_jsonl.zip`

**Source URL pattern:** `https://www.ato.gov.au/law/view/document?docid=AID/{auth_num}`

| Column | Type | Nullable | Description / Format |
|---|---|---|---|
| `ATO_ID_Number` | string | No | Primary key. ATO ID label as it appears in the document, e.g. `ATO ID 2024/1`. |
| `Status` | string | Yes | Current standing, e.g. `Current`, `Authoritative`, `Withdrawn`. Blank when not present. |
| `Title` | string | Yes | Short descriptive title of the decision. |
| `Issue` | string | Yes | The question or issue being decided, as a prose block. |
| `Decision` | string | Yes | The ATO's ruling on the issue, as a prose block. |
| `Facts` | string | Yes | Factual background underpinning the decision. |
| `Reasons_for_Decision` | string | Yes | Full reasons text. Where sub-headings exist they are preserved with ` \| ` separating each section. |
| `Date_of_Decision` | string | Yes | Date the decision was made. Verbatim from HTML label `Date of decision:`. Typical format: `D Month YYYY` (e.g. `15 March 2023`). |
| `Year_of_Income` | string | Yes | Income year(s) to which the decision applies. Tries both `Year of income` and `Year(s) of income` labels. Typical format: `Year ending 30 June 20XX`. |
| `Legislative_References` | string | Yes | Act and section references, pipe-separated. |
| `Related_Public_Rulings_and_Determinations` | string | Yes | Related ruling/determination identifiers, pipe-separated, e.g. `TR 2023/4 \| TD 2019/7`. |
| `Related_ATO_Interpretative_Decisions` | string | Yes | Cross-referenced ATO ID numbers, pipe-separated. |
| `Subject_References` | string | Yes | Topical subject references, pipe-separated. |
| `Case_References` | string | Yes | Case law citations from the `Case References` block, full text preserved (linked and non-linked parts), e.g. `White v Elmdene Estates Ltd [1960] 1 QB 1`. |
| `Other_References` | string | Yes | Other-references block — explanatory memoranda, dictionaries and external citations not captured by the legislative/case/related-document fields. |
| `Business_Line` | string | Yes | ATO business unit that produced the decision, e.g. `Public Groups and International`. |
| `Is_Archived` | boolean | No | `True` if the document carries an archival disclaimer; `False` otherwise. |
| `Source_URL` | string | No | Full document URL. Stable resume key. |
| `Unmatched_Content` | string | Yes | Fallback capture: article text blocks that none of the structured field extractors claimed (new/changed page formats, unusual sections), joined with ` \| `. Orphan blocks are prefixed with their nearest uncaptured heading. Empty when the structured fields captured everything. |

---

## 3. Public Rulings & Determinations

**Output files** (depends on `--folder` and `--type` flags):

| Flags | Output basename |
|---|---|
| *(default, both folders)* | `public_all` |
| `--folder rulings` | `public_rulings_all` |
| `--folder determinations` | `public_determinations_all` |
| `--type TR` (a ruling code) | `public_rulings_TR` |
| `--type TD` (a determination code) | `public_determinations_TD` |

Four artefacts per run are written to `EV_Data/`: `{basename}.csv`, `{basename}.jsonl`, `{basename}.zip`, and `{basename}_jsonl.zip`.

**Source URL pattern:** `https://www.ato.gov.au/law/view/document?docid={docid}`

**Known type codes:**

| Category | Codes |
|---|---|
| Rulings | `TR`, `GSTR`, `CR`, `PR`, `MT`, `SGR`, `FTR`, `SMSFR`, `ER`, `WTR`, `LCTR`, `MTR`, `TFNR`, `SFR`, `SCR` |
| Determinations | `TD`, `GSTD`, `LCTD`, `FTD`, `SGD`, `SMSFD`, `WTD`, `MTD`, `ED`, `TFND`, `SFD`, `SCD` |

| Column | Type | Nullable | Description / Format |
|---|---|---|---|
| `Document_Reference` | string | No | Primary identifier, e.g. `TR 2023/4`, `TD 2024/1`. Re-parsed from document HTML; falls back to the tree-walk value. |
| `Document_Type` | string | No | Top-level folder: `Ruling` or `Determination`. |
| `Document_Type_Code` | string | No | Short code derived from document title, e.g. `TR`, `GSTR`, `TD`. |
| `Title` | string | Yes | Full document title as it appears in the heading. |
| `Status` | string | Yes | Current standing, e.g. `Current`, `Draft`. Blank when not present. |
| `Date_of_Issue` | string | Yes | Date the ruling/determination was issued. Verbatim from HTML label `Date of issue`. Typical format: `D Month YYYY`. |
| `Date_of_Effect` | string | Yes | Date from which the ruling has effect. May be a prose sentence (e.g. `This Ruling applies from 1 July 2023`) rather than a bare date. |
| `Date_of_Withdrawal` | string | Yes | Date of withdrawal. Blank for current documents. |
| `What_This_Ruling_Is_About` | string | Yes | Introductory section describing the ruling's scope. |
| `Class_of_Entities_or_Scheme` | string | Yes | Description of the taxpayers or schemes to which the ruling applies. |
| `Ruling` | string | Yes | The ruling text (the operative paragraphs). |
| `Examples` | string | Yes | Any worked examples included in the ruling. |
| `Appendix_Explanation` | string | Yes | Content of the first (`Appendix — Explanation`) appendix. |
| `Other_Appendices` | string | Yes | Content of any additional appendices, pipe-separated by appendix heading. |
| `Previous_Rulings` | string | Yes | Identifiers of rulings/determinations this document replaces, pipe-separated. |
| `Related_Rulings` | string | Yes | Identifiers of related rulings/determinations, pipe-separated. |
| `Legislative_References` | string | Yes | Act and section references, pipe-separated. |
| `Case_References` | string | Yes | Case law citations, pipe-separated. |
| `Subject_References` | string | Yes | Topical subject references, pipe-separated. |
| `ATO_References` | string | Yes | ATO internal reference numbers, pipe-separated. |
| `Other_References` | string | Yes | Other-references block — explanatory memoranda, dictionaries and external citations distinct from the ATO file numbers in `ATO_References`. |
| `Is_Withdrawn` | boolean | No | `True` if the document has been formally withdrawn; `False` otherwise. |
| `Source_URL` | string | No | Full document URL. Resume key. |
| `Unmatched_Content` | string | Yes | Fallback capture: article text blocks that none of the structured field extractors claimed (new/changed page formats, unusual sections), joined with ` \| `. Orphan blocks are prefixed with their nearest uncaptured heading. Empty when the structured fields captured everything. |

---

## 4. Practical Compliance Guidelines

**Output files:**
- `EV_Data/practical_compliance_guidelines.csv`
- `EV_Data/practical_compliance_guidelines.jsonl`
- `EV_Data/practical_compliance_guidelines.zip`
- `EV_Data/practical_compliance_guidelines_jsonl.zip`

**Source URL pattern:** `https://www.ato.gov.au/law/view/document?docid={docid}`

**Date format:** PCG dates are extracted by a regex that accepts either `D Month YYYY` (e.g. `1 July 2024`) or `YYYY-MM-DD` (e.g. `2024-07-01`). Values are stored verbatim in whichever format appears in the source document.

| Column | Type | Nullable | Description / Format |
|---|---|---|---|
| `PCG_Number` | string | No | Primary identifier, e.g. `PCG 2024/1`, `Draft PCG 2023/D1`. |
| `Document_Type` | string | No | `Final PCG` (published guideline), `Draft PCG` (formal draft), `Compendium` (EC / Early Commentary consultation draft), or `Unknown` (unusual docid format). |
| `Title` | string | Yes | Full document title. |
| `Status` | string | Yes | `Current` (default for all published PCGs), `Draft` (draft and EC consultation docs), or `Withdrawn`. `Current` is inferred when no withdrawal or draft indicators are found in the document text. |
| `Date_of_Issue` | string | Yes | Date issued, extracted from the "Commissioner of Taxation [date]" sign-off paragraph at the end of each PCG. Format: `D Month YYYY`. Blank for EC (Early Commentary) consultation drafts (which have no Commissioner sign-off) and a small number of older PCGs without this footer. |
| `Date_of_Effect` | string | Yes | Date from which the PCG has effect. Same date formats as `Date_of_Issue`. |
| `Date_of_Withdrawal` | string | Yes | Date of withdrawal. Blank for current documents. |
| `Replaces` | string | Yes | Identifier(s) of prior PCG(s) this document supersedes, pipe-separated. |
| `Related_Rulings_and_Determinations` | string | Yes | Related ruling/determination identifiers, pipe-separated. |
| `Legislative_References` | string | Yes | Act and section references, pipe-separated. |
| `Other_References` | string | Yes | Other-references block — explanatory memoranda, dictionaries and external citations; `Related_Rulings_and_Determinations` pulls only the ruling-style references from this same block. |
| `Summary` | string | Yes | Summary section text. |
| `Purpose_and_Scope` | string | Yes | Purpose and scope section text. |
| `Background` | string | Yes | Background section text. |
| `Compliance_Approach` | string | Yes | Compliance approach section text (often the main operative content). |
| `Examples` | string | Yes | Worked examples. |
| `Appendices` | string | Yes | Appendix content. |
| `Other_Sections` | string | Yes | Content from any sections not covered by the named columns above, concatenated with ` \| ` between sections. |
| `Compendium_Reference` | string | Yes | Compendium reference if present, e.g. `PCG 2024/1EC`. |
| `Is_Archived` | boolean | No | `True` if the document carries an archival disclaimer; `False` otherwise. |
| `Is_Draft` | boolean | No | `True` for `Draft PCG` and `Compendium` (EC / Early Commentary) doc types, and for any `PCG_Number` ending in `EC`; `False` for finalized PCGs. |
| `Source_URL` | string | No | Full document URL. Resume key. |
| `Unmatched_Content` | string | Yes | Fallback capture: article text blocks that none of the structured field extractors claimed (new/changed page formats, unusual sections), joined with ` \| `. Orphan blocks are prefixed with their nearest uncaptured heading. Empty when the structured fields captured everything. |

---

## 5. Taxpayer Alerts

**Output files:**
- `EV_Data/taxpayer_alerts_all.csv`
- `EV_Data/taxpayer_alerts_all.jsonl`
- `EV_Data/taxpayer_alerts_all.zip`
- `EV_Data/taxpayer_alerts_all_jsonl.zip`

**Source URL pattern:** `https://www.ato.gov.au/law/view/document?docid=TPA/TA{year}{num}/NAT/ATO/00001`

| Column | Type | Nullable | Description / Format |
|---|---|---|---|
| `Alert_Number` | string | No | Primary identifier, e.g. `TA 2026/1`. Parsed from H2 on the document page. |
| `Title` | string | Yes | Full descriptive title of the alert, e.g. `Contrived property development arrangements…`. |
| `Date_of_Issue` | string | Yes | Date the alert was issued. Verbatim from page metadata. Typical format: `D Month YYYY`. Blank when not present. |
| `Status` | string | Yes | `Current` or `Withdrawn`. Derived from page header text. |
| `Overview` | string | Yes | Paragraphs under the "Overview" heading, joined with ` \| `. |
| `Description` | string | Yes | Paragraphs under the "Description" heading, joined with ` \| `. |
| `Example` | string | Yes | Paragraphs under the "Example" heading, joined with ` \| `. Blank when no example is included. |
| `Our_Concerns` | string | Yes | Paragraphs under the "Our concerns" heading. |
| `What_ATO_Is_Doing` | string | Yes | Paragraphs under the "What we are doing" heading. |
| `What_You_Should_Do` | string | Yes | Paragraphs under the "What you should do" heading. |
| `Related_Documents` | string | Yes | Pipe-separated document and legislative references from inline links in the article body (e.g. `PS LA 2008/15 \| ITAA 1997 Div 70`). Nav links and self-references excluded. |
| `Is_Withdrawn` | boolean | No | `True` if the alert appears in the "Withdrawn" sub-folder of the tree; `False` otherwise. |
| `Source_URL` | string | No | Full document URL. Resume key. |
| `Unmatched_Content` | string | Yes | Fallback capture: article text blocks that none of the structured field extractors claimed (new/changed page formats, unusual sections), joined with ` \| `. Orphan blocks are prefixed with their nearest uncaptured heading. Empty when the structured fields captured everything. |

---

## 6. Decision Impact Statements

**Output files:**
- `EV_Data/decision_impact_statements_all.csv`
- `EV_Data/decision_impact_statements_all.jsonl`
- `EV_Data/decision_impact_statements_all.zip`
- `EV_Data/decision_impact_statements_all_jsonl.zip`

**Source URL pattern:** `https://www.ato.gov.au/law/view/document?docid=LIT/ICD/{court_ref}/00001`

| Column | Type | Nullable | Description / Format |
|---|---|---|---|
| `Case_Name` | string | No | Full case citation, e.g. `Commissioner of Taxation v Bendel [2025] FCAFC 15`. Parsed from H2. |
| `Venue_Reference_No` | string | Yes | Court file number, e.g. `VID 903 of 2023`. Parsed from `Venue Reference No:` label. |
| `Venue` | string | Yes | Court or tribunal name, e.g. `Full Federal Court`. Parsed from `Venue:` label. |
| `Judgment_Date` | string | Yes | Date of the judgment. Format: `D Month YYYY`. |
| `Date_Published` | string | Yes | Date the DIS was published by the ATO. Parsed from the page title parenthetical `(Published DD Month YYYY)`. |
| `Document_Type` | string | No | `Decision Impact Statement` or `Interim Decision Impact Statement`. |
| `Summary_of_Decision` | string | Yes | Paragraphs under the "Summary of decision" heading, joined with ` \| `. |
| `Overview_of_Facts` | string | Yes | Paragraphs under the "Overview of facts" heading, joined with ` \| `. |
| `Issues_Decided` | string | Yes | All issue sub-sections concatenated: `Issue 1 – [topic]: [text] \| Issue 2 – [topic]: [text]`. |
| `ATO_View_of_Decision` | string | Yes | The ATO's response — whether it accepts, appeals, or will update guidance. This is the critical field. |
| `Administrative_Treatment` | string | Yes | Paragraphs under the "Administrative treatment" heading. |
| `Related_Documents` | string | Yes | Pipe-separated document and legislative references from inline links (e.g. `TD 2022/11 \| PCG 2022/2`). |
| `Legislative_References` | string | Yes | Legislative-references block, full text (Acts plus section numbers, linked and non-linked parts preserved). |
| `Case_References` | string | Yes | Case-references block — full case citations including the case name text the link-only `Related_Documents` list drops. |
| `Subject_References` | string | Yes | Topical subject-keyword taxonomy (not present in `Related_Documents` at all). |
| `Is_Interim` | boolean | No | `True` if H1 is "Interim Decision Impact Statement"; `False` otherwise. |
| `Source_URL` | string | No | Full document URL. Resume key. |
| `Unmatched_Content` | string | Yes | Fallback capture: article text blocks that none of the structured field extractors claimed (new/changed page formats, unusual sections), joined with ` \| `. Orphan blocks are prefixed with their nearest uncaptured heading. Empty when the structured fields captured everything. |

---

## 7. Law Administration Practice Statements

**Output files:**
- `EV_Data/law_admin_practice_statements_all.csv`
- `EV_Data/law_admin_practice_statements_all.jsonl`
- `EV_Data/law_admin_practice_statements_all.zip`
- `EV_Data/law_admin_practice_statements_all_jsonl.zip`

**Source URL pattern:** `https://www.ato.gov.au/law/view/document?docid=PSR/PS{year}{num}/NAT/ATO/00001` (regular); `DPS/DPSLA{year}{num}/NAT/ATO/00001` (draft)

| Column | Type | Nullable | Description / Format |
|---|---|---|---|
| `Statement_Reference` | string | No | Primary identifier, e.g. `PS LA 2026/1`. Parsed from H2. |
| `Title` | string | Yes | Subject line of the statement, e.g. `Self-managed superannuation funds — education directions…`. |
| `Date_of_Issue` | string | Yes | Date issued. Format: `D Month YYYY`. Parsed from `Date of Issue:` metadata block. |
| `Date_of_Effect` | string | Yes | Date from which the practice statement has effect. Same format as `Date_of_Issue`. |
| `Document_Type` | string | No | `Law Administration Practice Statement`, `Draft Law Administration Practice Statement`, or `Law Administration Practice Statement (GA)`. |
| `Has_Compendium` | boolean | No | `True` if the page references a compendium (e.g. `PS LA 2026/1EC`); `False` otherwise. |
| `Body` | string | Yes | All numbered sections concatenated: `1. [Heading]: [text] \| 2. [Heading]: [text] \| … \| Appendix: [text]`. |
| `Related_Documents` | string | Yes | Pipe-separated document and legislative references from inline links. |
| `Legislative_References` | string | Yes | Legislative references from the `Legislative References:` metadata block at end of page, pipe-separated. |
| `Subject_References` | string | Yes | Topical subject-keyword taxonomy from the `Subject References` block (not present in `Related_Documents`). |
| `Other_References` | string | Yes | Other-references block — explanatory memoranda, charters, dictionaries and external citations. |
| `Is_Draft` | boolean | No | `True` when `Document_Type` starts with `Draft`; `False` otherwise. |
| `Is_Withdrawn` | boolean | No | `True` if page header indicates the statement has been withdrawn; `False` otherwise. |
| `Source_URL` | string | No | Full document URL. Resume key. |
| `Unmatched_Content` | string | Yes | Fallback capture: article text blocks that none of the structured field extractors claimed (new/changed page formats, unusual sections), joined with ` \| `. Orphan blocks are prefixed with their nearest uncaptured heading. Empty when the structured fields captured everything. |

---

## 8. Legislative Instruments

**Output files:**
- `EV_Data/legislative_instruments_all.csv`
- `EV_Data/legislative_instruments_all.jsonl`
- `EV_Data/legislative_instruments_all.zip`
- `EV_Data/legislative_instruments_all_jsonl.zip`

**Source URL pattern:** `https://www.ato.gov.au/law/view/document?docid=OPS/{docid}/NAT/ATO/00001`

**Scope:** Legislative instruments, determinations, data standards, frameworks, and notices made by the Commissioner under tax and related legislation. Covers current, draft, and repealed/archived instruments. Documents are registered on the Federal Register of Legislation (FRLI). Payment Summary Notices (PYV doctype) and other old unstructured instruments are included but have blank `Title` fields.

| Column | Type | Nullable | Description / Format |
|---|---|---|---|
| `Instrument_Reference` | string | No | Primary identifier parsed from the H1, e.g. `LI 2024/19`, `ABRS 2021/1`. |
| `Title` | string | Yes | Formal name of the instrument as stated in its "1 Name" section, e.g. `Income Tax Assessment (Effective Life of Depreciating Assets) Amendment Determination 2024`. Blank for Payment Summary Notices and other old unstructured documents that have no standard "Name" section. |
| `Enabling_Act` | string | Yes | The empowering legislation, extracted from the first `<h3>` in the header sub-div, e.g. `Income Tax Assessment Act 1997`. |
| `Document_Type` | string | No | Nature of the instrument — first `<h3>` in the body sub-div when present (e.g. `Legislative Instrument`, `Determination`, `Data Standard`, `Framework`). Falls back to inferring from the "Name of instrument/determination/notice" heading pattern; defaults to `Legislative Instrument` when neither is available. |
| `Topic_Category` | string | Yes | Folder breadcrumb from the ATO tree walk, e.g. `Current > Income tax > Deductions > Motor vehicle expenses`. Reflects where the document sits in the ATO's hierarchy. |
| `Date_of_Issue` | string | Yes | Date signed by the Commissioner, extracted from the footer paragraph, e.g. `6 June 2024`. Blank for some older instruments that omit this. |
| `Signatory` | string | Yes | Name and title of the signatory following the date in the footer, e.g. `Ben Kelly Deputy Commissioner of Taxation`. |
| `Registration_Number` | string | Yes | FRLI registration number, e.g. `F2024L00712`. Present for registered instruments; blank for drafts and many repealed documents. |
| `Registration_Date` | string | Yes | Date of FRLI registration, e.g. `10 June 2024`. Present alongside `Registration_Number`; blank otherwise. |
| `Body` | string | Yes | All numbered sections concatenated: `1 Name: [text] \| 2 Commencement: [text] \| …`. Handles both modern format (`1  Name`) and older format (`1. Name of instrument`). For unstructured documents with no numbered sections (e.g. old Payment Summary Notices), falls back to all paragraph text joined with ` \| `. |
| `Related_Explanatory_Statements` | string | Yes | Pipe-separated references to associated explanatory statements or other linked documents, derived from inline links in the article body. |
| `Is_Draft` | boolean | No | `True` for documents under the ATO tree's "Drafts for comment" folder (docids typically contain a `D` marker, e.g. `LI2026D1`); `False` otherwise. |
| `Is_Repealed` | boolean | No | `True` for documents under the ATO tree's "Repealed or archived" folder; `False` otherwise. |
| `Source_URL` | string | No | Full document URL. Resume key. |
| `Unmatched_Content` | string | Yes | Fallback capture: article text blocks that none of the structured field extractors claimed (new/changed page formats, unusual sections), joined with ` \| `. Orphan blocks are prefixed with their nearest uncaptured heading. Empty when the structured fields captured everything. |


---

## 9. Principal Legislation (provisions)

**Output files:**
- `EV_Data/frl_principal_legislation_all.csv` / `.jsonl` / `.zip` / `_jsonl.zip`

**Source:** Federal Register of Legislation (legislation.gov.au), latest in-force compilation of each ATO-administered Act. **One row per provision (section).**

**Scope:** Every section of the 22 ATO-administered principal Acts (`ato_administered_acts.py`) — ITAA 1997 & 1936, TAA 1953, GST/LCT/WET/GSTT, FBTAA, super (SGAA/SGCA/SISA/SUMLMA), Fuel Tax, PRRTAA, TASA, etc. Crown copyright, reused under CC BY 4.0.

| Column | Type | Nullable | Description / Format |
|---|---|---|---|
| `Act_Short_Name` | string | No | Internal short code, e.g. `ITAA1997`, `FBTAA`, `GST`. |
| `Act_Title` | string | No | Full Act title, e.g. `Income Tax Assessment Act 1997`. |
| `Act_Year` | int | No | Year of the Act. |
| `Act_FRL_Id` | string | No | FRL series id, e.g. `C2004A05138`. |
| `Compilation_PiT` | string | No | Compilation point-in-time; `latest` for the current in-force compilation. |
| `Provision` | string | No | Section label, e.g. `s 6-5`, `s 136`, `s 102UQ`. |
| `Provision_Key` | string | No | Normalised join key (`s 6-5`→`s6-5`, `Section 3`→`s3`, sub-provisions roll up to their section). Join key to the history datasets. |
| `Heading` | string | Yes | Section heading, e.g. `Income according to ordinary concepts (ordinary income)`. |
| `Text` | string | Yes | Full operative text of the section, subsections/notes/tables concatenated. Genuinely empty for reserved sections and navigational "Map of …"/"Diagram giving overview" headings (an ATO/FRL source characteristic, not a scrape gap). `*` marks defined terms, per FRL convention. |
| `Source_URL` | string | No | `https://www.legislation.gov.au/{Act_FRL_Id}/latest/text#{provision}`. Resume key. |

## 10. Legislation Amendment History

**Output files:**
- `EV_Data/frl_legislation_history_all.csv` / `.jsonl` / `.zip` / `_jsonl.zip`

**Source:** FRL compilation endnotes (Endnote 2 abbreviation key, Endnote 3 legislation history, Endnote 4 amendment history). **One row per amendment event** (a provision changed by an amending Act). Notes are reconstructed in our own wording from Crown-copyright FRL facts

| Column | Type | Nullable | Description / Format |
|---|---|---|---|
| `Act_Short_Name` / `Act_Title` / `Act_Year` / `Act_FRL_Id` | — | No | Identify the principal Act (as above). |
| `Provision` | string | No | Provision affected, e.g. `s 5B`, `s. 41-5(1)`, `Note 1 to s. 41-5(1)`. |
| `Provision_Key` | string | No | Normalised key; joins to the principal dataset. Sub-provision/note amendments roll up to their section. |
| `Action_Code` | string | Yes | Raw endnote code: `am` (amended), `ad` (added/inserted), `rep` (repealed), `rs` (repealed & substituted), etc. |
| `Action` | string | No | Expanded verb for the note, e.g. `Amended`, `Inserted`, `Repealed`. |
| `Amending_Act_Number` | int | No | Number of the amending Act. |
| `Amending_Act_Year` | int | No | Year of the amending Act. |
| `Amending_Act_FRL_Id` | string | Yes | Constructed/resolved FRL id of the amending Act, e.g. `C2013A00124`. |
| `Amending_Item` | string | Yes | Schedule item that made the change, e.g. `Sch 11 item 10` (best-effort; blank when run with `--no-items` or unattributable, e.g. inserted sections). |
| `Commencement` | string | Yes | Commencement detail from Endnote 3, e.g. `Sch 11 (items 10-25, 27): 30 June 2013`. May carry multiple schedule dates or `Royal Assent`. |
| `Application_Saving_Transitional` | string | Yes | Application/saving/transitional text for the amending Act, where present. |
| `History_Note` | string | No | Reconstructed note, e.g. `Amended by No 124 of 2013, effective 30 June 2013`. |
| `Source_URL` | string | No | `https://www.legislation.gov.au/{amending_act}#{Act}-{provision}-{code}-{item}` (with a `~N` suffix on rare identically-labelled events). Resume key. |

## 11. Legislation + History Notes (joined)

**Output files:**
- `EV_Data/frl_legislation_with_history_all.csv` / `.jsonl` / `.zip` / `_jsonl.zip`

**Source:** Left-join of datasets 9 and 10 on `(Act_FRL_Id, Provision_Key)`. **One row per current section**, carrying its full text plus the chronological history notes — the ATO Legal Database "see history notes" view, rebuilt from FRL. Repealed/renumbered provisions absent from the current compilation are history-only orphans (add `--include-orphans` to emit them).

| Column | Type | Nullable | Description / Format |
|---|---|---|---|
| `Act_Short_Name` / `Act_Title` / `Act_Year` / `Act_FRL_Id` | — | No | Identify the principal Act. |
| `Provision` / `Provision_Key` / `Heading` / `Text` | — | — | As in dataset 9 (current section text). |
| `Amendment_Count` | int | No | Number of amendment events for this section (0 = never amended). |
| `First_Amended` | string | Yes | Earliest amending Act, e.g. `No 139 of 1987`. Blank when unamended. |
| `Last_Amended` | string | Yes | Most recent amending Act. Blank when unamended. |
| `Amending_Acts` | string | Yes | Pipe-separated distinct amending Acts in order, e.g. `No 139 of 1987 \| No 145 of 1995 \| …`. |
| `History_Notes` | string | Yes | Chronologically-ordered reconstructed notes, pipe-separated, e.g. `Amended by No 139 of 1987, effective 18 Dec 1987 \| Amended by …`. Blank when unamended. |
| `Source_URL` | string | No | The provision's `/latest/text#{provision}` URL. |

> **Point-in-time text** (`frl_point_in_time_all`) — one row per (compilation version, provision) so you can reconstruct what any section said on any past date — is produced by `frl_point_in_time_scraper.py` and will be added here once the full all-versions run completes.
