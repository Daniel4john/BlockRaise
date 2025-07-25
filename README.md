

# BlockRaise - Clarity Crowdfunding Smart Contract

A smart contract for decentralized crowdfunding on the [Stacks blockchain](https://www.stacks.co/), written in [Clarity](https://docs.stacks.co/docs/clarity-language/overview/). This contract allows users to create fundraising campaigns, contribute STX tokens, claim funds, get refunds, view contributions, and manage campaigns transparently.

---

## 🚀 Features

* ✅ Campaign creation with funding goals and deadlines
* 💸 STX contributions from any user
* ⏳ Automatic deadline enforcement
* 📈 Campaign status tracking
* 🔒 Owner-only fund claiming (if goal met)
* 💰 Refunds for contributors (if goal not met)
* 🧾 Contributor history view
* ❌ Campaign cancellation (only if no contributions)

---

## 📄 Contract Overview

### Constants

| Constant                    | Value       | Description                                    |
| --------------------------- | ----------- | ---------------------------------------------- |
| `contract-owner`            | `tx-sender` | The deploying address                          |
| Various `ERR-...` constants | `err u1xx`  | Standardized error codes for contract failures |

---

## 📂 Data Structures

### `campaigns` (map)

Stores campaign data by `campaign-id`.

```clojure
{ 
  campaign-id: uint 
} => {
  owner: principal,
  goal: uint,
  deadline: uint,
  total-raised: uint,
  claimed: bool,
  status: (string-ascii 20),
  description: (optional (string-ascii 500))
}
```

### `contributions` (map)

Tracks contributions by campaign and contributor.

```clojure
{ campaign-id: uint, contributor: principal } => {
  amount: uint,
  timestamp: uint
}
```

### `campaign-counter` (data-var)

An auto-incrementing counter for new campaign IDs.

---

## 📤 Public Functions

### `create-campaign(goal uint, deadline uint) → (ok campaign-id)`

Creates a new campaign.

* 🔐 Owner: Any user
* ❗ Fails if:

  * `goal <= 0`
  * `deadline` is in the past
  * Campaign ID already exists

---

### `contribute(campaign-id uint, amount uint) → (ok true)`

Allows users to fund a campaign.

* 💸 Transfers `amount` STX to contract
* ❗ Fails if:

  * Invalid campaign
  * Amount is zero or negative
  * Deadline passed

---

### `claim-funds(campaign-id uint) → (ok true)`

Campaign owner can claim funds if the goal is met.

* 🔐 Only owner can call
* ❗ Fails if:

  * Not authorized
  * Goal not met
  * Already claimed

---

### `refund(campaign-id uint) → (ok true)`

Allows contributors to get refunds if the campaign failed.

* 🔁 Refunds STX back to sender
* ❗ Fails if:

  * Not a contributor
  * Deadline not passed
  * Goal was met

---

### `update-deadline(campaign-id uint, new-deadline uint) → (ok true)`

Allows the owner to extend the deadline.

* 🔐 Only campaign owner
* ❗ Fails if:

  * New deadline is in the past
  * New deadline is earlier than current one

---

### `cancel-campaign(campaign-id uint) → (ok true)`

Allows the owner to cancel a campaign **only if there are no contributions**.

* ❗ Fails if:

  * Unauthorized caller
  * Contributions already made

---

## 📚 Read-Only Functions

### `get-campaign-status(campaign-id uint) → (ok campaign-info)`

Returns campaign progress and metadata:

```clojure
{
  owner: principal,
  goal: uint,
  total-raised: uint,
  remaining-blocks: int,
  progress-percentage: uint,
  is-active: bool,
  is-successful: bool,
  is-claimed: bool
}
```

---

### `get-contribution-history(campaign-id uint, contributor principal) → (ok contribution)`

Returns contribution data for a specific user.

```clojure
{
  amount: uint,
  timestamp: uint,
  campaign-id: uint
}
```

---

## 🧪 Error Codes

| Error Code | Description                         |
| ---------- | ----------------------------------- |
| `u100`     | Not authorized                      |
| `u101`     | Campaign already exists             |
| `u102`     | Invalid amount                      |
| `u103`     | Deadline passed                     |
| `u104`     | Goal not met                        |
| `u105`     | Funds already claimed               |
| `u106`     | Invalid campaign                    |
| `u107`     | Invalid deadline                    |
| `u108`     | No contribution found               |
| `u109`     | Active campaign cannot be cancelled |

---
