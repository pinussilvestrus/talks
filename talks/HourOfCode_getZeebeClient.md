---
title: Hour of Code (9th February) - `zeebe-api#getZeebeClient`
author: pinussilvestrus
---

# Hour of Code

## [`app/zeebe-api#getZeebeClient`](https://github.com/zeebe-io/zeebe-modeler/blob/develop/app/lib/zeebe-api.js#L163-L209)

---

```js
getZeebeClient(endpoint) {
  const cachedInstance = this.zbClientInstanceCache;

  if (endpoint && isHashEqual(endpoint, this.endpointCache)) {
    return cachedInstance;
  }

  if (cachedInstance) {
    this.shutdownClientInstance(cachedInstance);
  }

  this.endpointCache = endpoint;

  if (endpoint.type === 'selfHosted') {

    this.zbClientInstanceCache = new this.ZB.ZBClient(endpoint.url, {
      retry: false
    });
  } else if (endpoint.type === 'oauth') {

    this.zbClientInstanceCache = new this.ZB.ZBClient(endpoint.url, {
      retry: false,
      oAuth: {
        url: endpoint.oauthURL,
        audience: endpoint.audience,
        clientId: endpoint.clientId,
        clientSecret: endpoint.clientSecret,
        cacheOnDisk: false
      },
      useTLS: true
    });
  } else if (endpoint.type === 'camundaCloud') {

    this.zbClientInstanceCache = new this.ZB.ZBClient({
      retry: false,
      camundaCloud: {
        clientId: endpoint.clientId,
        clientSecret: endpoint.clientSecret,
        clusterId: endpoint.clusterId,
        cacheOnDisk: false
      },
      useTLS: true
    });
  }

  return this.zbClientInstanceCache;
}
```

---

## Assessment (I)

Doing one, two, three .... things.

* Re-use cached client
* Cleanup outdated client
* Distinguish between different endpoint configurations
* Creating dat stuff

---

## Assessment (II)

What is _public_ & _private_ API?

```js

export default class ZeebeAPI {

  async checkConnectivity(parameters) { }

  async deploy(parameters) { }

  async run(parameters) { }

  getZeebeClient(endpoint) { }

  shutdownClientInstance(instance) { }
}
```

---

## Assessment (III)

What shall it do?

:arrow_right: Get a configured Zeebe Client for a given endpoint.

---

## Solution Sketch (I)

Make things explicit.

```js
getZeebeClient(endpoint) {
  
  const cachedInstance = this.zeebeNodeClientInstanceCache;

  // (1) use existing Zeebe Client for endpoint
  if (endpoint && isHashEqual(endpoint, this.endpointCache)) {
    return cachedInstance;
  }

  // (2) cleanup old client instance
  if (cachedInstance) {
    this.shutdownClientInstance(cachedInstance);
  }

  this.endpointCache = endpoint;

  // (3) create new Zeebe Client for endpoint configuration
  this.zeebeNodeClientInstanceCache = this.createZeebeClient(endpoint);

  return this.zeebeNodeClientInstanceCache;
}
```

---

## Solution Sketch (II)

Make it private API.

```js
_getZeebeClient(endpoint) {

  const cachedInstance = this._zeebeNodeClientInstanceCache;

  // (1) use existing Zeebe Client for endpoint
  if (endpoint && isHashEqual(endpoint, this._endpointCache)) {
    return cachedInstance;
  }

  // (2) cleanup old client instance
  if (cachedInstance) {
    this._shutdownClientInstance(cachedInstance);
  }

  this.endpointCache = endpoint;

  // (3) create new Zeebe Client for endpoint configuration
  this._zeebeNodeClientInstanceCache = this._createZeebeClient(endpoint);

  return this._zeebeNodeClientInstanceCache;
}
```

---

## Solution Sketch (II)

Make it private API.

```js
async run(parameters) {

  const {
    endpoint,
  } = parameters;

  const client = this._getZeebeClient(endpoint);

  // ...
}
```

---

## Solution Sketch (III)

Pull it out.

```js
_createZeebeClient(endpoint) {

  const {
    type
  } = endpoint;

  // define the default
  let options = {
    retry: false
  };

  if (!values(endpointTypes).includes(type)) {
    return;
  }

  if (type === endpointTypes.OAUTH) {
    options = {
      ...options,
      oAuth: {
        url: endpoint.oauthURL,
        audience: endpoint.audience,
        clientId: endpoint.clientId,
        clientSecret: endpoint.clientSecret,
        cacheOnDisk: false
      },
      useTLS: true
    };
  }

  /* ... */

  return new this.ZeebeNode.ZBClient(endpoint.url, options);
}
```

---

## Thanks!

Find results in [`camunda-modeler#2091`](https://github.com/camunda/camunda-modeler/pull/2091).