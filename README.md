# Lighthouse4u

LH4U provides [Google Lighthouse](https://github.com/GoogleChrome/lighthouse) as a service, surfaced by both a friendly UI+API, and backed by
[Elastic Search](https://www.elastic.co/products/elasticsearch) for all your query and [visualization needs](https://www.elastic.co/products/kibana).


![UI](./docs/example.gif)

## Usage

Start server via CLI:

```
npm i -g lighthouse4u
lh4u --config local --config-dir ./app/config --config-base defaults --secure-config ./app/config/secure --secure-file some-private.key -- server
```

Or locally to this repo via `npm start` (you'll need to create `test/config/local.json5` by copying `test/config/COPY.json5`).


## Configuration Options

| Option | Type | Default | Desc |
| --- | --- | --- | --- |
| elasticsearch | | | All options connected with Elasticsearch usage |
| elasticsearch.options | `ESOptions` | [See Defaults](./src/config/default-config.js#L3) | [ES driver options](https://www.npmjs.com/package/elasticsearch) |
| elasticsearch.indexName | `string` | `lh4u` | Index name in ES |
| elasticsearch.indexType | `string` | `lh4u` | Index type in ES |
| elasticsearch.index | `ESIndexOptions` | [See Defaults](./src/config/default-config.js#L12) | Options supplied to driver upon creation of ES index |
| amqp | | | [AMQP Options](https://www.npmjs.com/package/amqplib) to connect to RabbitMQ or any other AMQP-compatible interface |
| amqp.url | `string` | `amqp://guest:guest@localhost:5672/lh4u` | Connection string following the typical `{username}:{password}@{host}:{port}/{path}` format |
| amqp.idleDelayMs | `number` | `1000` | Time (in MS) between queue checks when queue is empty |
| amqp.secretKey | `string` | optional | Required only if encrypting/decrypting `secureHeaders`. Should be stored in [secure configuration](#secure-configuration) |
| amqp.queue | `AmqpOptions` | | Queue options |
| amqp.queue.enabled | `boolean` | `true` | To enable pulling/processing of queue |
| amqp.queue.name | `string` | `lh4u` | Name of queue |
| amqp.queue.options | | [See Defaults](./src/config/default-config.js#L40) | [Queue creation options](http://www.squaremobius.net/amqp.node/channel_api.html#channel_assertQueue) |
| http | | | HTTP(S) Server options |
| http.bindings | `Hash<HttpBinding>` | | An object of named bindings |
| http.bindings.{name} | `HttpBinding` | | With `{name}` being the name of the binding -- can be anything, ex: `myMagicBinding`. See value to `false` if you want to disable this binding. |
| http.bindings.{name}.port | `number` | `8994` | Port to bind to |
| http.bindings.{name}.ssl | [TLS Options](https://nodejs.org/dist/latest-v10.x/docs/api/tls.html#tls_new_tls_tlssocket_socket_options) | `undefined` | Required to bind HTTPS/2 |
| http.auth | | | Zero or more authorization options |
| http.auth.{authName} | `HttpAuth` | | A set of auth options keyed by any custom `{authName}` |
| http.auth.{authName}.type | `basic|custom` | **required** | Type of auth, be it built-in Basic auth, or custom provided by `customPath` module |
| http.auth.{authName}.groups | `string|array<string>` | `*` | **required** |  |
| http.auth.{authName}.customPath | `string` | | Path of the custom module |
| http.auth.{authName}.options | | | Options provided to the auth module |
| http.authRedirect | | `undefined` | URL to redirect UI to if auth is enabled and fails |
| http.routes | | | Optional set of custom routes |
| http.routes.{name} | `string|HttpRoute` | | If value of type string that'll be used as path |
| http.routes.{name}.path | `string` | | Path to resolve to connect middleware, relative to current working directory |
| http.routes.{name}.method | `string` | `GET` | Method to bind to |
| http.routes.{name}.route | `string` | Uses `{name}` if undefined | Route to map, ala `/api/test` |
| http.staticFiles | | | Optional to bind routes to static assets |
| http.staticFiles.{route} | [Express Static](https://expressjs.com/en/starter/static-files.html) | | Options to supply to static middleware |
| lighthouse | | | Options related to Lighthouse usage |
| lighthouse.config | [LighthouseOptions](https://github.com/GoogleChrome/lighthouse/blob/master/lighthouse-core/config/default-config.js) | | Google Lighthouse configuration options |
| lighthouse.config.extends | `string` | `lighthouse:default` | What options to default to |
| lighthouse.config.logLevel | `string` | `warn` | Level of logging |
| lighthouse.config.chromeFlags | `array<string>` | `[ '--headless', '--disable-gpu', '--no-sandbox' ]` | Array of CLI arguments |
| lighthouse.config.settings | [LighthouseSettings](https://github.com/GoogleChrome/lighthouse/blob/master/lighthouse-core/config/constants.js#L30) | [See Defaults](./src/config/default-config.js#L90) | Settings applied to Lighthouse |
| lighthouse.validate | `hash<string>` | | Zero or more validators |
| lighthouse.validate.{groupName} | `string` | | Path of validation module used to determine if the response is coming from the intended server. Useful in cases where you only want to measure results coming from an intended infrastructure |
| lighthouse.concurrency | `number` | `1` | Number of concurrently processed tasks permitted |
| lighthouse.samples | | | After N lighthouse samples are taken, only the best result is recorded |
| lighthouse.samples.default | `number` | `3` | Default number of lighthouse tests performed  |
| lighthouse.samples.range | `tuple<min,max>` | `[1, 5]` | Minimum and maximum samples taken before returning result |
| lighthouse.attempts | | | Number of attempts at running Google Lighthouse before giving up due to failures |
| lighthouse.attempts.default | `number` | `2` | Default number of attempts before giving up |
| lighthouse.attempts.range | `tuple<min,max>` | `[1, 10]` | Minimum and maximum attempts before giving up |
| lighthouse.attempts.delayMsPerExponent | `number` | `1000` | Exponential backoff after failure |
| lighthouse.delay | | | Time (in ms) before a test is executed |
| lighthouse.delay.default | `number` | `0` | Default time to wait before test can be run |
| lighthouse.delay.range | `tuple<min,max>` | `[0, 1000 * 60 * 60]` | Minimum and maximum time before test can be run |
| lighthouse.delay.delayMsPerExponent | `number` | `1000 * 30` | Maximum time before delayed messages will be requeued |


## API

### API - `GET /api/website`

Fetch zero or more website results matching the criteria.

#### Query String Options

| Option | Type | Default | Desc |
| --- | --- | --- | --- |
| format | `string` | `json` | Format of results, be it `json` or `svg` |
| scale | `number` | `1` | Scale of `svg` |
| documentId | `string` | optional | Query by document ID |
| requestedUrl | `string` | optional | Query by requested URL |
| finalUrl | `string` | optional | Query by final URL (after redirects) |
| domainName | `string` | optional | Query by full domain |
| rootDomain | `string` | optional | Query by root domain |
| group | `string` | optional | Query by group ID |


### API - `POST /api/website`

Submit to process website with Google Lighthouse. 

#### JSON Body Options

| Option | Type | Default | Desc |
| --- | --- | --- | --- |
| url | `string` | **required** | URL to process via Google Lighthouse |
| headers | `hash<string>` | optional | Collection of HTTP headers to supply in request |
| secureHeaders | hash<string> | optional | Same use as `headers`, but stored securely in queue, and never persisted to ElasticSearch |
| samples | `number` | (See `options.lighthouse.samples`) | Number of samples to take before recording result |
| attempts | `number` | (See `options.lighthouse.attempts`) | Number of failure attempts before giving up |
| hostOverride | `string` | optional | Map host of request to an explicit IP. [Not yet supported by Chrome in Headless mode](https://bugs.chromium.org/p/chromium/issues/detail?id=798793) |
| delay | `number` | (See `options.lighthouse.delay`) | Delay (in milliseconds) before test will be performed. Useful if the intended service or domain is not expected to be available for some time |
| group | `string` | `unknown` | Group to categorize result to. Useful when searching/filtering on different groups of results |



## Secure Configuration

Optionally you may run LH4U with the `--secure-config {secureConfigPath}` and `--secure-file {securePrivateFile}` powered by [Config Shield](https://github.com/godaddy/node-config-shield).

```
npm i -g config-shield
cshield ./app/config/secure some-private.key
> help
> set amqp { url: 'amqp://lh4u_user:someSuperSecretPassword@rmq.on-my-domain.com:5672/lh4u' }
> get amqp
: { "url": "amqp://lh4u_user:someSuperSecretPassword@rmq.on-my-domain.com:5672/lh4u" }
> save
> exit
```

The example above will allow you to merge sensitive credentials with your existing public configuration. Only options defined in the secure config will be deeply merged.
