# MockRequest helper for CodeceptJS

[![Build Status](https://img.shields.io/endpoint.svg?url=https%3A%2F%2Factions-badge.atrox.dev%2Fcodecept-js%2Fmock-request%2Fbadge&style=popout)](https://actions-badge.atrox.dev/codecept-js/mock-request/goto)

> Replacement for built-in "MockRequest" helper of CodeceptJS v2.

👻 Implements HTTP request mocking, record & replay via [PollyJS](https://netflix.github.io/pollyjs/#/) for [CodeceptJS](https://codecept.io). 

Use it to:

-   📼 Record & Replay HTTP requests
-   📲 Replace server responses
-   ⛔ Block third party integrations: analytics, chats, support systems. 

Works with Puppeteer & WebDriver helpers of [CodeceptJS](https://codecept.io).

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

-   [MockRequest](#mockrequest)
    -   [Installations](#installations)
    -   [Configuration](#configuration)
        -   [With Puppeteer](#with-puppeteer)
        -   [With WebDriver](#with-webdriver)
-   [Usage](#usage)
    -   [👻 Mock Requests](#-mock-requests)
    -   [📼 Record & Replay](#-record--replay)
    -   [Parameters](#parameters)
    -   [startMocking](#startmocking)
        -   [Parameters](#parameters-1)
    -   [mockServer](#mockserver)
        -   [Parameters](#parameters-2)
    -   [recordMocking](#recordmocking)
    -   [replayMocking](#replaymocking)
    -   [passthroughMocking](#passthroughmocking)
    -   [flushMocking](#flushmocking)
    -   [stopMocking](#stopmocking)
    -   [mockRequest](#mockrequest-1)
        -   [Parameters](#parameters-3)

### MockRequest

This helper allows to **mock requests while running tests in Puppeteer or WebDriver**.
For instance, you can block calls to 3rd-party services like Google Analytics, CDNs.
Another way of using is to emulate requests from server by passing prepared data.

MockRequest helper works in these [modes](https://netflix.github.io/pollyjs/#/configuration?id=mode):

-   passthrough (default) - mock prefefined HTTP requests
-   record - record all requests into a file
-   replay - replay all recorded requests from a file

Combining record/replay modes allows testing websites with large datasets. 

To use in passthrough mode set rules to mock requests and they will be automatically intercepted and replaced:

```js
// default mode
I.mockRequest('GET', '/api/users', '[]');
```

In record-replay mode start mocking to make HTTP requests recorded/replayed, and stop when you don't need to block requests anymore:

```js
// record or replay all XHR for /users page
I.startMocking();
I.amOnPage('/users');
I.stopMocking();
```

#### Installations

    npm i @codeceptjs/mock-request --save-dev

Requires Puppeteer helper or WebDriver helper enabled

#### Configuration

##### With Puppeteer

Enable helper in config file:

```js
helpers: {
   Puppeteer: {
     // regular Puppeteer config here
   },
   MockRequestHelper: {
     require: '@codeceptjs/mock-request',
   }
}
```

[Polly config options](https://netflix.github.io/pollyjs/#/configuration?id=configuration) can be passed as well:

```js
// sample options
helpers: {
  MockRequestHelper: {
     require: '@codeceptjs/mock-request',
     mode: record,
     recordIfMissing: true,
     recordFailedRequests: false,
     expiresIn: null,
     persisterOptions: {
       keepUnusedRequests: false
       fs: {
         recordingsDir: './data/requests',
       },
    },
  }
}
```

* * *

**TROUBLESHOOTING**: Puppeteer does not mock requests in headless mode: 

Problem: request mocking does not work and in debug mode you see this in output:

    Access to fetch at {url} from origin {url} has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

Solution: update Puppeteer config to include `--disable-web-security` arguments:

```js
 Puppeteer: {
   show: false,
   chrome: {
     args: [
       '--disable-web-security',
     ],
   },
 },
```

* * *

##### With WebDriver

This helper partially works with WebDriver. It can intercept and mock requests **only on already loaded page**.

```js
helpers: {
   WebDriver: {
     // regular WebDriver config here
   },
   MockRequestHelper: {
     require: '@codeceptjs/mock-request',
   }
}
```

> Record/Replay mode is not tested in WebDriver but technically can work with [REST Persister](https://netflix.github.io/pollyjs/#/examples?id=rest-persister)

### Usage

#### 👻 Mock Requests

To intercept API requests and mock them use following API

-   [startMocking()](#startMocking) - to enable request interception
-   [mockRequest()](#mockRequest) - to define mock in a simple way
-   [mockServer()](#mockServer) - to use PollyJS server API to define complex mocks
-   [stopMocking()](#stopMocking) - to stop intercepting requests and disable mocks.

Calling `mockRequest` or `mockServer` will start mocking, if it was not enabled yet.

```js
I.startMocking(); // optionally
I.mockRequest('/google-analytics/*path', 200);
// return an empty successful response 
I.mockRequest('GET', '/api/users', 200);
// mock users api
I.mockServer(server => {
  server.get('https://server.com/api/users*').
    intercept((req, res) => { res.status(200).json(users);
  });
});
I.click('Get users);
I.stopMocking();
```

#### 📼 Record & Replay

> At this moment works only with Puppeteer

Record & Replay mode allows you to record all xhr & fetch requests and save them to file.
On next runs those requests can be replayed. 
By default, it stores all passed requests, but this behavior can be customized with `I.mockServer`

Set mode via enironment variable, `replay` mode by default:

```js
// enable replay mode
helpers: {
 Puppeteer: {
   // regular Puppeteer config here
 },
 MockRequest: {
    require: '@codeceptjs/mock-request',
    mode: process.env.MOCK_MODE || 'replay',
 },
}
```

Interactions between `I.startMocking()` and `I.stopMocking()` will be recorded and saved to `data/requests` directory.

```js
I.startMocking() // record requests under 'Test' name
I.startMocking('users') // record requests under 'users' name
```

Use `I.mockServer()` to customize which requests should be recorded and under which name:

```js
I.startMocking();
I.mockServer((server) => {
  // mock request only from ap1.com and api2.com and
  // store recording into two different files
  server.any('https://api1.com/*').passthrough(false).recordingName('api1');
  server.any('https://api2.com/*').passthrough(false).recordingName('api2');
});
```

To stop request recording/replaying use `I.stopMocking()`.

🎥 To record HTTP interactions execute tests with MOCK_MODE environment variable set as "record":

    MOCK_MODE=record npx codeceptjs run --debug

📼 To replay them launch tests without environment variable:

    npx codeceptjs run --debug

#### Parameters

-   `config`  

#### startMocking

Starts mocking of http requests.
In record mode starts recording of all requests.
In replay mode blocks all requests and replaces them with saved.

If inside one test you plan to record/replay requests in several places, provide [recording name](https://netflix.github.io/pollyjs/#/api?id=recordingname) as the parameter.

```js
// start mocking requests for a test
I.startMocking(); 

// start mocking requests for main page
I.startMocking('main-page');
// do actions
I.stopMocking();
I.startMocking('login-page');
```

To update [PollyJS configuration](https://netflix.github.io/pollyjs/#/configuration) use secon argument:

```js
// change mode
I.startMocking('comments', { mode: 'replay' });

// override config
I.startMocking('users-loaded', {
   recordFailedRequests: true
})
```

##### Parameters

-   `title` **any**  (optional, default `'Test'`)
-   `config`   (optional, default `{}`)

#### mockServer

Use PollyJS [Server Routes API](https://netflix.github.io/pollyjs/#/server/overview) to declare mocks via callback function:

```js
// basic usage
server.get('/api/v2/users').intercept((req, res) => {
  res.sendStatus(200).json({ users });
});

// passthrough requests to "/api/v2"
server.get('/api/v1').passthrough();
```

In record replay mode you can define which routes should be recorded and where to store them:

```js
I.startMocking('mock');
I.mockServer((server) => {

  // record requests from cdn1.com and save them to data/recording/xml
  server.any('https://cdn1.com/*').passthrough(false).recordingName('xml');
  
  // record requests from cdn2.com and save them to data/recording/svg
  server.any('https://cdn2.com/*').passthrough(false).recordingName('svg');

  // record requests from /api and save them to data/recording/mock (default)
  server.any('/api/*').passthrough(false);
});
```

##### Parameters

-   `configFn`  

#### recordMocking

Forces record mode for mocking.
Requires mocking to be started.

```js
I.recordMocking();
```

#### replayMocking

Forces replay mode for mocking.
Requires mocking to be started.

```js
I.replayMocking();
```

#### passthroughMocking

Forces passthrough mode for mocking.
Requires mocking to be started.

```js
I.passthroughMocking();
```

#### flushMocking

Waits for all requests handled by MockRequests to be resolved:

```js
I.flushMocking();
```

#### stopMocking

Stops mocking requests.
Must be called to save recorded requests into faile.

```js
I.stopMocking();
```

#### mockRequest

Mock response status

```js
I.mockRequest('GET', '/api/users', 200);
I.mockRequest('ANY', '/secretsRoutes/*', 403);
I.mockRequest('POST', '/secrets', { secrets: 'fakeSecrets' });
I.mockRequest('GET', '/api/users/1', 404, 'User not found');
```

Multiple requests

```js
I.mockRequest('GET', ['/secrets', '/v2/secrets'], 403);
```

##### Parameters

-   `method` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** request method. Can be `GET`, `POST`, `PUT`, etc or `ANY`.
-   `oneOrMoreUrls` **([string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Array](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)>)** url(s) to mock. Can be exact URL, a pattern, or an array of URLs.
-   `dataOrStatusCode` **([number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number) \| [string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String) \| [object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object))** status code when number provided. A response body otherwise
-   `additionalData` **([string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String) \| [object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object))** response body when a status code is set by previous parameter. (optional, default `null`)
