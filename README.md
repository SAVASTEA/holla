## Information

<table>
<tr>
<td>Package</td>
<td>vein</td>
</tr>
<tr>
<td>Description</td>
<td>RPC/PubSub via WebSockets</td>
</tr>
<tr>
<td>Node Version</td>
<td>>= 0.4</td>
</tr>
</table>

## Example

### Server

```javascript
var Vein = require('vein');
var server = http.createServer().listen(8080);
var vein = new Vein(server);

vein.add('multiply', function (res, numOne, numTwo){
  res.send(numOne * numTwo);
});
```

### Client

```javascript
var vein = new Vein();
vein.ready(function (){
  vein.multiply(2, 5, function (num){
    // num === 10
  });
});
```

## Server Usage

### Create

```
-- Options --
path - prefix path (default: "/vein")
resource - change to allow multiple servers for one prefix (default: "default")
```

```javascript
var Vein = require('vein');
var vein = new Vein(8080, {options});
```

### Adding services

Arguments passed to res.send() will be applied to the callback on the client

```javascript
vein.add('getNumber', function (res, name, num) {
  res.send("Hey there " + name + " I got your number " + num);
});
```

### PubSub

You can use res.publish() to send a message to everyone subscribing to a service

```javascript
vein.add('publishMessage', function (res, msg) {
  res.publish(res.cookie('username'), msg);
});
```

### Cookies

The server can read and write cookies to the client via res.cookie()

```javascript
vein.add('login', function (res, username, password) {
  if (res.cookie('login')) {
    res.send('You already logged in!');
  } else if (username === 'username' && password === 'pass123') {
    res.cookie('login', 'success!');
    res.send();
  } else {
    res.send('Invalid username or password');
  }
});
```

### Middleware

You can use middleware to add layers in front of your services. Any arguments passed into next will be thrown as an error on the client and end the middleware chain.

```javascript
vein.use(function(req, res, next){
  if (req.service == 'login') {
    next();
  } else {
    if (res.cookie('login') == 'success!') {
      next();
    } else {
      next('Not authorized - piss off');
    }
  }
});
```

## Client Usage

### Create

```
-- Options --
host - server location (default: window.location.hostname)
port - server port (default: window.location.port)
secure - use SSL (default: window.location.protocol)
path - prefix path (default: "/vein")
resource - change to allow multiple servers for one endpoint (default: "default")
```

```javascript
var vein = new Vein({options});
```

### Ready

When the connection has been established your callback will be called with an array of services available.

```javascript
vein.ready(function (services) {
  //Start doing stuff!
});
```

### Calling services

When calling a service the format is vein.serviceName(args..., callback)

```javascript
vein.getNumber('john', 2, function (msg) {
  console.log(msg);
});
```

### PubSub

Prefixing a service with .subscribe allows the server to send unsolicited messages to the client.
Subscribing to a service does not communicate to the server in any way.

```javascript
vein.subscribe.publishMessage(function (user, message) {
  console.log(user, "just said:", message);
});
```
  
### Cookies

You can read/write cookies by using vein.cookie(). This is just nice sugar.

```javascript
vein.cookie('test', 'hello world'); //write
console.log(vein.cookie('test')); //read
vein.cookie('test', null); //delete
```
  
### Close

If the connection has been closed this will be called.

```javascript
vein.close(function (reason) {
  console.log("Connection lost due to", reason);
});
```

## Examples

You can view a chatroom example and more in the [example folder.](https://github.com/wearefractal/vein/tree/master/examples)

## LICENSE

(MIT License)

Copyright (c) 2012 Fractal <contact@wearefractal.com>

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
