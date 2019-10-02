# Front-End Development Meetup

10-02-2019

## Creating a cli tool

Let's create a simple CLI tool

### Stating / Installating Project

```
npm init -y
npm install shelljs yargs yargonaut
```

### Create bin directory

```
mkdir bin
```

### Add CLI Utility entry

```
cd bin
touch mycli.js
```

edit mycli:

```
#!/usr/bin/env node

const {exec} = require('shelljs')
const yargs = require('yargs')
const yargonaut = require('yargonaut')

const ROOT = exec('npm root', {silent: true}).replace('node_modules', '').trim()

console.log(ROOT)
```

`silent` option does not echo program output to console.

edit package.json and add:

```
  "bin": {
    "mycli": "bin/mycli.js"
  },
```

also add/change scripts in package.json:

```
  "scripts": {
    "install-cli": "npm install -g ."
  },
```

#### Change directory back to root

```
cd ..
```

#### Install cli tool using npm:

```
npm run install-cli
```

#### Run mycli:

```
mycli
```

This will output the ROOT dir

### Add Yargs

#### Basic usage:

edit bin/mycli.js:

```
#!/usr/bin/env node

const {exec} = require('shelljs')
const yargs = require('yargs')
const yargonaut = require('yargonaut')

const ROOT = exec('npm root', {silent: true}).replace('node_modules', '').trim()

if (yargs.argv.hello) {
  console.log(`Hello ${yargs.argv.hello}`)
} else {
  console.log('Hello Console World')
}
```

run:

```
mycli
mycli --hello=Yargs
```

#### Advanced usage:

edit bin/mycli.js:

```
#!/usr/bin/env node

const {exec} = require('shelljs')
const yargs = require('yargs')
const yargonaut = require('yargonaut')

const ROOT = exec('npm root', {silent: true}).replace('node_modules', '').trim()

yargs
  .command('hello [whom]', 'say hello to someone', (yargs) => {
    yargs
      .positional('whom', {
        describe: 'whom do we say hello to???',
        default: 'World'
      })
  }, (argv) => {
    if (argv.verbose) console.info(`Say hello to :${argv.whom}`)
    console.log(`Hello ${argv.whom}`)
  })
  .option('verbose', {
    alias: 'v',
    default: false
  })
  .argv
```

run:

```
mycli
mycli hello
mycli hello Friends
mycli --help
mycli hello --help
mycli hello Friends -v=true
```

### Organizing and Advanced setup

We will use organize commands inside of a commands directory.

