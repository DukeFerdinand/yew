## How to run

Make sure you're using a nightly Rust toolchain when compiling this project:

```bash
$ rustup override set nightly
```

Install all JS build dependencies with npm/yarn:

```sh
$ npm install # or yarn install
```

Then run in your terminal:

```sh
$ npm run start:dev # yarn start:dev
```

Then you can visit http://localhost:8000 in your favorite browser :)

### Editing note

If you're running VS Code, you may need to add this to your `.vscode/settings.json`:

```json
{
  "rust.target": "wasm32-unknown-unknown",
  "emmet.includeLanguages": {
    "rust": "html"
  }
}
```

## About

The purpose of this project is to show how to effectively pass state around your application in a reactive way, without throwing away the convenience of Yew's `Agent` system.

You can use this example as a minimal template for any of your web applications. The base for the state system here is a Yew `Agent` and a `Mutable` wrapper from `futures_signals`.

The flow is as follows:

- A component in the tree establishes a connection with the `Store` Agent.
- An instance of the `Store` agent is created and the `State` object is initialized with `Mutable` field(s).
- Store sends `StateInstance(State)` back to `App` (or other connected components on connect)
- Component can then subscribe to any updates it cares about

Below is an example showing how to subscribe to updates made to a `String` field.

```rust
impl App {
    fn register_state_handlers(&self) {
        let callback = self.link.callback(|ip| Msg::SetIp(ip));
        let state = self.state_ref.as_ref().unwrap();

        // use signal() for copy-able types
        // signal_cloned() for things like Strings that can't be copied
        let handler = state.ip.signal_cloned().for_each(move |u| {
            info!("{:?}", u); // from log crate
            callback.emit(u);
            ready(()) // from futures crate
        });

        // for_each converts the signals into futures
        // so you'll need to spawn the futures locally
        spawn_local(handler); // from wasm_bindgen_futures
    }
}

// ... rest of your component implementation
```

The corresponding `State` object in this case would look something like this:

```rust
struct State {
  ip: Mutable<Option<String>>
}
```

## Global Updates

The `Subscriber` component (along with add and remove buttons) in `App` show how the global state will be retained as long as a connection to `Store` is alive (the easiest way to do this is to maintain a connection to `Store` in `App` or the root component, even if `App` doesn't need to use anything in `State`).

Because of the nature of Rust's memory management, you'll need to keep a reference to your `Store` connection around in each component in order to start up your state subscriptions. I haven't come up with any ways around this yet, but please feel free to make a PR/issue regarding this :)

### For more info on `futures_signals`

You can use the tutorial for the library [here](https://docs.rs/futures-signals/0.3.15/futures_signals/tutorial/index.html).

There's a lot more to it than I've included, like `MutableVec` as a subscribable `Vec` type with its own set of update filters. You should check it out! :)