# Bank Statement Analysis Tool

## The Problem

When assessing loan applications, brokers analyse bank statements to understand a customer's financial position. The current industry tool (illion BankStatements) provides HTML/JSON exports but has significant UX limitations:

1. **Vertical-only layout**: Multiple accounts stacked vertically. Matching interbank transfers (e.g., finding where a $1,000 credit came from) requires manual Ctrl+F hunting.

2. **Poor categorisation**: Transactions often mislabelled. Genuine business income tagged as "Other Credit". No way to correct this.

3. **Limited analysis**: Basic income/expense summaries. No trend analysis, no cash flow timing, no irregular payment detection.

4. **No interactivity**: Static display. Can't filter, can't compare periods, can't annotate for underwriters.

5. **No persistence**: No way to save work, no collaboration, no deal tracking.

## Your Task

Build an **Electron desktop app** (Windows or macOS), backed by a small cloud API, that functions as both a **document viewer** and a **data extraction tool**. Brokers should be able to:

1. Upload bank statements (HTML or JSON) and have them parsed into structured data
2. View and analyse transactions with categorisation overrides
3. Annotate line items with notes and tags
4. Track assessment progress (team collaboration is a nice-to-have)
5. Generate assessment summaries and export reports

### Tech Stack Requirements

- **App**: Electron desktop app that runs on Windows or macOS, written in TypeScript with React for the UI
- **Backend API**: A small API on Google Cloud Run — the app talks only to this, never directly to the database or storage
- **Database**: Cloud SQL for PostgreSQL
- **Sign-in**: Google sign-in via Google Identity Platform, opened in the user's normal web browser
- **File Storage**: Cloud Storage (for uploaded statements), accessed through your API
- **Distribution**: An installer for Windows or macOS (whichever you use), attached to a GitHub Release

See the [root README](../README.md#desktop-app-requirements-both-projects) for the desktop app requirements that apply to both projects.

---

## Core Concepts

### Deal

A **Deal** is the central entity that groups everything together:
- Multiple bank statements (from different accounts, banks, or time periods)
- Annotations and tags applied to transactions
- Assessment notes and final recommendations
- Deal type: **Consumer** or **Commercial** (can be changed at any time)

A single deal can contain:
- Multiple transaction accounts (savings, everyday)
- Loan accounts
- Credit card accounts
- Accounts from different banks

### Assessment Workflow

Deals progress through these statuses:

| Status | Description |
|--------|-------------|
| **Uploaded** | Statement files have been uploaded, awaiting parsing |
| **Parsed** | Transactions extracted and stored, ready for review |
| **Checked** | Broker has reviewed and annotated transactions |
| **Completed** | Assessment finished with final recommendation |

Note: Assessments are always editable - status indicates progress, not a lock.

---

## Core Features

### 1. Sign-in & Team Collaboration

- [ ] Google sign-in via Google Identity Platform, opened in the user's normal web browser
- [ ] Until the user signs in, the app shows only the sign-in screen
- [ ] User can see all deals they have access to
- [ ] **Team collaboration** - multiple brokers can work on the same deal *(nice to have)*
- [ ] Activity tracking (who made what changes) *(nice to have)*

### 2. File Upload & Parsing

**Supported Formats:**
- illion HTML exports (`.html` files matching the V3-QVSF/V3-QVSN format)
- illion JSON exports (`.json` files matching the v2 schema)

**Requirements:**
- [ ] Open statements with the operating system's **file picker** (Electron's `dialog`), and support **dragging files in from Finder or File Explorer**
- [ ] Parse and extract transaction data into structured database format
- [ ] Support multiple file uploads per deal
- [ ] **Validation**: Only accept files matching the sample formats provided (hardcoded validation)
- [ ] Reject files that don't match illion format with clear error message
- [ ] Store original file in Cloud Storage for reference (uploaded through your API — no cloud keys in the app)
- [ ] Show parsing progress/status for large files

**Data Extraction:**
From each statement, extract and store:
- Account holder name
- Bank name and BSB
- Account number and type
- Opening/closing balance
- Available balance
- Transaction date, description, amount, balance
- Original illion categories/tags (preserved but overridable)

### 3. Multi-Account Side-by-Side View

- [ ] Display all accounts horizontally in columns
- [ ] Synchronised scrolling for comparing transactions across accounts
- [ ] Visual highlighting of related transactions (interbank transfers)
- [ ] Collapsible accounts for managing screen space
- [ ] Account summary cards showing key metrics

