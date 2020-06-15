# ▶ Day 1
# Hello node
To set headers :
```js
res.setHeader('Content-Type', 'application/json')
```

## Basic routing
`req.url` property contains relative path from the domain.
`req.originalURL` - full path with domain

## Express
the main feature of express for node developer is it's built in `router`
By switching to express we can take advantage of its router which simplifies development.

By calling `res.json()`, express will automatically send
the correct Content-Type header (app/json) and stringify our response body for us.

`req.query` - represents query parameters from the path (/?name=value)

In the next snippet
```js
app.get('/static/*', respondStatic)
```
asterisk signifies the whole left path.

# Express >>>
## Postman
1. download postman
2. use postman to test your api
3. How to save requests
    1. create a new collection
    2. save request into a collection (save button is near the send button)
    3. give the request some structured name (like: fine new item)

## setting up project
1. setup npm with `npm init`
2. setup express with `npm i express`

```js
const express = require('express');

const app = express();
app.get(`/`, (req, res) => {
    res.status(200).json({
        msg: `hello`,
        app: `my app`
    });
})
const port = 3000;
app.listen(port, () => {
    console.log(`app running ${port}`)
});
```

## REST architecture
1. Define resources
    Resource is a representation of something.
    
2. Define paths to resources
    URL paths should be structured and only define `what` resource should be accessed.
    www.xxx.addTour - bad path, because it contains a `verb`
    
3. Define what you want to do with resources with `http verbs`

To get resources that are "part" of another resource:
    `/users/4f2324/tours/53a7f8`

4. REST should always be `stateless`
    Meaning all the state should be handled on the `client` and not on the server

## Handling requests with Express
You can compose responses according to `JSend` spec. Which is a spec that controls 
how you should structure your responses.

1. GET request
    ```js
    const app = express();
    app.get(`/api/v1/tours`, (req, res) => {
        res.status(200).json({
            status: `success`,
            data: {
                tours
            }
        })
    })
    const port = 3000;
    app.listen(port, () => {
        console.log(`app running ${port}`)
    });
    ```
2. POST request
    ```js
    app.post(`/api/v1/tours`, (req, res) => {
        const newId = tours[tours.length - 1].id + 1;
        const newTour = Object.assign({id: newId}, req.body); // note req.body
        tours.push(newTour);

        fs.writeFile(`${__dirname}/dev-data/data/tours-simple.json`,
            JSON.stringify(tours), err => {
            res.status(201).send({
                status: `success`,
                data: {
                    tour: newTour
                }
            })
        })
    });
    ```

3. URL parameters
    To define parameters you need to use a `colon` (:)
    To make a parameter `optional` add a question mark to it like this (/:smth?)
    ```js
    app.get(`/api/v1/tours/:id`, (req, res) => {
        const tour = tours.find(t => t.id == req.params.id); // note req.params.id

        if(tour) res.status(201).send({ status: `success`, data: { tour } })
        else res.status(404).send({ status: 'fail' })
    })
    ```

4. PATCH requests
    ```js
    app.patch(`/api/v1/tours/:id`, (req, res) => {
        res.status(200).json({
            status: 'success',
            data: {tour: `Updated tour`}
        })
    });
    ```

5. Delete requests
    ```js
    app.delete(`/api/v1/tours/:id`, (req, res) => {
        res.status(204).json({
            status: 'success',
            data: {tour: `Deleted a tour`}
        })
    });
    ```

## Code structure
For now you have both the path and a handler (function) in a `single place`
which makes it harder to add more functions.

1. Export functions (lambda furnction with req and res params) into a distinct function.
2. Then you will see that some routes have the same base, like
        - /api/tours/:id
        - /api/tours/:id
        - /api/tours
    To refactor this, do the following
    ```js
    app.route(`/api/v1/tours`)
        .get(getAllTours)
        .post(createTour);
    app.route(`/api/v1/tours/:id`)
        .get(getTour)
        .patch(changeTour)
        .delete(deleteTour);
    ```

## Custom middleware
To create custom middleware create a function with three parameters: req, res and next.
```js
app.use((req, res, next) => {
    console.log('hello from the middleware');
    next();
})
```
These middleware functions will apply to every request.

The `order` of middleware matters.

You can `set properties` on the req object for use in later middleware.
```js
app.use((req, res, next) => {
    req.requestTime = new Date().toISOString();
    next();
})
```
-
                                    **Note**
In custom middleware when you send a response, you have to `return` after that.
If code will reach the call to `next()` function after you have returned a response
to a client you will get an `error`.

## 3rd party middleware
`Morgan` - logging middleware => npm i morgan
```js
app.use(morgan('dev'));
```

With this little setup you get some quite useful information in your console
on every request.

You can also use ('tiny')

## Implementing routes for User "resource"
```js
app.route(`/api/v1/users`)
    .get(getAllUsers)
    .post(createUser);

app.route(`api/v1/users/:id`)
    .get(getUser)
    .patch(updateUser)
    .delete(deleteUser);
```
**Note** in `postman` you can create collections (you know it) and in collections
you can create `folders` (may be useful to create a different folder `for different model`)


## Code structure +
You may want to divide `different routes for different resources`.
And different directories for routes and functions that are called by the routes.

1. You need to create one separate `router` for each resource.
    ```js
    const tourRouter = express.Router(); // one router for one resource
    tourRouter.route(`/tours/`) // use the router instead of app
        .get(getTours)
        .post(postTour);

    tourRouter.route(`/tours/:id`)
        .get(getTour)
        .delete(deleleTour)
        .patch(patchTour);

    app.use(`/api/v1/`, tourRouter); // when mounting route to the app, you should 
                                        // supply the base domain path
    ```
    Router is a `middleware` so you can easily moun it to your app with
    `app.use(someRouter)`
    You just define a router and then assign
    `different http verbs with middlewares` and relative `paths` for that router
    The process of mounting the router to the app is called `mounting the router`.

2. It may be easier to remember if you imagine that router is an ASP.NET `controller`
    that contains the base path to some model.
    And different verbs with different relative paths are `actions`.

3. Separate different routers into different files
    1. Create another folder called `Routes`
    2. In that folder create distinct file for each model and call them `ModelName`.
    3. In every "route" file define `express.Router` and assign functions to them.
    4. Export the routers and import them in app.js.
    
    This way you can have different routers for some specific base path and 
    some custom middleware for all requests in your app.js file.
    
4. Create another folder where you will store `functions`, and call it `actions`.
    1. Create different files for different models like this: `ModelName`
    2. Put all request handlers in there
    3. Export the actions for routers to use.

5. Create another file in the root folder called `server.js`
    1. import `app` from app.js
    2. create port variable
    3. call `app.listen()` in this file
    This file will be resposible for things like:
        1. Database configuration
        2. Error handling
        3. Env. vars setup
        4. etc

6. Create an npm script for running your app
    ```npm
    npm scripts>
        start: node server.js
    ```

## Param middleware - special type of middleware
Is a type of middleware that `only runs` when we have special parameters in `URL`

