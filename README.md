# bk-rabbitmq-rpc

This is a very opinionated abstraction over amqplib to help simplify the implementation of RPC messaging patterns on RabbitMQ.

> !IMPORTANT! - bk-rabbitmq-rpc needs nodejs >= 8.x.

### Features:
 * Creation of RPC services (client & server)
 * Attempt to gracefully handle lost connections and channels
 * Connections have a backoff policy of exponential delay randomized: ~1s, ~2s, ~4s, ~8s then stable at ~8s, ~8s...

## Installation

```(bash)
npm install bk-rabbitmq-rpc
```

## Getting started

On RPC server :

```javascript
const RabbitmqRPC = require('bk-rabbitmq-rpc');

const client = new RabbitmqRPC();

// create a your first service
const service = client.createService('myFirstService',{ autoStartConsume: true })

// handle a service method on rpc server
service.handle('myMethod', function(data){
	return data;
});

// or if you don't use option autoStartConsume for rpc server
service.startConsume();

```
On RPC Client :
```javascript
const RabbitmqRPC = require('bk-rabbitmq-rpc');

const client = new RabbitmqRPC();

// on rpc client ask the service method
client.request('myFirstService','myMethod',data).then((result) => {
	console.log(result);
})

```

## API Reference

### new RabbitmqRPC(options)
Return a new RabbitmqRPC client.
`options` are :
 * 	`url` : URL to connect to RabbitMQ (default : `amqp://guest:guest@localhost:5672/`)
 * `logLevel` : Log level (default : `info`)
 * `logName` : Log name for Bunyan = (default : `RabbitmqRPC`)
 * `exchangeName` : Exchange name for handle RPC request  (default : `RabbitmqRPC`)
 * `log` : Custom log instance (require to implement function trace, debug, info, warn and error)
 * `responseQueue` : If `true`, the responseQueue will be generated, it's possible to use a string to define. Use `false` for no responseQueue (default : `true`)
* `timeout` : Timeout for rpc request in ms (default : `10000`)

### {client} request(serviceName, method, data, options)
Request a method on a service.
* `serviceName` string for the service name.
* `method` is the method name, you have to use alphanum string for the method name.
* `data` is the data send to the client.
* `options` options for the request with the following properties :
	* `timeout` : specify timeout for rpc request in ms. Defautlt is the time specify in the constructor.
This function return a Promise.

### {client} apply(serviceName, method, data, options)
Request a method on a service but without response.
* `serviceName` string for the service name.
* `method` is the method name, you have to use alphanum string for the method name.
* `data` is the data send to the client.
* `options` options for the request with the following properties :
	* `timeout` : specify timeout for rpc request in ms. Warning : defautlt is no timeout !

This function return a Promise.

### {client} createService(serviceName, options)
Return a service object
* `serviceName` string for the service name.
* `options` are :
	* `autoStartConsume` : If the service start to consume RPC message at the beginning (default : `false`)
	* `limit` : Concurent limit request for the service. By default there is no limit.

### {service} handle(method,function(data) {})
Handle a method for the service.
* `method` is the method name, you have to use alphanum string for the method name.
* `function` is the function called. It can be a function or a promise. The return of the function is return to the client. The function take one arg whose is the data send by the client.

> !IMPORTANT! - if you don't use the option autoStartConsume: true on service constructor don't forget to call service.startConsume() to handle the RabbitmqRPC message.

## Contributing

First off, thanks for your interest and for wanting to contribute!

### Run tests

```bash
# With docker
npm run build-image #to build rabbitmq Image
npm run start-image #to start rabbitmq on localhost
# Or provide your own local rabbitmq install

# run tests
npm test

# run lint
npm run lint

# run coverage
npm run coverage
```

### TODO
 * Add better coverage test
 * Add better reconnection method => sometime the connections aren't really closed
 * ...

## License
MIT License

Copyright (c) 2017 Beekast
