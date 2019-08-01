# axios-retry

[![Build Status](https://travis-ci.org/softonic/axios-retry.svg?branch=master)](https://travis-ci.org/softonic/axios-retry)

Axios plugin that intercepts failed requests and retries them whenever possible.

## Installation

```bash
npm install axios-retry
```

## Usage

```js
// CommonJS
// const axiosRetry = require('axios-retry');

// ES6
import axiosRetry from 'axios-retry';

axiosRetry(axios, { retries: 3 });

axios.get('http://example.com/test') // The first request fails and the second returns 'ok'
  .then(result => {
    result.data; // 'ok'
  });

// Exponential back-off retry delay between requests
axiosRetry(axios, { retryDelay: axiosRetry.exponentialDelay});

// Custom retry delay
axiosRetry(axios, { retryDelay: (retryCount) => {
  return retryCount * 1000;
}});

// Works with custom axios instances
const client = axios.create({ baseURL: 'http://example.com' });
axiosRetry(client, { retries: 3 });

client.get('/test') // The first request fails and the second returns 'ok'
  .then(result => {
    result.data; // 'ok'
  });

// Allows request-specific configuration
client
  .get('/test', {
    'axios-retry': {
      retries: 0
    }
  })
  .catch(error => { // The first request fails
    error !== undefined
  });

// Allows retries on status-code-200 responses to work around poorly-written APIs

const shouldRetrySuccessResponse = (response) => {
  return response.errorMessage && response.errorMessage.includes('too many requests');
};

axiosRetry(client, { shouldRetrySuccessResponse });

client.get('/test')  // Retries until shouldRetrySuccessResponse returns false or another no-retry condition is met
  .then(result => {
    result.data; // 'ok'
  });
```

**Note:** Unless `shouldResetTimeout` is set, the plugin interprets the request timeout as a global value, so it is not used for each retry but for the whole request lifecycle.

## Options

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| retries | `Number` | `3` | The number of times to retry before failing. |
| retryCondition | `Function` | `isNetworkOrIdempotentRequestError` | A callback to further control if a request should be retried.  By default, it retries if it is a network error or a 5xx error on an idempotent request (GET, HEAD, OPTIONS, PUT or DELETE). |
| retryDelay | `Function` | `function noDelay() { return 0; }` | A callback to further control the delay between retried requests. By default there is no delay between retries. Another option is exponentialDelay ([Exponential Backoff](https://developers.google.com/analytics/devguides/reporting/core/v3/errors#backoff)). The function is passed `retryCount` and `error`. |
| shouldResetTimeout | `Boolean` | false | Defines if the timeout should be reset between retries |
| shouldRetrySuccessResponse | `Function` | `function shouldRetrySuccessResponse(response) { return false; }` | A callback to control if a response with a 200 status code should be retried.  Useful when you need to use APIs that send 200 status codes when an error status code (e.g. 429) is more appropriate. The function is passed the server's response.  |  

## Testing

Clone the repository and execute:

```bash
npm test
```

## Contribute

1. Fork it: `git clone https://github.com/softonic/axios-retry.git`
2. Create your feature branch: `git checkout -b feature/my-new-feature`
3. Commit your changes: `git commit -am 'Added some feature'`
4. Check the build: `npm run build`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request :D
