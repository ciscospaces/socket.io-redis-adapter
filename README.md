# Socket.IO Redis adapter

The `@ciscospaces/redis-adapter` package allows unicast packets between multiple Socket.IO servers.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/adapter_dark.png">
  <img alt="Diagram of Socket.IO packets forwarded through Redis" src="./assets/adapter.png">
</picture>


## Installation

```
npm install @ciscospaces/redis-adapter
```

## Usage

### With the `redis` package

```js

import { createClient } from "redis";
import { Server } from "socket.io";
import { createAdapter } from "@ciscospaces/redis-adapter";

const pubClient = createClient({ url: "redis://localhost:6379" });
const subClient = pubClient.duplicate();
const runningNodeInstanceId = EC2InstanceId;
const externalNamespace = '/api/v1/namespace';

await Promise.all([
  pubClient.connect(),
  subClient.connect()
]);


/**
 * Example: 
 * instanceRoutingFullNameSpace = 'socket.io#/instanceid-i-0ee1956361f484f09#room:123#
 * instanceRoutingNameSpace = 'socket.io#/instanceid-i-0ee1956361f484f09#
 * 
 * 
 */
function instanceBasedMessageRouteFunc (instanceRoutingFullNameSpace, instanceRoutingNameSpace) {
  let overrideChannel;
  if (instanceRoutingFullNameSpace.indexOf(cachedRoutingNamespaceVar) !== -1) {
    overrideChannel = instanceRoutingFullNameSpace.replace(runningNodeInstanceId, externalNamespace);
  }
  return overrideChannel;
}
const io = new Server({
  adapter: createAdapter(pubClient, subClient, { instanceBasedMessageDelivery: instanceBasedMessageRouteFunc})
});

io.of(runningNodeInstanceId);

io.of(externalNamespace).on('connection', (socket) => {
  console.log('connected to instance namespace');
  socket.on('message', (data) => {
    console.log('message received', data);
  });
})

io.listen(3000);
```

## License

[MIT](LICENSE)