[https://github.com/yargs/yargs/blob/1b477454f87fd125184b3514360e23964a009478/docs/advanced.md#commanddirdirectory-opts](https://github.com/yargs/yargs/blob/1b477454f87fd125184b3514360e23964a009478/docs/advanced.md#commanddirdirectory-opts)

```
cd bin
mkdir commands
cd ..
```

Edit bin/mycli.js:

```
#!/usr/bin/env node

const {exec} = require('shelljs')
const yargs = require('yargs')
const yargonaut = require('yargonaut')

const ROOT = exec('npm root', {silent: true}).replace('node_modules', '').trim()

yargs
  .commandDir(`${ROOT}bin/commands/`)
  .option('verbose', {default: false})
  .argv

```

If we want to add the option for verbose for all commands we can add here.
[https://github.com/yargs/yargs/blob/1b477454f87fd125184b3514360e23964a009478/docs/api.md#optionskey-opt](https://github.com/yargs/yargs/blob/1b477454f87fd125184b3514360e23964a009478/docs/api.md#optionskey-opt)


Add first command:

```
cd bin/commands/
touch hello.js
cd ../../
```

Edit bin/commands/hello.js:

```
const builder = Object.freeze({
  whom: {
    default: 'World'
  }
})

const handler = (args) => console.log(`Hello ${args.whom || args.h}`)

module.exports = {
  command: ['hello [whom]', 'h'],
  describe: 'Says hello to a special someone',
  builder: builder,
  handler: handler,
}
```

Say hello:

```
mycli help
mycli hello help
mycli hello
mycli hello Friends
```

### Customize Help

Edit bin/mycli.js:

```
#!/usr/bin/env node

const {exec} = require('shelljs')
const yargs = require('yargs')
const yargonaut = require('yargonaut')

const ROOT = exec('npm root', {silent: true}).replace('node_modules', '').trim()

yargs
  .commandDir(`${ROOT}bin/commands/`)
  .option('verbose', {default: false})
  .help('Halp!')
  .argv

```

run

```
mycli help
mycli Halp!
```

Not useful in our case, let's remove it. Edit bin/mycli.js

```
#!/usr/bin/env node

const {exec} = require('shelljs')
const yargs = require('yargs')
const yargonaut = require('yargonaut')

const ROOT = exec('npm root', {silent: true}).replace('node_modules', '').trim()

yargs
  .commandDir(`${ROOT}bin/commands/`)
  .option('verbose', {default: false})
  .argv

```

### Validate that Arguments exist

Edit bin/mycli.js:

```
#!/usr/bin/env node

const {exec} = require('shelljs')
const yargs = require('yargs')
const yargonaut = require('yargonaut')

const ROOT = exec('npm root', {silent: true}).replace('node_modules', '').trim()

const validate = (argv, opts) => {
  if (argv._.length > 0) return true
  throw new Error('Please make sure to specify at least one command')
}

yargs
  .check(validate)
  .commandDir(`${ROOT}bin/commands/`)
  .option('verbose', {default: false})
  .argv

```

The underscore (`_`) property of argv in validate function is an array of commands passed
into Yargs, and allows us to check if any were given.

run

```
mycli
```

The last line should be the error.

### Add Better Error Handling

Edit bin/mycli.js:

```
#!/usr/bin/env node

const {exec} = require('shelljs')
const yargs = require('yargs')
const yargonaut = require('yargonaut')

const ROOT = exec('npm root', {silent: true}).replace('node_modules', '').trim()

const errorHandler = (msg, err, args) => {
  if (msg || err) {
    console.error(msg || err)
    console.log(args.help())
  }

  if (args.verbose) console.error(err.stack)
  process.exit(1)
}

const validate = (argv, opts) => {
  console.log(argv)
  if (argv._.length > 0) return true
  throw new Error('Please make sure to specify at least one command')
}

yargs
  .check(validate)
  .commandDir(`${ROOT}bin/commands/`)
  .option('verbose', {default: false})
  .fail(errorHandler)
  .argv

```

Here we added the `errorHandler` function and added `.fail(errorHandler)` to the method
chain for Yargs.

run

```
mycli
```

You should now see the errors moved to the beginning, before the help docs.

### Make it pretty

Add yargonaut to add some color. Edit bin/mycli.js

```
#!/usr/bin/env node

const {exec} = require('shelljs')
const yargs = require('yargs')
const yargonaut = require('yargonaut')

const ROOT = exec('npm root', {silent: true}).replace('node_modules', '').trim()

const errorHandler = (msg, err, args) => {
  if (msg || err) {
    console.error(yargonaut.chalk().red(msg || err))
    console.log(args.help())
  }

  if (args.verbose) console.error(yargonaut.chalk().red(err.stack))
  process.exit(1)
}

const validate = (argv, opts) => {
  if (argv._.length > 0) return true
  throw new Error('Please make sure to specify at least one command')
}

yargonaut.style('blue')
  .style('yellow', 'required')
  .helpStyle('green')
  .errorsStyle('red.bold')


yargs
  .check(validate)
  .commandDir(`${ROOT}bin/commands/`)
  .option('verbose', {default: false})
  .fail(errorHandler)
  .argv

```

In order for our error to show up in red, we had to hack around and directly use
chalk (available by itself or through yargonaut, as in the example).

### Simplifying Validation

We can use `demandCommand` chained method with the minimum of 1 item set as an option.
This will allow us to remove our custom validation.

Edit bin/mycli.js:

```
#!/usr/bin/env node

const {exec} = require('shelljs')
const yargs = require('yargs')
const yargonaut = require('yargonaut')

const ROOT = exec('npm root', {silent: true}).replace('node_modules', '').trim()

const errorHandler = (msg, err, args) => {
  if (msg || err) {
    console.error(yargonaut.chalk().red(msg || err))
    console.log(args.help())
  }

  if (args.verbose) console.error(yargonaut.chalk().red(err.stack))
  process.exit(1)
}

yargonaut.style('blue')
  .style('yellow', 'required')
  .helpStyle('green')
  .errors('Calvin S')
  .errorsStyle('red.bold')

yargs
  .commandDir(`${ROOT}bin/commands/`)
  .option('verbose', {default: false})
  .demandCommand(1)
  .argv

```

By using demandCommmand without setting a custom error message makes Yargonaut automatically
pick up the default color / font setting for errors.

run:

```
mycli
```

Unfortunately this is a limitation of Yargonaut. If we want a custom message and color, we need
to add the following:

```
.demandCommand(1, yargonaut.chalk().red.bold('You need at least one command before moving on'))
```

Edit bin/mycli.js:

```
#!/usr/bin/env node

const {exec} = require('shelljs')
const yargs = require('yargs')
const yargonaut = require('yargonaut')

const ROOT = exec('npm root', {silent: true}).replace('node_modules', '').trim()

const errorHandler = (msg, err, args) => {
  if (msg || err) {
    console.error(yargonaut.chalk().red(msg || err))
    console.log(args.help())
  }

  if (args.verbose) console.error(yargonaut.chalk().red(err.stack))
  process.exit(1)
}

const validate = (argv, opts) => {
  if (argv._.length > 0) return true
  throw new Error('Please make sure to specify at least one command')
}

yargonaut.style('blue')
  .style('yellow', 'required')
  .helpStyle('green')
  .errorsStyle('red.bold')

yargs
  .commandDir(`${ROOT}bin/commands/`)
  .option('verbose', {default: false})
  .demandCommand(1, yargonaut.chalk().red.bold('You need at least one command before moving on'))
  .argv

```

### Widen the output

Using `.wrap(num)` we can widen the output.

Edit bin/mycli.js:

```
#!/usr/bin/env node

const {exec} = require('shelljs')
const yargs = require('yargs')
const yargonaut = require('yargonaut')

const ROOT = exec('npm root', {silent: true}).replace('node_modules', '').trim()

const errorHandler = (msg, err, args) => {
  if (msg || err) {
    console.error(yargonaut.chalk().red(msg || err))
    console.log(args.help())
  }

  if (args.verbose) console.error(yargonaut.chalk().red(err.stack))
  process.exit(1)
}

const validate = (argv, opts) => {
  if (argv._.length > 0) return true
  throw new Error('Please make sure to specify at least one command')
}

yargonaut.style('blue')
  .style('yellow', 'required')
  .helpStyle('green')
  .errorsStyle('red.bold')

yargs
  .commandDir(`${ROOT}bin/commands/`)
  .option('verbose', {default: false})
  .demandCommand(1, yargonaut.chalk().red.bold('You need at least one command before moving on'))
  .wrap(100)
  .argv

```

### Add ROOT to Config and App Version

Edit bin/mycli.js:

```
#!/usr/bin/env node

const {exec} = require('shelljs')
const yargs = require('yargs')
const yargonaut = require('yargonaut')

const ROOT = exec('npm root', {silent: true}).replace('node_modules', '').trim()

const errorHandler = (msg, err, args) => {
  if (msg || err) {
    console.error(yargonaut.chalk().red(msg || err))
    console.log(args.help())
  }

  if (args.verbose) console.error(yargonaut.chalk().red(err.stack))
  process.exit(1)
}

const validate = (argv, opts) => {
  if (argv._.length > 0) return true
  throw new Error('Please make sure to specify at least one command')
}

yargonaut.style('blue')
  .style('yellow', 'required')
  .helpStyle('green')
  .errorsStyle('red.bold')

yargs
  .config({root: ROOT})
  .commandDir(`${ROOT}bin/commands/`)
  .option('verbose', {default: false})
  .demandCommand(1, yargonaut.chalk().red.bold('You need at least one command before moving on'))
  .version(require(`${ROOT}package.json`).version)
  .wrap(100)
  .argv

```

### Serve a Local Website

Create a simple html file.

```
mkdir src
cd src
touch index.html
cd ..
```

Edit src/index.html:

```
<!doctype html>
<html>
<head>
  <title>My App</title>
</head>
<body>
  <h1>Hello Website World</h1>
</body>
</html>

```

Install `serve` package

```
npm install serve
```

Add new command

```
cd bin/commands
touch serve.js
cd ../../
```

Edit bin/serve.js:

```
const serveHandler = require('serve-handler')
const http = require('http');

const builder = Object.freeze({
  port: {
    alias: 'p',
    default: 1337,
  },
})

const handler = (args) => {
  const port = args.port || args.p
  const server = http.createServer((request, response) => {
    serveHandler(request, response, {
      public: `${args.root}src/`,
      renderSingle: true,
    })
  })

  server.listen(port, () => {
    const address = server.address().address
    const networkHost = address === '::' ? '0.0.0.0' : address

    console.log(`Local:             http://localhost:${port}`)
    console.log(`Network Address:   http://${networkHost}:${port}`)
  })
}

module.exports = {
  command: ['serve', 'start', 's'],
  describe: 'starts a local server',
  builder: builder,
  handler: handler,
}
```
