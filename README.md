# Credit Risk & Loan Portfolio Analysis: LendingClub (2013-2015)

**Live dashboard:** https://public.tableau.com/views/LendingClubCreditRiskAnalysis_17850323817260/Dashboard1

## Context
LendingClub is a peer-to-peer lender, meaning investors fund loans, borrowers repay principal plus interest, and the lender only profits if enough loans are repaid to offset the ones that default. I took on the role of a credit analyst to find where losses concentrate across the loan book and how approval criteria could reduce them without giving up too much lending volume.

## The question
Which borrower segments drive our loan losses, and how should we adjust approval criteria to cut the default rate without killing loan volume?

## Source
LendingClub public loan data from Kaggle. The full dataset holds 2.26M loans, which I scoped down to the 791,537 loans issued between 2013 and 2015.

## Data cleaning
I labeled Charged Off, Default, and Late (31-120 days) loans as bad, and Fully Paid loans as good. I excluded the 59,929 still-open (Current) loans, since an active loan hasn't had its chance to default yet and counting it would understate the risk.

The raw data needed real cleaning before analysis:
- `int_rate` and `revol_util` were stored as text with `%` signs, so I stripped the sign and cast them to numbers.
- `term` was stored as " 36 months", so I parsed it down to an integer.
- `emp_length` was inconsistent text ("< 1 year", "10+ years", blanks), so I mapped it to 0-10 and kept blanks as NULL.
- `issue_d` was stored as "Dec-2011", so I parsed it into a proper date.
- I found and excluded 4 malformed rows 
- I dropped the mostly-empty joint-applicant, hardship, and settlement columns since they were out of scope.

## Findings

**1. Loan grade drives risk, a clean 9x ramp.**
Default rates climb steadily from 5.4% for grade A to 48.6% for grade G. Each grade step roughly doubles the odds of default relative to the top grades. LendingClub's grading system genuinely sorts risk.

**2. Loan purpose predicts default, but volume matters more than rate.**
Small-business loans default most often (27.7%), followed by moving (22.5%) and house (21.9%), while car and wedding loans are the safest (around 15%). But the highest-rate categories are tiny in volume. Debt consolidation, at a moderate 20.3% default rate, represents 436,640 loans meaning the real loss sits in debt consolidation's scale, not small business's rate.

**3. Losses concentrate in the middle grades, not the worst ones.**
Total net loss across all defaulted loans was about $1.28B. The largest dollar losses fall on grades C ($350M), D ($318M), and E ($253M), the high-volume middle of the book, not on F and G. Despite their bigger default rates, F and G only account for around $138M combined.

## Recommendation
I built a scenario model to quantify the tradeoff of declining the worst grades:

| Policy | Losses removed | Volume given up |
|--------|---------------|-----------------|
| Decline F, G | 10.8% | 9.1% |
| Decline E, F, G | 30.6% | 26.8% |
| Decline D, E, F, G | 55.5% | 50.8% |

The percentages favor cutting deeper, since the gap between losses removed and volume given up widens at each step. But declining all the way down to grade D means abandoning half the entire lending business to remove 55% of losses, which isn't commercially viable.

My recommendation is to decline grades F and G outright, removing 10.8% of losses for only 9.1% of volume. Instead of cutting grades D and E, I'd re-price them, since they carry the largest dollar losses but too much volume to leave behind.

## Limitations
- The expected-loss model measures loss avoided but not interest earned, so it can't value the profitable loans a cut would forgo. The real cutoff depends on net margin per grade, which would need revenue data.
- The analysis only covers 2013-2015 issuance. More recent vintages or macroeconomic shifts could change the picture.
- The next step would be a simple predictive model (logistic regression) to score default probability at the individual-loan level rather than the segment level.
