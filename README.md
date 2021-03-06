# feathers-authentication-hooks

[![Greenkeeper badge](https://badges.greenkeeper.io/feathersjs-ecosystem/feathers-authentication-hooks.svg)](https://greenkeeper.io/)

[![Build Status](https://travis-ci.org/feathersjs-ecosystem/feathers-authentication-hooks.png?branch=master)](https://travis-ci.org/feathersjs-ecosystem/feathers-authentication-hooks)
[![Dependency Status](https://img.shields.io/david/feathersjs-ecosystem/feathers-authentication-hooks.svg?style=flat-square)](https://david-dm.org/feathersjs-ecosystem/feathers-authentication-hooks)
[![Download Status](https://img.shields.io/npm/dm/feathers-authentication-hooks.svg?style=flat-square)](https://www.npmjs.com/package/feathers-authentication-hooks)

> Useful hooks for authentication and authorization

```
$ npm install feathers-authentication-hooks --save
```

`feathers-authentication-hooks` is a package containing some useful hooks for authentication and authorization. For more information about hooks, refer to the [chapter on hooks](../hooks.md). 

> **Note:** Restricting authentication hooks will only run when `params.provider` is set (as in when the method is accessed externally through a transport like [REST](../rest.md) or [Socketio](../socketio.md)).


## queryWithCurrentUser

The `queryWithCurrentUser` **before** hook will automatically add the user's `id` as a parameter in the query. This is useful when you want to only return data, for example "messages", that were sent by the current user.

```js
const hooks = require('feathers-authentication-hooks');

app.service('messages').before({
  find: [
    hooks.queryWithCurrentUser({ idField: 'id', as: 'sentBy' })
  ]
});
```

#### Options

- `idField` (default: '_id') [optional] - The id field on your user object.
- `as` (default: 'userId') [optional] - The id field for a user on the resource you are requesting.
- `expandPaths` (default: true) [optional] - Prevent path expansion when the DB Adapter doesn't support it. Ex: With this option set to false, a value like 'foo.userId' won't be expanded to a nested `{ "foo": { "userId": 51 } }` but instead be set as `{ "foo.userId": 51 }`.

When using this hook with the default options the `User._id` will be copied into `context.params.query.userId`.


## restrictToOwner

`restrictToOwner` is meant to be used as a **before** hook. It only allows the user to retrieve or modify resources that are owned by them. It will return a _Forbidden_ error without the proper permissions. It can be used on *any* method.

For `find` method calls and `patch`, `update` and `remove` of many (with `id` set to `null`), the [queryWithCurrentUser](#queryWithCurrentUser) hook will be called to limit the query to the current user. For all other cases it will retrieve the record and verify the owner before continuing.

```js
const hooks = require('feathers-authentication-hooks');

app.service('messages').before({
  remove: [
    hooks.restrictToOwner({ idField: 'id', ownerField: 'sentBy' })
  ]
});
```

#### Options

- `idField` (default: '_id') [optional] - The id field on your user object.
- `ownerField` (default: 'userId') [optional] - The id field for a user on your resource.
- `expandPaths` (default: true) [optional] - Prevent path expansion when the DB Adapter doesn't support it. Also see [queryWithCurrentUser](#queryWithCurrentUser).


## associateCurrentUser

The `associateCurrentUser` **before** hook is similar to the `queryWithCurrentUser`, but works on the incoming **data** instead of the **query** params. It's useful for automatically adding the userId to any resource being created. It can be used on `create`, `update`, or `patch` methods.

```js
const hooks = require('feathers-authentication-hooks');

app.service('messages').before({
  create: [
    hooks.associateCurrentUser({ idField: 'id', as: 'sentBy' })
  ]
});
```

#### Options

- `idField` (default: '_id') [optional] - The id field on your user object.
- `as` (default: 'userId') [optional] - The id field for a user that you want to set on your resource.

## License

Copyright (c) 2018

Licensed under the [MIT license](LICENSE).