E.g you can write a middleware that will only run when there's an `id` parameter in URL.
You can create a param middleware by calling `param` instead of "route" on the 
router object and passing to the function a callback with `four` parameters, the last
one is value of the parameter.
```js
// modelRoutes.js file, router = express.Router()
router.param('id', (req, res, next, val) => {
    console.log(`Tour id for this request is ${val}`);
    next();
});
```
This middleware should be put `before any other` router paths for a specific model 
router, because it should get the request first.

Apparently this middleware will only run for specific models.
It won't affect other models.

It's good to use this function for `validation` (to avoid repetition).

This way every function has only `one responsibility`, because you now 
don't need to do validation in every function.

## Chaining multiple middleware functions (for the same route)
Before now for every http verb and according path we only executed a single 
middleware function at a time.
```js
router.route(`/`)
    .get(getTours)  // single function per verb
    .post(postTour);// single function per verb
```

You may want to do it for some repetitive activities (like validation), that is not 
passed via params. e.g `validate body of the request`

To chain multiple middlewares to the same verb just `enumerate them via comma`.
```js
router.route(`/`)
    .get(getTours)
    .post(validateTour, postTour);

// in another file, in Actions directory
const validateTour = (req, res, next) => { // middleware 
    let {name, price} = req.body;
    if(name && price) 
        next();
    else 
        res.status(400).send("Invalid request body");
};
```

## Serving static files
You need to do that using middleware.
```js
// in app.js
app.use(express.static(`${__dirname}/public`)); // to serve files from the public folder
                                                // that is located in your root folder.
```

Then to access the file you need to access `domain/rpf` where `rpf` is the relative
path `from public` directory to the file, inluding the file name.

## Env variables
In `app.js` you should do everything that is only relevant to `express`
Env var is not about express so you should configure them in `server.js`

To get environment you can type
```js
app.get('env'); // return the string that represents the environment
                // by default it's "development" and is stored in env var NODE_ENV
```

`env` only tells you what environment you are running your program in.
Environment is stored in enviroment variable. And there `a lot of environment variables`.

To see all environment variables you can use this property
    `process.env`
if you console.log it you will see all the environemt variables there are


You can set `environment manually` by setting the `NODE_ENV` environment variable 
when you start the app.
```cli
NODE_ENV=development node server.js
```
But when you need to set `multiple` environment variables you will need a distinct 
file specifically for env vars.

`Uppercase` is a convention for environment variables.


1. Create `config.env` file in your root folder
    ```config.env
    NODE_ENV=development
    ```
2. Install a package called `dotenv` (--save)
3. Add this module to your server
    ```js
    const dotenv = require('dotenv');
    dotenv.config({ path: './config.env' }); // this add all the variables from the file
                                            // to node env vars
    ```
    You need this package `because` variables in config.env are not environmental, they
    are just text and this package sets environment variables of your system 
    accordingly to this file.

4. E.g to add logging only in `development` environment
    ```js
    // app.js
    if(proccess.env.NODE_ENV == "development" )
        app.use(morgan('dev')); 
        
    // or 
    app.use(morgan(process.env.LOGGING)); 
    ```

5. Port fron env vars
    ```js
    const port = process.env.PORT || 3000;
    ```

**Note** you only need to call `dotenv.config` once! in the server file.
**Note** it's important to call `app after` you configured your env vars.
    if you call app first, app won't know about proper env vars and may be configured
    inappropriately. `Always call app after` you have set up env vars with `dotenv`.

To run the app with different environments (not env vars), you can setup 
npm like `this`:
```json
"scripts": {
    "start": "node server.js",
    "start:prod": "NODE_ENV=production node server.js"
    // last one doesn't work on windows, need to use cross-env package
    /*
    npm i -D cross-env
    
    "start:prod": "cross-env NODE_ENV=production node server.js"
    */
}
```
Then you call it like this: `npm run start:dev`

## Setting up ESLint
```
npm i
    eslint
    eslint-config-airbnb
    eslint-plugin-node
    eslint-plugin-import
    eslint-plugin-jsx-a11y
    eslint-plugin-react -D
```

# ▶ Day 2
# Mongo
`Collection` is like a "table" of data.
Collections can contain `documents` which represent "rows" in RDBs.

1. Main features
    1. Stores data in documents.
    2. Easily `scalable`
    3. Flexible => you can change schema without migrations
    4. `Performant` due to indexing, sharding and many more.
    5. Free and open source

    In mongo you can have:
        1. `Multiple` values per field (store arrays)
        2. `Embedded` documents, which happens by the process called `denormalizing`

2. Installing MongoDB
    1. Install community server
    2. Create a directory for mongoDB to store data
        1. On a disk where you have installed mongo create a folder `data`
        2. In "data" folder create another folder called `db`
        3. Go to `program files/mongo/server/bin`
        4. Execute `mongod.exe` => this will start the server
        5. To connect to the server, you will need to start `mongo.exe`
        6. In the shell that has opened, write `db` to list databases, 
            you should get "test" in return.
        7. Add mongod to the PATH
            environment variables > system variables > find `PATH` > add `"lalala/bin"` of 
                mongo to that variable
                
3. Creating a Local database
    1. type `mongo` in cmd
    2. create a database with the `use` command
        `use dbName`
        This command is also used to switch to an existing database.
    3. Each `db` has `collections` that in turn have `documents`
    4. To insert a document into a collection of some database you need to do:
        **Note** tours is a collection, you need to specify it before you insert a doc
        because any document must be in some collection
        `db.tours.insertOne({ name: "the forest hiker", price: 299 })`
    5. To see the document you have just created
        `db.tours.find()`
    6. To see the databases you have created => `show dbs`
    7. To quit shell => `quit()`

4. Creating documents in mongoDB
    1. To create multiple documents at the same time
        `db.tours.insertMany([{name: "some name 1"}, {name: "some name 2"}])`
        The function takes an `array of objects`.

5. Querying a database
    1. Get all
        `db.tours.find()`
    2. Get those which have specific, exact property value
        `db.tours.find({ name: "the forest hiker" })`
    3. Pass predicate
        `db.tours.find({ price: {$lte: 500} })`
        Predicates are passed `as an object` with the property that
        starts with the `dollar sign` and has some value.
    4. Multiple predicates at the same time
        `db.tours.find({ price: {$lt: 500}, rating: {$gt: 3.9 } })`
        Just separate them with a comma.
    5. One of multiple predicates
        `db.tours.find({ $or [price: {$lt: 500}, rating: {$gt: 3.9 }] })` 
        It should be the `$or` operator which value is an `array of predicates`.
    6. Map the output 
        `db.tours.find(SOME_FILTER_ABOVE, {name: 1})` 
        name: 1 => means that you are only `interested in name fields` of 
        the objects you get as a result of applying filter.

6. Updating documents
    `db.tours.updateOne(FILTER_OBJECT, {$set : {price: 433}})`
    if filter returns multiple objs, only the first one will be updated.
    
    Set accepts an object that may have multiple parameters.
    (in this case it has only one). If the object that is affected 
    by the query `doesn't have` the property, it will be `added`.
    
    `Why set`? Why not just pass an object? It's because `there are other`
    useful `operators` like increment, decrement, etc.

    `updateMany` to update multiple documents.
    
    `replaceOne/Many` to completely replace documents with new doc.

