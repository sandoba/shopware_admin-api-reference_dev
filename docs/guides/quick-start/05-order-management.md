---
stoplight-id: fdd24cc76f22d
---

# Order Overview

The order resource enables merchants to process orders they receive from customers.

You can use the orders resource to do the following:

* Retrieve orders and their line-items
* Create orders
* Update order statuses
* Manage returns

## Retrieve Orders

A list of all customer orders is obtained using the below route:

```json http
{
  "method": "post",
  "url": "/api/search/order",
  "headers": {
    "Content-Type": "application/json",
    "Accept": "application/json",
    "Authorization": "Bearer Your_API_Key"
  }
}
```

[Sorting](https://shopware.stoplight.io/docs/store-api/cf710bf73d0cd-search-queries#sort), [aggregating](https://shopware.stoplight.io/docs/store-api/cf710bf73d0cd-search-queries#aggregations) and [filtering](https://shopware.stoplight.io/docs/store-api/cf710bf73d0cd-search-queries#filter) order results is possible using a [search criteria](https://shopware.stoplight.io/docs/store-api/cf710bf73d0cd-search-queries).

### Order line-items

An order can have more other items or child items of `type` - `product`, `promotion`, `credit` or `custom`. To fetch line items for a particular order, try the below route:

```json http
{
  "method": "get",
  "url": "http://localhost/api/order/558efc15fe604829b4d0607df75187e0/line-item",
  "headers": {
    "Content-Type": "application/json",
    "Accept": "application/json",
    "Authorization": "Bearer Your_API_Key"
  }
  }
```

## Create an order

You can manually create test orders in the admin panel to record orders made outside Shopware or send customer invoices. However, no payments, invoices, etc., are created automatically.

You can create orders for existing or new customers.

```json http
{
  "method": "post",
  "url": "http://localhost/api/order",
  "headers": {
    "Content-Type": "application/json",
    "Accept": "application/json",
    "Authorization": "Bearer Your_API_Key"
  },
  "body": {
    "billingAddressId": "c90e05c82c5a4457844cba7403c7ef96",
    "currencyId": "b7d2554b0ce847cd82f3ac9bd1c0dfca",
    "languageId": "2fbb5fe2e29a4d70aa5854ce7ce3e20b",
    "salesChannelId": "98432def39fc4624b33213a56b8c944d",
    "orderDateTime": "2021-05-31T14:13:14.866+00:00",
    "currencyFactor": 1.0,
    "stateId": "a75eb89b4abe41f9bade83b2f07d874e"
   }
  }
```

```json json_schema
{
  "type": "object",
  "description": "Parameters for order creation.",
  "properties": {
    "billingAddressId": {
      "description": "ID of the billing address.",
      "type": "string"
    },
    "currencyId": {
      "description": "ID of the currency to which the price belongs.",
      "type": "string"
    },
    "languageId": {
      "description": "Unique ID of language.",
      "type": "string"
    },
    "salesChannelId": {
      "description": "Unique ID of the defined sales channel.",
      "type": "string"
    },
    "orderDateTime": {
      "description": "Timestamp of the order placed.",
      "type": "string"
    },
    "currencyFactor": {
      "description": "Rate factor for currencies.",
      "type": "string"
    },
    "stateId": {
      "description": "Unique ID of transition state as defined by the state machine.",
      "type": "string"
      }
  }
}
```

Orders created are associated with payment and delivery transitions. The following section provides you with more details.

## Order state handling

Every order in Shopware has three states `order.state`, `order_delivery.state`, and `order_transaction.state`. For each state, there is a state machine that holds the status of the order, delivery, and payment status, respectively. The `state_machine_transition` is a collection of all defined transitions.

The `transition` method handles the order transition from one state to another.

### Order

On creating a new order, the order state is set to *open* by default. Order state can be transitioned among `cancel`, `complete`, `reopen`, and `process`.

Below is a sample request to change the state of order to *complete*:

```json http
{
  "method": "post",
  "url": "http://localhost/api/_action/order/558efc15fe604829b4d0607df75187e0/state/complete",
  "headers": {
    "Content-Type": "application/json",
    "Accept": "application/json",
    "Authorization": "Bearer Your_API_Key"
  },
  "body": {
  }
}
```

A *canceled* order cannot change to an *in-progress* state unless it is reopened again.

### Order delivery

The order delivery state represents the state of the delivery. `reopen`, `ship`, `ship_partially`, `cancel`, `retour`, and `retour_partially` are the states associated with order delivery.

Below is a sample request that sets the delivery state to *fail*:

```json http
{
  "method": "post",
  "url": "http://localhost/api/_action/order_delivery/558efc15fe604829b4d0607df75187e0/state/fail",
  "headers": {
    "Content-Type": "application/json",
    "Accept": "application/json",
    "Authorization": "Bearer Your_API_Key"
  }
}
```

### Order transaction

The order transaction state represents the state of the payment. `open`, `fail`, `authorize`, `refund_partially`, `refund`, `do_pay`, `paid`, `paid_partially`, `remind`,  and `cancel` are the states associated with order transaction.

Below is a sample request that sets the payment state to *open*:

```json http
{
  "method": "post",
  "url": "http://localhost/api/_action/order_transaction/558efc15fe604829b4d0607df75187e0/state/open",
  "headers": {
    "Content-Type": "application/json",
    "Accept": "application/json",
    "Authorization": "Bearer Your_API_Key"
  }
  }
```  

## Refund Payment

Initiating and capturing payment is handled by [Store API](https://shopware.stoplight.io/docs/store-api/8218801e50fe5-handling-the-payment), whereas the admin API deals with refund payment.

The refund payment method can be called only for transactions that are claimed to be successful.

Generally, refunds are linked to a specific order transaction capture ID. An order can have one or more line items and amounts. Each of these line items signifies refund positions. Based on the number of line items requested for refund, the payment can either be partially or completely refunded.

<!-- theme: danger -->
> **Refund handling internals**
>
> To allow easy refund handling, have your payment handler implement the `RefundPaymentHandlerInterface`. Your refund handler implementation will be called with the `refundId` to call the PSP or gateway.
>
> For more information, check our documentation on [implementing payment refund handlers](https://developer.shopware.com/docs/guides/plugins/plugins/checkout/payment/add-payment-plugin#refund-example)

When you refund a payment, the API will change the refund state to *complete*. If you want to fail the refund in your RefundHandler implementation, simply throw a `RefundException`, and the state of the refund will transition to *fail*.
  
> Your payment extensions must write their own captures and refunds into the order_transaction_capture and order_transaction_capture_refund tables, respectively, before calling the refund handler, as the Shopware Core does not carry this out.
