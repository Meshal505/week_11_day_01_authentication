# Passport Local Strategy Starter

## Intro to Passport.js

From the [passport website](http://passportjs.org/docs):

"_Passport is authentication Middleware for Node. It is designed to serve a singular purpose: authenticate requests. When writing modules, encapsulation is a virtue, so Passport delegates all other functionality to the application. This separation of concerns keeps code clean and maintainable, and makes Passport extremely easy to integrate into an application._

_In modern web applications, authentication can take a variety of forms. Traditionally, users log in by providing a username and password. With the rise of social networking, single sign-on using an OAuth provider such as Facebook or Twitter has become a popular authentication method. Services that expose an API often require token-based credentials to protect access._"

### Strategies

The main concept when using passport is to register _Strategies_.  A strategy is a passport Middleware that will create some action in the background and execute a callback; the callback should be called with different arguments depending on whether the action that has been performed in the strategy was successful or not. Based on this and on some config params, passport will redirect the request to different paths.

Because strategies are packaged as individual modules, we can pick and choose which ones we need for our application. This logic allows the developer to keep the code simple - without unnecessary dependencies - in the controller and delegate the proper authentication job to some specific passport code.

There are currently more than [300 Passport.js strategies](http://www.passportjs.org/packages/).

### Why Passport.js?

[Passport.js](http://www.passportjs.org/) is a middleware that simplifies implementing user authentication in JS applications.

Building your own authentication is hard - really hard. It's easy to not do it well. Doing it the wrong way can have major consquences, so it's almost always better to rely on something popular and prebuilt. 

Passport is a good solution for javascript applications. It's popular, well-tested, and secure.

When we use Passport and bcrypt together, it takes care of making sure that passwords are properly stored in our database (i.e. not in plain text).

> Note: We will talk about bcrypt shortly

<details>
  <summary><strong>Why might storing passwords in the database be a bad idea?</strong></summary>

  - Because of password reuse, if an attacker were to get access to our database, other accounts of our users may be threatened.
  - [Plain Text Offenders](http://plaintextoffenders.com/faq/devs)

</details>

<details>
  <summary><strong>What are the alternatives to storing passwords in plaintext?</strong></summary>

  - Hashing
  - Encryption

</details>

### You Do: Hashing vs. Encryption

Pair up, then read through [this blog post](http://www.securityinnovationeurope.com/blog/whats-the-difference-between-hashing-and-encrypting) together.

In pairs, post the answers to these questions in your own words as an issue on this repo.

<details>
  <summary><strong>What is the difference between encryption and hashing?</strong></summary>

  > **Encryption** takes a string and, using a key/algorithm, converts it to another string of variable length. The key/algorithm can be used to revert -- or "decrypt" -- the new string back to the original. In other words, it is a two-way process.
  >
  > **Hashing** is similar to encryption in that it converts a string to a different one using an algorithm. It cannot, however, be reverted back to the original, making it a one-way process.

</details>

<details>
  <summary><strong>When would you choose one over the other?</strong></summary>

  > Encryption should only be used if somebody or something needs to see the original string.

</details>

<details>
  <summary><strong>Which one should be used for passwords?</strong></summary>

  > **Hashing**. For security reasons, there is no need to store or gain access to the original password. All an application needs to do is to take in a password attempt, pass it through the hashing algorithm and see if it matches with the hash generated when the particular user signed for an account with an app.

</details>

> Some more resources...
>
> - [Encoding vs. Encryption vs. Hashing vs. Obfuscation](https://danielmiessler.com/study/encoding-encryption-hashing-obfuscation/#gs.7lUOAZI)
> - [Why are salted hashes more secure for password storage?](http://security.stackexchange.com/questions/51959/why-are-salted-hashes-more-secure-for-password-storage)

## Sessions / Flash 

### Sessions

Many of us have been to several webpages that only allow us to access content if we are logged in users of the webpage. Why do you think this would be necessary for many websites? Also, how is this concept of "being signed in" done programmatically? How is that information persisted from request to request? Enter sessions.

Most applications need to keep track of the state of a particular user. Is a user logged-in? Is there information that is unique to this user's instance of being signed in?

HTTP is by nature, stateless. That is, the program is not keeping record of previous interactions. Without state, a user would have to identify themselves after every request. Our shopping carts in Amazon couldn't keep their contents.

A session is just a way to store data in the browser between requests. The session in JS is an object which holds the properties we need. Today in Express we will create a new session automatically when a new user signs into an application.

Sessions are used mostly for user authentication and authorization, shopping carts, and setting a current model to persist throughout requests.

A contrived example of a session object might look like this:

```js
{
  sessionID: 'shdfsk3@234jsd!!@#jx...',
  expires: '6/18/2018:00-00-00UTC'
}
```

Note that sessions shouldn't contain any sensitive information about the user.

#### Real example

- Open up your browser to a website that has authentication (like github). Make sure you're logged in.
- Open the inspector and go to the `Network` tab. Make sure you have it set to "All" requests (or to not filter any requests).
- Refresh the page, then click on the first request (it should be similar to the url of the site you're on).
- Click on the `Headers` tab and you can see all of the http headers that get sent and received with that request.

> Which one of these looks like it might contain session information?

## Installation

Fork and clone this repo. By the end of the lesson, our app should be able to:

- A User can sign-up and login using [Passport's `local-strategy`](http://www.passportjs.org/).

<br>

## Set-up

> Let set up a nodejs app on port 4000

Passport's local strategy should be functional. Run the following to test it out.

```bash
$ npm i ejs express-ejs-layouts passport passport-local dotenv --save
$ nodemon server
```

Goto `localhost:4000`

Passport requires a secret to encode the sessions. 
```bash
$ npm i express-session
```

Note the following lines of code in `server.js`:

```js
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: true
}));
```


We've required the `dotenv` module to hide our `ENV` variables in development. So, `touch .env` and add a `SESSION_SECRET=mikedangisthebest`

<br>

```bash
$ mkdir helper && cd helper
$ touch passport.js
```

In `passport.js` include the following
```js
const passport = require('passport')
const LocalStrategy = require('passport-local').Strategy

/*
 Initialize passport local strategy,
 basic logic is collect email and password, findOne n user by email
 if user exist check if password is a match if password is a match return user else return error messages.
*/
passport.use(new LocalStrategy({
    usernameField: 'email',
    passwordField: 'password'
  },
  (email, password, done) => {
    User.findOne({
      email: email
    }, function (err, user) {

      if (err) {
        return done(err);
      }
      if (!user) {
        return done(null, false, { message : "User doesnt exist" });
      }

      user.verifyPassword(password, user.password, (err, match)=>{

        if(err) { return done(null, false, {message : "somethings wrong"}) }

        if(!match) { return done(null, false, {message : "passwords dont match"})}

        return done(null, user);
      })

    });
  }
));



//visual flow
/*
passport.serializeUser(function(user, done) {
    done(null, user.id);
});              │
                 │ 
                 │
                 └─────────────────┬──→ saved to session
                                   │    req.session.passport.user = {id: '..'}
                                   │
                                   ↓           
passport.deserializeUser(function(id, done) {
                   ┌───────────────┘
                   │
                   ↓ 
    User.findById(id, function(err, user) {
        done(err, user);
    });            └──────────────→ user object attaches to the request as req.user   
});

*/
```



#### YOU DO

- Read through the [Passport documentation](http://passportjs.org/docs).
- With a partner, walk through the starter code and add comments describing the flow of the app.

<br>


## SignUp

Goto `localhost:3000/auth/signup` and create a user

![](https://i.imgur.com/LqXiaHr.png)

This will automatically log you in and you should see the following:

![](https://i.imgur.com/GzKqI14.png)

You should have one user in your database.

<br>

## Add `isLoggedIn` to protected routes

Goto the `localhost:3000/profile` URL and checkout the `/profile` route in `index.js`. This route is protected with custom middleware code named `isLoggedIn`:

```js
// server..js
app.get('/profile', isLoggedIn, function(req, res) {
  res.render('profile');
});
```

This simple code can be found in the `middleware/isLoggedIn.js` file. These lines of code merely check for a `req.user` object, which passport creates for us upon logged in. Try removing the middleware from the route and see if you can access it without logging in.


Add `isLoggedIn` to ANY route that requires log in to access.

<br>

## Client has a `currentUser` object

Passport gives you a `currentUser` object for your views. Add this to your `profile.ejs` view, log in, and go to `localhost:4000/profile`

```html
<h2><%= currentUser.name %>'s Profile Page</h2>

<a href="/">Home</a>

<br>

Current User: <%= currentUser.name %>
```

<br>

## Server has a `req.user` object

Add a `console.log` the `/profile` route and check your server.

```js
app.get('/profile', isLoggedIn, function(req, res) {
  console.log(req.user);
  res.render('profile');
});
```
<br>


## Create a `usersroutes.js`

1. `touch usersroutes.js` and add the following:

```js
var express = require('express');
var router = express.Router();
var isLoggedIn = require('../middleware/isLoggedIn');
const user = require('../models/user');

module.exports = router;
```

1. Don't forget to add the route to your `server.js`

```js
app.use('/users', require('./controllers/usersroutes'));
```

<br>


## Create a `Pet` model

1. `mkdir models`
2. `cd models && touch user.js pet.js`
3. import the model in the `usersroutes.js`

```js
const user = require('../routes/usersroutes');
const pet = require('../routes/petsroutes');
```

<br>

### `User.hasMany(Pet)` and `Pet.belongsTo(User)`

1. `models/user.js`

```js
...
const userSchema = new Schema({
 firstname : String,
 lastname : String,
 username: String,
 pets : [{ type: Schema.Types.ObjectId, ref: 'Pet'}]
})
...
```

2. `models/pet.js`

```js
...
const petSchema = new Schema({
 name : String
})
...
```
<br>

### Add some pets
> Student make create user and add pets

<br>

## Create a User Show route with Pets

```js
const express = require('express');
const router = express.Router();
//var isLoggedIn = require('../middleware/isLoggedIn');
const User = require('../models/user');


router.get('/:id', isLoggedIn, (req, res) => {
  User.findById(req.user.id) 
  .populate('pets')
  .then(user => {
    res.render('users/show', { user })
    // res.send(user);
  });
});

module.exports = router;
```

<br>

## Create a view

1. `views/users/show.ejs`
	
```html
<h4><%= currentUser.name %>'s Pets!</h4>
<ul>
  <% user.pets.forEach(pet => { %>
    <li>  Pet Name: <%= pet.name %> | Pet Breed: <%= pet.breed %></li>
  <% }) %>
</ul>
```
1. Goto `http://localhost:4000/users/:mongoid`


Note...

- `currentUser` is only the User object. You'll need to use the mongoose `user` object to grab the associated pets array.
- You *should not* be able to access your 2nd user.

<br>

## What is Flash?

[You can create custom error messages.](https://www.npmjs.com/package/connect-flash)

<br>