7. Deleting documents
    `db.tours.deleteMany(FILTER_OBJECT)`
    It returns documents that were deleted
    
    `db.tours.deleteMany({})` => delete ALL

8. Using compass for CRUD
    You can connect to the local database by following the steps:
        1. Run the server
        2. Run compass
        3. Choose to fill fields individually to connect
        4. Connect
    It's easy to do CRUD operations with compass.
    You can allso click `options` in compass to add projection/filtering/sorting
    
9. Creating a remote database with Atlas
    Atlas takes all the pain of managing and scaling a database aways from us.

    1. Create free account
    2. Create a cluster
    3. Create a project

10. Connecting to remote database
    1. Click connect in your cluster
    2. Add your ip and mongo user credentials
    3. Save credentials in your `config.env`
    4. Choose the connection method
        It should give you the `connection string`.
    5. If you choose **compass**, in compass:
        1. Click `connect to` from top left menu
        2. Choose to fill fields individually
        3. Paste password and connect
        4. You will have preconfigured databases but you should `create a new one` 
            and call it as you like.
        5. Fill in `database and collection`(remember collection = table) names
        6. Congratulations! You can go to the collection and insert your first document.
            You can go to your collection on the website and see what you have added 
        8. You added your ip address for security, but if you want to access
            the db from another computer you may want to `allow all ip's` to access
            your database.
            1. Click `network access`
            2. Edit
            3. allow access from everywhere
            
    6. If you choose **mongoshell**
        1. Go back to cluster on the website
        2. Click connect
        3. Choose "connect with the mongo shell"
        4. Copy the connection string
        5. Paste the connections stirng into the terminal `don't forget to paste db name`
            `mongo "mongodb+srv://natours-3w9fv.mongodb.net/Natours" --username francois`
        6. Insert password from the database
        7. Woila!
        8. show dbs => lists all databases you have

# Mongoose >>>
## Connect express to the mongoDB
1. Go to the website
2. Click cluster
3. Click connect => `connect your application`
4. Copy the connection string and fill the dbname and password
    (or you can substitute the passwords `programmatically`)
5. Create env var in config.env with the connection string

Connections string for `local` database is simple:
    `mongodb://localhost:27017/<dbname>`

After connectiong to the database you should install a `database driver`.
```cli
npm i mongoose
```

Then connect to the database usign `mongoose.connect`
```js
// in server.js
const dotenv = require('dotenv');       // first env vars
dotenv.config({ path: './config.env' });

const mongoose = require('mongoose');
const DBCS = process.env.DBSTRING.replace('<PASS>', process.env.DBPASS);

mongoose.connect(DBCS, {
    useNewUrlParser: true,
    useCreateIndex: true,
    useFindAndModify: false
}).then(connection => {
    //console.log(connection.connections);
    console.log(`Connected to the database: success!`);
});
```

Note, "mongoose.connect" returns a promise which contains `connection` object 
with different useful properties. If there is an error while connecting
to the database, you can always `catch` it with .catch or try_catch.

## Creating a model with mongoose
`Model` is like a class in OOP, a 'blueprint' for further objects(rows).
We create models `to create documents` from it.

To create a model you need a `schema`.
Schema is used to describe a model.

When describing model's properties, mongoose allows you to use `native JS type objects`
as types.
```js
const tourSchema = new mongoose.Schema({
    name: { // instead of a type you can specify OPTIONS
        type: String,
        required: [true, "A tour must have a name"],
        unique: true
    },
    rating: Number,
    price: Number
});

const Tour = mongoose.model('Tour', tourSchema); // m - small, not capital in "model"
```

## Creating documents and testing the Model
Now you can create `instances` from you model, by calling the model with the `new` 
operator. 
```js
const testTour = new Tour({
    name: "The Forest Hiker",
    rating: 4.4,
    price: 400
});
```
`Instances` have mehtods on them that allow you to interact with the database.
```js
testTour
    .save() // save document(instance) to the database
    .then(tour => console.log(`successfully saved the tour named ${tour.name}`))
    .catch(e => console.error(`Error: ${e}`));
```

## Architecture
1. Application vs Business logic
    Applications logic is logic that concerns `implementation` of something.
        - how to get something from the database
        - how to sort something efficiently
        
    Business logic is about the `meaning` of an action
        - what does it mean to register (what is needed for that)
        - what does it mean a "valid" user
        - what does it mean to pay the bill

    There's a philisophy called `fat model thin controller` that means that 
    you should offload as much logic as possible to the model and leave 
    controllers as small as possible. This way you can make your servers more performant.
    E.g `don't do sorting or filtering` on the server, let DB do that.

2. Refactorign into MVC
    1. We must already have `routes` and `actions` separated.
    2. Create a `Models` folder
    3. This folder shouls contain both the `schema` and `model` of mongoose.
    4. You should only export the model.
    5. Import the model into Actions if you need

## Another way to create documents
```js
// instead of 
const newTour = new Tour({/*...*/})
newTour.save();

// we can do the following
Tour.create({/*...*/});
```

**Note** to use async/await with eslint you may want to set node version in your 
package.json
```json
root {
    engines {
        node : ">=10.0.0"
    }
}
```
**Note** properties of the object that you want to add to the database
`that are not in the schema`, will be completely `ignored` on save or Model.create.
and won't be saved in the database.

```js
// part of the Tour Actions file.
const {Tour} = require('../Models/Tour');

const createTour = async (req, res) => {
    try {
        const tour = await Tour.create(req.body);
        res.status(200).json({status: "success", data: tour});
    }
    catch(e) {
        res.status(400).json({status: "failed", data: e.message});
    }
};
```

## Reading the documents
You can use `find` method `on a Model` to look for differnt documents.
```js
const getAllTours = async (req, res) => {
    try {
        const tours = await Tour.find();

        res.status(200).json({
            status: `success`,
            data: tours,
            dataLength: tours.length
        })
    }
    catch (e) {
        res.status(404).json({status: `failed`, message: e.message});
    }
};
```

```js
const getTour = (req, res) => {
    try {
        const tour = Tour.findById(req.params.id); // note findBY ID

        res.status(200).json({
            status: 'success',
            data: tour
        });
    }
    catch(e) {
        res.status(404).send("Not found");
    }
};
```
When you want to get `a single file` you should always use `findById` as it's indexed
and therefore `faster`. 

## Updating documents
```js
const updateTour = async (req, res) => {
    try {
        const tour = await Tour
            .findByIdAndUpdate(req.params.id, req.body, { // findByIdAndUpdate
                new: true,          // return updated document
                runValidators: true // validate after update
            });

        res.status(200).json({
            status: 'success',
            data: tour
        });
    } catch(e) {
        res.status(404).send("Not found");
    }
};
```
**Note** it's simultaneously find and update : `findByIdAndUpdate` it `takes`
    - id
    - object with properties to set
    - `options` object

