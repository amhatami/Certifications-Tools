# <i class="icon-file"></i> Tutorial-OAuth
How to Implement Sign-In via OAuth



# This tutorial is broken down in the following sections: [![Build Status](https://travis-ci.org/andreareginato/oauth-ng.svg?branch=master)](https://travis-ci.org/andreareginato/oauth-ng)

* What is OAuth?
* OAuth and Javascript
* Set-Up
* Create an Application
* Add Business Logic
* Client-side implementation
* Troubleshooting

## What is OAuth?

OAuth is an open protocol to allow secure authorization in a simple and standard method from web, mobile, and desktop applications. As an application developer, services that provide HTTP APIs supporting OAuth, let you access parts of their service on behalf of your users. For example, when accessing a social network site, if your user gives you permission to access his account, you might be able to import pictures, friends lists, or contact information to your application.

This tutorial focuses on using OAuth to implement Sign-In through the following providers: Facebook, Google+, LinkedIn, and Twitter. These providers use either OAuth1.0a (LinkedIn, Twitter) or OAuth2.0 (Facebook, Google+).

To use OAuth, you first have to register an application with that provider. The application will be assigned credentials, needed when performing the OAuth flow.

## OAuth1.0a
OAuth1.0a provides a method to access a protected resource at the provider, on behalf of the resource owner (the user). This process consists of the following steps:

1. The client obtains an unauthorized request token.
2. The client redirects the user to a login dialog at the provider.
3. The user authorizes the request token, associating it with their account.
4. The provider redirects the user back to the client.
5. The client exchanges the request token for an access token.
6. The access token allows the client to access a protected resource at the provider, on behalf of the user.

![alt text](https://raw.githubusercontent.com/amhatami/Tutorial-OAuth/master/img/OAuth10a_img.JPG "OAuth1.0a steps")

## OAuth2.0
OAuth2.0 is similar to the OAuth1.0a protocol explained above, but the steps to be followed are different:

1. The client redirects the user to a login dialog at the provider.
2. The user authorizes the client.
3. The provider redirects the user back to the client, additionally returning an access_token.
4. The client validates the access token.
5. The access token allows the client to access a protected resource at the provider.

![alt text](https://github.com/amhatami/Tutorial-OAuth/blob/master/img/OAuth20a_img.JPG?raw=true "OAuth1.0a steps")

Both OAuth1.0a and OAuth2.0 use (part of) the application credentials to perform the flow described above.

## OAuth and JavaScript
Implementing OAuth in client-side JavaScript is challenging, mainly because:
* The nature of OAuth forces the client to expose application credentials. This is a serious security vulnerability.
* The provider may not support CORS. This makes it impossible for the client to communicate with the provider.

Furthermore, implementing multiple OAuth protocols for the different providers is a lot of work. Instead, a simpler and safer solution is needed. One that:
* Never reveals application credentials to the client.
* Functions even if the provider does not support CORS.
* Requires little effort to implement and is easy to use.

The remainder of this tutorial will adhere to these three requirements. By using any Business Logic feature, in combination with the library, you can provide your users with a safe Sign-In via OAuth through Facebook, Google+, LinkedIn, and Twitter.


# Sample code : Using Oauth 2.0 In With AngularJS

Oauth Implicit Grant Type via OauthLib:

`The implicit grant type is used to obtain access tokens (it does not support the issuance of refresh tokens) and is optimized for public clients known to operate a particular redirection URI. These clients are typically implemented in a browser using a scripting language such as JavaScript.
Unlike the authorization code grant type, in which the client makes separate requests for authorization and for an access token, the client receives the access token as the result of the authorization request.`

You’ll know the provider supports the implicit grant type when they make use of <font color="red">response_type=token</font> rather than `response_type=code`.

So there are going to be a few requirements to accomplish this in AngularJS:

1. We are going to be using the <b>AngularJS UI-Router</b> library
2. We are going to have a stand-alone index.html page with multiple templates
3. We are going to have a stand-alone oauth_callback.html page with no AngularJS involvement

With that said, let’s go ahead and create our project to look like the following:


