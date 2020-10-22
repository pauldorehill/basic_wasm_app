# Basic Rust Wasm App

This is an example of a simple way to build and run a `rust` + `wasm` app using only `cargo` and `wasm-bindgen`. When working this way there is one thing you have to remember: **the `wasm-bindgen` used by your app and the `wasm-bindgen-cli` versions must be the same.**
So I'd suggest fixing your `wasm-bindgen` in the `Cargo.toml`:

```toml
wasm-bindgen = "=0.2.68"
```
then installing `wasm-bindgen-cli` a matching version with cargo:
```
cargo install --version 0.2.68 -- wasm-bindgen-cli
```
Details of the [wasm-bindgen-cli](https://rustwasm.github.io/docs/wasm-bindgen/reference/cli.html) are here or after installing just run `wasm-bindgen -h`.

## Compiling to Wasm
First use `cargo` to compile to the `wasm32-unknown-unknown` target:
```
cargo build --target wasm32-unknown-unknown
```
This will generate a `.wasm` file at `./target/wasm32-unknown-unknown/debug/basic_wasm_app.wasm`.

Then use `wasm-bindgen-cli` to create the required `js` glue code for loading wasm in web browser:
```
wasm-bindgen ./target/wasm32-unknown-unknown/debug/basic_wasm_app.wasm --target web --no-typescript --out-dir ./dist/js
```
Which generates the following two files in the `dist/js` folder:
- `basic_wasm_app_bg.wasm` - your app compiled to wasm
- `basic_wasm_app.js` - required js glue code to load a wasm module in a browser

Now all you need to do is use some very basic `js` / `html` to load and run the wasm:

```html
<script type="module">
    import init from './js/basic_wasm_app.js';
    init();
</script>
```
## Running

Now you just need to serve the `dist/index.html` file into a browser and check the `console` for "Hello world".

### Open the file directly

If you want to just open the file url (no server) you will likely run into a [CORS error](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/Errors/CORSRequestNotHttp). This is an easy fix with `Firefox`: browse to `about:config` and set `privacy.file_unique_origin = false`. Other browsers can be done... just search the internet.

Then just browse to `file:///...your_local_path.../basic_wasm_app/dist/index.html`.

### Use a server

Spin up your favourite server and navigate to `/dist/index.html`. An easy rust based way to start is [basic-http-server](https://github.com/brson/basic-http-server).

```
cargo install basic-http-server
basic-http-server
```
And go to `http://127.0.0.1:4000/dist/index.html`

## Notes

### Running WebAssembly
Loading [WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly/Loading_and_running) in a browser is explained here. The `wasm-bindgen-cli` generates all this code for you and it exports a final
```async function init(input) {...}``` in the generated `js` glue code: in the script above all we did was import this function and then call it.

### Why the app starts upon loading
As the `fn main()` is annotated with [#[wasm_bindgen(start)]](https://rustwasm.github.io/docs/wasm-bindgen/reference/attributes/on-rust-exports/start.html?highlight=start#start) as soon as the wasm loads it runs. If you wanted to do it yourself:

`lib.rs`
```rust
#[wasm_bindgen]
pub fn my_manual_start() {
    web_sys::console::log_1(&"hello world".into());
}
```
`index.html`
```html
<script type="module">
    import init, { my_manual_start } from './js/basic_wasm_app.js';
    async function run() {
        await init();
        my_manual_start();
    }
    run();
</script>
```

