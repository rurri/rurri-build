---
title: "Creating a custom authentication service in Meteor "
posted: 2015-06-16T14:53
tags: [meteor, authentication]
track: Technical
thumbnail: meteor-key.jpg
---
There are many reasons someone might need to create a custom authentication method in meteor. For our project it was the use of a custom CAS authentication server for which there was no available module since it was a completely custom fork of CAS.

If you are attempting to connect to anything non-standard and for which someone has not yet released an authentication module, you will most likely end up creating your own authentication functions and attaching them to the login hooks.

## Where is the documentation? ##
Unfortunately this process is not very well documented yet. Most of what I learned here came from looking at source code and how other internal and external authentication systems connected to the built-in authentication hooks.

A good place to start looking at code if you get stuck is the code that actually calls all the authentication hooks. This can be found [here on github](https://github.com/meteor/meteor/blob/devel/packages/accounts-base/accounts_server.js).

Of course there is some official documentation on accounts on [Meteor’s website](http://docs.meteor.com/#/full/accounts_api), which can be useful if you just want to hook into some existing authentication processes to change their behavior instead of creating your own.

## Connecting to Accounts hooks ##
If you are just looking to do certain tasks on user creation or to add custom validation, or user lockout, there is no need to create an entire custom user authentication service. You can instead just connect to the built-in hooks documented on Meteor’s website.

You can connect functions to run in response to several events:

* [Accounts.validateNewUser(func)](http://docs.meteor.com/#/full/accounts_validatenewuser)
* [Accounts.onCreateUser(func)](http://docs.meteor.com/#/full/accounts_oncreateuser)
* [Accounts.validateLoginAttempt(func)](http://docs.meteor.com/#/full/accounts_validateloginattempt)
* [Accounts.onLogin(func)](http://docs.meteor.com/#/full/accounts_onlogin)
* [Accounts.onLoginFailure(func)](http://docs.meteor.com/#/full/accounts_onloginfailure)

## First the data model ##
As you may already know, each meteor user is represented by an entry in the users collection. You can access this collection directly using [Meteor.users](http://docs.meteor.com/#/full/meteor_users). 

It is worth taking a moment to get to know the users collection as there are a few fields that have special behaviors which will effect security.

From Meteor’s documentation:

> username: a unique String identifying the user.
> emails: an Array of Objects with keys address and verified; an email address may belong to at most one user. verified is a Boolean which is true if the user has verified the address with a token sent over email.
> createdAt: the Date at which the user document was created.
> profile: an Object which the user can create and update with any data. Do not store anything on profile that you wouldn’t want the user to edit unless you have a deny rule on the Meteor.users collection.
> services: an Object containing data used by particular login services. For example, its reset field contains tokens used by forgot password links, and its resume field contains tokens used to keep you logged in between sessions.

#### username ####
A String.

A field with a unique constraint in the db. Optional. If you are using emails and not passwords you can ignore this field. But if you want each user to also have a username then you will be storing it here.

For our uses we didn’t want or need username. Many sites these days use email as the unique identifier and nothing more.

My personal thoughts on usernames is that unless you need a username as a type of handle for keeping user’s anonymous, then there is no need to burden users with the selection and maintenance of a username.

#### emails ####
An array of objects that look like:

```javascript
[{email:’jason@example.com’, verified: true},
{email:’rurri@example.com’, verified: false}]
```
Just like username, the email addresses are also unique. As you can see each entry also keeps track of whether the email has been verified or not. This is currently used by the password module in tracking which e-mails still need to be verified. The password module will mark it as verified: true once a link in an e-mail is clicked on.

There are a number of ways we can use this email list. If we know the e-mail address of users we are creating, and we add an entry here, it will prevent them from creating accounts with other services besides the one we created. If we send out our own e-mails we can embed codes in the links to also verify their e-mail when they click on an invitation or other type of link.

#### profile ####
A blackbox object.

This is an interesting “blackbox” type of object that can store anything. The most important thing to keep in mind with this special attribute is that any information that is stored here is automatically visible to the client upon login, AND any of this information can be modified by client.
> **Do not put any confidential or proprietary information in this field**
> Users can both see and modify this information be default.

So what do you put here? Easy! Things that you want the user to be able to see and modify be default. Things that you might show on say a profile or settings page that they could then modify such as their first and last name, settings and options for opting out of mailings, etc.

#### services #### 
An object keyed by service names.

This is where things get interesting for our custom authentication service. The services field is where each service stores its information. Each service picks a namespace/key value and then stores an object under that key containing its values.

By convention, and this will allow you to use some of Meteor’s built-in helper methods if you do follow it, one of the fields under services is called ‘id’. This id should be unique to each user you are authenticating.

Of course if you don’t have a unique id, or other id for your service you can always just use the username or emails fields to make sure you don’t make more than one user.

Some of the current built-in services and their keys:

* password
* resume
* google

This means a user might look like this (from Meteor’s documentation):

```javascript
{
  _id: “bbca5d6a-2156-41c4-89da-0329e8c99a4f”,  // Meteor.userId()
  username: “cool_kid_13”, // unique name
  emails: [
    // each email address can only belong to one user.
    { address: “cool@example.com”, verified: true },
    { address: “another@different.com”, verified: false }
  ],
  createdAt: Wed Aug 21 2013 15:16:52 GMT-0700 (PDT),
  profile: {
    // The profile is writable by the user by default.
    name: “Joe Schmoe”
  },
  services: {
    facebook: {
      id: “709050”, // facebook id
      accessToken: “AAACCgdX7G2…AbV9AZDZD”
    },
    resume: {
      loginTokens: [
        { token: “97e8c205-c7e4-47c9-9bea-8e2ccc0694cd”,
          when: 1349761684048 }
      ]
    }
  }
}
```

## Meteor Hooks and Helper ##
This is actually really all you need to know to use the Meteor.users collection to create a custom authentication. With the knowledge of how the users collection is organized you could create custom authentication code that worked only by modifying the collection directly.

This is not ideal, and next we will go over the undocumented Meteor helpers and hooks that make creating a custom authentication service quite easy.

### Accounts.registerLoginHandler ###
`Accounts.registerLoginHandler(“servicename”, callback);`
Currently have to look [at the source](https://github.com/meteor/meteor/blob/devel/packages/accounts-base/accounts_server.js) to see exactly how this hook works.

This is the main (and only method) you have to register with accounts to use the accounts framework to create a custom authentication service. When registering a custom service, create a custom service name that you will use, and a callback function to be called on login.

You callback will be called with one parameter called “options”.

When your function is called, any information that came with the login request will be inside an object called “options”. This will vary by login method. For an OATH login it will contain information different than a password login. When you attempt to log into the system on the client you can pass any login information along that you want. Whatever is passed in, is passed merrily along to your callback.

You have a few options when your callback is called:

#### Return Nothing ####
You can return nothing. If you don’t know how to handle this type of login (based on the information available in the options), you can assume the login call is not for you, but for a different service. In this case just return nothing and the system will continue looking for a handler that wants to do something with it.

#### Return an object ####
You can return an object. This means that you are logging the user into the system. The object must contain **at least** a value for the key `user_id`. **This is not the user’s primary key**. Remember above how each service inside of a user object should have an id field? If you followed this convention, then this is for you. You just return the user_id *for your service* of whomever you just authenticated.

In the example user object above which contained a service for Facebook, the Facebook service might return an object that looked like this:
```javascript
{user_id: “709050”}
```
Notice when we registered our callback we told the Accounts module what our service name was. This combined with the unique id for our service is enough for the Accounts module to lookup the user.

Also as part of this object we are returning we can return a blackbox object called `options`. Anything inside of options will be passed along to other callbacks such as the `validateResult` callback.

If during login you gather other information about the user, such as maybe a profile picture or real name, you could pass it along in options for other hooks to do something with if they wanted.

Our return object might start to look like this:
```javascript
{
  user_id: “709050”,
  options: {profile: {name: ‘Example’, pic_url:’example.com/1211'}}
}
```

The default post create user hook takes the contents of options.profile and copies it over to the user’s profile. *This is only done on user creation*. This is to prevent information the user has updated from being overwritten every time they log in.

If you want different behavior you can add your own post login hooks to copy over any information you want.
#### Return or throw an error ####
You should only return or throw an error if you know this request for meant for your service but that you couldn’t handle it. Remember that the Accounts module is not smart enough to know which callback will be handling each login request. It just calls them all until it gets a response back.

This means if you are not sure this request is for you, and you don’t have enough information it is probably best to return nothing.

If however, you can tell this login request was for your service but that there is something wrong with either the log-in or the amount of information you received, you can either return or throw an error.

Anything thrown will be caught by accounts, it will be caught and returned gracefully to the client.

Alternatively you can create an object with the key `error` and return that object. Personally I would opt to the throw an exception, but this could just a personal preference.

## Use updateOrCreate helper function

Accounts provides a nice helper method that you can drop right inside your login hook. The Accounts.updateOrCreateUserFromExternalService is again only documented [in the source](https://github.com/meteor/meteor/blob/devel/packages/accounts-base/accounts_server.js), but is quite handy for doing a lot of the legwork common to a lot of authentication services.

You call this method with three parameters (serviceName, serviceData, options).

The service name should be the name of your service. 

Service data should be all the information you want to store in the users collection under your service. This **must** contain *at least* a value for the field `id`.

>The value for id can be a complex object if your unique identifier is more than just a single id. The id could for example be: {account_number: 111, user_number: 222}. This will be handled gracefully by the helper.

Options will contain the additional information you would be returning if you were handling this yourself.

The `Accounts.updateOrCreateUserFromExternalService` method will return an object suitable for returning from your loginHandler. That is, it will have a user_id, and the options will be a part of the returned result.

This means the last thing your login handler does can be to just return the result of the `Accounts.updateOrCreateUserFromExternalService` method.

## Logging in from the client ##
Now that all your code is ready to go server side, you are ready to call the Accounts module login method to log a user in. 

This is also not yet documented in Meteor’s main documentation, but you can find the code and explanation [in source](https://github.com/meteor/meteor/blob/devel/packages/accounts-base/accounts_client.js).

From Meteor’s source:
```
// Call a login method on the server.
//
// A login method is a method which on success calls `this.setUserId(id)` and
// `Accounts._setLoginToken` on the server and returns an object with fields
// ‘id’ (containing the user id), ‘token’ (containing a resume token), and
// optionally `tokenExpires`.
//
// This function takes care of:
//   - Updating the Meteor.loggingIn() reactive data source
//   - Calling the method in ‘wait’ mode
//   - On success, saving the resume token to localStorage
//   - On success, calling Accounts.connection.setUserId()
//   - Setting up an onReconnect handler which logs in with
//     the resume token
//
// Options:
// - methodName: The method to call (default ‘login’)
// - methodArguments: The arguments for the method
// - validateResult: If provided, will be called with the result of the
//                 method. If it throws, the client will not be logged in (and
//                 its error will be passed to the callback).
// - userCallback: Will be called with no arguments once the user is fully
//                 logged in, or with the error on error.
//
```

To call your new login service you place a call to the Accounts.callLoginMethod.

A call might look something like this:

```javascript
Accounts.callLoginMethod({
  methodArguments: [
  {
    custom: ‘example’,
    entries: this.params.query.email,
    here: this.params.query.account
  }],
  validateResult: function (result) {
    //Custom validation of login on client side can go here
  },
   userCallback: function(error) {
   if (error) {
     console.log(error);
   }
   //Login happened in pop-up. Close the window after success.
	 close();
}
```

### Create a custom login method ###
This is everything needed to cause the login to happen, but that login code on the client is quite ugly. Notice there are only two items that vary during login based on query parameters in this case. It would be quite unfortunate if anyplace we wanted to kick off a login action we had to copy and paste this code.

By convention, but by no means required, when creating a new service you also create a method called logwinWith<Service> which takes the parameters that vary in this case the email and account, and then handles passing the login along.

## Summary ##
There is definitely a lot of small pieces to put together to make Meteor’s authentication operate just exactly as you want it to; however, once these pieces are understood, making a new authentication service becomes quite easy. In fact, when using the helpers to do most of the work, they can be done with quite a small amount of code.
