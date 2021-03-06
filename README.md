# ng6-socket.io

[![npm version](https://badge.fury.io/js/ng6-socket-io.svg)](https://www.npmjs.com/package/ng6-socket-io)

[Socket.IO](http://socket.io/) module for Angular 6 and RxJS 6.

This is a fork of [ng-socket-io](https://github.com/bougarfaoui/ng-socket-io) as it wasn't being actively maintained. Credit to original author Bougarfaoui El Houcine.

## Install

`npm install ng6-socket-io`

## How to use

#### Angular 6 Note:

For use with Angular 6+ and RxJS 6+ add the following line to `polyfills.ts`

```ts
// src/polyfills.ts
(window as any).global = window;
```

### Import and configure SocketIoModule

```ts
//...
import { SocketIoModule, SocketIoConfig } from 'ng6-socket-io';

const config: SocketIoConfig = { url: 'http://localhost:8988', options: {} };

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, SocketIoModule.forRoot(config)],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

We need to configure `SocketIoModule` module using the object `config` of type `SocketIoConfig`, this object accepts two optional properties they are the same used here [io(url[, options])](https://github.com/socketio/socket.io-client/blob/master/docs/API.md#iourl-options).

Now we pass the configuration to the static method `forRoot` of `SocketIoModule`

### Using your socket Instance

The `SocketIoModule` provides now a configured `Socket` service that can be injected anywhere inside the `AppModule`.

```ts
import { Injectable } from '@angular/core';
import { Socket } from 'ng6-socket-io';

@Injectable()
export class ChatService {
  constructor(private socket: Socket) {}

  sendMessage(msg: string) {
    this.socket.emit('message', msg);
  }

  getMessage() {
    return this.socket.fromEvent('message').map(data => data.msg);
  }
}
```

### Using multiple sockets with different end points

In this case we do not configure the `SocketIoModule` directly using `forRoot`. What we have to do is: extend the `Socket` service, and call `super()` with the `SocketIoConfig` object type (passing `url` & `options` if any).

```ts
import { Injectable, NgModule, NgZone } from '@angular/core';
import { Socket } from 'ng6-socket-io';

@Injectable()
export class SocketOne extends Socket {
  constructor(ngZone: NgZone) {
    super(ngZone);
    this.init({ url: 'http://url_one:portOne', options: {} });
  }
}

@Injectable()
export class SocketTwo extends Socket {
  constructor(ngZone: NgZone) {
    super(ngZone);
    this.init({ url: 'http://url_two:portTwo', options: {} });
  }
  sendMessage(msg: string) {
    this.emit('message', msg);
  }

  getMessage() {
    return this.fromEvent('message').map(data => data.msg);
  }
}

@NgModule({
  declarations: [
    //components
  ],
  imports: [
    SocketIoModule
    //...
  ],
  providers: [SocketOne, SocketTwo],
  bootstrap: [
    /** AppComponent **/
  ]
})
export class AppModule {}
```

Now you can inject `SocketOne`, `SocketTwo` in any other services and / or components.

## API

Most of the functionality here you are already familiar with.

The only addition is the `fromEvent` method, which returns an `Observable` that you can subscribe to.

### `socket.on(eventName: string)`

Takes an event name and callback.
Works the same as in Socket.IO.

### `socket.removeListener(eventName: string, callback: Function)`

Takes an event name and callback.
Works the same as in Socket.IO.

### `socket.removeAllListeners(eventName: string)`

Takes an event name.
Works the same as in Socket.IO.

### `socket.emit(eventName:string, message: any, [callback: Function])`

Sends a message to the server.
Optionally takes a callback.
Works the same as in Socket.IO.

### `socket.fromEvent<T>(eventName: string): Observable<T>`

Takes an event name and returns an Observable that you can subscribe to.

### socket.fromEventOnce<T>(eventName: string): Promise<T>

Takes an event name, and returns a Promise instead of an Observable.
Works the same as `once` in Socket.IO.

You should keep a reference to the Observable subscription and unsubscribe when you're done with it.
This prevents memory leaks as the event listener attached will be removed (using `socket.removeListener`) ONLY and when/if you unsubscribe.

If you have multiple subscriptions to an Observable only the last unsubscription will remove the listener.

##### Example

You can also see this [example](https://github.com/vrn-dev/ng6-socket-io/tree/master/examples/chat-app) with express.js.

Go to /examples/chat-app/public and run `ng build`

Start both servers in separate terminals with `node app1.js` and access the page from `http://localhost:8988/`

```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule, NgZone } from '@angular/core';
import { AppComponent } from './app.component';

import { SocketIoModule, SocketIoConfig, Socket } from 'ng6-socket-io';

import { Socket1Service } from './socket1.service';
import { Socket2Service } from './socket2.service';

const config: SocketIoConfig = { url: 'http://localhost:8988', options: {} };

@Injectable({
  providedIn: 'root'
})
class ChatService {
  constructor(private socket: Socket) {}

  sendMessage(msg: string) {
    this.socket.emit('message', msg);
  }

  getMessage() {
    return this.socket.fromEvent<any>('message').map(data => data.msg);
  }

  close() {
    this.socket.disconnect();
  }
}

@Injectable({
  providedIn: 'root'
})
export class Socket1Service extends Socket {
  constructor(ngZone: NgZone) {
    super(ngZone);
    this.init({ url: 'http://localhost:8989', options: {} });
  }

  getMessage() {
    return this.fromEvent<any>('msg').pipe(map(data => data.msg));
  }

  sendMessage(msg: string) {
    this.emit('msg', msg);
  }
}

@Injectable({
  providedIn: 'root'
})
export class Socket2Service extends Socket {
  constructor(ngZone: NgZone) {
    super(ngZone);
    this.init({ url: 'http://localhost:8989', options: {} });
  }

  getMessage() {
    return this.fromEvent<any>('msg').pipe(map(data => data.msg));
  }

  sendMessage(msg: string) {
    this.emit('msg', msg);
  }
}

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, SocketIoModule.forRoot(config)],
  providers: [Socket1Service, Socket2Service],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

## LICENSE

MIT
