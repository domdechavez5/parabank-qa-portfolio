# Test Environment Notes

Last updated: October 22, 2026

## 1. Environment

| Item | Value |
|---|---|
| Operating system | Windows 11 |
| Java | Temurin 21.0.12.1 |
| Maven | 3.10.0 |
| Application | ParaBank (parasoft/parabank on GitHub, master branch), run locally |
| URL | http://localhost:8080/parabank |
| Browser | Mozilla Firefox 153.0.1 (64-bit) |
| Start command | `mvn -DskipTests clean package cargo:run` |
| Date set up | October 20, 2026 |

## 2. Starting, stopping, and resetting

- **Start:** run the start command from the `parabank` folder. The app is ready when the console stops scrolling and the URL loads.
- **Stop:** pressing Ctrl+C does not stop the application; the port stays in use.
  Command used instead (stops whatever is listening on port 8080):
  `Get-NetTCPConnection -LocalPort 8080 -State Listen -ErrorAction SilentlyContinue | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }`
  Verification: the URL no longer loads.
- **Reset to baseline:** stop, then start again. No other action is needed.

## 3. Data behavior across restarts

| Test | What I did | Result |
|---|---|---|
| A: restart without rebuild | Logged in with a user I had registered before the restart | Failed: "The username and password could not be verified." |
| B: restart with clean rebuild | Same check | Same result |
| Re-registration | Registered the same username again | Accepted, so the earlier user no longer existed |
| Built-in customer | Logged in as `john` / `demo` after each restart (2 restarts) | Works every time; same accounts and balances |
| State change as john | Transferred $10.45 from account 12456 to 12900, then restarted | Balances returned to the original values |

**Conclusion:** user-created data and changes to seeded data are lost on restart, and the seeded john data is re-created. Every restart returns the application to a known baseline.

## 4. Baseline data: customer john (present after every start)

| Account | Balance | Available amount | Type |
|---|---|---|---|
| 12345 | -$2,300.00 | $0.00 | CHECKING |
| 12456 | $10.45 | $10.45 | CHECKING |
| 12567 | $100.00 | $100.00 | CHECKING |
| 12678 | -$100.00 | $0.00 | SAVINGS |
| 12789 | $100.00 | $100.00 | CHECKING |
| 12900 | $0.00 | $0.00 | CHECKING |
| 13011 | $100.00 | $100.00 | CHECKING |
| 13122 | $1,100.00 | $1,100.00 | CHECKING |
| 13233 | $100.00 | $100.00 | CHECKING |
| 13344 | $1,231.10 | $1,231.10 | SAVINGS |
| 54321 | $1,351.12 | $1,351.12 | CHECKING |
| **Total** | **$1,692.67** | | |

Source: Accounts Overview page, customer id 12212 (from console log).
Notes:
- Two seeded accounts have negative balances (12345 and 12678), and their available amount shows $0.00.
- The Balance column header carries an asterisk. Footnote on the page: *Balance includes deposits that may be subject to holds.

## 5. Evidence: state change and reset

Transfer made as john: $10.45 from account 12456 to account 12900.

| Account | Baseline | After transfer | After restart |
|---|---|---|---|
| 12456 | $10.45 | $0.00 | $10.45 |
| 12900 | $0.00 | $10.45 | $0.00 |

Overview total after the transfer: $1,692.67

## 6. Observations

| ID | Observation | Notes |
|---|---|---|
| OBS-01 | Seeded accounts 12345 and 12678 have negative balances with $0.00 available. | Not yet known whether this is intended (for example, credit or loan-type accounts). Carried to open questions in the feature inventory; not a defect yet. |
| OBS-02 | The console log label reads "cutomer=" instead of "customer=". | Cosmetic, server-side only. Very low priority. |

## 7. Implications for testing

- Each test session starts from a known baseline, created by restarting the application.
- Read-only checks can use john, since his data is identical every start.
- Tests that change data should use a freshly registered user or accept that john's data resets on restart.
- Tests are not repeatable across sessions unless setup is repeated; automated tests must create their own user and accounts in a setup step.
- The seeded negative-balance accounts are useful test data for checks on overdrafts, transfers from accounts with no available funds, and balance display.