## Deleting Documents
```js
const deleteTour = async (req, res) => {
    try{
        let tour = await Tour.findByIdAndDelete(req.params.id); // note findByIdAndDelete

        res.status(200).json({
            status: 'success',
            data: tour          // deleted tour, (no need to specify options)
        });
    } catch (e) {
        res.status(404).send("Not found");
    }
};
```

# Modelling the app
## Adding more properties and constraints
```js
summary: {
    type: String,
    trim: true // automatically removes whitespace on the sides
    required: [true, "Summary is requred"] // plays the role of TUPLE
    // lowercase: true, // automatically lowercases the input
},
createdAt: {
    type: Date,
    default: Date.now(), // sets the date automatically
},
startDates: {
    type: [Date], // an ARRAY of Dates
}
```

## Creating script to import data from a file
```js
const fs = require('fs');
const {Tour} = require('../../Models/Tour');
const dotenv = require('dotenv');
dotenv.config({ path: `${__dirname}/../../config.env` });

// Connect to the database
const mongoose = require('mongoose');
const DBCS = process.env.DBSTRING.replace('<PASS>', process.env.DBPASS);

mongoose.connect(DBCS, { 
    useNewUrlParser: true, // db options
    useCreateIndex: true,
    useFindAndModify: false
}).then(_ => {
    //console.log(connection.connections);
    console.log(`Connected to the database: success!`);
});

let tours = fs.readFileSync(`${__dirname}/tours.json`, 'utf-8');
tours = JSON.parse(tours);
const importTours = async (tours) => {
    try{
        const addedTours = await Tour.create(tours);
    } catch(e) {
        console.log(`Error occured while importing from json: ${e}`);
    }
};

if([...process.argv].some(param => param == '-d')) // contains parameters 
    Tour.deleteMany({})
        .then(_ => console.log(`Current data has been successfully deleted`));
if ([...process.argv].some(param => param == '-i'))
    importTours(tours);
```

You can pass parameters to node like this `node app --param`, and access them
via `process.argv` which is an array of parameters.

To delete all tours we used `Tour.deleteMany({})`
To import tours we used `Tour.create(tours)` method that can receive both
single model as well as an `array` of models

# ▶ Day 3
# Making API Better
## Filtering
Allowing a user to filter data with a query string.
To access query parameters you can use `req.query` which is an object that
contains parameters and their values.

Simple way of filtering
```js
const tours = await Tour.find({
    ...req.query
});
```

More sophisticated way, which you should use more often as it gives you `more control`
and more different functions (not only equal)
```js
const tours = await Tour
    .find()                                         // first just find
    .where('duration').equals(req.query.duration)   // then something like LINQ
    .where('difficulty').equals(req.query.difficulty);
```
`where` and `equals` are both methods of a `Query` object of mongoose.
You can look up documentation on that.

On top of that they perform equally well

Another way that you may want to use is filtering the query itself into a new obj
```js
let query = {...req.query}; // new object from query
let extraQuery = ['page', 'sort', 'limit', 'fields'];
extraQuery.forEach(ext => delete query[ext]); // filtering the new query object properties
```

## Advanced filtering
When you want to specify for example greater that opeartor, on the client 
it looks like this: `tours?duration[gte]=5`, which transforms into the following 
req.query = `{duration: { gte: 5 }}`

`But` for this query to work with mongoose it should be `$gte` instead of just gte
```js
let queryStr = JSON.stringify(query);
queryStr = queryStr
    .replace(/\b(gte|lte|gt|lt)\b/g, match => `$${match}`); // \b - word boundary
                                                            //g - global
query = JSON.parse(queryStr);
const tours = await Tour.find(query); // and now it works
```

## Sorting
Client Queries : 
    1. `?sort=property`
    2. `?sort=-property` (ascending order)
    3. `?sort=property1,-property2` (sortBy, thenBy)
    
Server receives a string like this (`req.query.sort` is equal to):
    1. `property`
    2. `-property` (ascending order)
    3. `property1,-property2` (sortBy, thenBy)
    
To make it work with mongoose all you need is `replace comma with a space`
```js
let dbQuery = Tour.find(query); // query to mongo, WITHOUT await

if(req.query.sort){
    const sortBy = req.query.sort.replace(',', ' '); // replace comma
    dbQuery = dbQuery.sort(sortBy);                 // append sorting to query
}

const tours = await dbQuery; // and then you AWAIT
```

## Filtering Fields
Client Queries : 
    - `?fields=duration,price`  - choose fields to return
    
To `permanently remove` a property from the result on the `schema level`:
```js
// in a schema
createdAt: {
    type: Date,
    default: Date.now(),
    select: false  // <-------- select: false
},
```

To `map` response
```js
if(req.query.fields){
    const getOnly = req.query.fields
        .split(',')
        .join(' ');      // commas into spaces (`replace` replaces only the FIRST)
    dbQuery = dbQuery.select(getOnly); // mongoose's SELECT methods
} else {
    dbQuery = dbQuery.select('-__v'); // excluding the fields
}
```

## Pagination
Client Queries : 
    - `?page=2&limit=10`  - choose fields to return

You have two mongoose methods for pagination
    1. `skip()` - skip first N
    2. `limit()` - amount
    
```js
let {page = 1, limit = 10} = req.query; // X = N -> default value
const skip = (page-1) * limit;

// if skipped too many
if(skip > await Tour.countDocuments()) // Note Model.countDocuments() method
    throw new Error('Skipped too many pages'); // throw an error (actually you shouldn't)

dbQuery = dbQuery
    .skip((skip)
    .limit(+limit); // convert limit to number
```

## Providing alias for frequent routes
Create middleware that will `set query paramters for user`

```js
// in Aliases file
const topFiveTours = (req, res, next) => {
    req.query = {
        sort: 'rating',
        limit: '5',
        fields: 'name,difficulty,summary'
    };
    next();
}
module.exports = {topFiveTours};


// in Tour file that is in Routes directory
const Aliases = require('./Aliases');
//...
router.route('/top5')
    .get(Aliases.topFiveTours, getAllTours); // FIRST call the alias
```

# ▶ Day 4
# Refactoring API features
At the time we have `too much of functionality` in a single method. `getAllTours`
hadndles not only getting the tours but also parsing every possible query and 
structuring data accordingly.

Furthermore in case you want to parse it in some other place you would need to `copy`
all the logic from this method to another, which violates one of the most important
rules of coding - DRY. 

We gonna create a `distinct class` for every feature that we gonna use throughtout 
our api (sorting, filtering, pagination, etc), call it `APIResult` and place it into
`Utils` folder that will be located in the root directory.

This class will receive a query from client : `req.query` as `query`
and a query to the database to `pipe onto` : `<Model>.find()` as `dbQuery`

