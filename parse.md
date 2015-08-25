# Parse

## Promise

* `Parse.Promise.as("Hello")`

```js
Parse.Promise.as("Hello").then(function (hello) {
  console.log(hello);
});
```

* `Parse.Promise.when(helloPromise, worldPromise)`

```js
var helloPromise = Parse.Promise.as("Hello");
var worldPromise = Parse.Promise.as(", world!");
Parse.Promise.when(helloPromise, worldPromise).then(function (hello, world) {
  console.log(hello + world);
});
```

flat and zip:

```js
var helloPromise = Parse.Promise.as("Hello");
var worldPromise = Parse.Promise.as(", world!");
helloPromise.then(function (hello) {
  return Parse.Promise.when(Parse.Promise.as(hello), worldPromise);
}).then(function (hello, world) {
  console.log(hello + world);
});
```

Error handling:

```js
/**
 * Returns email.
 *
 * @param {String} accessToken
 * @returns {Promise<String>} email
 */
function getEmailAlternative(accessToken) {
    return getEmail(accessToken).then(function (email) {
        if (!email) return Parse.Promise.error("Invalid email");

        return Parse.Promise.as(email);
    }, function (error) {
        return getUid(accessToken).then(function (uid) {
            return Parse.Promise.as(uid + "@weibo.com");
        });
    });
}

/**
 * Returns email
 *
 * @param {String} accessToken
 * @returns {Promise<String>} email
 */
function getEmail(accessToken) {
    return Parse.Cloud.httpRequest({
        url: "https://api.weibo.com/2/account/profile/email.json",
           params: {
               access_token: accessToken
           }
    }).then(function (httpResponse) {
        return JSON.parse(httpResponse.text)[0].email; // [ { email: "abc@example.com" } ]
    });
};

/**
 * Returns uid.
 *
 * @param {String} accessToken
 * @returns {Promise<String>} uid
 */
function getUid(accessToken) {
    return Parse.Cloud.httpRequest({
        url: "https://api.weibo.com/2/account/get_uid.json",
           params: {
               access_token: accessToken
           }
    }).then(function (httpResponse) {
        return JSON.parse(httpResponse.text).uid; // { uid: 5647447265 }
    });
}
```

ref.

* https://gist.github.com/yongjhih/e196e01fc7da9c03ce7e
