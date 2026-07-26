# Credit Risk & Loan Portfolio Analysis — LendingClub (2013–2015)

**Live dashboard:** [https://public.tableau.com/views/LendingClubCreditRiskAnalysis_17850323817260/Dashboard1?:language=en-US&publish=yes&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link]

## Context
LendingClub is a peer-to-peer lender meaning investors fund loans, borrowers repay principal plus interest, and the lender profits only if enough loans are repaid to offset the ones that default. Assuming the role as a credit analyst, I set out to find where losses concentrate across the loan book and how approval criteria could reduce them without sacrificing too much lending volume.

## The question
*Which borrower segments drive our loan losses, and how should we adjust approval criteria to cut the default rate without killing loan volume?*

## Data
- Source: LendingClub public loan data — 2.26M loans total, scoped to **791,537 loans issued 2013–2015**.

## Data Cleaning

I labeled Charged Off/ Default/Late (31–120 days) as *bad* loans and Fully Paid as *good*, and excluded still-open (Current) loans
59,929 of them, since an active loan hasn't yet had its chance to default. Counting them would have understated risk.

The raw data required real cleaning before analysis:
- `int_rate` and `revol_util` stored as text with `%` signs → stripped and cast to numeric.
- `term` stored as " 36 months" → parsed to an integer.
- `emp_length` inconsistent text ("< 1 year", "10+ years", blanks) → mapped to 0–10, blanks preserved as NULL.
- `issue_d` stored as "Dec-2011" → parsed to a proper date.
- Identified and excluded 4 malformed rows caused by delimiter issues in free-text fields (a stray date and blank values had shifted into the `loan_status` column).
- Dropped mostly-empty joint-applicant, hardship, and settlement columns as out of scope.

## Findings

**1. Loan grade drives risk — a clean 9x ramp.**
Default rates climb steadily from **5.4% for grade A to 48.6% for grade G**. Each grade step roughly doubles the odds of default relative to the top grades. LendingClub's grading system genuinely sorts risk.

**2. Loan purpose predicts default, but volume matters more than rate.**
Small-business loans default most often (27.7%), followed by moving (22.5%) and house (21.9%); car and wedding loans are safest (~15%). But the highest-*rate* categories are tiny in volume. **Debt consolidation, a moderate 20.3% default rate, represents 436,640 loans, the single largest slice of the book.** The real dollar loss exposure sits in debt consolidation's scale, not small business's rate.

3. Losses concentrate in the middle grades, not the worst ones
Total net loss across all defaulted loans was **~$1.28B**. The largest dollar losses fall on grades **C ($350M), D ($318M), and E ($253M)** — the high-volume middle of the book — not on F and G, which despite the scariest default rates account for only ~$138M combined.

## Recommendation
A scenario model quantified the tradeoff of declining the worst grades:

| Policy | Losses removed | Volume given up |
|--------|---------------|-----------------|
| Decline F, G | 10.8% | 9.1% |
| Decline E, F, G | 30.6% | 26.8% |
| Decline D, E, F, G | 55.5% | 50.8% |

While the percentages favor cutting deeper (the loss-removed vs. volume-given-up gap widens at each step), declining down to grade D means abandoning **half the entire lending business** to remove 55% of losses — commercially unviable, since interest from performing loans is what makes lending profitable.
Decline grades F and G outright — a clean win removing 10.8% of losses for only 9.1% of volume. Rather than cutting grades D and E, **re-price or tighten underwriting** on them, since they carry the largest dollar losses but too much volume to abandon.

## Limitations
- The expected-loss model measures loss avoided but not interest earned, so it can't value the profitable loans a cut would forgo — the real cutoff depends on net margin per grade, which would require revenue data.
- Analysis is limited to 2013–2015 issuance; more recent vintages or macroeconomic variables could shift the picture.
- The next step is a simple predictive model (logistic regression) to score default probability at the individual-loan level rather than the segment level.