```js
// APIResult in Utils
class APIResult {
    constructor(dbQuery, query) {  // note constructor
        this.query = query;
        this.dbQuery = dbQuery;
    }
    
    Result() {
        return this.dbQuery; // result returns the current query for awaiting
    }
    
    OnlyProperties() {
        if(this.query.fields){
            const getOnly = this.query.fields.split(',').join(' ');
            this.dbQuery = this.dbQuery.select(getOnly);
        } else {
            this.dbQuery = this.dbQuery.select('-__v'); // note ADDING to dbQuery
        }
        
        return this; // <---- note return THIS for PIPING
    }
    
    Filter() {
        let query = {...this.query};
        let extraQuery = ['page', 'sort', 'limit', 'fields'];
        extraQuery.forEach(ext => delete query[ext]);

        let queryStr = JSON.stringify(query);
        queryStr = queryStr
            .replace(/\b(gte|lte|gt|lt)\b/g, match => `$${match}`);
        query = JSON.parse(queryStr);

        this.dbQuery = this.dbQuery.find(query);
        
        return this;
    }
    //...
}


// in Model Actions 
const {APIResult} = require('../Util/APIResult');
//...
const getAllTours = async (req, res) => {
    try {
        const resultQuery = new APIResult(Tour.find(), req.query); // note parameters
        const tours = await resultQuery
            .Filter()                                   // note piping
            .OnlyProperties()
            .Pagination()
            .Sort()
            .Result();

        res.status(200).json({
            status: `success`,
            data: tours,
            dataLength: tours.length
        })
    }
    catch (e) {
        res.status(404).json({status: `failed`, message: e.message});
    }
};
```

## Aggregation pipeline
Is a `mongoDB framework` for data aggregation.

We define a pipeline that all documents `from a certain collections go through`
in order to transform them into `aggregated results`.

E.g it may be used to calculate averages, minimum or max. values.

1. Create another function in your actions.
```js
const getTourStats = async (req, res) => {
    try {
        const stats = Tour.aggregate() // note the aggregate function
```

2. To that function you have to pass an `array of steps` that you want to be performed
    on every model. Every step is a distinc `object`. (complete list can be found in
    mongodb documentation)


```js
const stats = await Tour.aggregate([            // note await
    {
        $match: { 'ratingsAverage' : { $gte: 4.5 }}
    },
    {
        $group: {
            _id: null,              // grouping by some property, in this case NONE
            avgRating: { $avg: '$ratingsAverage'}, // calculate average for a group
            avgPrice: { $avg: '$price' },       // avgPrice is a new Property, price 
            minPrice: { $min: '$price' },       // is a property from a Model
            maxPrice: { $max: '$price'  },      // note dollar in front of price
            numRatings: { $sum: '$ratingsQuantity' }, // [1]
            num: { $sum: 1 } // 1 will be added to the sum for every document
        }
    }
]);
```

[1] it will add ratingsQuantity (which is a number) to the `numRatings` stats property
    for every document in the `group`.

3. To `group` by some property you need to specify a property to group by.
    ```js
    $group: {
        _id: '$difficulty',    // grouping by difficulty, _id is the id of THE GROUP
        avgRating: { $avg: '$ratingsAverage'}, // calculate average for a group
        avgPrice: { $avg: '$price' },       // avgPrice is a new Property, price 
        minPrice: { $min: '$price' },       // is a property from a Model
        maxPrice: { $max: '$price'  },      // note dollar in front of price
        numRatings: { $sum: '$ratingsQuantity' }, // [1]
        num: { $sum: 1 } // 1 will be added to the sum for every document
    }
    ```

4. To `sort`
    ```js
    {
        $sort: { avgRating: 1 } // specify the property and the 1 or 0 (asc/desc)
    }
    ```
    Note that depending on the order in the pipeline you may have access to
    different properties to sort by. In this case you had a grouping above so in this case
    `you only have access` to the properties that were defined in that group


5. `Match` again, don't forget, you can `repeat` diferent aggregation objects, i.e
    you may want to repeat match. But note the properties that you can access.
    ```js
    {
        $match: { '_id' : { $ne: 'easy' }} // $ne - not equal
    }
    ```
    In the case above you just filtered out the group with id "easy".
    

6. Example: finding the most busy months in an array of tours that have starting dates.
    1. We need to count amount of tours for every single month. We are going to use an
        `unwind` mehtod which will create a  `distinct document` for every value in
        an array of some value of that document
            ```js
            let someDoc = {name: 'aviato', someVal: [val1, val2]};
            ```
            In this case it will create two distinct documents, both with 
            the name `aviato` for every value in the `someVal` array. Result:
            ```js
            doc1 = {name: 'aviato', someVal: val1};
            doc2 = {name: 'aviato', someVal: val2};
            ```
            So now we `have distinct tour for every date` => result
            
    2. Now we need to match the correct year, let's use `match` for that 
        ```js
        const plan = await Tour.aggregate([
            {
                $unwind: '$startDates'
            },
            {
                $match: { 
                    startDates: { 
                        $gte: new Date(`${year}-01-01`),
                        $lte: new Date(`${year}-12-31`)
                    }
                }
            }
        ]);
        ```
    3. Then we need to `group` by months, but how can we do that considering 
        we don't have months of tours but only dates? 
        We will need a special operator that you can find in the documentation.
        They are called `Aggregation pipeline operators`
        The one we are going to use here is called `$month`
        
        But then after we have them, how do we get the name of all aff them per month?
        We would like to create an `array` of all the names that we have in group,
        to do that we `have to use` another operator, as we can't just pass an array
        This operator is called `$push`, which creates an array from some property
        ```js
        $group: {
            _id: { $month: `$startDates` },
            numOfTours: { $sum: 1 },
            tours: { $push: `$name` }
        }
        ```
    4. Final result
        ```js
        const plan = await Tour.aggregate([
            {
                $unwind: '$startDates'
            },
            {
                $match: { 
                    startDates: { 
                        $gte: new Date(`${year}-01-01`),
                        $lte: new Date(`${year}-12-31`)
                    }
                }
            },
            {
                $group: {
                    _id: { $month: `$startDates` },
                    numOfTours: { $sum: 1 },
                    tours: { $push: `$name` }
                }
            },
            {
                $sort: {
                    numOfTours: -1
                }
            },
            {
                $addFields: { month: `$_id` } // add aliases to fields and more
            },
            {
                $project: { _id: 0 }   // allows you to add or remove fields from result
            },
            {
                $limit: 1  // get only the most busy month
            }
        ]);
        ```

# ▶ Day 5
# Advanced Modelling
## Virtual properties
Virtual properties are the fields that you define on the model schema but which
`won't be persisted in db`.

They make sence when you don't want to store some properties
`that can be direved from other properties`

To set a virtual property on a schema you need to call it's `virtual` method
which takes just the name of the property you want to add. Then you need to
pipe the `get` getter method for the property.

By default, virtual properties are `neither` persisted nor returned from a query.
To get the property on "get" you need to specify `options` in schema constructor.
```js
const tourSchema = new mongoose.Schema({
    name: {
        type: String,
        required: [true, "A tour must have a name"],
        unique: true,
        trim: true
    },
    /* ... */
}, { // schema options
    toJSON: {virtuals: true},
    toObject: {virtuals: true}
});

tourSchema
    .virtual('durationWeeks')
    .get(function() {   // < ------- GETTER for the property
        return (this.duration / 7).toFixed(2); 
    });
```

**Note**, you `can't` use virtual properties in `queries`.

