# Hasura NodeJS Express TypeScript Boilerplate
This is the Hasura bolerplate that works with nodejs-express-typescript-hasura-starter

# Features

|   Name         | Description |
| -------------- | ----------- | 
| Authentication | Creating new users and verifying their identity |
| Authorisation  | Role based access managment on each route and resource |
| Mailing client | Ready to use mailing client that uses fully customisable embedded                            javascript templates |
| Request validation | Setting up request rules for each route that will be loaded                                  automatically from file that follows naming convention |
| Authentication with socials | TO DO |
| File upload | TO DO |
| Realtime, bi-directional communication | This is accomplished using socket.io library |

# Folder Structue 
1. **~** - root
     - **/ custom** -- custom storage rules ( to do ) and email templates
       - Second nested list item
     - **/ src**
       - **/ controllers** -- Class based controllers whos methods get called from router 
         - all controllers get loaded from `index.ts` one
         - naming convention - `/{route name directory-lowercase}/{ Route Name }Controller` - camel case 
         - ex. **/auth/AuthController**
       - **/ database** - Database driver configuration file, migrations and seeds
         - enviroment variables required - **DATABASE_CLIENT**, **DATABASE_URL** - connection string
         - commands
           - **npm run knex migrate:latest** - apply migrations
           - **npm run knex migrate:rollback** - revert latest migration
           - **npm run knex migrate:make migration_name** - create new migration
           - **npm run knex seed:run** - run all seeds from /seeds directory
           - **npm run knex seed:run --specific=seed-filename.ts** - Runs a specific seed file
          - knex cheatsheet - https://devhints.io/knex
       - **/ middleware** - gloabl and route level middleware
         - `auth.ts` - global authentication middleware that validates user and his access permission
         - `requestValidator.ts` - global middleware that automatically loads `{route name}.router.rules.ts` file and validates incoming request based on its rules
       - **/ models** - For each database table there should be corresponding **ts** file.
       - **/ plugins** - Directory with extensions such as socket.io, redis... - socket.io - inital setup is created in index.ts and socket events and listeners             in events_listeners.ts and imported to the index. Inital setup is imported in root dir index.js and instantiated.
       - **/ routes** - API routes defintions
         - naming convention - `/{route name directory}/` - lowercase
           - `{route name}.router.ts` - place where you define all routes for router ( ex. auth.router.ts ) - lowercase
           - `{route name}.router.rules.ts` - rules for the router routes (ex. auth.router.rules.ts )
             - exports object whos keys are route definitions which have value of validation rules as array
             - object key naming convention - `{ method }_{ route name }` (ex. auth_register):[] 
           -  all `router.ts` files should be imported into `routes/index.ts` file
       - **/ services** - Microservices for each db model of separate feature
         - naming convention - `/{ service name directory }` - lowercase  (ex. auth)
           - `{service name}Service.ts` (ex. AuthService)
           - `I{service name}Service.ts` - interface that service above implements (ex. IAuthService)
       - **/ shared** - plugins, types, db instance, helper function - all things that need to be globally available 
         - `types` - directory with all global types 
         - `db.ts` - database instance using which we make all queries
         - `email.ts` - mailing client 
         - `jwt.ts` - JWT helper functions
         - `lodash.ts` - export lodash instance
         - `serviceResponseHandler.ts`  - handler for response call in controller which will return api response from its call
       - **/ tests** - to be written

# How to run

### Run the Hasura, Postgres and Mailhog instances
In `hasura-starter`  directory
```
git clone https://github.com/borisa99/hasura-starter.git
cd hasura-starter
cp docker-compose.yml.example docker-compose.yml
docker-compose up --build -d
```
In `nodejs-express-typescript-hasura-starter`  directory

### Run NodeJS app
```
git clone https://github.com/borisa99/nodejs-express-typescript-hasura-starter. git
cp .env.example to .env 
npm install
npm run knex migrate:latest
npm run knex seed:run
npm run dev - if you want to run the app in development mode 
npm run start - run the app in production, requires additional configuration in .env
```

### Other commands

```
npm run build - typescript complier
npm run lint - eslint
npm run format - prettier format
```

# How to extend

### Create new service, controller and router
1. Create `{service name}` directory in services
2. Create `I{service name}Service.ts` in it and define methods
3. Create `{service name}Service.ts` that extends interface and implement business logic
4. Create `{controller name}` directory in controllers
5. Create `{controller name}Controller.ts` in that directory and define methods that call service methods wrapped in **serviceResponseHandler**
6. Import that controller in `index.ts` file in controllers and export it from there
4. Create `{router name}` directory in routers
5. Create `{route name}.router.ts` in that directory and define routes using the **router_wrapper**
6. Create `{route name}.router.rules.ts` and export object from it with route names and methods as keys and arrays of request rules as values
7. import `{route.name}.router.ts` in `index.ts` in routes and use it in app

All global plugins and helper functions are created in `shared` directory in separate directories and files

# API reference 

______________________

## Authentication
### /auth/invite
Invite new user to the system 
Method : `POST`
**parameters**
these params should be inside input object beacuse of Hasura Actions 
- email: string (required)
- role: string (required) 

### /auth/register
Register new user 
Method : `POST`
**parameters**
these params should be inside user object inside input object beacuse of Hasura Actions 
- ticket: string (required)
- first_name: string (required) 
- last_name: string (required) 
- email: string (required) 
- avatar_url: string (required) 
- password: string (required) 

### /auth/login
Login existing user 
Method : `POST`
**parameters**
these params should be inside input object beacuse of Hasura Actions 
- email: string (required)
- password: string (required) 

### /auth/refresh
Refresh JWT with refresh_token 
Method : `POST`
**parameters**
these params should be inside input object beacuse of Hasura Actions 
- refresh_token: string (required)

### /user/me
Get current user
Method : `POST`

### /auth/request_reset_password
Request reset password for user
Method : `POST`
**parameters**
these params should be inside input object beacuse of Hasura Actions 
- email: string (required)

### /auth/reset_password
Reset password using the ticket recieved in email       
Method : `POST`
**parameters**
these params should be inside input object beacuse of Hasura Actions 
- ticket: string (required)
- password: string (required)


### /file/upload
Upload file to cloud storage       
Method : `POST`
**parameters**
this api should be called directly, not over the Hasura action
- base64String: string (required)



*Non public routes require 'Bearer' + JWT to be passed in headers in Authorization key
# Database query - in progress

This application is using [knex](https://knexjs.org/#Builder-knex) as driver for database. You can use knex directly
from `shared/db.ts` file or you can create model and use it as instance.

### DB Model based - in progress

When using it over model. Class in question needs to extend base class, lest say we have class named user class will
look like:

```
import Model from '@shared/Model'
class User extends Model{
    constructor() {
        super()
    }
}
```

From example above we need to follow rules:

| Table name     | Writing query                 |
| -------------- | ----------------------------- |
| users (plural) | new User().db.<knex_function> |

in case of multiple words in class name example:

```
class UsersInRole extends Model {
    constructor() {
        super()
    }
}
```

| Table name             | Writing query                         |
| ---------------------- | ------------------------------------- |
| user_in_roles (plural) | new UsersInRoles().db.<knex_function> |

# Tests

This project for testing is using [Jest](https://jestjs.io/docs/getting-started)
for examples about jest and node express
use [this link](https://www.albertgao.xyz/2017/05/24/how-to-test-expressjs-with-jest-and-supertest/) <br />
**Tests should not cover 100% of code**, just API calls or functionalities like
db connection.<br />
Running tests: `npm t`
