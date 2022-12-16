# Intro

Saifu is an open-source financial ledger server that enables you build financial products easily.

## Use Cases

- Banking
- Digital wallets
- Card processing
- Brokerage systems


# How Saifu works

# Ledgers
Yes, ledgers are a common feature of financial systems, and are used to record and track transactions and balances for a particular entity. In a system such as Saifu, ledgers might be used to track the financial transactions and balances of customers, as well as other financial entities such as banks, businesses, or governments.

To create a new ledger in a system like Saifu, you would typically need to specify an identity for the ledger. This identity could be the customer's name or account number, or the name or identification number of another financial entity. The identity of the ledger is used to uniquely identify the ledger within the system, and to associate it with the appropriate transactions and balances.

Once a ledger has been created, all transactions and balances for that ledger can be recorded and tracked within the system. This might include recording and updating the balances for various accounts or financial assets, as well as tracking the flow of assets between accounts. By maintaining accurate and up-to-date ledgers, it is possible to track the financial activity of a particular entity and to ensure the integrity and accuracy of financial transactions.

# Balances
Balances are typically calculated for each  ledger, and represent the total amount of that asset that is available for use or transfer. Balances are typically updated every time a new transaction is recorded in the ledger, and can be used to track the flow of assets between accounts and to ensure that the ledger remains in balance.

### A balance consist of Balances three sub balances.

| Name | Description |
| ------ | ------ |
| Credit Balance | Credit balance holds the sum of all credit transactions recorded|
| Debit Balance | Debit balance holds the sum of all debit transactions recorded  |
| Balance | The actual Balance is calculated by summing the Credit Balance and Debit Balance|


### Computing Balances
Balances are calculated for very new transaction entry to a ledger.

A ledger can have multiple balances, depending on the types of accounts and assets that it tracks. For example, a ledger might have separate balances for different currencies.

### Example

**Sample Transaction Entries**

| LedgerID | Currency | Amount | DRCR 
| ------ | ------ | ------ | ------ |
| 1 | USD| 100.00| CR
| 1 | USD| 50.00| CR 
| 1 | NGN| 50,000.00| CR 
| 1 | NGN| 1,000.00| CR 
| 1 | GHS| 1,000.00| CR
| 1 | USD| 50.00| DR 
| 1 | BTC| 1| DR


**Computed Balances**

| LedgerID | BalanceID | Currency | Credit Balance | Debit Balance | Balance
| ------ | ------ | ------ | ------ | ------ | ------ |
| 1 | 1 | USD  | 150.00 | 50.00 | 100.00
| 1 | 2 | NGN  | 51,000.00 | 0.00 | 150.00
| 1 | 3 | GHS  | 1,000.00 | 0.00 | 150.00
| 1 | 4 | BTC  | 1 | 0.00 | 1


### Balance Multiplier
Multipliers are used to convert balance to it's lowest currency denomination. Balances are multiplied by the multiplier and the result is stored as the balance value in the database.

**Before multiplier is applied**

| BalanceID | Currency | Credit Balance | Debit Balance | Balance | Multiplier
| ------ | ------ | ------ | ------ | ------ | ------ |
| 1 | USD  | 150.00 | 0.00 | 150.00 | 100
| 1 | BTC  | 1 | 0 | 1 | 100000000


**After multiplier is applied**

| BalanceID | Currency | Credit Balance | Debit Balance | Balance | Multiplier
| ------ | ------ | ------ | ------ | ------ | ------ |
| 1 | USD  | 15000 | 0 | 15000 | 100
| 2 | BTC  | 100000000 | 0 | 1 | 100000000

### Grouping Balance
Group balances by using a common group identifier (such as a ```group_id```) can be a useful way to associate related balances together.

**For example, if you have a wallet application that enable a customer have multiple ```wallet balances``` you can use grouping to merge or fetch all balances associated with a customer**

Overall, grouping balances using a common group_id can be a useful way to manage and track related balances, and can help to make it easier to view and analyze balances in your system.


### Balance Properties

| Property | Description | Type |
| ------ | ------ | --- |
| id | Balance ID | string |
| ledger_id | The Ledger the balance belongs to | string |
| created | Timestamp of when the balance was created. | Time |
| currency | Balance currency | String
| balance | Derived from the summation of ```credit_balance``` and ```debit_balance``` | int64 |
| credit_balance | Credit Balance  | int64 |
| debit_balance |  Debit Balance | int64 |
| multiplier | Balance Multiplier | int64 |
| group | A group identifier | string |