## Document hooks
`Mongo` has a concept of `hooks` which allows you to run actions before and after
a certain event. There are four types of middleware in mongoose: 
    1. document     3. aggregate
    2. query        4. model

Hook is to be defined `on the schema`.

1. First type is `document hook`
    It runs before `.save()` and `.create()` command,
    but `not` for insertMany or any other
    
    In the callback function you have access to `this` which points to the `document`
    ```js
    tourSchema.pre('save', function() {
        console.log(this);
    });
    ```
    
    You can use this hook, to create a `slug` for example. Slug is a string that
    you can put in the url, based on the name of the document.
    
    Finish result
    ```js
    tourSchema.pre('save', function(next) {
        this.slug = slugify(this.name, {lower: true}); //
        
        next(); // don't forget to call next
    });

    tourSchema.post('save', function(doc, next) { // post(meaning after) hooks you have
        console.log(`The result document : ${doc}`); // access to doc instead of "this"
        
        next();
    });
    ```
    Document middleware can `also` run for validate, init and remove.

2. Second type - `query middleware`, they allow us to run actions before/after
    `a certain query` is executed. The difference between query and document hooks is in 
    the string you are passing to the `pre/post` functions.
    Now, `this` will point to the query, instead of the document.
    ```js
    tourSchema.pre('find', function(next) { // note pre('find', ...)
        
        next();
    });
    ```
    
    Imagine you want to create some "premium" tours that not everyone has access to.
    You would create a special field in the Tour schema for that.
    
    As we have access to `this` which is the query object, we can pipe more methods to it.
    ```js
    tourSchema.pre('find', function(next) {
        this.find({isSecret: {$eq: true}});
        
        next();
    });
    ```
    **Note** although this hook works for `find` it doens't really work for any other 
    method like "findOne" or "findMany" and you may want to fix this behaviour.
    The best way to fix this is by using a `regular expression`.
    
    Find middleware can run for much more different queries, see the docs.
    
    **Note** in the post middleware you have access to `this` as well which points 
    to the same document you had in "pre" hook, which allows you to do the following:
    finish result
    ```js
    tourSchema.pre(/^find.*/, function(next) {
        this.find({isSecret: {$ne: true}});
        this.start = Date.now();                                // set the timer to this
        
        next();
    });

    tourSchema.post(/^find.*/, function(docs, next) {
        console.log(`The result count - ${docs.length}`);
        console.log(`Query took ${Date.now() - this.start} milliseconds!`); // get time
        
        next();
    });
    ```

3. Third type - `aggregation middleware`
    This type allows us to run hooks before or after `aggregation`
    Using this hook you can decide for example which documents should take part
    in aggregation and which shouldn't.
    
    In aggregation middlewre `this` points to the `aggregation object`.
    For example you can exmine the aggregate pipeline with the method called `pipeline()`.
    It returns you an `array of aggregation objects`.
    
    E.g you can add another aggregation object at the beginning with `unshift` or
    at the end with `push`.
    ```js
    tourSchema.pre('aggregate', function(next) {
        this.pipeline().unshift({$match: {isSecret: {$ne: true}}}); // adding another 
        this.pipeline().pop(); // removing an aggr. obj             // aggregation obj.
        console.log(this.pipeline());// see the pipeline

        next(); // dont forget about NEXT
    });
    ```

# Validation
## Built-in
1. `Validation` - checking that the input is in accordance to the schema.
2. `Sanitization` - checking for malicious code in the input.

Some `built-in` validators:
```js
// in a schema
name: {
    type: String,
    trim: true,
    maxlength: [40, 'A tour name must have less or equal than 40 characters'], // length
    minlength: [10, 'A tour name must have more or equal than 10 characters'],
},

difficulty: {
    type: String,
    required: [true, 'A tour must have a difficulty'],
    enum: {                                                 // enum for strings
        values:['easy', 'medium', 'difficult'],
        message: 'Difficulty can only be easy, medium or difficult',
    },
},
ratingsAverage: {
    type: Number,
    default: 3,
    min: [1, 'Rating must be above or equal to 1.0'],       // max/min numbers
    max: [5, 'Rating must be below or equal to 5.0']
},
```

## Custom validators
Validator is a function that returns a boolean.
To specify a validator, use `validate` property of `options object`
```js
priceDiscount: {
    type: Number,
    validate: {
        validator: function(value) {
            // "this" is the doc ("this" works only for NEW documents)
            // fixing this is cumbersome and is not worth it
            return value < this.price; // this.price => access to another field
        },
        // {VALUE} is the value of the property
        message: 'Discount price should be less than the price, invalid value - {VALUE}'
    }
},
```

Popular npm package for `string` validation is called `validator`.
But you can use default mongoose validators in most cases.

# Error handling with express >>>
## Debugging node.js
Google developed an npm package called `NBM`
```
npm i -g nbm
```
Then add new script to your `npm package`
```json
"debug": "ndb server.js"
```
 
This package opens your project in a `special chrome-like window`, in which
you can make changes.

1. Setting up `breakpoints`
    1. In the window, click on a `line number`.
    2. If the program has already gone pass this line you can right-click on the 
        file and click `Run this script`

2. Degugging piped functions
    In order to debug a certain function in a pipeline of multiple functions 
    you need to stop at the function call and then press `step`, wich will 
    let you step into function call.
    ```js
    const tours = await resultQuery
            .Filter()
            .OnlyProperties()
            .Pagination()
            .Sort() // set breakpoint here to see what this function does.
            .Result();
    ```

## Handling unhandled routes
If you have defined routes without any action method, when you try to access the route
you get an `error`. But the problem here is that the erorr is returned in the `html form`

To fix this we need to create a route that will catch everything that `was not catched`
by any other route. To do that we just have to create a route that will be executed
`last`, that why we put it in the `bottom of` the middleware `pipeline`.
```js
// all = any verb, * = any path
app.all('*', (req, res, next) => {
    res.status(404).json({
        status: 'fail',
        message: `Can't find ${req.originalUrl} on this server :(`
    });
});         

module.exports = app; 
```

## Overview
Errors should be handled in some `central error handling place`.

There are two types of errors: `Operational errors` and `Programming errors`.
1. Operational errors
    Problems that `we can predict` will happen, like 
        - failed to connect to a db
        - invalid user input
        - invalid route access
    There problems are `not` about `bugs in our code` but rather, bad external conditions.
    (bad user, bad server, bad connection)
    We should strive to make all possible errors in our app operational.

2. Programming errors
    Actual `bugs` that are difficult to track and debug.
        - using wrong arguments
        - null access
        - fail to parse a value properly
    
Express allows you to easily catch and process `operational errors`, by using 
an `error handling middleware` that will be responsible for catching all kinds
of errors in your app from different places.

This kind of error handling is nice as it allows you to `separate` error handling
from actual business logic in your code.

# ▶ Day 6
## Implementing global error handling middleware 
Express comes with error handlers out of the box. You just need to create an
error handling middleware, this middleware has `four` arguments: error, res, req and next.

`But` when you have a single middleware for handling all types of errors, you
can't know what `status code` should you send to the client.

You can create an error in your middleware as well as statusCode and status.
This way you create an error in your middleware but `not handle it`.
After that you call next() but with an `error argument`.

Next receives a parameter, this parameter is an `error`. Whatever you pass to next(), is
assumed to be an error. So when you pass an error, all the middlewares are `skipped`
but not error middleware.

Result:
```js
app.all('*', (req, res, next) => {
    const error = new Error(`Can't find ${req.originalUrl} on this server`);
    error.status = 'fail';
    error.statusCode = 404;

    next(error);
})

