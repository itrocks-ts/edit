[![npm version](https://img.shields.io/npm/v/@itrocks/edit?logo=npm)](https://www.npmjs.org/package/@itrocks/edit)
[![npm downloads](https://img.shields.io/npm/dm/@itrocks/edit)](https://www.npmjs.org/package/@itrocks/edit)
[![GitHub](https://img.shields.io/github/last-commit/itrocks-ts/edit?color=2dba4e&label=commit&logo=github)](https://github.com/itrocks-ts/edit)
[![issues](https://img.shields.io/github/issues/itrocks-ts/edit)](https://github.com/itrocks-ts/edit/issues)
[![discord](https://img.shields.io/discord/1314141024020467782?color=7289da&label=discord&logo=discord&logoColor=white)](https://25.re/ditr)

# edit

Generic action-based object edit form in HTML and JSON.

*This documentation was written by an artificial intelligence and may contain errors or approximations.
It has not yet been fully reviewed by a human. If anything seems unclear or incomplete,
please feel free to contact the author of this package.*

## Installation

```bash
npm i @itrocks/edit
```

## Usage

`@itrocks/edit` provides a generic `Edit` action that renders a form to
view and update an existing domain object in HTML or JSON.

You typically:

1. Define a domain class (for example `User`).
2. Attach an `Edit<User>` action to this class.
3. Use the `html` method for the classic HTML form and the `json` method
   for API‑driven or SPA front‑ends.

### Minimal example

```ts
import { Edit } from '@itrocks/edit'
import type { Request } from '@itrocks/action-request'

class User {
	name  = ''
	email = ''
}

// Action responsible for editing existing User instances
const editUser = new Edit<User>()

// HTML edit form endpoint
async function editUserHtml (request: Request<User>) {
	return editUser.html(request)
}

// JSON endpoint (for SPA / XHR based UI)
async function editUserJson (request: Request<User>) {
	return editUser.json(request)
}
```

The `Request<User>` is generally created by
[@itrocks/action-request](https://github.com/itrocks-ts/action-request)
from an incoming HTTP request and contains all the information needed to
load the object and apply edits.

## API

### `class Edit<T extends object = object> extends Action<T>`

`Edit` is a subclass of
[@itrocks/action](https://github.com/itrocks-ts/action)'s `Action`.
It focuses on the "edit existing object" use case and is reused by
other higher‑level actions such as
[`New`](https://github.com/itrocks-ts/new#class-newt-extends-object--object-extends-editt).

From the type declarations you can see that it works with a
`Request<T>` and returns responses from
[@itrocks/core-responses](https://github.com/itrocks-ts/core-responses).

#### Type parameter

- `T` – The domain object type being edited (for example `User`).

#### Methods

##### `html(request: Request<T>): Promise<HtmlResponse>`

Builds an HTML form for editing an existing instance of `T`.

Typical usage in a route handler:

```ts
fastify.get('/users/:id/edit', async (req, reply) => {
	const request = toActionRequest<User>(req)
	const response = await editUser.html(request)
	reply
		.status(response.status)
		.headers(response.headers)
		.type('text/html')
		.send(response.body)
})
```

Internally, `Edit` relies on the it.rocks action stack to:

- locate and load the object to edit (for example by ID),
- bind incoming data to this object,
- prepare the view model and HTML template for the form.

##### `json(request: Request<T>): Promise<JsonResponse>`

Builds a JSON representation of the same edit form/state. This is
useful for SPA or mobile clients that render their own UI while keeping
validation and persistence logic on the server.

```ts
fastify.get('/api/users/:id/edit', async (req, reply) => {
	const request = toActionRequest<User>(req)
	const response = await editUser.json(request)
	reply
		.status(response.status)
		.headers(response.headers)
		.send(response.body)
})
```

## Typical use cases

- Provide a conventional "edit" screen for any business entity
  (users, products, orders, etc.).
- Expose a JSON API for editing existing objects from rich front‑end
  applications while reusing the same server‑side action.
- Factor common edit behaviour once in `Edit<T>` and reuse it in
  specialised actions (for example `New<T>`).