### 4. Transaction Categorisation System

**Show existing illion categories but allow override.**

The tool should display illion's original categorisation (e.g., "Groceries", "Dining Out", "External Transfers") but allow users to:
- Override with their own tags
- Apply multiple tags to a single transaction
- Create custom tag categories
- Save tag presets for reuse

**Default Tag Categories:**

| Consumer Tags | Commercial Tags |
|---------------|-----------------|
| Primary Income / Salary | Genuine Revenue |
| Centrelink Benefits | Interbank Transfers |
| Loan Repayments | Loan Repayments |
| Debt Collections | ATO Payments |
| Gambling | Owner Drawings |
| BNPL (Buy Now Pay Later) | Wages Paid |
| ATM Withdrawals | Debt Collections |
| Large Transfers | Significant Expenses (5-digit+) |
| Recurring Expenses | Crypto/Digital Assets |
| Crypto/Digital Assets | |

**Custom Categories:**
- [ ] Users can create their own tag categories
- [ ] Save category presets per user/team
- [ ] Import/export category configurations

### 5. Transaction Annotations

**Tags vs Annotations:**
- **Tag**: A category label (e.g., "Gambling", "Genuine Revenue") - can have multiple
- **Annotation**: A free-text note (e.g., "Needs explanation - large payment to unknown recipient")

**Requirements:**
- [ ] Click on any transaction to add tags and/or annotations
- [ ] One transaction can have multiple tags AND an annotation
- [ ] Annotations are timestamped with author name
- [ ] Filter transactions by tag
- [ ] Search within annotations
- [ ] Bulk tagging (select multiple transactions, apply same tag)

### 6. Analysis Lens - Consumer vs Commercial

The deal type determines which analysis checklist is displayed.

#### Consumer Analysis Checklist

| Question | What to Look For |
|----------|------------------|
| Primary Income | Salary/wage frequency, amount, employer consistency |
| Are declared expenses true? | Match against application declarations |
| Any income subsidies not declared? | Centrelink, Family Benefits, Carers payments |
| Signs of good conduct? | Regular repayments, savings behaviour |
| Any debt collections? | Recovery agencies, collection notices |
| Bad account conduct? | Gambling, excessive withdrawals, dishonours, days in negative |
| Arrears, knockbacks, days in negative? | Payment failures, negative balance periods |
| Any accounts not declared? | Hidden savings, undisclosed accounts |
| Large payments to/from? | Significant transfers requiring explanation |
| Consistent/recurring expenses? | International transfers, insurances, loan repayments |

#### Commercial Analysis Checklist

| Question | What to Look For |
|----------|------------------|
| Revenue Per Month? | Total business income, breakdown by source |
| Genuine revenue and days? | Real trading income vs interbank transfers |
| Last Month Revenue? | Most recent month's performance |
| Any interbank transfers inflating credits? | Circular transfers between related accounts |
| Any 5-digit or significant expenses/debts? | Large payments, supplier invoices |
| Most significant debtor? | Largest customer paying the business |
| Any debt collections? | Recovery agencies indicating credit issues |
| Bad account conduct? | ATM withdrawals, gambling, dishonours |
| Arrears, knockbacks, days in negative? | Cash flow stress indicators |
| Current Loans? | Existing finance commitments |
| ATO payments? | Tax obligations, payment patterns |
| Savings taken, owner drawings? | Money withdrawn from business |

### 7. Assessment Output & Export

**Output Format:**
The final output should be a structured notes table containing:
- Tagged transactions summary (grouped by tag category)
- Annotations requiring follow-up
- **Final suggestion**: Questions to ask the applicant OR acceptability statement
- Mitigants and explanations for any concerns

**Export Options:**
- [ ] **Consumer Summary Export** - formatted for consumer loan applications
- [ ] **Commercial Summary Export** - formatted for commercial loan applications
- [ ] Copy to clipboard using the system clipboard (for pasting into loan submission systems)
- [ ] PDF export (optional)

**Example Output Structure:**

