Lab 5
=====

In this lab, we'll monkey patch the project from Lab 4 so that it can also receive
and return json.

HTTP protocol includes a `content-type` header. In theory, we could have one
route on flask which could return different responses depending on whether the
incoming request content-type was application/json or if it was html. Okay fine,
we'll do it that way. Fine! But in the past, I have written separate routes
that exclusively receive and return json requests and responses.

When HTML forms are submitted, the content-type header is usually
`application/x-www-form-urlencoded`, which means that the form's field-values
are submitted in the form of a url:

  key1=val1&key2=val2&key3=val3

Flask can parse these out and store them in its `request.form` property.

But many pages on the internets make asynchronous form submission, where a user's
browser does not need to refresh to load the response. Instead, javascript is
executed in the user's browser which can make a request, receive the response,
and update just a small piece of the displayed HTML page, instead of having to
reload the entire thing. This is what can make webpages feel dynamic, even if
they rely on connections to an external server.

Whatever. Just look at this new repo that I made. Specifically, look at
[this commit](https://github.com/deargle/security-analytics-lab-5/commit/9cb378df7f3bfbdecb4f320479f718dab04af398).

I'll use this lab to give you a crash course in javascript, and an orientation
to content-types.


Deliverables
------------

Modify your app from lab 4, and javascript-ize it. You can optionally make a brand
new repo for this lab, but you don't have to. You don't
have to do exactly what I did, but at the most basic, your flask app needs
to be able to receive a json request, and return a json response. And you need
to do _something_ with javascript in your predict.html which can make and handle
the response.

Deploy your new app to heroku. If you just modified your lab 4 repo, you will be
using the same heroku app. If you make a brand new git repo for this lab, you will
want to make a new heroku app.
