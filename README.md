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

At this point you should be able to start the server and view the Phoenix default page. You can test it with:

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

Open the file `/lib/chat_web/channels/user_socket.ex`. Change the line 11:

```elixir
channel "room:*", ChatWeb.RoomChannel
```

to:

```elixir
channel "room:lobby", ChatWeb.RoomChannel
```

Your file should be like this: [user_socket.ex](https://github.com/ManuelBilbao/elixir-chat/blob/5a9b51136da37a620a57f00400ab303e3ba1e1dd/lib/chat_web/channels/user_socket.ex)
