# CopyFactory trade copying API for javascript (a member of [metaapi.cloud](https://metaapi.cloud) project)

CopyFactory is a powerful copy trading API which makes developing forex trade copying applications as easy as writing few lines of code.

CopyFactory API is a member of MetaApi project ([https://metaapi.cloud](https://metaapi.cloud)), a powerful cloud forex trading API which supports both MetaTrader 4 and MetaTrader 5 platforms.

MetaApi is a paid service, however we may offer a free tier access in some cases.

The [MetaApi pricing](https://metaapi.cloud/#pricing) was developed with the intent to make your charges less or equal to what you would have to pay
for hosting your own infrastructure. This is possible because over time we managed to heavily optimize
our MetaTrader infrastructure. And with MetaApi you can save significantly on application development and
maintenance costs and time thanks to high-quality API, open-source SDKs and convenience of a cloud service.

## Why do we offer CopyFactory trade copying API

We found that developing reliable and flexible trade copier is a task
which requires lots of effort, because developers have to solve a series
of complex technical tasks to create a product.

We decided to share our product as it allows developers to start with a
powerful solution in almost no time, saving on development and
infrastructure maintenance costs.
## Frequently asked questions (FAQ)
FAQ is located here: [https://metaapi.cloud/docs/copyfactory/faq/](https://metaapi.cloud/docs/copyfactory/faq/)

## CopyFactory copytradging API features

Features supported:

- low latency trade copying API
- reliable trade copying API
- suitable for large-scale deployments
- suitable for large number of subscribers
- connect arbitrary number of strategy providers and subscribers
- subscribe accounts to multiple strategies at once
- select arbitrary copy ratio for each subscription
- configure symbol mapping between strategy providers and subscribers
- apply advanced risk filters on strategy provider side
- override risk filters on subscriber side
- provide multiple strategies from a single account based on magic or symbol filters
- supports manual trading on subscriber accounts while copying trades
- synchronize subscriber account with strategy providers
- monitor trading history
- calculate trade copying commissions for account managers
- support portfolio strategies as trading signal source, i.e. the strategies which include signals of several other strategies (also known as combos on some platforms)
- webhooks for external systems to create trading signals on strategies

Please note that trade copying to MT5 netting accounts is not supported in the current API version

Please check Features section of the [https://metaapi.cloud/docs/copyfactory/](https://metaapi.cloud/docs/copyfactory/) documentation for detailed description of all settings you can make

## REST API documentation
CopyFactory SDK is built on top of CopyFactory REST API.

CopyFactory REST API docs are available at [https://metaapi.cloud/docs/copyfactory/](https://metaapi.cloud/docs/copyfactory/)

## FAQ
Please check this page for FAQ: [https://metaapi.cloud/docs/copyfactory/faq/](https://metaapi.cloud/docs/copyfactory/faq/).

## Code examples
We published some code examples in our github repository, namely:

- Javascript: [https://github.com/metaapi/metaapi-copyfactory-javascript-sdk/tree/master/examples](https://github.com/metaapi/metaapi-copyfactory-javascript-sdk/tree/master/examples)

## Installation
```bash
npm install --save metaapi.cloud-sdk
```

## Installing SDK in browser SPA applications
```bash
npm install --save metaapi.cloud-sdk
```

## Installing SDK in browser HTML applications
```html
<script src="unpkg.com/metaapi.cloud-sdk"></script>
<script>
    const token = '...';
    const api = new CopyFactory(token);
</script>
```

## Retrieving API token
Please visit [https://app.metaapi.cloud/token](https://app.metaapi.cloud/token) web UI to obtain your API token.

## Configuring trade copying

In order to configure trade copying you need to:

- add MetaApi MetaTrader accounts with CopyFactory as application field value (see above)
- create CopyFactory provider and subscriber accounts and connect them to MetaApi accounts via connectionId field
- create a strategy being copied
- subscribe subscriber CopyFactory accounts to the strategy

```javascript
import MetaApi, {CopyFactory} from 'metaapi.cloud-sdk';

const token = '...';
const metaapi = new MetaApi(token);
const copyfactory = new CopyFactory(token);

// retrieve MetaApi MetaTrader accounts with CopyFactory as application field value
// provider account must have PROVIDER value in copyFactoryRoles
const providerMetaapiAccount = await api.metatraderAccountApi.getAccount('providerMetaapiAccountId');
if(!providerMetaapiAccount.copyFactoryRoles || !providerMetaapiAccount.copyFactoryRoles.includes('PROVIDER')) {
  throw new Error('Please specify PROVIDER copyFactoryRoles value in your MetaApi account in ' +
    'order to use it in CopyFactory API');
}
// subscriber account must have SUBSCRIBER value in copyFactoryRoles
const subscriberMetaapiAccount = await api.metatraderAccountApi.getAccount('subscriberMetaapiAccountId');
if(!subscriberMetaapiAccount.copyFactoryRoles || !subscriberMetaapiAccount.copyFactoryRoles.includes('SUBSCRIBER')) {
  throw new Error('Please specify SUBSCRIBER copyFactoryRoles value in your MetaApi account in ' +
    'order to use it in CopyFactory API');
}

let configurationApi = copyfactory.configurationApi;

// create a strategy being copied
let strategyId = await configurationApi.generateStrategyId();
await configurationApi.updateStrategy(strategyId.id, {
  name: 'Test strategy',
  description: 'Some useful description about your strategy',
  accountId: providerMetaapiAccount.id,
  maxTradeRisk: 0.1,
  timeSettings: {
    lifetimeInHours: 192,
    openingIntervalInMinutes: 5
  }
});

// subscribe subscriber CopyFactory accounts to the strategy
await configurationApi.updateSubscriber(subscriberMetaapiAccount.id, {
  name: 'Demo account',
  subscriptions: [
    {
      strategyId: strategyId.id,
      multiplier: 1
    }
  ]
});
```

See esdoc in-code documentation for full definition of possible configuration options.

### Retrieving paginated lists

There are two groups of methods to retrieve paginated lists:
- with pagination in infinite scroll style
- with pagination in a classic style which allows you to calculate page count

They are applied to following entities:
- strategies: `getStrategiesWithInfiniteScrollPagination` and `getStrategiesWithClassicPagination`
- provider portfolios: `getPortfolioStrategiesWithInfiniteScrollPagination` and `getPortfolioStrategiesWithClassicPagination`
- subscribers: `getSubscribersWithInfiniteScrollPagination` and `getSubscribersWithClassicPagination`

Example of retrieving strategies with pagination in infinite scroll style:
```javascript
// paginate strategies, see esdoc for full list of filter options available
const strategies = await api.metatraderAccountApi.getStrategiesWithInfiniteScrollPagination({limit: 10, offset: 0});

// get strategies without filter (returns 1000 strategies max)
const strategies = await api.metatraderAccountApi.getStrategiesWithInfiniteScrollPagination();
const strategy = strategies.find(strategy => strategy._id === 'strategyId');
```

Example of retrieving strategies with paginiation in classic style:
```javascript
// paginate strategies, see esdoc for full list of filter options available
const strategies = await api.metatraderAccountApi.getStrategiesWithClassicPagination({limit: 10, offset: 0});
const strategy = strategies.items.find(strategy => strategy._id === 'strategyId');
// number of all strategies matching filter without pagination options
console.log(strategies.count);

// get strategies without filter (returns 1000 strategies max)
const strategies = await api.metatraderAccountApi.getStrategiesWithClassicPagination();
```

## Retrieving trade copying history

CopyFactory allows you to monitor transactions conducted on trading accounts in real time.

### Retrieving trading history on provider side
```javascript
let historyApi = copyfactory.historyApi;

// retrieve trading history, please note that this method support pagination and limits number of records
console.log(await historyApi.getProvidedTransactions(new Date('2020-08-01'), new Date('2020-09-01')));
```

### Retrieving trading history on subscriber side
```javascript
let historyApi = copyfactory.historyApi;

// retrieve trading history, please note that this method support pagination and limits number of records
console.log(await historyApi.getSubscriptionTransactions(new Date('2020-08-01'), new Date('2020-09-01')));
```

## Resynchronizing subscriber accounts to providers
There is a configurable time limit during which the trades can be opened. Sometimes trades can not open in time due to broker errors or trading session time discrepancy.
You can resynchronize a subscriber account to place such late trades. Please note that positions which were
closed manually on a subscriber account will also be reopened during resynchronization.

```javascript
let accountId = '...'; // CopyFactory account id

// resynchronize all strategies
await copyfactory.tradingApi.resynchronize(accountId);

// resynchronize specific strategy
await copyfactory.tradingApi.resynchronize(accountId, ['ABCD']);
```

## Sending external trading signals to a strategy
You can submit external trading signals to your trading strategy.

```javascript
const accountId = '...';
const tradingApi = copyfactory.tradingApi;
const signalId = '...';

// get signal client
const strategySignalClient = await tradingApi.getStrategySignalClient(strategyId);

// add trading signal
await strategySignalClient.updateExternalSignal(signalId, {
  symbol: 'EURUSD',
  type: 'POSITION_TYPE_BUY',
  time: new Date(),
  volume: 0.01
});

// get external signals
console.log(await signalClient.getStrategyExternalSignals(strategyId));

// remove signal
await strategySignalClient.removeExternalSignal(signalId, {
  time: new Date()
});
```

## Retrieving trading signals

```javascript
const accountId = '...';
const subscriberSignalClient = await tradingApi.getSubscriberSignalClient(accountId);

// retrieve trading signals
console.log(await subscriberSignalClient.getTradingSignals());
```

## Managing stopouts
A subscription to a strategy can be stopped if the strategy have exceeded allowed risk limit.
```javascript
let tradingApi = copyfactory.tradingApi;
let accountId = '...'; // CopyFactory account id
let strategyId = '...'; // CopyFactory strategy id

// retrieve list of strategy stopouts
console.log(await tradingApi.getStopouts(accountId));

// reset a stopout so that subscription can continue
await tradingApi.resetSubscriptionStopouts(accountId, strategyId, 'daily-equity');
```

## Managing stopout listeners
You can subscribe to a stream of stopout events using the stopout listener.
```javascript
import {StopoutListener} from 'metaapi.cloud-sdk';

let tradingApi = copyfactory.tradingApi;

// create a custom class based on the StopoutListener
class Listener extends StopoutListener {

  // specify the function called on event arrival
  async onStopout(strategyStopoutEvent) {
    console.log('Strategy stopout event', strategyStopoutEvent);
  }

  // specify the function called on error event
  async onError(error) {
    console.log('Error event', error);
  }

}

// add listener
const listener = new Listener();
const listenerId = tradingApi.addStopoutListener(listener);

// remove listener
tradingApi.removeStopoutListener(listenerId);
```

## Retrieving subscriber trading logs
```javascript
let tradingApi = copyfactory.tradingApi;
let accountId = '...'; // CopyFactory account id

// retrieve subscriber trading log
console.log(await tradingApi.getUserLog(accountId));

// retrieve paginated subscriber trading log by time range
console.log(await tradingApi.getUserLog(accountId, new Date(Date.now() - 24 * 60 * 60 * 1000), undefined, 20, 10));
```

## Log streaming
You can subscribe to a stream of strategy or subscriber log events using the user log listener.

### Strategy logs
```javascript
import {UserLogListener} from 'metaapi.cloud-sdk';

let tradingApi = copyfactory.tradingApi;

// create a custom class based on the UserLogListener
class Listener extends UserLogListener {

  // specify the function called on event arrival
  async onUserLog(logEvent) {
    console.log('Strategy user log event', logEvent);
  }

  // specify the function called on error event
  async onError(error) {
    console.log('Error event', error);
  }

}

// add listener
const listener = new Listener();
const listenerId = tradingApi.addStrategyLogListener(listener, 'ABCD');

// remove listener
tradingApi.removeStrategyLogListener(listenerId);
```

### Subscriber logs
```javascript
import {UserLogListener} from 'metaapi.cloud-sdk';

let tradingApi = copyfactory.tradingApi;

// create a custom class based on the UserLogListener
class Listener extends UserLogListener {

  // specify the function called on event arrival
  async onUserLog(logEvent) {
    console.log('Subscriber user log event', logEvent);
  }

  // specify the function called on error event
  async onError(error) {
    console.log('Error event', error);
  }

}

// add listener
const listener = new Listener();
const listenerId = tradingApi.addSubscriberLogListener(listener, 'accountId');

// remove listener
tradingApi.removeSubscriberLogListener(listenerId);
```

## Transaction streaming
You can subscribe to a stream of strategy or subscriber transaction events using the transaction listener.

### Strategy transactions
```javascript
import {TransactionListener} from 'metaapi.cloud-sdk';

let historyApi = copyfactory.historyApi;

// create a custom class based on the TransactionListener
class Listener extends TransactionListener {

  // specify the function called on event arrival
  async onTransaction(transactionEvent) {
    console.log('Strategy transaction event', transactionEvent);
  }

  // specify the function called on error event
  async onError(error) {
    console.log('Error event', error);
  }

}

// add listener
const listener = new Listener();
const listenerId = historyApi.addStrategyTransactionListener(listener, 'ABCD');

// remove listener
historyApi.removeStrategyTransactionListener(listenerId);
```

### Subscriber transactions
```javascript
import {TransactionListener} from 'metaapi.cloud-sdk';

let historyApi = copyfactory.historyApi;

// create a custom class based on the TransactionListener
class Listener extends TransactionListener {

  // specify the function called on event arrival
  async onTransaction(transactionEvent) {
    console.log('Subscriber transaction event', transactionEvent);
  }

  // specify the function called on error event
  async onError(error) {
    console.log('Error event', error);
  }

}

// add listener
const listener = new Listener();
const listenerId = historyApi.addSubscriberTransactionListener(listener, 'accountId');

// remove listener
historyApi.removeSubscriberTransactionListener(listenerId);
```

## Webhooks

Webhooks can be created on specific strategies and their URLs can be provided to external systems to create external trading signals. The URL contains a secret webhook ID, so no extra authorization is required on a REST API invocation to a webhook.

```javascript
const strategyId = '...';

let webhook = await copyfactory.configurationApi.createWebhook(strategyId);
let url = webhook.url;
```

For example, if `webhook.url` is `https://copyfactory-api-v1.london.agiliumtrade.ai/webhooks/yMLd8aviewgFfS4NBxZETkoVPbWAJ92t` then a request can be sent to it to create an external signal:

```bash
curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{
  "symbol": "EURUSD",
  "type": "POSITION_TYPE_BUY",
  "time": "2024-12-19T06:52:19.679Z",
  "volume": 0.1
}' 'https://copyfactory-api-v1.london.agiliumtrade.ai/webhooks/yMLd8aviewgFfS4NBxZETkoVPbWAJ92t'
```

## Related projects:
Take a look at our website for the full list of APIs and features supported [https://metaapi.cloud/#features](https://metaapi.cloud/#features)

Some of the APIs you might decide to use together with MetaStats API are:

1. MetaApi cloud forex API [https://metaapi.cloud/docs/client/](https://metaapi.cloud/docs/client/)
2. MetaTrader account management API [https://metaapi.cloud/docs/provisioning/](https://metaapi.cloud/docs/provisioning/)
3. MetaStats forex trading metrics API [https://metaapi.cloud/docs/metastats/](https://metaapi.cloud/docs/metastats/)
4. MetaApi MT manager API [https://metaapi.cloud/docs/manager/](https://metaapi.cloud/docs/manager/)
5. MetaApi risk management API [https://metaapi.cloud/docs/risk-management/](https://metaapi.cloud/docs/risk-management/)
