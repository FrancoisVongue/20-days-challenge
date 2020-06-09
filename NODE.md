# ▶ Day 1
# Hello node
To set headers :
```js
res.setHeader('Content-Type', 'application/json')
```

## Basic routing
`req.url` property contains relative path to the resource from the domain.

## Express
the main feature of express for node developer is it's built in `router`
By switching to express we can take advantage of its router which simplifies development.

By calling `res.json()`, express will automatically send
the correct Content-Type header and stringify our response body for us.

`req.query` - represents query parameters from the path (/?name=value)

In the next snippet
```js
app.get('/static/*', respondStatic)
```
asterisk signifies the whole left path.

## Real time
For real time communication (e.g messaging app) you can use `server sent Events`

# Express
## Postman
1. download postman
2. use postman to test your api
3. How to save requests
    1. create a new collection
    2. save request into a collection (save button is near the send button)
    3. give the request some structured name (like: fine new item)

## setting up project
1. setup npm with npm init
2. setup express with npm i express
3. setup eslint

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
    URL paths should be structured and only define what resource should be accessed.
    www.xxx.addTour - bad path, because it contains a `verb`
    
3. Define what you want to do with resources with `http verbs`

To get resources that are "part" of another resource:
    `/users/4f2324/tours/53a7f8`

4. REST should always be `stateless`
    Meaning all the state should be handled on the `client` and not on the server

## Handling requests with Express
You can compose responses according to JSend spec. Which is a spec that controls 
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
        const newTour = Object.assign({id: newId}, req.body);
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
        const tour = tours.find(t => t.id == req.params.id);

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

You can `set properties` on the req object for use in later middlewares.
```js
app.use((req, res, next) => {
    req.requestTime = new Date().toISOString();
    next();
})
```
-
                                    **Note**
In custom middleware when you send a response, you have to return after that.
If code will reach the call to `next()` function after you have returned a response
to a client you will get an error.

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
you can create folders (may be useful to create a different folder for different model)

## Code structure +
You may want to divide `different routes for different resources`.
And different directories for routes and functions that are called by the routes.

1. You need to create one separate `router` for each resource.
    ```js
    const tourRouter = express.Router(); // one router for one resource
    tourRouter.route(`/`) // use the router instead of app
        .get(getTours)
        .post(postTour);

    tourRouter.route(`/:id`)
        .get(getTour)
        .delete(deleleTour)
        .patch(patchTour);

    app.use(`/api/v1/tours`, tourRouter); // when mounting route to the app, you can 
                                        // supply the base path
    ```
    Router is a `middleware` so you can easily moun it to your app with
    `app.use(someRouter)`
    You just define a router and then assign
    `different http verbs` and relative `paths` for that router
    The process of mounting the router to the app is called `mounting the router`.

2. It may be easier to remember is you imagine that router is an ASP.NET `controller`
    that contains the base path. 
    And different verbs with different relative paths are just `actions`.

3. Separate different routers into different files
    1. Create another folder called `Routes`
    2. In that folder create distinct file for each model and call them `modelRoute`.
    3. In every "route" file define express.Router's and assign functions to them.
    4. Export the routers and import them in app.js.
    This way you can have different routers for some specific base path and 
    some custom middleware for all requests in your app.js file.
    
4. Create another folder where you will store `functions`, and call it `actions`.
    1. Create different files for different models like this: `modelActions`
    2. Put all request handlers in there
    3. Import the actions into routers to use.

5. Create another file in the root folder called `server.js`
    1. import `app` from app.js
    2. create port variable
    3. call `app.listen()` in this file
    This file will be resposible for things like:
        1. Database configuration
        2. Error handling
        3. Env. vars
        4. etc

6. Create an npm script for running your app
    ```npm
    npm scripts>
        node server.js
    ```

## Param middleware - special type of middleware
Is a type of middleware that `only runs` when we have special parameters in `URL`

E.g you can write a middleware that will only run when there's an `id` parameter in URL.
You can create a param middleware by calling `param` instead of "route" on the 
router object and passing to the function a callback with `four` paramers, the last
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

// in another file, in Acitons directory
const validateTour = (req, res, next) => {
    let keys = new Set(Object.keys(req.body));
    if(keys.has('name') && keys.has('price')) {
        next();
    } else {
        res.status(400).send("Invalid request body");
    }
};
```

## Serving static files
You need to do that using middleware.
```js
// in app.js
app.use(express.static(`${__dirname}/public`)); // to serve files from the public folder
                                                // that is located in your root folder.
```

Then to access the file you need to access `domain/rpf` where rpf is the relative
path from public directory to the file, inluding the file name.

## Env variables
In `app.js` you should do everything that is only relevant to `express`
Env var is not about express so you should configure them in `server.js`

To get environment you can type
```js
app.get('env'); // return the string that represents the environment
                // by default it's "development"
```

"env" only tells you what environment you are running your program in.
Environment is stored in enviroment variable. And there `a lot of environment variables`.

To see all environment variables you can use this property
    `process.env`
if you console.log it you will see all the environemt variables there are


You can set `environment manually` by setting the `NODE_ENV` environment variable 
when you start the app.
```cli
NODE_ENV=development node server.js
```
But when you need to set multiple environment variables you will need a distinct 
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

4. E.g to add logging only in `development` environment
    ```js
    // app.js
    if(proccess.env.NODE_ENV == "development" )
        app.use(morgan('dev')); 
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
the npm like `this`:
```json
"scripts": {
    "start:dev": "node server.js",
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
































































































