google-measurement-protocol
===========================

A Python implementation of Google Analytics Measurement Protocol

Transaction handling depends on the `prices` library.


Generating a client ID
----------------------

Google strongly encourages using UUID version 4 as unique user handles.
It's up to you to generate and persist the ID between actions, just make
sure that all actions performed by the same user are reported using the
same client ID.

```python
import uuid

client_id = uuid.uuid4()
```


Reporting a page view
---------------------

There are two ways to construct a `PageView` object:
```python
PageView(path[, host_name=None][, title=None][, referrer=None])
```
```python
PageView(location='http://example.com/my-page/?foo=1'[, title=None][, referrer=None])
```

Example:
```python
from google_measurement_protocol import PageView, report

view = PageView(path='/my-page/', title='My Page', referrer='http://example.com/')
report('UA-123456-1', client_id, view)
```


Reporting an event
------------------

Use the `Event` object:
```python
Event('category', 'action'[, label=None][, value=None])
```

Example:
```python
from google_measurement_protocol import Event, report

event = Event('profile', 'user_registered')
report('UA-123456-1', client_id, event)
```


Reporting a transaction
-----------------------

First create `Item`s to describe the contents of the transaction:
```python
Item(name, unit_price[, quantity=None][, item_id=None])
```

Then the `Transaction` itself:
```python
Transaction(transaction_id, items[, revenue=None][, shipping=None][, affiliation=None])
```
If `revenue` is given, it will override the total that is otherwise calculated
from items and shipping.

Example:
```python
from google_measurement_protocol import Item, report, Transaction
from prices import Money, TaxedMoney

transaction_id = '0001'  # any string should do
items = [
    Item(
        'My awesome product', TaxedMoney(
            net=Money(90, currency='EUR'), gross=Money(100, currency='EUR')),
            quantity=2),
    Item(
        'Another product', TaxedMoney(
            net=Money(27, currency='EUR'), gross=Money(30, currency='EUR'))]
transaction = Transaction(transaction_id, items)
report('UA-123456-1', client_id, transaction)
```


Reporting an extended ecommerce purchase
----------------------------------------

For Extended Ecommerce we have implemented Purchase tracking, please note
this will add an event automatically, as required by Google Analytics

First create `EnhancedItem`s to describe the contents of the transaction:
```python
EnhancedItem(name, unit_price[, quantity=None][, item_id=None]
             [, category=None][, brand=None][, variant=None])
```

Then the `EnhancedPurchase` itself:
```python
EnhancedPurchase(transaction_id, items, url_page[, revenue=None][, tax=None]
                 [, shipping=None][, host=None][, affiliation=None])
```
If `revenue` is given, it will override the total that is otherwise calculated
from items, taxes and shipping. please note you have to add an explicit path
when creating your `EnhancedPurchase` instance.

Example:
```python
from google_measurement_protocol import EnhancedItem, report, EnhancedPurchase

transaction_id = '0001'  # any string should do
items = [EnhancedItem('My awesome product', 90, quantity=2),
         EnhancedItem('Another product', 30))]
transaction = EnhancedPurchase(transaction_id, items, '/cart/')
report('UA-123456-1', client_id, transaction)
```


Reporting extra data
--------------------

You can pass `extra_info` and `extra_headers` to `report()` function to submit
additional information.

`extra_headers` is passed directly as additional headers to `requests`
library. This is currently the only way to pass `User-Agent`.

`extra_info` should be an instance of `SystemInfo`. Currently only language
reporting is supported:

```python
SystemInfo([language=None])
```

Example:
```python
from google_measurement_protocol import PageView, report, SystemInfo

view = PageView(path='/my-page/', title='My Page', referrer='http://example.com/')
headers = {'user-agent': 'my-user-agent 1.0'}
info = SystemInfo(language='en-us')
report('UA-123456-1', client_id, view, extra_info=info, extra_header=headers)
```
