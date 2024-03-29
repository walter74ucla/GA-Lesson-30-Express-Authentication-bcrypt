# https-git.generalassemb.ly-JimHaff-bcrypt-blob-master-README.md

# Express - Authentication

## Lesson Objectives
1. Explain what bcrypt does
1. Include bcrypt package
1. Hash a string using bcrypt
1. Compare a string to a hashed value to see if they are the same

## Explain what bcrypt does

bcrypt is a package that will encrypt passwords so that if your database gets hacked, people's passwords won't be exposed

## Include bcrypt package

Standard install

```
npm install bcryptjs --save
```

and require bcrypt in the file you want to use it in

```javascript
const bcrypt = require('bcryptjs');
```

## Hash a string using bcrypt

bcrypt does this thing called "salting" a string.  It requires you to generate a salt which is used in the encryption process.  This must be generated each time you hash a string.  If you don't do this, the same string will get hashed to the same value each time.  If this were to happen, someone with a common password could hack the database and see who else's hashed password had the same value as theirs and know that they have the same password as them.

```javascript
const hashedString = bcrypt.hashSync('yourStringHere', bcrypt.genSaltSync(10));
```

## Compare a string to a hashed value to see if they are the same

Because the same string gets encrypted differently every time, we have no way of actually seeing what the value of the string is.  We can compare it to another string and see if the two are "mathematically" equivalent.

```javascript
bcrypt.compareSync('yourGuessHere', hashedString); //returns true or false
```


## Lets implement bcrypt into our registration route and set a session

```javascript
// at top of file
const bcrypt = require('bcryptjs');

router.post('/register', (req, res, next) => {

  // first we are going to hash the password
  const password = req.body.password;
  const passwordHash = bcrypt.hashSync(password, bcrypt.genSaltSync(10));

  // lets create a object for our db entry;
  const userDbEntry = {};
  userDbEntry.username = req.body.username;
  userDbEntry.password = passwordHash

  // lets put the password into the database
  User.create(userDbEntry, (err, user) => {
    console.log(user)

    // lets set up the session in here we can use the same code we created in the login
    req.session.username = user.username;
    req.session.logged   = true;
    res.redirect('/authors')
  });

})

```


## Alrite lets try to implement the compare function in the login route

1.  first query the database to see if the user exists
2.  if the users exists use the bcrypt compare password to make sure the passwords match
3.  set the session if bcrypt compare equals true
4.  if the user doesn't exist send a message back to the user on the login view and tell them that the username or password don't exist, (hint maybe save a message to the session object)

```javascript
router.post('/login', (req, res, next) => {

  User.findOne({username: req.body.username}, (err, user) => {

      if(user){
                     //now compare hash with the password from the form
            if(bcrypt.compareSync(req.body.password, user.password)){
                req.session.message  = '';
                req.session.username = req.body.username;
                req.session.logged   = true;
                console.log(req.session, req.body)

                res.redirect('/authors')
            } else {
              console.log('else in bcrypt compare')
              req.session.message = 'Username or password are incorrect';
              res.redirect('/sessions/login')

            }

      } else {

          req.session.message = 'Username or password are incorrect';
          res.redirect('/sessions/login')

      } //end of if user
  });

})

```
