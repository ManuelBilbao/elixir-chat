⚠️ATTENTION!⚠️ This README is incomplete and in progress. Please, refer to the [latest version of this repo](https://github.com/ManuelBilbao/elixir-chat).

---

A very simple (but persistent) online chat tutorial, made with Elixir and Phoenix.

# 0. Requisites

1. Elixir ([installation page](http://elixir-lang.org/install.html)). 
  - Tested on version `1.13.2`.
  - Run `elixir -v` to check your version.
2. Phoenix framework ([installation page](https://hexdocs.pm/phoenix/installation.html)).
  - Tested on version `1.6.9`.
  - Run `mix phx.new --version` to check your version.

# 1. Create the app

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

# 2. Create the (WebSocket) _Channel_

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

# 3. Update the UI

## 3.1 Update the main content

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

## 3.2 Update the layout

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

# 4. Update the "Client" code in JS.

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