app.use((err, req, res, next) => {
    err.statusCode = err.statusCode || 500;
    err.status = err.status || 'error';
    res.status(err.statusCode).json({
        status: err.status,
        message: err.message
    })
});
```

But passing multiple properties to the error object may be cumbersome, so it might 
be better to create `your own error class`.

## Creating a custom Error class and refactoring
We need to create an error class that will extend the base Error class and have
properties that we want. Also it should specify whether the error is `operational` or not.
Because we don't want client to know about error that occured because of a bug in 
our system, we only want to know what we have prepared for them and `no more`.

This class should also allow us to capture the `stack trace` so that we could know 
where the error happened.
`Note` you can access error's stack trace with the `stack` error property.

```js
class AppError extends Error {
    constructor(message, statusCode) {
        super(message);                 // this property already exists on the base class
        this.statusCode = statusCode;
        this.status = statusCode < 500 ? 'failed' : 'server error';
        this.isOperational = true;

        Error.captureStackTrace(this, this.constructor); // the way to capture the stack
    }
}

module.exports = AppError;
```

```js
// You can put it inside actions. maybe?
// it's an error handling action-middleware
const errorHandler = (err, req, res, next) => {
    err.statusCode = err.statusCode || 500;
    err.status = err.status || 'error';
    res.status(err.statusCode).json({
        status: err.status,
        message: err.message
    });
};

module.exports = errorHandler;
```

```js
// app.js
app.all('*', (req, res, next) => {
    const error = new AppError(`Can't find ${req.originalUrl}`, 404);
    next(error);
});

app.use(errorHandler);
```

## Catching errors in a single function
For now we have multiple actions, and each action handles erros by itself.
So we want `to get try/catch out of actions and put it in one specialized place`.

So we should create a function that does just that and place it inside our 
`Utils` folder.
```js
// in tourActions.js that is in Actions directory
module.exports = {
    createTour: catchAsync(createTour),
    updateTour: catchAsync(updateTour),
    deleteTour: catchAsync(deleteTour),
    getTour: catchAsync(getTour),
    getAllTours: catchAsync(getAllTours),
    getTourStats: catchAsync(getTourStats),
    getMonthPlan: catchAsync(getMonthPlan),
}


// catchAsync.js
const catchAsync = fn => (req, res, next) => {
    fn(req, res).catch(next);
}

module.exports = catchAsync;

// that's how actions look like now
const createTour = async (req, res) => {
    const tour = await Tour.create(req.body); // note, no try/catch
    res.status(200).json({status: "success", data: tour});
    // if an error occurs it will be handled by the catchAsync function
};
```

## Adding 404 not found
For now if something goes wrong we get 500 internal error. Which is not what we want.
To make errors meaningful we need to `throw them inside our actions`.

```js
// inside an Action method
const tour = await Tour.findById(req.params.id);
if(!tour) 
    throw new AppError(`Tour with id ${req.params.id} does not exist`, 404);
```

- **So, as of now** we have the following `architecture`:
    1. We have async methods that throw `custom errors`.
    2. We have a wrapper method, which `modifies` every other async method
        in such a way that it `redirects` errors into some 
        `central custom error handling middleware.` by calling `next(err)` in `catch`
    3. We have custom middleware that handles custom errors the way we like it to
        in app.js

<!-- here -->
## Errors in development vs production
You probably want to see more meaningful errors when you develop the program and
don't want the client to see the same errors in production.

And **remember** we only want to send to the client the errors `that are operational`.
```js
// in Actions/errorActions.js
const operError = (err, res) => {
    res.status(err.statusCode).json({
        status: err.status,
        message: err.message
    });
}
const devError = (err, res) => {
    if(err.isOperational) {
        operError(err, res);
    } else {
        res.status(err.statusCode).json({
            status: err.status,
            message: err.message,
            stackTrace: err.stack,
            error: err
        });
    }
}
const prodError = (err, res) => {
    if(err.isOperational){
        operError(err, res);
    } else {
        console.error(err);                 // we wanna see the error on our side
        res.status(err.statusCode).json({
            status: 'error',
            message: 'Something went wrong!'
        });
    }
}

const errorHandler = (err, req, res, next) => { // our errorn hadling middleware
    err.statusCode = err.statusCode || 500;
    err.status = err.status || 'error';

    if(process.env.NODE_ENV == 'development' ) {       // different responce for dif. env.
        devError(err, res); 
    } else if (process.env.NODE_ENV == 'production') { // different responce for dif. env.
        prodError(err, res);
    }
};
```

## Handling "invalid database ID" error
When user tries to access an invalid id, it won't get any meaningful feedback in
production. We need to `make` this type of error `operational`.

Same with validation errors (at this point they are not operational and can only
be seen in development environment.)

That's how easy it is to make it:
```js
const prodError = (err, res) => {
    let appError = err;
    
    if(err.isOperational)
        return operError(appError, res);
    
    switch (err.name) {  // just make a SWITCH on error name, and define 
        case 'CastError': // operational error on any name you want
            appError = new AppError(`Invalid ${err.path}: ${err.value}`, 400);
            return operError(appError, res);
        
        default:
            appError = new AppError(`Something went wrong :(`, 500);
            return operError(appError, res);
    }
}
```
You can get error `names` from docs or just by experimenting. 

## Handling "Duplicate field" error
This will create a little problem as it `doesn't have a name`.
It doesn't have a name because it isn't a mongoose error but rather 
MongoDB error. So to identify the error we are going to use `code` property instead.

