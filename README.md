=====================================================================================

# Full Stack Responsive React Movies App With API | MERN Project | React Tutorial

PART ONE: `https://www.youtube.com/watch?v=j-Sn1b4OlLA&t=2421s&ab_channel=TuatTranAnh`
PART TWO: `https://www.youtube.com/watch?v=Q_uLi4f27Lc&t=2603s&ab_channel=TuatTranAnh`

GH: `https://github.com/trananhtuat/fullstack-mern-movie-2022`
DEMO: `https://moonflix.vercel.app/`

=====================================================================================

# SERVER SETUP

1. Add moonflix folder
2. in it add client and server folders
3. setup `server` folder for typescript
4. install dependencies
5. add dev script in package.json
6. add `.env` in server folder and add `TMDB base url` and api key
   See `https://www.themoviedb.org/settings/api`

7. in src folder create `index.ts` and add code inlcuding `middleware` and `mongoose connnect` to database code
8. in src folder create the following folders: `axios`, `controllers`, `models`, `routes`, `tmdb`
   `handlers` and `middlewares`

////////////////////////////////////////////////////////

# Creating API endpoints and Axios configuration

9. in `axios` folder create `axios.client.ts` file and add code
10. in `src` create `interfaces` folder and in it add `types.ts` for now just what's required
11. in `axios.client.ts` import from `types.ts` and use in code
12. in `tmdb` folder create `tmdb.config.ts` and add code
13. Now in `tmdb` folder create `tmdb.endpoints` and add code
14. In `tmdb` folder create `tmdb.api.ts` and add code

Notes on the above files

The high-level flow starts from the application code that uses the tmdbApi methods to request
media-related information from the TMDb API.

In `tmdb.api.ts` The `mediaList()` method calls `tmdbEndpoints.mediaList()` to construct the appropriate URL for
the API request based on the provided parameters.

Inside `tmdbEndpoints.mediaList()`, it uses `tmdbConfig.getUrl()` to get the fully constructed URL by appending
the parameters to the base URL and filtering them to include only valid string values.

Once the URL is constructed, the `tmdbEndpoints.mediaList()` method calls the `axiosClient.get()` (as is the case with
higher order methods) to make the actual HTTP GET request to the TMDb API.

The `axiosClient.get()` method sends the HTTP GET request using Axios and returns the API response data
as a Promise.

The Promise is resolved, and the API response data is returned back through the chain of function calls until
it reaches the application code that originally made the request.

# high-level flow

`tmdbApi.ts` -> `tmdbEndpoints.ts` -> `tmdbConfig.ts` -> `tmdbApi.ts` -> `axiosClient.ts` -> API request to TMDb API ->
Response back to `axiosClient.ts` -> Response back to `tmdbConfig.ts` -> Response back to `tmdbEndpoints.ts`->
Response back to `tmdbApi.ts` -> Response back to the application code.

Example of URL from: `${baseUrl}${endpoint}?api_key=${key}&${qs}`
`https://api.themoviedb.org/3/movie/comedy?api_key=your_api_key&page=1`

In the above URL, `qs` is the part: `&page=1`, which represents the page query parameter in the URL.
The page value, which is 1, is appended to the URL after the & symbol to signify the start of the
query parameters section.

/////////////////////////////////////////////////////////////////

# Adding Models Model options and interfaces

1. in `models` folder add `model.options.ts` and add code and new interface

Notes on `model.options.ts`

`toJSON:` This property is an object that contains two sub-properties:

- `virtuals:` It's a boolean value set to true. This indicates that `virtual properties` (computed properties that
  do not exist in the database but are dynamically generated by the model), should be included when converting
  the model instance to JSON.

- `transform:` It's a function that takes two parameters, `error` and `obj`. The purpose of this function
  is to modify the JSON representation of the model instance before it is converted to ` JSON`. In this case, it
  removes the `_id` property from the object and then returns the modified object.

- `toObject:` This property is similar to toJSON and also contains two sub-properties:

- `virtuals:` It's a boolean value set to true. This indicates that virtual properties should be included
  when converting the model instance to a regular JavaScript object (not JSON). For example, you might have a
  `fullName` virtual property that combines the `firstName` and `lastName` fields from the model.

- `transform:` It's a function similar to the one in the toJSON property. It takes two parameters, `error`
  and `obj`, and its purpose is to modify the JavaScript object representation of the model instance
  before it is returned. In this case, it removes the `_id` property from the object and then returns
  the modified object. this may be useful for example: `data privacy`, `consistant data structure` etc

- `versionKey`: It's a boolean value set to false. This property controls whether Mongoose includes a `__v` field
  in the saved documents. The `__v` field is used to track the version of a document.

/////////////////////////////////////////////////////////

2. in models folder create `user.model.ts` file and add code and add new interface
3. in models folder create `review.model.ts` file and add code and add new interface
4. in models folder create `favorite.model.ts` file and add code and

# Request handlers with express validator, and token middleware

1. in handlers folder add two files:
   `request.handler.ts` and `response.handler.ts` and add code

NOTES ON: `request.handler.ts` and `response.handler.ts`

`request.handler.ts`

This file contains middleware to handle request validation using `express-validator`. The main purpose of this
file is to ensure that incoming requests are validated before proceeding to the actual route handler. It exports a
single function named `validate`, which takes three parameters: `req (Request object)`, `res (Response object)`, and
`next (NextFunction).`