# Transactions
Transactions record all ledger events. Transaction are recorded as either  ```Debit(DR)``` ```Credit(CR)```.


### Debit/Credit

```Debits``` and ```Credits``` are used to record all of the events that happen to a ledger, and to ensure that the ledger remains in balance. By using debits and credits, it is possible to track the movement of money between balances and to maintain an accurate record of financial transactions.

### Transaction Properties

| Property | Description | Type |
| ------ | ------ | --- |
| id | Transaction ID | string |
| amount | Transaction Amount| int64 |
| DRCR | Credit or Debit indicator| string |
| currency | Transaction currency | String
| ledger_id | The Ledger the transaction belongs to | string |
| balance_id | The balance the belongs to | string |
| status | The status of the transaction. Transaction status are grouped into ```Successful```, ```Pending```, ```Reversed``` | string |
| reference | Unique Transaction referecence | string |
| group | A group identifier | string |
| description | Transaction description | string |
| meta_data | Custom metadata | Object |

### Immutability
Transactions are immutable, this means that the records of the transaction cannot be altered or tampered with once they have been recorded. This is an important feature of transactions, as it ensures that the record of a transaction remains accurate and unchanged, even if the transaction itself is modified or reversed at a later time.

### Idempotency
Transactions are idempotent, "idempotent" means that the effect of a particular operation will be the same, no matter how many times it is performed. In other words, if a transaction is idempotent, then repeating the transaction multiple times will have the same result as performing it just once.

Idempotence is an important property of transactions, as it ensures that the outcome of the transaction is predictable and consistent, regardless of how many times it is performed. This can be particularly useful in situations where a transaction may be repeated due to network errors or other issues that may cause the transaction to fail.

**For example, consider a transaction that involves transferring money from one bank account to another. If the transaction is idempotent, then it will not matter how many times the transaction is repeated – the end result will always be the same. This helps to prevent unintended consequences, such as multiple transfers occurring or funds being mistakenly credited or debited multiple times.**

Saifu ensures Idempotency by leveraging ```refereces```. Every transaction is expected to have a unique reference. Saifu ensures no two transactions are stored with the same reference. This helps to ensure that the outcome of the transaction is consistent, regardless of how many times the transaction is performed.

### Grouping Transactions
Group transactions by using a common group identifier (such as a ```group_id```) can be a useful way to associate related transactions together. This can be particularly useful when dealing with transactions that have associated fees, as it allows you to easily track and manage the fees that are associated with a particular transaction.

**For example, if you have a system that processes financial transactions, you might use a ```group_id``` to link a main transaction with any associated fees. This would allow you to easily fetch all transactions that are associated with a given group, allowing you to view the main transaction and all associated fees in a single view.**

Using a group_id to link transactions can also be useful in other contexts, such as when dealing with transactions that are part of a larger process or workflow. By using a group_id, you can easily track and manage all of the transactions that are associated with a particular process, making it easier to track the progress of the process and identify any issues that may arise.

Overall, grouping transactions using a common group_id can be a useful way to manage and track related transactions, and can help to make it easier to view and analyze transactions in your system.

# Fault Tolerance
Fault tolerance is a key aspect of any system design, as it helps ensure that the system can continue to function even in the event of failures or errors

**By ```enabling fault tolerance in the config```, Saifu temporarily writes transactions to disk if they cannot be written to the database. This can help ensure that no transaction records are lost and that the system can continue to function even if the database experiences issues.**


# How To Install

## Option 1: Docker Image
```bash
$ docker run \
	-p 5005:5005 \
	--name saifu \
    --network=host \
	-v `pwd`/saifu.json:/saifu.json \
	docker.cloudsmith.io/saifu/saifu:latest
```

