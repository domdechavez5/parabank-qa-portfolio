# ParaBank Test Plan (v0.1)

Last updated: October 24, 2026

## 1. Introduction

This plan describes how the ParaBank demo banking application will be tested. ParaBank is a sample online banking app from Parasoft, run locally. The testing is a portfolio project that covers manual testing first, then API testing, data validation, and automation in later phases. This document covers the manual testing phase.

There is no formal requirements document for ParaBank. Expected behavior is inferred from the page text, the Services and OpenAPI documentation, and common banking rules. Where the correct behavior is unclear, the question is recorded in Section 9 instead of being guessed.

## 2. Scope

### In scope
- Functional features F-01 to F-11 in the feature inventory: Register, Login, Open New Account, Accounts Overview, Transfer Funds, Bill Pay, Find Transactions, Update Contact Info, Request Loan, Contact Us, Log Out.
- Balance consistency across features (for example, transfers, bill payments, new accounts, and loans keep account balances and the overview total consistent).
- Behavior on invalid input in the money-related features.

### Out of scope
- External sites (Products, Locations pages): not part of the application.
- Bookstore services: not part of the banking application.
- Load and performance testing: not part of the manual phase.
- Security penetration testing: outside the scope of this phase. Obvious security observations are recorded but not exploited.
- Admin Page settings: tested only at their default values. Settings are not changed during this phase.

## 3. Test approach

### Test levels
- **Functional:** each feature is checked against its visible behavior and the documented rules.
- **Integration:** actions that affect more than one account or page are checked end to end (for example, a transfer changes both balances, and the change appears in Accounts Overview and Find Transactions).
- **System integration (SIT):** UI actions are compared with the results of the matching service calls, in the API testing phase.
- **Regression:** the high-priority cases are re-run after any environment change (restart, setting change) to confirm results are repeatable, and repeatedly through the automated suite later in the project. ParaBank is a third-party application and will not be modified, so defects found are documented but not fixed or retested against a fix.
- **User acceptance (UAT):** 5-6 scenarios written from a customer's point of view, in plain business language (for example, "pay a bill from my checking account and confirm it shows in my history").

### Test design techniques
- **Boundary value analysis:** amounts at and around limits, such as 0, 0.01, the account balance, and the balance plus 0.01 for Transfer Funds and Bill Pay; down payment compared with account balance for Request Loan; the $100.00 minimum for Open New Account.
- **Equivalence partitioning:** valid and invalid classes for inputs such as amounts (positive, zero, negative, non-numeric, too many decimals), date formats in Find Transactions, and account types.
- **Decision tables:** Request Loan approval, based on loan amount, down payment, available funds, and the loan processor setting.
- **State transition:** account balance states (positive, zero, negative) and session states (logged out, logged in, logged out again).
- **Exploratory testing:** time-boxed sessions with a stated goal, for areas the scripted cases don't cover.

## 4. Risks and priorities

| ID | Risk | Related items | Priority |
|---|---|---|---|
| R-01 | Transfers and bill payments accept negative amounts and amounts above the balance, which could allow unauthorized overdrafts or reversed payments | F-05, F-06, EO-01, EO-02 | High |
| R-02 | Balances or the overview total become inconsistent after transfers, bill payments, new accounts, or loans | F-03, F-04, F-05, F-06, F-09 | High |
| R-03 | Loan approval rules are unclear, depend on admin settings, and behave inconsistently with other features | F-09, EO-05 | High |
| R-04 | Invalid amounts (too many decimals, empty, non-numeric) cause a generic internal error, and the error persists when moving to other sections, leaving the site not working properly | F-05, F-06, F-09, EO-03, EO-04 | High |
| R-05 | Find Transactions returns results that don't match the selected account | F-07, EO-06 | Medium |
| R-06 | Personal data fields accept any text, so invalid data may be stored | F-01, F-08, EO-08 | Medium |
| R-07 | Session handling: no lockout rule seen for failed logins, and page access after logout is unverified | F-02, F-11 | Medium |
| R-08 | Test results are not repeatable without a reset, because user-created data is lost on restart | Environment notes | Low |

Priority reflects the potential impact if the feature fails (especially anything involving money) and how likely a failure is, based on early observations.

## 5. Test environment and data

- Application: ParaBank run locally on Windows 11 (Java 21, Maven), at http://localhost:8080/parabank, default Admin settings.
- A restart resets the application to a known baseline: built-in user `john` with seeded accounts, and all other data is removed.
- Read-only checks use `john`, whose seeded data is the same after every restart. Tests that change data use a freshly registered user, so each test starts from a known state and doesn't depend on changes made by earlier tests. Using `john` for a data-changing test is possible if ParaBank is restarted afterward to reset his data.
- All data is made up. No real personal or financial information is used.
- Details are in `environment-notes.md`.

## 6. Entry and exit criteria

**Entry (testing can start when):**
- The application runs on default settings and the baseline data is confirmed.
- The feature inventory and this plan are committed.
- The test case template is ready.

**Exit (testing is finished when):**
- All high-priority test cases have been executed.
- At least 90% of planned test cases have been executed (pass, fail, and blocked all count as executed). Unexecuted cases are listed with the reason.
- All defects found are logged and triaged.
- No Critical defect is left unresolved without a recorded explanation.
- The test summary report is written.

## 7. Defect management

- Defects are logged in a Jira project. Each report includes a title, steps to reproduce, expected result, actual result, environment details, and evidence (screenshot or log excerpt).
- ParaBank will not be modified, so defects are documented and not retested against a fix.
- Severity: **Critical** (money handled wrongly or the app is unusable), **Major** (a feature fails or gives a wrong result), **Minor** (cosmetic or low impact).
- Priority (High, Medium, Low) is set separately and reflects how urgently a fix is needed.
- Early observations (EO items) are only logged as defects after they have been reproduced with repeatable steps.

## 8. Deliverables

- Test cases (40-60) with a traceability matrix linking features to cases
- Defect reports for genuine, reproduced defects
- UAT scenarios
- Test summary report with a release-readiness recommendation
- Screenshots and other execution evidence

## 9. Open questions

1. What is the intended rule when a transfer or bill payment exceeds the available balance, and for negative amounts?
2. What are the approval rules for Request Loan under each Loan Processor mode and the threshold setting?
3. How should the new loan account appear in the Accounts Overview total?
4. Why do seeded accounts 12345 and 12678 have negative balances with $0.00 available?
5. What is the intended behavior for amounts with more than two decimals?
6. Should the account selector in Find Transactions limit the results in every search mode?
7. What format rules should apply to SSN, phone, and zip code?
8. Does Contact Us actually send or store the message?
