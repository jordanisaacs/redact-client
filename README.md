# redact-client
[![License: GPL-3.0](https://badgen.net/github/license/pauwels-labs/redact-client?color=blue)](https://opensource.org/licenses/GPL-3.0) [![crates.io](https://badgen.net/crates/v/redact-client?color=blue)](https://crates.io/crates/redact-client) [![docs.rs](https://img.shields.io/docsrs/redact-client?style=flat)](https://docs.rs/redact-client) [![Coverage Status](https://badgen.net/coveralls/c/github/pauwels-labs/redact-client/main)](https://coveralls.io/github/pauwels-labs/redact-client?branch=main)

redact-client is a binary that runs locally on a user's device and responds to requests for encrypted data from third-party websites.

It achieves this by listening for HTTP requests on port 8080. When a website places a reference to private data on their webpage, what it's truly doing is pointing an iframe to localhost with a request path corresponding to the requested data, something like `GET /data/.profile.firstName`. The client also provides some convenient query parameters listed below for various functionality.

## Encryption

Behind the scenes, the client performs several operations to get the data secured and on the page. It first fetches the appropriate data from storage, which may come to it as another reference, an encrypted set of bytes, or an unencrypted set of bytes. If the bytes come encrypted, the decryption key may itself be retrieved as a reference, an encrypted set of bytes, or an unencrypted set of bytes. The client will resolve the entire chain of references/encryption to get the final decrypted value, deserialize it into its final type, and serve it up in a secure iframe. The bulk of the retrieval and resolution work is performed by [redact-crypto](https://github.com/pauwels-labs/redact-crypto), which contains all of the abstractions that power Redact's encrypted type system.

## Opaque Data Display

The last core component of redact-client is iframe security. It must ensure that data is only served within a secure context, that is, within a webpage it controls, in order to block any other domain from being able to request it. It achieves this by splitting the request process into two requests: an unsecure one and a secure one. 

The unsecure URL is the one placed in the third-party website's iframe, it requires no tokens or authentication and simply requests that a piece of data be placed at that location. During this phase, the client generates a token and sets it in its session store. It then responds to the request with an HTML page containing another iframe pointing to the same URL with the token appended as a path parameter. It also sets a cookie at localhost with the session ID of the previously created session. During the secure route phase, the query parameter token and session token are compared for equality before data is served in the returned HTML. A third-party website could not simultaneously provide both a valid query parameter and valid cookie if it attempted to make the request itself.

## Run
1. `git clone https://github.com/pauwels-labs/redact-client`
2. Set your storage URL in config/config.yaml. You can go to [redact-store](https://github.com/pauwels-labs/redact-store) to set up your own storage.
3. `cargo r`

## Usage
Refer to the [Redact Client Docs](https://docs.redact.ws/en/latest/client.html) for API documentation.

## Test
To run unit tests:
1. `cargo t`

To run unit tests+code coverage output (does not work on macos or windows):
1. `cargo install tarpaulin`
2. `cargo tarpaulin -o html`

## Docs & Support
Docs are available at [docs.redact.ws](https://docs.redact.ws).

Join us in our Keybase channel! You can download the keybase client [here](https://keybase.io/download).

Once there, click on Teams, select Join a team, and our team name is pauwelslabs.

Once you're in, Redact discussion happens in the #redact channel.

Discussions in the Keybase team should be respectful, focused on Redact, and free of profanity.
