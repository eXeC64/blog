+++
date = "2016-03-20T12:03:00Z"
title = "Websockets with Redux"

+++

## Introduction

Lately, I've been devoting a fair bit of my time to working on a single page
web application, for which I've been using the excellent [React.js](https://facebook.github.io/react/)
and [Redux](http://redux.js.org) frameworks for managing my UI, and the client
side state. An honourable mention goes to [Material UI](http://material-ui.com)
for the nice React components I built the interface with.

For various reasons, the server component of this application had to be built in
C++, and it was required that the server could push events or data to the client.
While at first I toyed with the idea of a RESTful HTTP API with either long-polling
or websockets for events, I quickly abandoned this approach in favour of a pure
websocket API.

Combined with the power of React+Redux, using websockets for the application led
to a very smooth, and pain free development experience. The purpose of this post
is to share the approach I took to handling the websocket connection with Redux,
and my experiences developing with this architecture.

This post assumes some rudimentary knowledge of Facebook's Flux architecture,
and how Redux implements that, specifically the unidirectional data flow.

## Websocket Middleware

In a Redux based application, the typical path of user interaction is as follows:

1. User presses button
2. Action is created
3. Action is delivered to the reducver
4. The reducer updates the store's state
5. The UI is updated with the new state

For user interactions that have to communicate with the server, a detour is
clearly needed in step 3. Fortunately, Redux provides a way to hook into actions
at that step, namely [middleware](http://redux.js.org/docs/advanced/Middleware.html).

I created a custom middleware responsible for opening a websocket to the server,
maintining it, and intercepting or dispatching actions as appropriate. That is,
any actions that needed to be dealt with by the server were intercepted at
step 3, and the appropriate message sent to the server. When a message is
received from the server, an equivalent action would be dispatched to the reducer
to update the state, as per the new information received from the server.

The new path of user interaction is as follows:

1. User presses button
2. Action is created
3. Action is delivered to socket middleware. If needed, the middleware re-routes
   the action through the websocket, and does not deliver it to the reducer.
   Otherwise, action is given to reducer as usual.
4. The reducer updates the store's state
5. The UI is updated with the new state

With additional path for when a message is received from the server:

1. The socket middleware receives a message from the websocket
2. The socket middleware creates an action, and dispatches it to the reducer
3. The reducer updates the store's state
4. The UI is updated with the new state

This design is clean, and easy to reason about. For both the user and redux, the
detour and interception of remote actions is invisible. Additionally, since all
redux actions are serializable, you can record and replay actions that are
delivered to the reducer, or to the middleware, providing excellent debugging
opportunities.

To help demonstrate this solution, I've adapted my middleware to provide an
example websocket middleware anyone can base theirs off:

### socketMiddleware.js
```javascript

import actions from './actions'

const socketMiddleware = (function(){ 
  var socket = null;

  const onOpen = (ws,store,token) => evt => {
    //Send a handshake, or authenticate with remote end

    //Tell the store we're connected
    store.dispatch(actions.connected());
  }

  const onClose = (ws,store) => evt => {
    //Tell the store we've disconnected
    store.dispatch(actions.disconnected());
  }

  const onMessage = (ws,store) => evt => {
    //Parse the JSON message received on the websocket
    var msg = JSON.parse(evt.data);
    switch(msg.type) {
      case "CHAT_MESSAGE":
        //Dispatch an action that adds the received message to our state
        store.dispatch(actions.messageReceived(msg));
        break;
      default:
        console.log("Received unknown message type: '" + msg.type + "'");
        break;
    }
  }

  return store => next => action => {
    switch(action.type) {

      //The user wants us to connect
      case 'CONNECT':
        //Start a new connection to the server
        if(socket != null) {
          socket.close();
        }
        //Send an action that shows a "connecting..." status for now
        store.dispatch(actions.connecting());

        //Attempt to connect (we could send a 'failed' action on error)
        socket = new WebSocket(action.url);
        socket.onmessage = onMessage(socket,store);
        socket.onclose = onClose(socket,store);
        socket.onopen = onOpen(socket,store,action.token);

        break;

      //The user wants us to disconnect
      case 'DISCONNECT':
        if(socket != null) {
          socket.close();
        }
        socket = null;

        //Set our state to disconnected
        store.dispatch(actions.disconnected());
        break;

      //Send the 'SEND_MESSAGE' action down the websocket to the server
      case 'SEND_CHAT_MESSAGE':
        socket.send(JSON.stringify(action));
        break;

      //This action is irrelevant to us, pass it on to the next middleware
      default:
        return next(action);
    }
  }

})();

export default socketMiddleware
```

This fairly straightforward middleware handles our entire websocket with ease.
It produces and consumes actions as needed, and allows us to effortlessly
update the local state based on information received from the server, and to
inform the server of any user actions, all without relying on messy callbacks
or violating the unidirectional data flow.

I have found this approach to be very powerful, and would highly recommend it
to anyone considering building a web application with websockets.