```
DEAL: J Smith - Consumer Loan Application
STATUS: Checked
ACCOUNTS: 3 (ANZ Transaction, Bendigo Home Loan, QCCU Savings)

ASSESSMENT SUMMARY:
------------------
Primary Income: $7,375.30 avg monthly from AUSTRALIAN COUNT (fortnightly)
Conduct: Good - consistent loan repayments, no gambling detected
Concerns: 1 large transfer requiring explanation

FLAGGED TRANSACTIONS:
--------------------
| Date       | Amount     | Tag              | Annotation                    |
|------------|------------|------------------|-------------------------------|
| 2025-11-15 | $6,530.00  | Large Transfer   | Transfer to SAV 64277377      |
| 2025-10-22 | $1,765.86  | Loan Repayment   | FMC Direct Debit - verify     |

QUESTIONS TO ASK:
-----------------
1. Please explain the $6,530 transfer on 15/11 - is this savings or expense?
2. Confirm FMC Direct Debit is the disclosed auto loan

RECOMMENDATION: Acceptable subject to clarifying questions above
```

### 8. Deal Management

- [ ] Create new deals with name and type (Consumer/Commercial)
- [ ] Upload multiple statements to a single deal
- [ ] Change deal type at any time (Consumer ↔ Commercial)
- [ ] View all deals with status and last modified date
- [ ] Search and filter deals
- [ ] Archive/delete completed deals
- [ ] Duplicate a deal (for similar applications)

---

## Database Design

You are responsible for designing your own database schema. We intentionally do not provide a suggested schema - your schema design decisions will be evaluated as part of your submission.

**Key Entities to Consider:**

- **Users** - authenticated users with team membership
- **Teams** - groups of users who can collaborate on deals
- **Deals** - the central entity grouping statements and assessments
- **Statements** - uploaded files with parsing status
- **Accounts** - bank accounts extracted from statements
- **Transactions** - individual transactions with amounts, dates, descriptions
- **Tags** - category labels (both default and custom)
- **Annotations** - free-text notes attached to transactions
- **Tag Presets** - saved category configurations

**Security Requirements:**
- [ ] Your API checks the signed-in user on every request
- [ ] Row Level Security (RLS) policies for all tables
- [ ] Users can only access deals they own or are shared with
- [ ] Team members can view/edit shared deals (if you build team collaboration)

---

## Technical Requirements

### Must Have

