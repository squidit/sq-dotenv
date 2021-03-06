# sq-dotenv

Identical to [`dotenv`](https://github.com/motdotla/dotenv) and [`dotenv-safe`](https://github.com/motdotla/dotenv-safe), but ensures that all necessary environment variables are defined after reading from `.env` and `./kub/kub_deployment.yml.template`.
These needed variables are read from `.env.example`, which should be commited along with your project.

# Installation

```
npm install --save github:squidit/sq-dotenv
```

# Example

```dosini
# .env.example, committed to repo
SECRET=
TOKEN=
KEY=
```

```dosini
# .env, private
SECRET=topsecret
TOKEN=
```

```js
// index.js
require('sq-dotenv').config();
```

Since the provided `.env` or `./kub/kub_deployment.yml.template` file does not contain all the variables defined in
`.env.example`, an exception is thrown:

```
MissingEnvVarsError: The following variables were defined in .env.example but are not present in the environment:
  TOKEN, KEY
Make sure to add them to .env or directly to the environment.

If you expect any of these variables to be empty, you can use the allowEmptyValues option:
require('sq-dotenv').config({
  allowEmptyValues: true
});

If you expect only one variable to be empty, only set in `.env.example` file the variable value to `empty` as:

EMPTY=empty
```

Not all the variables have to be defined in `.env`, they can be supplied externally.
For example, the following would work:

```
$ TOKEN=abc KEY=xyz node index.js
```

# Usage

Requiring and loading is identical:

```js
require('sq-dotenv').config();
```

This will load environment variables from `.env` as usual, but will also read any variables defined in `.env.example`.
If any variables are already defined in the environment before reading from `.env`, they will not be overwritten.
If any variables are missing from the environment, a [`MissingEnvVarsError`](MissingEnvVarsError.js) will be thrown, which lists the missing variables.
Otherwise, returns an object with the following format:

```js
{
  parsed: { SECRET: 'topsecret', TOKEN: '' },          // parsed representation of .env
  required: { SECRET: 'topsecret', TOKEN: 'external' } /* key-value pairs required by .env.example
                                                          and defined by environment */
}
```

If all the required variables were successfully read but an error was thrown when trying to read the `.env` file, the error will be included in the result object under the `error` key.

`sq-dotenv` compares the actual environment after loading `.env` (if any) with the example file, so it will work correctly if environment variables are missing in `.env` but provided through other means such as a shell script.

## Continuous integration (CI)

It can be useful to depend on a different set of example variables when running in a CI environment.
This can be done by checking if the `CI` environment variable is defined, which is supported by virtually all CI solutions.
For example:

```js
require('sq-dotenv').config({
  example: process.env.CI ? '.env.ci.example' : '.env.example'
});
```

# Options

[Same options and methods supported by `dotenv`](https://github.com/motdotla/dotenv#options).

```js
require('sq-dotenv').config({
    allowEmptyValues: true,
    example: './.my-env-example-filename'
});
```

## `hasKub`

Set `true` if the project needs to validate `./kub/kub_deployment.yml.template` file variables.

## `allowEmptyValues`

If a variable is defined in the example file and has an empty value in the environment, enabling this option will not throw an error after loading.
Defaults to `false`.

## `example`

Path to example environment file.
Defaults to `.env.example`.
