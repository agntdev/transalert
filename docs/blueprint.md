# Personal Finance Tracker — Bot specification

**Archetype:** finance

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot that securely imports and categorizes transactions from multiple accounts in the same currency, notifying the owner instantly for every transaction. It supports manual entry, category overrides, and time-based summaries.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual user

## Success criteria

- Owner receives instant Telegram notifications for every new transaction
- Transactions are auto-categorized with editable overrides
- Manual transaction entry works via /add command
- Summary commands /today /week /month show accurate totals

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **/add** (command, actor: user, command: /add) — Manually add a transaction with amount, category, account, and date
- **/today** (command, actor: user, command: /today) — Show today's transaction summary by category and account
- **/week** (command, actor: user, command: /week) — Show this week's transaction summary by category and account
- **/month** (command, actor: user, command: /month) — Show this month's transaction summary by category and account
- **Edit category** (button, actor: user, callback: transaction:edit_category) — Change the category of a transaction
  - inputs: transaction_id, category
  - outputs: updated transaction record
- **Ignore** (button, actor: user, callback: transaction:ignore) — Mark a transaction as ignored
  - inputs: transaction_id
  - outputs: updated transaction status
- **Show details** (button, actor: user, callback: transaction:show_details) — View full transaction details
  - inputs: transaction_id
  - outputs: transaction details message

## Flows

### Setup
_Trigger:_ /start

1. Display welcome message
2. Request bank credentials securely
3. Store encrypted credentials
4. Confirm account addition

_Data touched:_ User, Bank Account

### Transaction Notification
_Trigger:_ New transaction detected

1. Send Telegram notification with transaction details
2. Display [Edit category] [Ignore] [Show details] buttons

_Data touched:_ Transaction

### Edit Category
_Trigger:_ transaction:edit_category

1. Display category selection menu
2. Save category override
3. Confirm change

_Data touched:_ Transaction

### Manual Add
_Trigger:_ /add

1. Prompt for amount
2. Prompt for category
3. Prompt for account
4. Prompt for date
5. Create new transaction record

_Data touched:_ Transaction

### Summary Commands
_Trigger:_ /today /week /month

1. Calculate totals by category and account
2. Format summary message
3. Send summary to user

_Data touched:_ Transaction

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Single owner/admin account
  - fields: telegram_id, currency
- **Bank Account** _(retention: persistent)_ — Bank account metadata with masked number
  - fields: bank_name, account_number_masked, credentials_encrypted
- **Transaction** _(retention: persistent)_ — Individual transaction record with categorization
  - fields: amount, date_time, merchant, raw_description, auto_category, edited_category, account_id, ignored
- **Category List** _(retention: persistent)_ — Predefined categories + 'Other'
  - fields: category_name

## Integrations

- **Telegram** (required) — Bot API messaging
- **Bank APIs** (required) — Transaction import
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Add/remove bank accounts
- Edit transaction categories
- View transaction history
- View summary reports

## Notifications

- Instant Telegram DM for every new transaction
- Summary reports on demand

## Permissions & privacy

- Bank credentials stored encrypted
- Only owner has access to data
- Transactions retained until deleted

## Edge cases

- Failed bank API connection
- Duplicate transaction detection
- Invalid category input during manual add

## Required tests

- End-to-end transaction notification flow
- Manual transaction addition and categorization
- Summary command accuracy

## Assumptions

- Owner will provide valid bank credentials
- Bank APIs support live transaction imports
- Single currency mode is sufficient for owner's needs
