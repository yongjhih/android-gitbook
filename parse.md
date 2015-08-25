# Parse

## Promise

* `Parse.Promise.as("Hello")`

```js
Parse.Promise.as("Hello").then(function (hello) {
  console.log(hello);
});
```

* `Parse.Promise.when(helloPromise, helloPromise)`

```js
var helloPromise = Parse.Promise.as("Hello");
Parse.Promise.when(helloPromise, helloPromise).then(function (hello, hello2) {
  console.log(hello + hello2);
});
```

以 parse-cloud weibo.js 的 `signInWithWeibo()` 為例:

```js
/**
 * Returns the session token of available parse user via weibo access token within `request.params.accessToken`.
 *
 * @param {Object} request Require request.params.accessToken
 * @param {Object} response
 * @returns {String} sessionToken
 */
function signInWithWeibo(request, response) {
    promiseResponse(signInWithWeiboPromise(request.user, request.params.accessToken, request.params.expiresTime), response);
}

/**
 * Returns the session token of available parse user via weibo access token.
 *
 * @param {Parse.User} user
 * @param {String} accessToken
 * @param {Number} expiresTime
 * @returns {Promise<String>} sessionToken
 */
function signInWithWeiboPromise(user, accessToken, expiresTime) {
   // ...
   return Parse.Promise.as("Hello-sessionToken");
}

function promiseResponse(promise, response) {
    promise.then(function(o) {
        response.success(o);
    }, function(error) {
        response.error(error);
    })
}

/**
 * @param {Function(request, response)} func
 */
function defineParseCloud(func) {
    Parse.Cloud.define(func.name, func);
}

defineParseCloud(signInWithWeibo);
```

* https://gist.github.com/yongjhih/e196e01fc7da9c03ce7e