So here we have done some little refactoring (switch true)
```js
switch (true) { // use switch true noe
    case err.name == 'CastError':
        appError = new AppError(`Invalid ${err.path}: ${err.value}`, 400);
        return operError(appError, res);
        
    case err.code == 11000: // check for err code instead of name
        let property = err.errmsg.split('"')[1]; // get the property
        appError = new AppError(`Duplicate property: ${property}`, 400);
        return operError(appError, res);
    
    default:
        appError = new AppError(`Something went wrong :(`, 500);
        return operError(appError, res);
}
```

## Handling Mongoose's validation errors
We have multiple `properties` in error object.
Every some property corresponds to the property that ruined validation.
You wanna take these properties and create a single string out of it.

```js
case err.name == 'ValidationError':
    let error = Object.values(err.errors) // get error messages for every property
        .reduce((b,v,i) => b + (i && ', ') + v.message, ''); // and combine them
    appError = new AppError(`Invalid input data: ${error}`, 400);
    return operError(appError, res);
```

## Errors outside of Express: Promise Rejections
E.g your database doesn't work. You need to have some way of handling these errors.
Database is outside of your express app so no middleware will catch them.

When your database doesn't work you get `unhandled promise rejection`, 
which you can easily fix, by adding `.catch` to your mongoose connection.

But you may want to handle other types of simillar errors `globally`.

When an unhandled rejection happens, the `process` object emits an `event` called
`UnhandledRejection`, so we can easily subscribe to the event and handle it 
as it occurs.
```js
const server = app.listen(port, () => { // SAVE THE SERVER INTO A VARIABLE
    console.log(`app running at port: ${port}, in environment: ${process.env.NODE_ENV}`);
});

process.on('unhandledRejection', err => { // note 'unhandledRejection' and process.on
    console.error(err.name, err.message);
    console.log("Unhandled rejection! Shutting down the server...");
    server.close(() => { // PROPER WAY OF CLOSING AN APP on Rejected promise
        process.exit(1); // gives server time to finish ongoing requests
    });
});
```

This way wherever in the app async error occurs it will be handled by 
this event listener.

## Errors outside of Express: Synchronous Rejections (Uncaught Exceptions)
we can listen to `process.uncaughtException` this time, but everything else is 
the same, except that you don't need now to wait for the server to close.

Although you do have these central error handling listeners, you `shouldn't rely on them`
It's just there in case you missed something occasionally, but in general
you should really `handle` error and not just rely on there listeners that
will shut down your server.

**Note** this `synchronous` listener should be put on `top` of the server.js file
as it needs to listen `before` any error occurs.

**Note** sync error that occur `inside` of express `middleware` are handled by
error handling middleware.

# ▶ Day 7
# Authentication, Authorization and Security >>>
## Modelling Users
Creating a user:
```js
const mongoose = require('mongoose');

const userSchema = mongoose.Schema({
    name: {
        type: String,
        required: [true, 'Please, provide a name'],
    },
    email: {
        type: String,
        validate: {
            validator(val) {
                return val.match(/^.+@.+\.[a-z]{2}/);
            },
            message: 'Invalid email address!'
        },
        unique: true,
        lowercase: true
    },
    photo: String,
    password: {
        type: String,
        minlength: 8,
        required: [true, 'Please, provide a password'],
    },
    passwordConfirm: String,
});

const User = mongoose.model('User', userSchema);
module.exports = {User};
```

## Creating new users
1. We need to create another actions class : `AuthActions`.

```js
// in UserRoute.js of Routes directory
router.post('/signup', signUp); // signUp from AuthActions
```

```js
// authActions.js in Actions directory
const signUp = async (req, res, next) => {
    const newUser = await User.create(req.body);

    res.status(201).json({
        status: 'success',
        data: newUser,
    });
};

module.exports = {
    signUp: catchAsync(signUp),
}
```

## Managing User passwords
**Note** `Remember` `not` to use arrow functions in `validator` function in mongoose 
schema.

And `Remember` that `validator only runs` on save and create and not on update.
```js
// confirmPassword field validation
validate: {
    validator: function(field) {
        return field == this.password;
    },
    message: 'Passwords are different!'
}
```

You should `never` save passwords as is.

If you want to stick to `fat model thin controller` philosophy, you should implement
the password hashing functionality `on the model`

Install `bcryptjs` npm package

```js
// mongoose hook on user scheme
userSchema.pre('save', async function(next) {
    if(!this.isModified('password'))      // check whether the field has been modified
        return next();                    // to not to rehash the pass on UPDATE
    else {
        // bcrypt returns a promise
        // 12 means how much computer power to waste on salting and encryption 
        this.password = await bcrypt.hash(this.password, 12);
        this.passwordConfirm = undefined; // delete this as we don't need it to persist, 
        return next();                    // only to check on model creation
    }
});
```

## JWT token Auth basics
1. There's `no need` to store `session state` on the server.

Auth **workflow**
1. User posts it's login and password
2. JWT is created for the user if it passed validation on the server.
3. Server sends JWT to the client
4. Each time user wants to access a protected route, he supplies his token along with
    the request.
5. App verifies the token

Note this communication `must` happen over `https` not http.

JWT `consists of` three parts :
    1. header
        info about the JWT itself
    2. payload
        data that we can encode
    3. signature
        encoded string for verification
        The `signing algorithm` takes the header, payload and `unique secret` to 
        create a `unique signature` which is then passed together with
        header and payload to verify them.
    
## Signing up Users
Proper authorizatoin and authentication is important.
`Passport` library may help you with this but you still need to be careful.

`Preventing` users from setting the role:
```js
const userData = Object.entries(req.body)
    .filter(v => v[1] != 'role')
    .reduce((b, v) => (b[v[0]] = v[1], b), {});
```

`Automatically loggin a user`
1. Install `jsonwebtoken` npm package
```js
const newUser = await User.create(userData);
const token = jwt.sign(
    {id: newUser._id},      // payload (user id)
    process.env.JWT_SECRET, // secret (from env vars)
    {
        expiresIn: process.env.JWT_EXPIRATION // additional JWT options (expiration time)
    }
);

res.status(201).json({
    status: 'success',
    token,                  // sending the toke back to the client in HEADERS
    data: newUser,
});

// env vars
JWT_SECRET=v10i0h2104v9ksad$d0fsd9fjkdsf%%df // note the secret should be 32+ characters
JWT_EXPIRATION=90d  (or 50h, or whatever)    // you can specify expiration like this 
```

## Logging users IN
1. We need to check that the user with the email exists
2. Check that the password is correct
3. Return token

**Note**
At this point when you return a user, you also return `it's hashed password`.
Which is not a good practice. The hashed password should stay on ther server,
user shoul only have the token.

To fix this problem you can `just` set `select:false` option on the field.
```js
// on User model
password: {
    type: String,
    //...
    select: false       // prevents the property from showing up in Find* queries
}
```

We want to check whether user provided the right password, 
for that we are going to create an `instance method` on the User model.

```js
userSchema.methods.verifyPassword = async function(attempt, password) {
    return await bcrypt.compare(attempt, password);
} // please, note that this is an async method
```

`Loggin the user in`
In some cases we want `password` for ourselves, not for the client 
in this case we will have to `explicitly select` the property with `select('+prop')`
```js
const login = async (req, res) => {
    const {email, password} = req.body; 
    if(!email || !password)
        throw new AppError(`Invalid credentials: you need to specify both password and login`,
            400);
    let user = await User
        .findOne({email})
        .select('+password'); // explicitly selecting the property
    let correctPassword = user 
        ? await user.verifyPassword(password, user.password) 
        : false;

    if(!user || !correctPassword)
        throw new AppError(`Invalid credentials: invalid password or email`, 400);

    const token = createToken({id: user._id});
    res.status(201).json({
        status: 'success',
        token,
        data: user,
    });
}


// the createToken mehtod that was defined besides middlewares, as a supportive method
const createToken = (payload) => {
    const token = jwt.sign(payload, process.env.JWT_SECRET, { // JWT config
        expiresIn: process.env.JWT_EXPIRATION
    });
    return token;
}
```

## <30> Protecting Routes






































































































