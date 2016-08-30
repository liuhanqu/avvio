# boot-in-the-arse &nbsp;&nbsp;[![Build Status](https://travis-ci.org/mcollina/boot-in-the-arse.svg)](https://travis-ci.org/mcollina/boot-in-the-arse)

Asynchronous bootstrapping made easy. Wait for all components/plugins to start, and then start your whole app.

`boot-in-the-arse` is fully reentrant and graph-based. You can load
components/plugins _within_ plugins, and be still sure that things will
happen in the right order.

[![JavaScript Style Guide](https://cdn.rawgit.com/feross/standard/master/badge.svg)](https://github.com/feross/standard)

## Example

```js
'use strict'

const boot = require('boot-in-the-arse')()

boot
  .use(first, { hello: 'world' })
  .after((cb) => {
    console.log('after first and second')
    cb()
  })
  .use(third, (err) => {
    if (err) {
      console.log('something bad happened')
      console.log(err)
    }

    console.log('third plugin loaded')
  })
  .ready(function () {
    console.log('application booted!')
  })

function first (instance, opts, cb) {
  console.log('first loaded', opts)
  instance.use(second, cb)
}

function second (instance, opts, cb) {
  console.log('second loaded')
  process.nextTick(cb)
}

function third (instance, opts, cb) {
  console.log('third loaded')
  cb()
}
```

## API

### boot([instance], [started])

Start a booting sequence.

`instance` will be used as the first
argument of all plugins loaded and `use`, `after` and `ready` 
function will be
added to that object, keeping the support for the chainable API:

```js
const server = {}

require('boot-in-the-arse')(server)

server.use(function first (s, opts, cb) {
  // s is the same of server
  s.use(function second (s, opts, cb) {
    cb()
  }, cb)
}).after(function (cb) {
  // after first and second are finished
  cb()
})
```

Options:

* `expose`: a key/value property to change how `use`, `after` and `ready` are exposed.

Events:

* `'error'`  if something bad happens
* `'start'`  when the application starts

The `boot` function can be used also as a
constructor to inherits from.

### app.use(func, [opts], [cb])

Loads a functions asynchronously. The function must have the
signature:

```js
function plugin (server, opts, done) {
  done()
}
```

`done` must be called only once.

Returns the instance on which `use` is called, to support a
chainable API.

### app.after(func([done]), [cb])

Calls a functon after all the previously defined plugins are loaded, including
all their dependencies. The `'start'` event is not emitted yet.

```js
boot.after(function (done) {
  done()
})
```

`done` must be called only once.

Returns the instance on which `after` is called, to support a
chainable API.

### app.ready(func([done]))

Calls a functon after all the plugins and `after` call are
completed, but befer `'start'` is emitted. `ready` callbacks are
executed one at a time.

```js
boot.ready(function (done) {
  done()
})
```

`done` must be called only once.

Returns the instance on which `ready` is called, to support a
chainable API.

### boot.express(app)

Same as:

```js
const app = express()

boot(app, {
  expose: {
    use: 'load'
  }
})
```

## License

MIT
