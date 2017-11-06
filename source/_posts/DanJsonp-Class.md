---
title: DanJsonp Class
date: 2017-11-05 19:19:38
tags:
---

[JSONP](https://www.w3schools.com/js/js_json_jsonp.asp) is a method for sending JSON data without worrying about cross-domain issues.

It does not use the XMLHttpRequest object, but the _script_ tag instead.

Requesting an external script from another domain does not have cross-domain policy problems.

**[DanJsonp](https://github.com/evildead/vanilla-chuck/blob/master/js/danJsonp.js)** is a pure Javascript ES6 class which implements JSONP protocol inside a web page.

It contains a static method which receives two parameters: an uri and a callbackName
```js
static invokeJsonp(uri, callbackName = null) {
```

So this methods considers two modalities:
1. The first one is when an external callback name is provided.
1. The second one is when no callback name is provided.

The method creates a script element first
```js
let s = document.createElement("script");
```

Then, if a callback name is provided, it executes:
```js
if(callbackName) {
    finalAddress += separator + "callback=" + callbackName;
    s.src = finalAddress;

    // append script node to document body
    document.body.appendChild(s);
}
```
and does not return anything.

Else, if a callback name is provided, it executes:
```js
else {
    // create the promise to return
    var willPromiseFinish = new Promise(
        (resolve, reject) => {
            // assign static method danJsonpCallback to class DanJsonp
            Object.assign(DanJsonp, {
                danJsonpCallback(myObj) {
                    // resolve promise after callback invocation
                    if(myObj) {
                        resolve(myObj);
                    }
                    else {
                        reject("No value returned by server");
                    }
                }
            });
        }
    );

    finalAddress += separator + "callback=DanJsonp.danJsonpCallback";
    s.src = finalAddress;

    // append script node to document body
    document.body.appendChild(s);

    return willPromiseFinish;
}
```
The very interesting part here is the creation of a Promise object, which will be returned at method completion.

A **[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)** object represents the eventual completion (or failure) of an asynchronous operation, and its resulting value.

A function that is passed with the arguments _resolve_ and _reject_ to the Promise contructor is called _executor_. The executor function is executed immediately by the Promise implementation, passing _resolve_ and _reject_ functions (the executor is called before the Promise constructor even returns the created object). The resolve and reject functions, when called, resolve or reject the promise, respectively.

In the _invokeJsonp_ method, the executor creates a static method named _danJsonpCallback_ inside _DanJsonp_ class, by using **Object.assign** function.

This static method (_DanJsonp.danJsonpCallback_) will be invoked at the end of API execution, and only then it will "resolve" the promise, so to make the JSON data available to client.

In order to do that, a query parameter `finalAddress += separator + "callback=DanJsonp.danJsonpCallback";` is concatenated to the final uri, and the script element is appended to the document body.

## Example

Let's consider a simple PHP/Apache server, which provides a simple API, and is running on http://localhost/php-jsonp/getJsonpData.php:
```php
<?php
$phpData = (object) [
    'lang' => 'php',
    'name' => 'phpJsonp',
    'payload' => "Let's test these data coming from the PHP/Apache server"
];

$callbackName = filter_input(INPUT_GET, 'callback', FILTER_SANITIZE_SPECIAL_CHARS);

// JSONP modality
if(($callbackName !== false) && ($callbackName !== null)) {
    echo("/**/ typeof ".$callbackName." === 'function' && ".$callbackName.'('.json_encode($phpData).');');
}
// JSON modality
else {
    echo(json_encode($phpData));
}
```

To get the simple JSON object provided by the PHP/Apache server, by using DanJsonp class, the client should simply invoke:
```js
DanJsonp.invokeJsonp("http://localhost/php-jsonp/getJsonpData.php")
    .then((myObj) => {
        console.log(myObj);
    })
    .catch((reason) => {
        console.log('Promises catch: ' + reason);
    });
```
