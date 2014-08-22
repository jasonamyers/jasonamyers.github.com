---
layout: post
title: "Testing Bluebird promises with Mocha"
description: "a look at some things you might encounter testing with bluebird
and Mocha"
tags: [Javascript, Bluebird, Mocha, Testing]
comments: true
share: true
---

So I have a node.js application that contains the getAccountInfoFromToken
function below, and I want to test it with Mocha. This function has been gutted
and the account details are embedded now instead of called in to make the
example clearer.

    Promise = require('bluebird'),
    getAccountInfoFromToken: function(token) {
        return new Promise(function(resolve, reject){
            // Get Account details
            var sample_return = {
                "accounts": [
                    {
                        "account_id": 1725389
                    }
                ],
            };
            var accounts = sample_return["accounts"];
            if (accounts.length > 0) {
                resolve(accounts);
            } else {
                reject('No Accounts');
            }
        });
    }

So initially I took a very naive approach to the test because I wasn't clear on
how javascript compared arrays. So I had the test you see below.

    "use strict";

    describe("token_vending_machine", function () {
        var tvm = require("../src/server"),
            sinon = require("sinon"),
            assert = require("assert"),
            expected_token_return = [
                {"account_id": 1725389}
            ];

        describe("#getAccountInfoFromToken()", function () {
            it("should return the right details when called with a token", function () {
                tvm.getAccountInfoFromToken("test").then(function (data) {
                    assert.equal(data, expected_token_return);
                })
            });
        });
    });

However Mocha would report that as a pass but then I got an error from Bluebird
about a possibly unhandled AssertionError... like you see in the output below.

    token_vending_machine
        #getAccountInfoFromToken()
        ✓ should return the right details when called with a token
    Possibly unhandled AssertionError: [{"account_id":1725389}] == [{"account_id":1725389}]
        at /Users/jmyers/dev/token_vending_machine/test/tvm.js:40:24
        at tryCatch1 (/Users/jmyers/dev/token_vending_machine/node_modules/bluebird/js/main/util.js:43:21)
        at Promise$_callHandler [as _callHandler] (/Users/jmyers/dev/token_vending_machine/node_modules/bluebird/js/main/promise.js:627:13)
        at Promise$_settlePromiseFromHandler [as _settlePromiseFromHandler] (/Users/jmyers/dev/token_vending_machine/node_modules/bluebird/js/main/promise.js:641:18)
        at Promise$_settlePromiseAt [as _settlePromiseAt] (/Users/jmyers/dev/token_vending_machine/node_modules/bluebird/js/main/promise.js:804:14)
        at Promise$_settlePromises [as _settlePromises] (/Users/jmyers/dev/token_vending_machine/node_modules/bluebird/js/main/promise.js:938:14)
        at Async$_consumeFunctionBuffer [as _consumeFunctionBuffer] (/Users/jmyers/dev/token_vending_machine/node_modules/bluebird/js/main/async.js:75:12)
        at Async$consumeFunctionBuffer (/Users/jmyers/dev/token_vending_machine/node_modules/bluebird/js/main/async.js:38:14)
        at process._tickDomainCallback (node.js:463:13)


    1 passing (8ms)

That possible assertion error is a real thing because I thought my test was
passing when it wasn't because Mocha wasn't getting the information it needed.
Next I tried to catch the error and throw it. So I edited the test like so...

        describe("#getAccountInfoFromToken()", function () {
            it("should return the right details when called with a token", function () {
                tvm.getAccountInfoFromToken("test").then(function (data) {
                    assert.equal(data, expected_token_return);
                }).catch(function (err) {
                    console.error(err);
                })
            });
        });

That lead to a timeout error, which actually made Mocha show the test as
failing. It also logged what the actual error was. That output is below

    token_vending_machine
        #getAccountInfoFromToken()
    { [AssertionError: [{"account_id":1725389}] == [{"account_id":1725389}]]
    name: 'AssertionError',
    actual: [ { account_id: 1725389 } ],
    expected: [ { account_id: 1725389 } ],
    operator: '==',
    message: '[{"account_id":1725389}] == [{"account_id":1725389}]' }
        1) should return the right details when called with a token


    0 passing (2s)
    1 failing

    1) token_vending_machine #getAccountInfoFromToken() should return the right details when called with a token:
        Error: timeout of 2000ms exceeded
        at null.<anonymous> (/Users/jmyers/dev/token_vending_machine/node_modules/mocha/lib/runnable.js:157:19)
        at Timer.listOnTimeout [as ontimeout] (timers.js:112:15)

Now I knew I needed to use a deepEqual to get the test to pass; however, I still
didn't wanna have to wait 2 seconds per test for my test to get marked as
a failure.  Enter Mocha's done function which tells Mocha that the test is
complete. It needs to be called after the assertion, and in the error handler
in order to work properly.  Here is the updated test.

    describe("#getAccountInfoFromToken()", function () {
        it("should return the right details when called with a token", function (done) {
            tvm.getAccountInfoFromToken("test").then(function (data) {
                assert.equal(data, expected_token_return);
                done();
            }).catch(function (err) {
                done(err);
            });
        });
    });

the clearer Mocha output:

    token_vending_machine
        #getAccountInfoFromToken()
        1) should return the right details when called with a token


    0 passing (6ms)
    1 failing

    1) token_vending_machine #getAccountInfoFromToken() should return the right details when called with a token:
        AssertionError: [{"account_id":1725389}] == [{"account_id":1725389}]
        at /Users/jmyers/dev/token_vending_machine/test/tvm.js:35:24
        at tryCatch1 (/Users/jmyers/dev/token_vending_machine/node_modules/bluebird/js/main/util.js:43:21)
        at Promise$_callHandler [as _callHandler] (/Users/jmyers/dev/token_vending_machine/node_modules/bluebird/js/main/promise.js:627:13)
        at Promise$_settlePromiseFromHandler [as _settlePromiseFromHandler] (/Users/jmyers/dev/token_vending_machine/node_modules/bluebird/js/main/promise.js:641:18)
        at Promise$_settlePromiseAt [as _settlePromiseAt] (/Users/jmyers/dev/token_vending_machine/node_modules/bluebird/js/main/promise.js:804:14)
        at Async$_consumeFunctionBuffer [as _consumeFunctionBuffer] (/Users/jmyers/dev/token_vending_machine/node_modules/bluebird/js/main/async.js:75:12)
        at Async$consumeFunctionBuffer (/Users/jmyers/dev/token_vending_machine/node_modules/bluebird/js/main/async.js:38:14)
        at process._tickDomainCallback (node.js:463:13)

Now to actually make the two arrays compare properly and getting a passing test.
All that we need to do is to do a deepEqual instead of an equal.


    describe("#getAccountInfoFromToken()", function () {
        it("should return the right details when called with a token", function (done) {
            tvm.getAccountInfoFromToken("test").then(function (data) {
                assert.deepEqual(data, expected_token_return);
                done();
            }).catch(function (err) {
                done(err);
            });
        });
    });

And our happy Mocha output:

    token_vending_machine
      #getAccountInfoFromToken()
        ✓ should return the right details when called with a token


    1 passing (6ms)

