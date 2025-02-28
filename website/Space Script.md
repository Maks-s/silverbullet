Space Script allows you to extend SilverBullet with JavaScript from within your space using `space-script` [[Blocks]]. It’s script... in (your) [[Spaces|space]]. Get it?

> **warning** **Experimental**
> This is an experimental feature that is under active development and consideration. Its APIs are likely to evolve, and the feature could potentially be removed altogether. Feel free to experiment with it and give feedback on [our community](https://community.silverbullet.md/).

> **warning** **Security**
> Space script allows for arbitrary JavaScript to be run in the client and server, there are security risks involved **if malicious users get write access to your space (folder)** or if you copy & paste random scripts from the Internet without understanding what they do.
> If this makes you very queazy, you can disable Space Script by setting the `SB_SPACE_SCRIPT` environment variable to `off`

# Creating scripts
Space scripts are defined by simply using `space-script` fenced code blocks in your space. You will get JavaScript [[Markdown/Syntax Highlighting]] for these blocks.

Here is a trivial example:

```space-script
silverbullet.registerFunction({name: "helloYeller"}, (name) => {
  return `Hello ${name}!`.toUpperCase();
})
```

You can now invoke this function in a template or query:

```template
{{helloYeller("Pete")}}
```

Upon client and server boot, all indexed scripts will be loaded and activated. To reload scripts on-demand, use the {[System: Reload]} command (bound to `Ctrl-Alt-r` for convenience).

If you use things like `console.log` in your script, you will see this output either in your server’s logs or browser’s JavaScript console (depending on where the script will be invoked).

# Runtime Environment & API
Space script is loaded directly in the browser environment on the client, and the Deno environment on the server.

While not very secure, some effort is put into running this code in a clean JavaScript environment, as such the following global variables are not available: `this`, `self`, `Deno`, `window`, and `globalThis`.

Depending on where code is run (client or server), a slightly different JavaScript API will be available. However, code should ideally primarily rely on the following explicitly exposed APIs:

* `silverbullet.registerFunction(definition, callback)`: registers a custom function (see [[#Custom functions]]).
* `silverbullet.registerCommand(definition, callback)`: registers a custom command (see [[#Custom commands]]).
* `syscall(name, args...)`: invoke a syscall (see [[#Syscalls]]).

Many useful standard JavaScript APIs are available, such as:

* [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) (making fetch calls directly from the browser on the client, and via Deno’s fetch implementation on the server)
* [Temporal](https://tc39.es/proposal-temporal/docs/) (implemented via a polyfill)

# Custom functions
SilverBullet offers a set of [[Functions]] you can use in its [[Template Language]]. You can extend this set of functions using space script using the `silverbullet.registerFunction` API.

Since template rendering happens on the server (except in [[Client Modes#Synced mode]]), this logic is typically executed on the server.

The `silverbullet.registerFunction` API takes two arguments:

* `options`: with currently just one option:
  * `name`: the name of the function to register
* `callback`: the callback function to invoke (can be `async` or not)

## Example
Even though a [[Functions#readPage(name)]] function already exist, you could implement it in space script as follows (let’s name it `myReadPage`) using the `syscall` API (detailed further in [[#Syscalls]]):

```space-script
silverbullet.registerFunction({name: "myReadPage"}, async (name) => {
  const pageContent = await syscall("space.readPage", name);
  return pageContent;
})
```

Note: this could be written more succinctly, but this demonstrates how to use `async` and `await` in space script as well.

This function can be invoked as follows:

```template
{{myReadPage("internal/test page")}}
```

# Custom commands
You can also define custom commands using space script. Commands are _always_ executed on the client.

Here is an example of defining a custom command using space script:

```space-script
silverbullet.registerCommand({name: "My First Command"}, async () => {
  await syscall("editor.flashNotification", "Hello there!");
});
```

You can run it via the command palette, or by pushing this [[Markdown/Command links|command link]]: {[My First Command]}

The `silverbullet.registerCommand` API takes two arguments:

* `options`:
  * `name`: Name of the command
  * `key` (optional): Keyboard shortcut for the command (Windows/Linux)
  * `mac` (optional): Mac keyboard shortcut for the command
  * `hide` (optional): Do not show this command in the command palette
  * `requireMode` (optional): Only make this command available in `ro` or `rw` mode.
* `callback`: the callback function to invoke (can be `async` or not)

# Syscalls
The primary way to interact with the SilverBullet environment is using “syscalls”. Syscalls expose SilverBullet functionality largely available both on the client and server in a safe way.

In your space script, a syscall is invoked via `syscall(name, arg1, arg2)` and usually returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) with the result.

Here are all available syscalls:

```template
{{#each @module in {syscall select replace(name, /\.\w+$/, "") as name}}}
## {{@module.name}}
{{#each {syscall where @module.name = replace(name, /\.\w+$/, "")}}}
* `{{name}}`
{{/each}}

{{/each}}
```