## Option 2: Building from source
To build saifu from source code, you need:
* Go [version 1.16 or greater](https://golang.org/doc/install).

```bash
$ git clone https://github.com/jerry-enebeli/saifu && cd saifu
$ make build
```

# Get Started with Saiffu
Saifu is a RESTFUL server. It exposes interaction with your Saifu server. The API exposes the following endpoints


## Create Config file ``saifu.json``

```json
{
  "port": "4100",
  "project_name": "MyWallet",
  "default_currency": "NGN",
  "data_source": {
    "name": "MONGO",
    "dns":""
  }
}
```

| Property | Description |
| ------ | ------ |
| port | Preferred port number for the server. default is  ```4300``` |
| project_name | Project Name. |
| default_currency |  The default currency for new transaction entries. This would be used if a currency is not passed when creating a new transaction record. |
| enable_wal | Enable write-ahead log. default is false. |
| data_source | Database of your choice.  |
| data_source.name | Name of preferred database. Saifu currently supports a couple of databases. |
| data_source.name | DNS of database|


### Supported Databases
| Data Base | Support |
| ------ | ------ |
| Postgres | ✅ |
| MYSQL | ✅ |
| MongoDB | ✅ |
| Redis | ✅ |


[comment]: <> (## Endpoints)

[comment]: <> (### Create ledger ```POST```)

[comment]: <> (```/ledgers```)

[comment]: <> (**Request**)

[comment]: <> (```json)

[comment]: <> ({)

[comment]: <> (  "id": "cu_ghjoipeysnsfu24")

[comment]: <> (})

[comment]: <> (```)

[comment]: <> (**Response**)

[comment]: <> (```json)

[comment]: <> ({)

[comment]: <> (  "id": "cu_ghjoipeysnsfu24")

[comment]: <> (})

[comment]: <> (```)

[comment]: <> (### Get Ledgers ```GET```)

[comment]: <> (```/ledgers```)

[comment]: <> (**Response**)

[comment]: <> (```json)

[comment]: <> ([{)

[comment]: <> (  "port": "4100",)

[comment]: <> (  "project_name": "MyWallet",)

[comment]: <> (  "default_currency": "NGN",)

[comment]: <> (  "data_source": {)

[comment]: <> (    "name": "MONGO",)

[comment]: <> (    "dns":"")

[comment]: <> (  })

[comment]: <> (}])

[comment]: <> (```)

[comment]: <> (### Get Ledger Balances ```GET```)

[comment]: <> (```/ledgers/balances/{ID}```)

[comment]: <> (**Response**)

[comment]: <> (```json)

[comment]: <> ({)

[comment]: <> (  "port": "4100",)

[comment]: <> (  "project_name": "MyWallet",)

[comment]: <> (  "default_currency": "NGN",)

[comment]: <> (  "data_source": {)

[comment]: <> (    "name": "MONGO",)

[comment]: <> (    "dns":"")

[comment]: <> (  })

[comment]: <> (})

[comment]: <> (```)

[comment]: <> (### Record Transaction ```POST```)

[comment]: <> (```/transactions```)

[comment]: <> (**Request**)

[comment]: <> (```json)

[comment]: <> ({)

[comment]: <> (  "port": "4100",)

[comment]: <> (  "project_name": "MyWallet",)

[comment]: <> (  "default_currency": "NGN",)

[comment]: <> (  "data_source": {)

[comment]: <> (    "name": "MONGO",)

[comment]: <> (    "dns":"")

[comment]: <> (  })

[comment]: <> (})

[comment]: <> (```)

[comment]: <> (**Response**)

[comment]: <> (```json)

[comment]: <> ({)

[comment]: <> (  "port": "4100",)

[comment]: <> (  "project_name": "MyWallet",)

[comment]: <> (  "default_currency": "NGN",)

[comment]: <> (  "data_source": {)

[comment]: <> (    "name": "MONGO",)

[comment]: <> (    "dns":"")

[comment]: <> (  })

[comment]: <> (})

[comment]: <> (```)


[comment]: <> (### Get Recorded Transactions ```GET```)

[comment]: <> (```/transactions```)

[comment]: <> (**Response**)

[comment]: <> (```json)

[comment]: <> ({)

[comment]: <> (  "port": "4100",)

[comment]: <> (  "project_name": "MyWallet",)

[comment]: <> (  "default_currency": "NGN",)

[comment]: <> (  "data_source": {)

[comment]: <> (    "name": "MONGO",)

[comment]: <> (    "dns":"")

[comment]: <> (  })

[comment]: <> (})

[comment]: <> (```)