login-engine
============

The authentication core that ties all the rest of Engine, Inc.'s offerings together.

Getting and validating session key
============
The ``express`` middleware will get session keys for you, and provide you to tools to validate them.  Samples from [howtonode](http://howtonode.org/socket-io-auth).
```
var app = express();

app.configure(function () {
    app.use(express.cookieParser());
    app.use(express.session({secret: 'secret', key: 'express.sid'}));
});
```
Note that the server and client will both need that same secret provided to them through config file or environment variable.
```
app.get('/', function (req, res) {
    res.sendfile(__dirname + '/index.html');
});

server = http.createServer(app)
server.listen(3000);
```
Validation (socket.io sample):
```
io = io.listen(server);

io.set('authorization', function (handshakeData, accept) {

  if (handshakeData.headers.cookie) {

    handshakeData.cookie = cookie.parse(handshakeData.headers.cookie);

    handshakeData.sessionID = connect.utils.parseSignedCookie(handshakeData.cookie['express.sid'], 'secret');

    if (handshakeData.cookie['express.sid'] == handshakeData.sessionID) {
      return accept('Cookie is invalid.', false);
    }

  } else {
    return accept('No cookie transmitted.', false);
  } 

  accept(null, true);
});
```

Using sessions with this auth module:
==========
1. POST ``/create``
	Creates a new user account.  Submit the following as content:
	```
	{
		"session": <session key>,
		"encryptedSession": <encrypted session key for validation>,
		"password": <password or other token used for login (username is in the profile)>,
		"profile": {
			"username": <user name or whatever ID is appropriate>,
			// Absolutely anything else can go in here.
		}
	}
	```
	In failure, returns:
	```
	{
		"error": <whatever error message>
	}
	```
	In success, returns: 
	```
	{
		"profile": {
			"username": <username provided in profile originally>,
			// Whatever else was in here.
		}
	}
	```
2. POST ``/login``
	Logs in a user to an existing account.  Submit the following as content:
	```
	{
		"session": <session key>,
		"encryptedSession": <encrypted session key for validation>,
		"password": <password or other token used for login (username is in the profile)>,
		"username": <user name or whatever ID is appropriate>
	}
	```
	In failure, returns:
	```
	{
		"error": <whatever error message>
	}
	```
	In success, returns: 
	```
	{
		"profile": {
			"username": <username provided in profile originally>,
			// Whatever else was in here.
		}
	}
	```

3. GET ``/loggedIn``
	Creates a new user account.  Submit the following as content:
	```
	?session=<session key>
	&encrypted=<encrypted session key for validation>
	```
	In failure, returns:
	```
	{
		"error": <whatever error message>
	}
	```
	In success, returns: 
	```
	{
		"profile": {
			"username": <username provided in profile originally>,
			// Whatever else was in here.
		}
	}
	```
