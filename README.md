# CRUD-checklist

* * *

### Database Setup

1. npm install
2. Add these files to gitignore:

    node_modules/
    .DS_Store
    .env
    yarn.lock
    package-lock.json

3. createdb 'messasge_dev'
& 'message_test'

4. Set up knexfile.js with the following:
```javascript
module.exports = {
development: {
client: 'pg',
connection: 'postgres://localhost/dbname-dev'
},
test: {
  client: 'pg',
  connection: 'postgres://localhost/dbname-test'
},
production: {
client: 'pg',
connection: process.env.DATABASE_URL
}
}
```
5. Be sure to add in the name for your database in place of 'dbname-dev' and 'dbname-test'
6. Run nodemon (should say 'listening on port')
7. Should be able to run NPM test and see tests, if so, git add commit

### Migrations

1. knex migrate:make 'messages'
2. fill out migration to match schema

```javascript
exports.up = (knex, Promise) => {
  return knex.schema.createTable('messages', (table) => {
    //Other stuff goes here, but dont forget these things
    table.dateTime('created_at').notNullable().defaultTo(knex.raw('now()'))
    table.dateTime('updated_at').notNullable().defaultTo(knex.raw('now()'))
  })
};

exports.down = (knex, Promise) => knex.schema.dropTableIfExists('messages')
```
3. knex migrate:latest
4. Check database with 'psql message_dev'
5. If tests pass, git add and commit

###Seeds

1. knex seed:make 001_messages
2. Build out seed with with data:
```javascript
exports.seed = function(knex, Promise) {
  // Deletes ALL existing entries
  return knex('messages').del()
    .then(function() {
      // Inserts seed entries
      return knex('messages').insert([{
          //Other stuff goes here, but dont forget these things!!
          created_at: new Date('2016-06-26 14:26:16 UTC'),
          updated_at: new Date('2016-06-26 14:26:16 UTC')

        },
      ])
      //ESPECIALLY this thing!!
      .then(() => {
           return knex.raw(`SELECT setval('messages_id_seq', (SELECT MAX(id) FROM messages));`)
         })
    });
};
```
3. knex seed:run
4. check database with 'psql message_dev'
5. if tests pass, git add commit

###Simple routes and server

1. Add these into server.js

```javascript
'use strict'

const express = require('express')
const app = express()
const bodyParser = require('body-parser')
const cookieParser = require('cookie-parser')
const path = require('path')

app.use(bodyParser.json())
app.use(cookieParser())

const messages = require('./routes/messages')

app.use(express.static(path.join('.')))


```

2. Be sure bodyParser comes BEFORE the const messages variable
3. Create index.html file

4. Build Out simple routes like so:
```javascript
const express = require('express')
const router = express.Router()
const knex = require('../knex')
// READ ALL records for this table
router.get('/', (req, res, next) => {
  res.send('ALL RECORDS')
})
// READ ONE record for this table
router.get('/:id', (req, res, next) => {
  res.send('ONE RECORD')
})
// CREATE ONE record for this table
router.post('/', (req, res, next) => {
  res.send('CREATED RECORD')
})
// UPDATE ONE record for this table
router.put('/:id', (req, res, next) => {
  res.send('UPDATED RECORD')
})
// DELETE ONE record for this table
router.delete('/:id', (req, res, next) => {
  res.send('DELETED RECORD')
})
module.exports = router
```
5. Test routes in terminal, if all work git add commit

### Routes with Knex

1. Time to make the routes work with Knex

```javascript
// READ ALL records for this table
router.get('/', (req, res, next) => {
  knex('messages')
  .select('id', 'name', 'message')
  .then((rows) => {
    res.json(rows)
  })
  .catch((err) => {
      next(err)
    })
})

// READ ONE record for this table
router.get('/:id', (req, res, next) => {
  knex('messages')
  .select('id', 'name', 'message')
  .where('id', req.params.id)
  .then((rows) => {
    res.json(rows[0])
  })
  .catch((err) => {
      next(err)
    })
})

// CREATE ONE record for this table
router.post('/', (req, res, next) => {
  knex('messages')
  .insert({
    name: req.body.name,
    message: req.body.message
  })
  .returning(['name','message'])
  .then((data) => {
    res.json(data[0])
  })
  .catch((err) => {
      next(err)
    })

})

// UPDATE ONE record for this table
router.patch('/:id', (req, res, next) => {
  knex('messages')
  .update({
    message: req.body.message
  })
  .where('id', req.params.id)
  .returning(['id','name','message'])
  .then((data) => {
    res.json(data[0])
  })
  .catch((err) => {
    next(err)
  })
})
// DELETE ONE record for this table
router.delete('/:id', (req, res, next) => {
  knex('messages')
  .del()
  .where('id', req.params.id)
  .returning(['name', 'id', 'message'])
  .then((data) => {
    res.json(data[0])
  })
  .catch((err) => {
    next(err)
  })
})
```
1. Be sure to git add and commit after each route test passes. If they do; bing, bang BOOM!!
