# ParaBank Feature Inventory

Last updated: October 22, 2026

Application: ParaBank, local install (see environment-notes.md). All data used is made up. Admin Page settings were read only and are at their defaults.

How to read this document:
- Section 1 lists the functional features that will be tested.
- Section 2 lists supporting and technical pages that are not customer features.
- Section 3 lists early observations from exploring the app. These are leads to verify later, not confirmed defects.
- Section 4 summarizes the Services and Admin Page contents for reference.
- Pages that open external websites, and the Bookstore services (not part of the banking application), are out of scope.

## 1. Functional features

| ID | Feature | What it does | Inputs and visible rules | Money moves? | Questions / ambiguities |
|---|---|---|---|---|---|
| F-01 | Register | Creates a new customer account | Fields: First Name, Last Name, Address, City, State, Zip Code, Phone #, SSN, Username, Password, Confirm. Text fields accept any characters with a minimum length of 1. | No | Are there format rules for SSN, phone, and zip code? Is a duplicate username rejected within the same session? Is password strength enforced? |
| F-02 | Login | Logs in a registered customer | Accepts registered users and the built-in john / demo. A failed login shows "The username and password could not be verified." | No | Is there any lockout after repeated failed logins? |
| F-03 | Open New Account | Opens a new checking or savings account | Dropdown for account type (CHECKING, SAVINGS). Dropdown to choose the existing account that funds the new account. Page text states a minimum of $100.00 must be deposited. The Accounts Overview total was the same before and after opening an account, consistent with the deposit being moved from the funding account. | Yes (from the chosen existing account to the new account) | Is the new account's starting balance always $100.00? Does the Admin Page setting "Min. Balance" control the $100.00 minimum? What is the intended rule when the funding account has less than $100.00? (see EO-10) |
| F-04 | Accounts Overview | Lists all accounts with balance and available amount, plus a total balance | Clicking an account opens Account Details (account number, type, balance, available) with Account Activity, filterable by Activity Period and Type. Clicking a transaction opens Transaction Details (ID, date, description, type, amount). | No | Why do seeded accounts 12345 and 12678 have negative balances with $0.00 available? What does the asterisk on "Balance*" mean? |
| F-05 | Transfer Funds | Moves an amount between two of the customer's existing accounts | Accounts are chosen from existing accounts. Amount accepts numbers with up to two decimals. | Yes | What is the intended rule for amounts above the available balance? For negative amounts? For more than two decimals? |
| F-06 | Bill Pay | Sends a payment from one of the customer's accounts to a payee | Payee Name, Address, City, State, Zip Code, and Phone # accept any characters with a minimum length of 1. Account # and Verify Account # accept numbers only and must match. From account is chosen from existing accounts. Amount accepts numbers only. | Yes | What are the intended rules for amount limits and negative values? Is the payee account validated against anything? |
| F-07 | Find Transactions | Searches an account's transactions | Four search modes: by Transaction ID (numbers only), by Date (MM-DD-YYYY), by Date Range (MM-DD-YYYY), by Amount (numbers only). Account selector dropdown. | No | Should the account selector limit results in every search mode? (see EO-06) |
| F-08 | Update Contact Info | Edits the customer's profile details | Fields: First Name, Last Name, Address, City, State, Zip Code, Phone #. Text fields accept any characters with a minimum length of 1. | No | Where are the updated details displayed? Are there format rules for phone and zip code? |
| F-09 | Request Loan | Applies for a loan | Fields: Loan Amount (numbers only), Down Payment (numbers only), and a From account selector. On approval, a new account is created whose balance equals the loan amount, and the down payment is deducted from the selected From account (new From balance = previous balance minus down payment). Messages seen: "We cannot grant a loan in that amount with your available funds" (for example, a negative loan amount) and "You do not have sufficient funds for the given down payment." (shown when the down payment exceeds the From account's balance). The Admin Page has a Loan Processor setting (Available Funds by default). | Yes | What are the approval rules under each Loan Processor mode (Available Funds, Down Payment, Combined) and the 20% threshold setting? How should the new loan account appear in the Accounts Overview total, given its balance equals the loan amount? What is the intended behavior for a loan amount smaller than the down payment, or a negative down payment? Why does the loan reject a down payment above the account balance when Transfer Funds, Bill Pay, and Open New Account accept amounts above the balance? |
| F-10 | Contact Us | Customer care page with an email support form | Fields: Name, Email, Phone, Message. Text fields accept any characters with a minimum length of 1. | No | Does the form actually send or store the message? Is the email field validated? |
| F-11 | Log Out | Logs the user out and returns to the Home page with the Customer Login section | None | No | Can the previous pages be reached with the browser Back button after logging out? |

## 2. Supporting and technical pages

| ID | Page | What it is | Notes |
|---|---|---|---|
| P-01 | Home | Landing page with links to services | Several links (Withdraw Funds, Transfer Funds, Check Balances, Make Deposits, Bill Pay, Account History, Transfer Funds) open XML pages. See EO-07. |
| P-02 | About Us | Describes the Parasoft demo website | Static content. |
| P-03 | Products | Link to Parasoft's products website | External site, out of scope. |
| P-04 | Locations | Link to Parasoft's solutions website | External site, out of scope. |
| P-05 | Site Map | Lists the site's pages and links | Useful as a checklist that every page was covered. |
| T-01 | Services | Lists the SOAP and RESTful services (Bookstore and ParaBank) | Not a customer feature. Reference for API testing. See Section 4. |
| T-02 | Admin Page | Administration page for environment settings | Environment control, not a customer feature. Contents summarized in Section 4. No settings changed. |

## 3. Early observations (to verify during test execution)

| ID | Area | Observation | Status |
|---|---|---|---|
| EO-01 | Transfer Funds, Bill Pay | Negative amounts are accepted. | To verify |
| EO-02 | Transfer Funds, Bill Pay | Amounts greater than the account balance are accepted. | To verify |
| EO-03 | Transfer Funds, Bill Pay, Request Loan | An amount with more than two decimals shows "An internal error has occurred and has been logged." | To verify |
| EO-04 | Request Loan | Empty or non-numeric input shows the same generic internal error message. | To verify |
| EO-05 | Request Loan | A loan amount smaller than the down payment is accepted, and a negative down payment is accepted. | To verify |
| EO-06 | Find Transactions | The account selector does not always limit results; a Transaction ID search showed transactions from other accounts. | To verify |
| EO-07 | Home | Several service links open XML pages. May be intended, since they may point to service descriptions. | To verify |
| EO-08 | Register, Bill Pay, Update Contact Info, Contact Us | Text fields accept any characters with a minimum length of 1; no format checks seen for SSN, phone, or zip code. | To verify |
| EO-09 | Open New Account | Resolved: the Accounts Overview total is unchanged after opening an account, so the $100.00 deposit is moved from the funding account, not created. | Resolved |
| EO-10 | Open New Account | An account can be opened when the funding account has less than $100.00, and the funding account balance goes negative. | To verify |

These are leads only. Each one is confirmed with repeatable steps during test execution, and only then is a defect report written.

## 4. Technical reference

### 4.1 Services page (http://localhost:8080/parabank, Services link)

- **Bookstore SOAP services** (out of scope): several Bookstore endpoints, some secured with WS-Security (username token, signature, encryption). Not part of the banking application.
- **ParaBank SOAP services:** LoanProcessor service (requestLoan) and ParaBank service. Endpoints under /parabank/services/.
- **ParaBank service methods:** getAccount, getAccounts, getCustomer, login, createAccount, deposit, withdraw, transfer, requestLoan, updateCustomer, getTransaction, getTransactions, getTransactionsOnDate, getTransactionsByToFromDate, getTransactionsByAmount, position methods (buyPosition, sellPosition, getPosition, getPositions, getPositionHistory), and administrative methods (initializeDB, cleanDB, setParameter, startupJmsListener, shutdownJmsListener).
- **RESTful services:** endpoint /parabank/services/bank, with a WADL description and an OpenAPI page at /parabank/api-docs/index.html.
- **Relevance:** the banking methods and the REST endpoints are candidates for the API testing work. The administrative methods change the environment and need care.

### 4.2 Admin Page (read only; no settings changed)

| Section | Contents |
|---|---|
| Database | Initialize and Clean buttons |
| JMS Service | Status: Running, with a Shutdown button |
| Data Access Mode | SOAP, REST (XML), REST (JSON), JDBC (default) |
| Web Service | ParaBank service endpoints (SOAP and REST) and LoanProcessor endpoint; leave blank to use the default |
| Application Settings | Init. Balance (default 515.50), Min. Balance (default 100.00), Loan Provider (JMS, Web Service as default, Local), Loan Processor (Available Funds as default, Down Payment, Combined), Threshold (default 20%) |

Notes:
- Testing is done with the default settings; any change will be documented here.
- The Loan Processor, Loan Provider, and Threshold settings probably affect Request Loan behavior, so loan test results depend on them.
- Init. Balance and Min. Balance may relate to Open New Account (see F-03).