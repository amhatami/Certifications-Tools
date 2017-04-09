# Tutorial-OAuth
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