The function checks for validation errors using `validationResult` from `express-validator` and
responds with a `JSON object` containing the first validation error message if there are any errors. If there are
no errors, it calls the next() function to pass control to the next middleware or route handler.

The file also includes an interface named `Validate`, which defines the shape of the validation error object.

`response.handler.ts`

This file contains helper functions for sending different types of responses from the server using the
`Response object in Express`. It exports several functions: `error`, `badrequest` `ok`, `created`,
`unauthorize`, and `not found`. Each function takes a Response object as its parameter and uses the
`responseWithData` function to send a JSON response with the provided data and the appropriate status code.

The `responseWithData function` itself takes a `Response object`, a `status code`, and an object conforming to the
`ApiResponse` interface (imported from the `types.ts` file).

These helper functions allow you to send standardized responses for common scenarios like errors, successful
responses, created resources, unauthorized access, and not found resources.

2. in middlewares folder create `token.middleware.ts` and add code

////////////////////////////////////////////////////////////////

`token.middleware.ts`

This file holds two middlware functions `tokenDecode` and `auth`

1. The `auth` middleware is used with the `update-password` route as well as other routes that
   require authentication

2. It recieves the `entire req object` containing the request headers
3. Then it uses the `tokenDecode` function, passing in the request
4. the `tokenDecode` function then extracts & validates the token passed in the `req`, and if all ok returns it
   to the `auth` function
5. Now the auth function checks if the returned token is missing or invalid
6. if ok, the auth function then searches the db using `findById` for the `userId` that matches the one in the token
7. if the `userId` is found the `auth` function then `attaches it to the request object` for further processing

Now add the`updatePassword.ts` controller in `user.controller.ts`

1. once the `auth` middleware function has completed, the `updatePassword` controller can accept a request from the user

2. it extracts the required fields `passsword` & `updatePassword` from the body entered by the user
3. It searches the database with the `findById` method, using the `req.user.id`, which holds the `userId` extracted
   from the authenticated JWT token by the `auth` middleware.
4. if found it validates the password, sets it to the `updatePassword` and saves to the db

////////////////////////////////////////////////////////////////

# User Controllers

1. in `controllers` folder add `user.controller.ts` and add `signup` controller function

Notes on `signup` Controller

The `signup function` is responsible for handling user registration. It first checks if the provided
`username` already exists in the database. If the username is not already taken, a `new user is created` with
the provided `display name`, `username`, and `hashed password`.

# jsonwebtoken logic

2. Create a `utils` folder and in it add `jwtUtils.ts` and add `generateToken` function

3. now in `user.controller.ts` import and call the `generateToken` function and assign to token.
4. use the `responseHandler` to create the response object passing in the token, the user payload
   and id

`responseHandler.created(res, { token, ...(user.toJSON() as UserDocument), id: user.id });`

`...(user.toJSON()...` The line above uses the `spread operator` (...) to include all the properties from the user
object. However, it is important to note that `user` is a Mongoose document, and by default, Mongoose documents have a
lot of internal properties that we might not want to expose to the client.

So, before spreading the user object, it is
first converted to a `plain JavaScript object` using the `toJSON()` method provided by Mongoose. This ensures that
only the relevant data is included in the response.

5. In `user.controller.ts` and the signin controller and code
6. In user.controller.ts add the `getInfo` controller

# Media Controllers

1. in controllers folder add `mediaController.ts` and add controllers

   It's common to face challenges in accurately typing API responses when dealing with external APIs that
   might not provide clear or consistent type information. In cases where the external API doesn't provide proper typings,
   you might need to rely on TypeScript's any type for certain properties. However, you can still improve the type safety of
   your code by narrowing down the types by trying dofferent types for the varaibles in the types file as was done here

////////////////////////////////////////////////////////////////////////////////////////////////

# NOTES on `req.query`, `req.params` & the `getList` function

`req.query` and `req.params` are both properties of the `Request object` in the Express.js framework, but they serve
different purposes and are used to extract different types of data from the incoming request.

`req.query`:
req.query allows you to access the query parameters from the URL. Query parameters are `key-value pairs` that are
typically included in the URL after a `?` character.

For example, in the URL `http://example.com/api/resource?page=2&limit=10`, the query parameters are `page` and `limit`.
To access these parameters, you would use `req.query.page` and `req.query.limit`.

`req.params`
req.params allows you to access `route parameters`. Route parameters are defined in the URL pattern and are placeholders
that are extracted when the URL matches the defined pattern.

For example, if you have a route defined as `/users/:id`, where `:id` is a route parameter, you can access the value of `id`
using `req.params.id`.

...so in the case of the getList controller we use:
url `http://example.com/api/mediaType/mediaCategory?page=2` where `/mediaType/mediaCategory` are the route
parameters accessed by `req.params` and `page` is the query parameter accessed with `req.query`

In summary, `req.query` is used to access query parameters, while `req.params` is used to access route parameters.
Both mechanisms allow you to extract data from the incoming request, but they are used in different contexts and for
different types of data.

////////////////////////////////////////////////////////////////////////////////////////////////

2. in controllers folder add `favorite.controller.ts` and add `addFavorite` and `removeFavorite` controllers

UPTO: `https://youtu.be/j-Sn1b4OlLA?t=4185`