- [ ] Electron app meeting the [desktop app requirements](../README.md#desktop-app-requirements-both-projects)
- [ ] Backend API on Cloud Run, with Cloud SQL (PostgreSQL), Google Identity Platform and Cloud Storage integration
- [ ] Google sign-in working correctly from the desktop app
- [ ] **HTML and JSON parsing** - extract transactions from both formats
- [ ] **Format validation** - reject non-illion files with clear error
- [ ] Row Level Security (RLS) policies for all database tables
- [ ] Multi-account side-by-side display
- [ ] Transaction tagging with click-to-tag functionality
- [ ] Free-text annotations on transactions
- [ ] Consumer vs Commercial analysis mode toggle
- [ ] Deal status tracking (uploaded → parsed → checked → completed)
- [ ] Basic income/expense summary per account
- [ ] Search and filter functionality
- [ ] Tests validating parsing against the sample files

### Should Have

- [ ] Automatic interbank transfer detection (matching amounts/dates)
- [ ] Visual linking of related transactions across accounts
- [ ] Custom tag categories with save/load presets
- [ ] Export summary for loan applications (Consumer and Commercial formats)
- [ ] Bulk tagging (select multiple, apply tag)

### Nice to Have

- [ ] **Team collaboration** - share deals with other users

- [ ] Trend visualisation (charts/graphs)
- [ ] Anomaly detection (unusual transactions, gambling patterns)
- [ ] Income consistency analysis (regular vs irregular)
- [ ] Direct comparison against lending criteria
- [ ] PDF export
- [ ] Activity log showing who made what changes

---

## Resources Provided

```
resources/
├── statements/           # Sample illion exports (HTML + JSON)
│   ├── a-t/             # Consumer - 3 bank accounts (ANZ, Bendigo, QCCU)
│   ├── t-k-pty-ltd/     # Commercial - 1 bank account (CBA)
│   ├── h-c-pty-ltd/     # Commercial - 1 bank account (Westpac)
│   └── j-v-w/           # Consumer - 1 bank account (ING)
└── test-cases.md        # Scenarios to validate against
```

### Sample Data Overview

| Case | Type | Accounts | Key Features |
|------|------|----------|--------------|
| **A-T** | Consumer | 3 (ANZ, Bendigo, QCCU) | Multi-bank, interbank transfers, regular income |
| **T-K-PTY-LTD** | Commercial | 1 (CBA) | Business account, needs full analysis |
| **H-C-PTY-LTD** | Commercial | 1 (Westpac) | Large revenue, multiple debtors, debt collections |
| **J-V-W** | Consumer | 1 (ING) | Multiple income sources, Centrelink, large unexplained payment |

---

## Data Format

### JSON Format (v2)

```json
{
  "dataVersion": 20170401,
  "reference": "XXXX",
  "submissionTime": "2025-12-27T08:34:31",
  "bankData": {
    "bankName": "ANZ",
    "bankSlug": "anz",
    "bankAccounts": [
      {
        "id": 0,
        "accountType": "transaction",
        "accountHolder": "Account Holder",
        "accountName": "Account Name",
        "bsb": "XXXXXX",
        "accountNumber": "XXXXXXXXX",
        "currentBalance": "0.86",
        "availableBalance": "0.86",
        "transactions": [
          {
            "date": "2025-12-22",
            "text": "PAYMENT DESCRIPTION",
            "amount": -199,
            "balance": "0.86",
            "type": "General Payment",
            "tags": [{"thirdParty": "Merchant Name"}]
          }
        ]
      }
    ]
  }
}
```

### HTML Format (V3)

The HTML files are illion's visual export format. They contain the same data as JSON but require HTML parsing to extract. Key elements:
- Account headers with BSB/account number
- Transaction tables with date, description, amount, balance
- Category badges on transactions
- Summary sections with totals

**Your parser must handle both formats and produce identical structured data.**

---

## Key Challenges

1. **HTML Parsing**: Extracting structured data from illion's HTML format reliably
2. **Format Validation**: Detecting non-illion files and rejecting gracefully
3. **Layout Complexity**: Multiple accounts must be viewable simultaneously while remaining readable
4. **Transfer Matching**: Same amount transferred between accounts may have different dates (banking delays), different descriptions
5. **Category Override**: Maintaining both original and user-applied categories
6. **Performance**: Statements can have 1000+ transactions; must remain responsive
7. **Collaboration** *(nice to have)*: Sync between team members

---

## Validation Scenarios

Your tool passes validation when a broker can:

1. **Upload & Parse**: Drag an HTML file in from Finder/File Explorer (or open it with the file picker), see it parsed and displayed within seconds
2. **Multi-Account View**: Open a statement with 4+ accounts and view all side-by-side
3. **Transfer Detection**: Find a $1,000 transfer between accounts in under 5 seconds
4. **Tagging**: Tag a transaction (e.g., "Genuine Revenue" or "Gambling") with a single click
5. **Annotation**: Add a note to a transaction that persists after logout/login
6. **Custom Categories**: Create a new tag category, apply it, save as preset
7. **Mode Toggle**: Switch between Consumer and Commercial analysis modes
8. **Assessment Status**: Move a deal from "Uploaded" to "Completed" through the workflow
9. **Export**: Generate a summary showing analysis against the assessment checklist
10. **Persistence**: Sign out, sign back in (or quit and reopen the app), see all previous work intact
11. **Collaboration** *(nice to have)*: Share a deal with a team member, see their annotations

---

## Submission Requirements

### Deployment & Distribution

- [ ] Backend API deployed to the **Google Cloud** project provisioned for you
- [ ] **Cloud SQL for PostgreSQL** for the database, **Cloud Storage** for uploaded statements
- [ ] A **GitHub Release** with an installer for Windows or macOS — send us the link (installers for both are a nice-to-have)

**Code due Tue 29 Sep, 11:59 pm IST.**

### Video Explanation (Required)

A **10-minute video**, face on camera, walking through your code and the decisions behind it. Due **Wed 30 Sep, 11:59 pm IST**.

Cover:

- **Schema design** - how you structured deals, statements, transactions and annotations, how you would handle team sharing, and the trade-offs you made
- **Application architecture** - how you structured the codebase, how the main process, preload bridge and UI communicate, and how you approached HTML/JSON parsing and the categorisation override system
- **Security** - how sign-in works, where the session is stored, how your API and RLS policies protect each user's data, and how you kept secrets out of the app
- **Packaging** - how the app is built into an installer
- **Testing** - E2E tests? Unit tests? How you validated parsing accuracy

### Code Repository

- [ ] Private GitHub repository
- [ ] Add `SauraPG72` and `s2dmad` as collaborators
- [ ] Tests
- [ ] README covering how to run and build the app, architecture decisions, which operating system your installer is for, and known limitations

---

## Sample Assessments

The following demonstrate the kind of analysis the tool should facilitate:

### A-T (Consumer - 3 Accounts)

**Account Summary:** ANZ Transaction, Bendigo Home Loan, QCCU Savings

| Question | Analysis |
|----------|----------|
| **Primary Income** | Monthly average: $7,375.30 from AUSTRALIAN COUNT (fortnightly pay cycle) |
| **Are declared expenses true?** | To be verified against application |
| **Any income subsidies not declared?** | No subsidies detected |
| **Signs of good conduct?** | Yes - consistent loan repayments being made |
| **Any debt collections?** | No debt collections found |
| **Bad account conduct?** | No gambling, no excessive withdrawals |
| **Arrears/knockbacks/days in negative?** | None |
| **Undeclared accounts?** | Account 64277377 receives transfers - possible additional savings |
| **Large payments to/from** | $6,530.00 transfer to SAV 64277377 - may be savings, not expense |
| **Recurring expenses** | FMC Direct Debit: $1,765.86/month, Yamaha Motor Finance: $276.30/month |

**Notes:** Finances spread across 3 banks. Credits from other accounts make reconciliation complex.

---

### H-C-PTY-LTD (Commercial - 1 Account)

**Account Summary:** Westpac Business One Plus - 180 days - 0 days negative - $102k available

| Question | Analysis |
|----------|----------|
| **Revenue Per Month** | Hume City Council: $180,294 avg pm, Downer: $10,431 avg pm, Other: $85,009 avg pm |
| **Genuine revenue (last 30 days)** | Hume City Council: $264,754, Downer: $6,479, Other: $1,738 |
| **Last Month Revenue** | Oct: Hume City Council $235,087, Downer $6,479, Other $3,388 |
| **Interbank transfers?** | Check RS Trading and Hume Contracting NAB transfers |
| **5-digit expenses** | Barling's Wood ($15k Jun, $10k Aug), Bluedog Fences ($18k Oct), Lokan Nominees ($13k Jun, $16k Aug), Rent $2,166 pm, Yard Rent $6,555 pm |
| **Most significant debtor** | HUME CITY COUNCIL |
| **Debt collections** | Recoveries Corp $1,888 (Jun), ARMA $484 (Jul) |
| **ATO payments** | $7,467 avg pm - latest $7,071 (Sep for Jun quarter) |
| **Owner drawings** | RS Trading transfers: $75k-$150k multiple payments, Hume Contracting NAB: $90k-$125k transfers |

---

### J-V-W (Consumer - 1 Account)

**Account Summary:** ING Transaction Account - 0 days in negative

| Question | Analysis |
|----------|----------|
| **Primary Income** | Comsol wages: $1,957.54, Centrelink Family Benefits: $1,673.96, Centrelink Pension: $1,084.99, Carers Benefit: $682.71, 2x DH: $167.03 |
| **Are declared expenses true?** | Yes |
| **Income subsidies not declared?** | No - opposite issue: Centrelink not visible in this account |
| **Signs of good conduct?** | Yes - strong savings behaviour |
| **Debt collections?** | None |
| **Bad account conduct?** | Minimal - $200 ATM (one-time $600) |
| **Arrears/days in negative?** | None |
| **Undeclared accounts?** | None found |
| **Large payments** | Adyen $18,835.95 (10 Dec) - requires explanation |
| **Non-SACC Loans** | Mortgage: $1,500, Plenti PL: $301.65 |

**Note:** Large Adyen payment needs explanation - could be e-commerce refund or business activity.

---

## Getting Started

1. Review sample data structure in `resources/statements/`
2. Build the HTML/JSON parser first as plain TypeScript with tests - validate against sample files
3. Set up an Electron app and get it building into an installer early
4. Set up Cloud SQL, Google Identity Platform and a Cloud Storage bucket on your provisioned Google Cloud project
5. Deploy a small API on Cloud Run and add Google sign-in from the desktop app
6. Build the transaction display and side-by-side view
7. Implement tagging and annotation system
8. Add deal management and status workflow
9. Build export functionality
10. Add team collaboration if time allows

---

## Questions to Consider

- How will you parse illion's HTML format reliably?
- How will you validate that an uploaded file is in the correct format?
- How will you handle statements with 10+ accounts visually?
- What's your matching algorithm for interbank transfers?
- How will you structure the database to support both tags and annotations efficiently?
- How would you handle team collaboration and concurrent edits?
- Should parsing happen in the desktop app or in your API — and why?
- What summary metrics are most valuable for loan assessment?
- How will you differentiate Consumer vs Commercial analysis workflows in the UI?
