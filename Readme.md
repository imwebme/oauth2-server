# Node OAuth2 Server

Complete, compliant and well tested module for implementing an OAuth2 Server/Provider with [express](http://expressjs.com/) in [node.js](http://nodejs.org/) [![Build Status](https://travis-ci.org/nightworld/node-oauth2-server.png?branch=master)](https://travis-ci.org/nightworld/node-oauth2-server)

## Installation

	$ npm install node-oauth2-server

## Quick Start

The module provides two middlewares, one for authorization and routing, another for error handling, use them as you would any other middleware:

```js
var express = require('express'),
	oauthserver = require('node-oauth2-server');

var app = express();

app.configure(function() {
	var oauth = oauthserver({
		model: {}, // See below for specification
		grants: ['password'],
		debug: true
	});
	app.use(express.bodyParser()); // REQUIRED
	app.use(oauth.handler());
	app.use(oauth.errorHandler());
});

app.get('/', function (req, res) {
	res.send('Secret area');
});

app.listen(3000);
```

After running with node, visting http://127.0.0.1:3000 should present you with a json response saying your access token could not be found.

Note: As no model was actually implemented here, delving any deaper, i.e. passing an access token, will just cause a server error. See below for the specification of what's required from the model.

## Options

- `model`	`Object`	Model object (see below)
- `allow`	`Array|Object`	Either an array (`['/path1', '/path2']`) or objects or arrays keyed by method (`{ get: ['/path1'], post: ['/path2'], all: ['/path3'] }`) of paths to allow to bypass authorisation. (Does not currently support regex)
- `grants`	`Array`	grant types you wish to support, currently the module only supports `password`
- `debug`	`Boolean` Whether to log errors to console
- `accessTokenLifetime`	`Number`	Life of access tokens in seconds (defaults to 3600)
- `refreshTokenLifetime` `Number`	Life of refresh tokens in seconds (defaults to 1209600)
- `authCodeLifetime`	`Number`	Lfe of auth codes in seconds (defaults to 30)
- `clientIdRegex`	`RegExp`	Regex to match auth codes against before checking model

## Model Specification

The module requires a model object through which some aspects or storage, retrieval and custom validation are abstracted.
The last parameter of all methods is a callback of which the first parameter is always used to indicate an error.
A model must provide the following methods:

### getAccessToken(bearerToken, callback)
- `bearerToken`	`String`	The bearer token (access token) that has been provided
- `callback`	`Function` callback(error, accessToken)
	- `error`	`Mixed`	Truthy to indicate an error
	- `accessToken`	Object|Boolean	The access token retrieved form storage or falsey to indicate invalid access token

`accessToken` should, at least, take the form:
- `expires` `Date`	The date when it expires
- `user_id` `String|Number`	The user_id (saved in req.user.id)

### getClient(clientId, clientSecret, callback)
- `clientId`	`String`
- `clientSecret`	`String`
- `callback`	`Function` callback(error, client)
	- `error`	`Mixed`	Truthy to indicate an error
	- `client`	`Object|Boolean`	The client retrieved from storage or falsey to indicate an invalid client (saved in req.client)

### grantTypeAllowed(clientId, grantType, callback)
- `clientId`	`String`
- `grantType`	`String`
- `callback`	`Function` callback(error, allowed)
	- `error`	`Mixed`	Truthy to indicate an error
	- `allowed`	`Boolean`	Indicates whether the grantType is allowed for this clientId

### saveAccessToken(accessToken, clientId, userId, expires, callback)
- `accessToken` `String`
- `clientId`	`String`
- `userId`	`Mixed`
- `expires`	`Date`
- `callback`	`Function`
	- `error`	`Mixed`	Truthy to indicate an error

### saveRefreshToken(refreshToken, clientId, userId, expires, callback)
- `refreshToken` `String`
- `clientId`	`String`
- `userId`	`Mixed`
- `expires`	`Date`
- `callback`	`Function`
	- `error`	`Mixed`	Truthy to indicate an error

### getUser(username, password, callback)
- `username`	`String`
- `password`	`String`
- `callback`	`Function`
	- `error`	`Mixed`	Truthy to indicate an error
	- `user`	`Object|Boolean`	The user retrieved from storage or falsey to indicate an invalid user (saved in req.user)

## Credits

Copyright (c) 2013 NightWorld

Created by Thom Seddon

## License

[Apache, Version 2.0](https://github.com/nightworld/node-oauth2-server/blob/master/LICENSE)