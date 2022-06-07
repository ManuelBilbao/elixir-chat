A very simple (but persistent) online chat tutorial, made with Elixir and Phoenix.

# Index

- [Requisites](#requisites)
- [Steps](#steps)
  - [1. Create the app](#1-create-the-app)
  - [2. Create the (WebSocket) _Channel_](#2-create-the-websocket-channel)
  - [3. Update the UI](#3-update-the-ui)
  - [4. Update the "Client" code in JS.](#4-update-the-client-code-in-js)
  - [Checkpoint!](#checkpoint)
  - [5. Generate database schema](#5-generate-database-schema)
  - [6. Run the Ecto migration](#6-run-the-ecto-migration)
  - [7. Insert messages into database](#7-insert-messages-into-database)
  - [8. Load existing messages from database](#8-load-existing-messages-from-database)
  - [9. Send existing messages to the client when they join](#9-send-existing-messages-to-the-client-when-they-join)
- [Credits](#credits)

# Requisites

1. Elixir ([installation page](http://elixir-lang.org/install.html)). 
  - Tested on version `1.13.2`.
  - Run `elixir -v` to check your version.
2. Phoenix framework ([installation page](https://hexdocs.pm/phoenix/installation.html)).
  - Tested on version `1.6.9`.
  - Run `mix phx.new --version` to check your version.

# Steps

## 1. Create the app

Run the following command to create the app:

```bash
mix phx.new chat --database sqlite3
```

When asked to "_Fetch and install dependencies? [Yn]_", type _Y_ and Enter. The output should look like this:

```
Fetch and install dependencies? [Yn] y
* running mix deps.get
* running mix deps.compile

We are almost there! The following steps are missing:

    $ cd chat

Then configure your database in config/dev.exs and run:

    $ mix ecto.create

Start your Phoenix app with:

    $ mix phx.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phx.server
```

Enter the new directory (`cd chat`) and run the following command to setup:

```bash
mix setup
```

### Safe check

At this point, you should be able to start the server and view the Phoenix default page. You can test it with:

```bash
mix phx.server
```

The server should be started at `localhost:4000`.

## 2. Create the (WebSocket) _Channel_

Generate the (WebSocket) channel to be used in the chat app:

```bash
mix phx.gen.channel Room
```

> When prompted to confirm the creation, type _Y_.

Open the file `lib/chat_web/channels/user_socket.ex`. Change the line 11:

```elixir
channel "room:*", ChatWeb.RoomChannel
```

to:

```elixir
channel "room:lobby", ChatWeb.RoomChannel
```

Your file should be like this: [user_socket.ex](https://github.com/ManuelBilbao/elixir-chat/blob/5a9b51136da37a620a57f00400ab303e3ba1e1dd/lib/chat_web/channels/user_socket.ex)

Add the socket handler in `lib/chat_web/endpoint.ex` with this line:

```elixir
socket "/socket", ChatWeb.UserSocket, websocket: true, longpoll: false
```

Your file should be like this: [endpoint.ex](https://github.com/ManuelBilbao/elixir-chat/blob/a70acc431a7df8250ace15ec955eac50940db9ca/lib/chat_web/endpoint.ex)

## 3. Update the UI

### 3.1 Update the main content

Open file `lib/chat_web/templates/page/index.html.heex` and replace all the content with:

```html
<!-- The list of messages will appear here: -->
<ul id="msg-list" style="list-style: none; min-height:200px;">
</ul>

<div class="row">
  <div class="col-xs-3" style="width: 20%; margin-left: 0;">
    <input type="text" id="name" class="form-control" placeholder="Your Name" style="border: 1px black solid; font-size: 1.3em;" autofocus>
  </div>
  <div class="col-xs-9" style="width: 100%; margin-left: 1%; ">
    <input type="text" id="msg" class="form-control" placeholder="Your Message" style="border: 1px black solid; font-size: 1.3em;">
  </div>
</div>
```

Your file should be like this: [index.html.heex](https://github.com/ManuelBilbao/elixir-chat/blob/f7e4ad9bfced2e7c73943188b2dee3bd8ff63e67/lib/chat_web/templates/page/index.html.heex)

### 3.2 Update the layout

Open file `lib/chat_web/templates/layout/root.html.heex`. Replace all the content inside the `<header></header>` tags with:

```html
<section class="container">
  <nav role="navigation">
    <h1 style="padding-top: 15px">Chat Example</h1>
  </nav>
    <img src={Routes.static_path(@conn, "/images/phoenix.png")}
    width="500px" alt="Phoenix Framework Logo" />
</section>
```

Your file should be like this: [root.html.heex](https://github.com/ManuelBilbao/elixir-chat/blob/f7e4ad9bfced2e7c73943188b2dee3bd8ff63e67/lib/chat_web/templates/layout/root.html.heex)

### Safe check

At this point, if you run the server (remember, `mix phx.server`), you should see the new page with the "Chat Example" title and two input fields.

## 4. Update the "Client" code in JS.

Open file `assets/js/app.js`. Replace all the code with:

```js
// We import the CSS which is extracted to its own file by esbuild.
// Remove this line if you add a your own CSS build pipeline (e.g postcss).
import "../css/app.css"

import socket from "./user_socket.js"

let channel = socket.channel('room:lobby', {}); // connect to chat "room"

channel.on('shout', function (payload) { // listen to the 'shout' event
  let li = document.createElement("li"); // create new list item DOM element
  let name = payload.name || 'guest';    // get name from payload or set default
  li.innerHTML = '<b>' + name + '</b>: ' + payload.message; // set li contents
  ul.appendChild(li);                    // append to list
});

channel.join(); // join the channel.


let ul = document.getElementById('msg-list');        // list of messages.
let name = document.getElementById('name');          // name of message sender
let msg = document.getElementById('msg');            // message input field

// "listen" for the [Enter] keypress event to send a message:
msg.addEventListener('keypress', function (event) {
  if (event.keyCode == 13 && msg.value.length > 0) { // don't sent empty msg.
    channel.push('shout', { // send the message to the server on "shout" channel
      name: name.value || "guest",     // get value of "name" of person sending the message. Set guest as default
      message: msg.value    // get message text (value) from msg input field.
    });
    msg.value = '';         // reset the message input field for next message.
  }
});
```

Your file should be like this: [app.js](https://github.com/ManuelBilbao/elixir-chat/blob/c4390c46edc1a890d45a8d6f5142b885e730f84b/assets/js/app.js)

To avoid console errors in the browser, search this lines in `assets/js/user_socket.js` and comment it:

```js
let channel = socket.channel("topic:subtopic", {})
channel.join()
  .receive("ok", resp => { console.log("Joined successfully", resp) })
  .receive("error", resp => { console.log("Unable to join", resp) })
```

Your file should be like this: [user_socket.js](https://github.com/ManuelBilbao/elixir-chat/blob/c4390c46edc1a890d45a8d6f5142b885e730f84b/assets/js/user_socket.js)

## Checkpoint!

At this point, you have a totally functional online chat. Go ahead and check it! (`mix phx.server`). Now we are going to add it persistance so the messages don't get missed on reloads.

## 5. Generate database schema

Run the following command:

```bash
mix phx.gen.schema Message messages name:string message:string
```

With this, you have generated a file describing the database table.

## 6. Run the Ecto migration

Now, we have to apply those migrations to get it reflected in the database.

```bash
mix ecto.migrate
```

## 7. Insert messages into database

Open the `lib/chat_web/channels/room_channel.ex` file and inside the `handle_in/3` function, insert this line:

```elixir
Chat.Message.changeset(%Chat.Message{}, payload) |> Chat.Repo.insert
```

Your file should be like this: [room_channel.ex](https://github.com/ManuelBilbao/elixir-chat/blob/19f5b51b1186e9d12e93c8935812a60adf6dadb6/lib/chat_web/channels/room_channel.ex)

## 8. Load existing messages from database

Open the file `lib/chat/messages.ex`. Add an import to `Ecto.Query` and the following function:

```elixir
def get_messages(limit \\ 20) do
  Chat.Message
  |> limit(^limit)
  |> order_by(desc: :inserted_at)
  |> Chat.Repo.all()
end
```

Your file should be like this: [messages.ex](https://github.com/ManuelBilbao/elixir-chat/blob/main/lib/chat/message.ex)

## 9. Send existing messages to the client when they join

Open the `lib/chat_web/channels/room_channel.ex` file and create a new function:

```elixir
def handle_info(:after_join, socket) do
  Chat.Message.get_messages()
  |> Enum.reverse() # revers to display the latest message at the bottom of the page
  |> Enum.each(fn msg -> push(socket, "shout", %{
      name: msg.name,
      message: msg.message,
    }) end)
  {:noreply, socket} # :noreply
end
```

Then update the `join` function adding this line inside the `if` statement:

```elixir
send(self(), :after_join)
```

Your file should be like this: [room_channel.ex](https://github.com/ManuelBilbao/elixir-chat/blob/e12e67ed86f0ed7122d47844d70af7a64ab3e0ea/lib/chat_web/channels/room_channel.ex)

# Credits

This guide is based on [dwyl/phoenix-chat-example](https://github.com/dwyl/phoenix-chat-example), and slightly modified to work with the Elixir and Phoenix versions previously mentioned. There it's more info about each command and code lines, as well as other things like tests and deployment.