# morgan

[![NPM Version][npm-version-image]][npm-url]
[![NPM Downloads][npm-downloads-image]][npm-url]
[![Build Status][travis-image]][travis-url]
[![Test Coverage][coveralls-image]][coveralls-url]

HTTP request logger middleware for node.js

> Named after [Dexter](http://en.wikipedia.org/wiki/Dexter_Morgan), a show you should not watch until completion.

## API

<!-- eslint-disable no-unused-vars -->

```js
var morgan = require('morgan')
```

### morgan(format, options)

Create a new morgan logger middleware function using the given `format` and `options`.
The `format` argument may be a string of a predefined name (see below for the names),
a string of a format string, or a function that will produce a log entry.

The `format` function will be called with three arguments `tokens`, `req`, and `res`,
where `tokens` is an object with all defined tokens, `req` is the HTTP request and `res`
is the HTTP response. The function is expected to return a string that will be the log
line, or `undefined` / `null` to skip logging.

#### Using a predefined format string

<!-- eslint-disable no-undef -->

```js
morgan('tiny')
```

#### Using format string of predefined tokens

<!-- eslint-disable no-undef -->

```js
morgan(':method :url :status :res[content-length] - :response-time ms')
```

#### Using a custom format function

<!-- eslint-disable no-undef -->

``` js
morgan(function (tokens, req, res) {
  return [
    tokens.method(req, res),
    tokens.url(req, res),
    tokens.status(req, res),
    tokens.res(req, res, 'content-length'), '-',
    tokens['response-time'](req, res), 'ms'
  ].join(' ')
})
```

#### Options

Morgan accepts these properties in the options object.

##### immediate

Write log line on request instead of response. This means that a requests will
be logged even if the server crashes, _but data from the response (like the
response code, content length, etc.) cannot be logged_.

##### skip

Function to determine if logging is skipped, defaults to `false`. This function
will be called as `skip(req, res)`.

<!-- eslint-disable no-undef -->

```js
// EXAMPLE: only log error responses
morgan('combined', {
  skip: function (req, res) { return res.statusCode < 400 }
})
```

##### stream

Output stream for writing log lines, defaults to `process.stdout`.

#### Predefined Formats

There are various pre-defined formats provided:

##### combined

Standard Apache combined log output.

```
:remote-addr - :remote-user [:date[clf]] ":method :url HTTP/:http-version" :status :res[content-length] ":referrer" ":user-agent"
```

##### common

Standard Apache common log output.

```
:remote-addr - :remote-user [:date[clf]] ":method :url HTTP/:http-version" :status :res[content-length]
```

##### dev

Concise output colored by response status for development use. The `:status`
token will be colored green for success codes, red for server error codes,
yellow for client error codes, cyan for redirection codes, and uncolored
for information codes.

```
:method :url :status :response-time ms - :res[content-length]
```

##### short

Shorter than default, also including response time.

```
:remote-addr :remote-user :method :url HTTP/:http-version :status :res[content-length] - :response-time ms
```

##### tiny

The minimal output.

```
:method :url :status :res[content-length] - :response-time ms
```

#### Tokens

##### Creating new tokens

To define a token, simply invoke `morgan.token()` with the name and a callback function.
This callback function is expected to return a string value. The value returned is then
available as ":type" in this case:

<!-- eslint-disable no-undef -->

```js
morgan.token('type', function (req, res) { return req.headers['content-type'] })
```

Calling `morgan.token()` using the same name as an existing token will overwrite that
token definition.

The token function is expected to be called with the arguments `req` and `res`, representing
the HTTP request and HTTP response. Additionally, the token can accept further arguments of
it's choosing to customize behavior.

##### :date[format]

The current date and time in UTC. The available formats are:

  - `clf` for the common log format (`"10/Oct/2000:13:55:36 +0000"`)
  - `iso` for the common ISO 8601 date time format (`2000-10-10T13:55:36.000Z`)
  - `web` for the common RFC 1123 date time format (`Tue, 10 Oct 2000 13:55:36 GMT`)

If no format is given, then the default is `web`.

##### :http-version

The HTTP version of the request.

##### :method

The HTTP method of the request.

##### :referrer

The Referrer header of the request. This will use the standard mis-spelled Referer header if exists, otherwise Referrer.

##### :remote-addr

The remote address of the request. This will use `req.ip`, otherwise the standard `req.connection.remoteAddress` value (socket address).

##### :remote-user

The user authenticated as part of Basic auth for the request.

##### :req[header]

The given `header` of the request. If the header is not present, the
value will be displayed as `"-"` in the log.

##### :res[header]

The given `header` of the response. If the header is not present, the
value will be displayed as `"-"` in the log.

##### :response-time[digits]

The time between the request coming into `morgan` and when the response
headers are written, in milliseconds.

The `digits` argument is a number that specifies the number of digits to
include on the number, defaulting to `3`, which provides microsecond precision.

##### :status

The status code of the response.

If the request/response cycle completes before a response was sent to the
client (for example, the TCP socket closed prematurely by a client aborting
the request), then the status will be empty (displayed as `"-"` in the log).

##### :total-time[digits]

The time between the request coming into `morgan` and when the response
has finished being written out to the connection, in milliseconds.

The `digits` argument is a number that specifies the number of digits to
include on the number, defaulting to `3`, which provides microsecond precision.

##### :url

The URL of the request. This will use `req.originalUrl` if exists, otherwise `req.url`.

##### :user-agent

The contents of the User-Agent header of the request.

### morgan.compile(format)

Compile a format string into a `format` function for use by `morgan`. A format string
is a string that represents a single log line and can utilize token syntax.
Tokens are references by `:token-name`. If tokens accept arguments, they can
be passed using `[]`, for example: `:token-name[pretty]` would pass the string
`'pretty'` as an argument to the token `token-name`.

The function returned from `morgan.compile` takes three arguments `tokens`, `req`, and
`res`, where `tokens` is object with all defined tokens, `req` is the HTTP request and
`res` is the HTTP response. The function will return a string that will be the log line,
or `undefined` / `null` to skip logging.

Normally formats are defined using `morgan.format(name, format)`, but for certain
advanced uses, this compile function is directly available.

## Examples

### express/connect

Simple app that will log all request in the Apache combined format to STDOUT

```js
var express = require('express')
var morgan = require('morgan')

var app = express()

app.use(morgan('combined'))

app.get('/', function (req, res) {
  res.send('hello, world!')
})
```

### vanilla http server

Simple app that will log all request in the Apache combined format to STDOUT

```js
var finalhandler = require('finalhandler')
var http = require('http')
var morgan = require('morgan')

// create "middleware"
var logger = morgan('combined')

http.createServer(function (req, res) {
  var done = finalhandler(req, res)
  logger(req, res, function (err) {
    if (err) return done(err)

    // respond to request
    res.setHeader('content-type', 'text/plain')
    res.end('hello, world!')
  })
})
```

### write logs to a file

#### single file

Simple app that will log all requests in the Apache combined format to the file
`access.log`.

```js
var express = require('express')
var fs = require('fs')
var morgan = require('morgan')
var path = require('path')

var app = express()

// create a write stream (in append mode)
var accessLogStream = fs.createWriteStream(path.join(__dirname, 'access.log'), { flags: 'a' })

// setup the logger
app.use(morgan('combined', { stream: accessLogStream }))

app.get('/', function (req, res) {
  res.send('hello, world!')
})
```

#### log file rotation

Simple app that will log all requests in the Apache combined format to one log
file per day in the `log/` directory using the
[rotating-file-stream module](https://www.npmjs.com/package/rotating-file-stream).

```js
var express = require('express')
var morgan = require('morgan')
var path = require('path')
var rfs = require('rotating-file-stream') // version 2.x

var app = express()

// create a rotating write stream
var accessLogStream = rfs.createStream('access.log', {
  interval: '1d', // rotate daily
  path: path.join(__dirname, 'log')
})

// setup the logger
app.use(morgan('combined', { stream: accessLogStream }))

app.get('/', function (req, res) {
  res.send('hello, world!')
})
```

### split / dual logging

The `morgan` middleware can be used as many times as needed, enabling
combinations like:

  * Log entry on request and one on response
  * Log all requests to file, but errors to console
  * ... and more!

Sample app that will log all requests to a file using Apache format, but
error responses are logged to the console:

```js
var express = require('express')
var fs = require('fs')
var morgan = require('morgan')
var path = require('path')

var app = express()

// log only 4xx and 5xx responses to console
app.use(morgan('dev', {
  skip: function (req, res) { return res.statusCode < 400 }
}))

// log all requests to access.log
app.use(morgan('common', {
  stream: fs.createWriteStream(path.join(__dirname, 'access.log'), { flags: 'a' })
}))

app.get('/', function (req, res) {
  res.send('hello, world!')
})
```

### use custom token formats

Sample app that will use custom token formats. This adds an ID to all requests and displays it using the `:id` token.

```js
var express = require('express')
var morgan = require('morgan')
var uuid = require('node-uuid')

morgan.token('id', function getId (req) {
  return req.id
})

var app = express()

app.use(assignId)
app.use(morgan(':id :method :url :response-time'))

app.get('/', function (req, res) {
  res.send('hello, world!')
})

function assignId (req, res, next) {
  req.id = uuid.v4()
  next()
}
```

## License

[MIT](LICENSE)

[coveralls-image]: https://badgen.net/coveralls/c/github/expressjs/morgan/master
[coveralls-url]: https://coveralls.io/r/expressjs/morgan?branch=master
[npm-downloads-image]: https://badgen.net/npm/dm/morgan
[npm-url]: https://npmjs.org/package/morgan
[npm-version-image]: https://badgen.net/npm/v/morgan
[travis-image]: https://badgen.net/travis/expressjs/morgan/master
[travis-url]: https://travis-ci.org/expressjs/morgan
<hr>
### Info en español:

Claro, aquí tienes la traducción del documento sobre Morgan, un middleware de registro de solicitudes HTTP para Node.js.

---

# Middleware de registro de solicitudes HTTP para Node.js

Llamado así por Dexter, un programa que no deberías ver hasta completarlo.

## API

```javascript
var morgan = require('morgan')
morgan(format, options)
```

Crea una nueva función de middleware de registro de Morgan utilizando el formato y las opciones dadas. El argumento `format` puede ser una cadena de un nombre predefinido (ver más abajo los nombres), una cadena de un formato de cadena, o una función que producirá una entrada de registro.

La función de formato será llamada con tres argumentos: `tokens`, `req` y `res`, donde `tokens` es un objeto con todos los tokens definidos, `req` es la solicitud HTTP y `res` es la respuesta HTTP. Se espera que la función devuelva una cadena que será la línea de registro, o `undefined` / `null` para omitir el registro.

### Usando una cadena de formato predefinido

```javascript
morgan('tiny')
```

### Usando una cadena de formato de tokens predefinidos

```javascript
morgan(':method :url :status :res[content-length] - :response-time ms')
```

### Usando una función de formato personalizada

```javascript
morgan(function (tokens, req, res) {
  return [
    tokens.method(req, res),
    tokens.url(req, res),
    tokens.status(req, res),
    tokens.res(req, res, 'content-length'), '-',
    tokens['response-time'](req, res), 'ms'
  ].join(' ')
})
```

## Opciones

Morgan acepta estas propiedades en el objeto `options`.

### immediate

Escribe la línea de registro en la solicitud en lugar de la respuesta. Esto significa que las solicitudes serán registradas incluso si el servidor se cae, pero los datos de la respuesta (como el código de respuesta, longitud del contenido, etc.) no pueden ser registrados.

### skip

Función para determinar si se omite el registro, por defecto es `false`. Esta función será llamada como `skip(req, res)`.

```javascript
// EJEMPLO: solo registrar respuestas de error
morgan('combined', {
  skip: function (req, res) { return res.statusCode < 400 }
})
```

### stream

Flujo de salida para escribir líneas de registro, por defecto es `process.stdout`.

## Formatos Predefinidos

Hay varios formatos predefinidos disponibles:

### combined

Salida estándar de registro combinado de Apache.

```
:remote-addr - :remote-user [:date[clf]] ":method :url HTTP/:http-version" :status :res[content-length] ":referrer" ":user-agent"
```

### common

Salida estándar de registro común de Apache.

```
:remote-addr - :remote-user [:date[clf]] ":method :url HTTP/:http-version" :status :res[content-length]
```

### dev

Salida concisa con colores por estado de respuesta para uso en desarrollo. El token `:status` será coloreado verde para códigos de éxito, rojo para códigos de error del servidor, amarillo para códigos de error del cliente, cian para códigos de redirección, y sin color para códigos informativos.

```
:method :url :status :response-time ms - :res[content-length]
```

### short

Más corto que el formato predeterminado, también incluye el tiempo de respuesta.

```
:remote-addr :remote-user :method :url HTTP/:http-version :status :res[content-length] - :response-time ms
```

### tiny

La salida mínima.

```
:method :url :status :res[content-length] - :response-time ms
```

## Tokens

### Crear nuevos tokens

Para definir un token, simplemente invoca `morgan.token()` con el nombre y una función de devolución de llamada. Se espera que esta función de devolución de llamada devuelva un valor de cadena. El valor devuelto está disponible como `:type` en este caso:

```javascript
morgan.token('type', function (req, res) { return req.headers['content-type'] })
```

Llamar a `morgan.token()` usando el mismo nombre que un token existente sobrescribirá esa definición de token.

Se espera que la función de token sea llamada con los argumentos `req` y `res`, que representan la solicitud HTTP y la respuesta HTTP. Además, el token puede aceptar más argumentos a su elección para personalizar el comportamiento.

### :date[format]

La fecha y hora actual en UTC. Los formatos disponibles son:

- `clf` para el formato de registro común ("10/Oct/2000:13:55:36 +0000")
- `iso` para el formato de fecha y hora ISO 8601 (2000-10-10T13:55:36.000Z)
- `web` para el formato de fecha y hora RFC 1123 (Tue, 10 Oct 2000 13:55:36 GMT)

Si no se da formato, el predeterminado es `web`.

### :http-version

La versión HTTP de la solicitud.

### :method

El método HTTP de la solicitud.

### :referrer

El encabezado Referrer de la solicitud. Usará el encabezado Referer mal escrito estándar si existe, de lo contrario Referrer.

### :remote-addr

La dirección remota de la solicitud. Usará `req.ip`, de lo contrario el valor estándar `req.connection.remoteAddress` (dirección del socket).

### :remote-user

El usuario autenticado como parte de la autenticación básica para la solicitud.

### :req[header]

El encabezado dado de la solicitud. Si el encabezado no está presente, el valor se mostrará como "-" en el registro.

### :res[header]

El encabezado dado de la respuesta. Si el encabezado no está presente, el valor se mostrará como "-" en el registro.

### :response-time[digits]

El tiempo entre la solicitud que llega a Morgan y cuando se escriben los encabezados de la respuesta, en milisegundos.

El argumento `digits` es un número que especifica la cantidad de dígitos a incluir en el número, siendo 3 el valor predeterminado, lo que proporciona precisión de microsegundos.

### :status

El código de estado de la respuesta.

Si el ciclo de solicitud/respuesta se completa antes de que se envíe una respuesta al cliente (por ejemplo, el socket TCP se cerró prematuramente por un cliente que aborta la solicitud), entonces el estado estará vacío (mostrado como "-" en el registro).

### :total-time[digits]

El tiempo entre la solicitud que llega a Morgan y cuando la respuesta ha terminado de ser escrita en la conexión, en milisegundos.

El argumento `digits` es un número que especifica la cantidad de dígitos a incluir en el número, siendo 3 el valor predeterminado, lo que proporciona precisión de microsegundos.

### :url

La URL de la solicitud. Usará `req.originalUrl` si existe, de lo contrario `req.url`.

### :user-agent

El contenido del encabezado User-Agent de la solicitud.

## morgan.compile(format)

Compila una cadena de formato en una función de formato para ser utilizada por Morgan. Una cadena de formato es una cadena que representa una sola línea de registro y puede utilizar la sintaxis de tokens. Los tokens son referenciados por `:token-name`. Si los tokens aceptan argumentos, pueden ser pasados usando `[]`, por ejemplo: `:token-name[pretty]` pasaría la cadena 'pretty' como un argumento al token `token-name`.

La función devuelta de `morgan.compile` toma tres argumentos `tokens`, `req` y `res`, donde `tokens` es un objeto con todos los tokens definidos, `req` es la solicitud HTTP y `res` es la respuesta HTTP. La función devolverá una cadena que será la línea de registro, o `undefined` / `null` para omitir el registro.

Normalmente, los formatos se definen usando `morgan.format(name, format)`, pero para ciertos usos avanzados, esta función de compilación está directamente disponible.

## Ejemplos

### express/connect

Aplicación simple que registrará todas las solicitudes en el formato combinado de Apache a `STDOUT`.

```javascript
var express = require('express')
var morgan = require('morgan')

var app = express()

app.use(morgan('combined'))

app.get('/', function (req, res) {
  res.send('hello, world!')
})
```

### Servidor HTTP estándar

Aplicación simple que registrará todas las solicitudes en el formato combinado de Apache a `STDOUT`.

```javascript
var finalhandler = require('finalhandler')
var http = require('http')
var morgan = require('morgan')

// crear "middleware"
var logger = morgan('combined')

http.createServer(function (req, res) {
  var done = finalhandler(req, res)
  logger(req, res, function (err) {
    if (err) return done(err)

    // responder a la solicitud
    res.setHeader('content-type', 'text/plain')
    res.end('hello, world!')
  })
})
```

### Escribir registros a un archivo

#### Archivo único

Aplicación simple que registrará todas las solicitudes en el formato combinado de Apache en el archivo `access.log`.

```javascript
var express = require('express')
var fs = require('fs')
var morgan = require('morgan')
var path = require('path')

var app = express()

// crear un flujo de escritura (en modo de añadir)


var accessLogStream = fs.createWriteStream(path.join(__dirname, 'access.log'), { flags: 'a' })

// configurar el registrador
app.use(morgan('combined', { stream: accessLogStream }))

app.get('/', function (req, res) {
  res.send('hello, world!')
})
```

#### Rotación de archivos de registro

Aplicación simple que registrará todas las solicitudes en el formato combinado de Apache a un archivo de registro por día en el directorio `log/` usando el módulo `rotating-file-stream`.

```javascript
var express = require('express')
var morgan = require('morgan')
var path = require('path')
var rfs = require('rotating-file-stream') // versión 2.x

var app = express()

// crear un flujo de escritura rotativo
var accessLogStream = rfs.createStream('access.log', {
  interval: '1d', // rotar diariamente
  path: path.join(__dirname, 'log')
})

// configurar el registrador
app.use(morgan('combined', { stream: accessLogStream }))

app.get('/', function (req, res) {
  res.send('hello, world!')
})
```

### Registro dividido / dual

El middleware Morgan puede ser utilizado tantas veces como sea necesario, permitiendo combinaciones como:

- Registrar una entrada en la solicitud y otra en la respuesta
- Registrar todas las solicitudes en un archivo, pero los errores en la consola
- ... ¡y más!

Aplicación de ejemplo que registrará todas las solicitudes en un archivo usando el formato Apache, pero las respuestas de error se registran en la consola:

```javascript
var express = require('express')
var fs = require('fs')
var morgan = require('morgan')
var path = require('path')

var app = express()

// registrar solo respuestas 4xx y 5xx en la consola
app.use(morgan('dev', {
  skip: function (req, res) { return res.statusCode < 400 }
}))

// registrar todas las solicitudes en `access.log`
app.use(morgan('common', {
  stream: fs.createWriteStream(path.join(__dirname, 'access.log'), { flags: 'a' })
}))

app.get('/', function (req, res) {
  res.send('hello, world!')
})
```

### Usar formatos de token personalizados

Aplicación de ejemplo que usará formatos de token personalizados. Esto agrega un ID a todas las solicitudes y lo muestra usando el token `:id`.

```javascript
var express = require('express')
var morgan = require('morgan')
var uuid = require('node-uuid')

morgan.token('id', function getId (req) {
  return req.id
})

var app = express()

app.use(assignId)
app.use(morgan(':id :method :url :response-time'))

app.get('/', function (req, res) {
  res.send('hello, world!')
})

function assignId (req, res, next) {
  req.id = uuid.v4()
  next()
}
```

---