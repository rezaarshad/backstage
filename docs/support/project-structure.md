---
id: project-structure
title: Backstage Project Structure
description:
  Introduction to files and folders in the Backstage Project repository
---

Backstage is a complex project, and the GitHub repository contains many
different files and folders. This document aims to clarify what purpose of those
files and folders are.

## General purpose files and folders

In the project root, there are a set of files and folders which are not part of
the project as such, and may or may not be familiar to someone looking through
the code.

- [`.changeset/`](https://github.com/spotify/backstage/tree/master/.changeset) -
  This folder contains files outlining which changes occurred in the project
  since the last release. These files are added manually, but managed by
  [changesets](https://github.com/atlassian/changesets) and will be removed at
  every new release. They are essentially building-blocks of a CHANGELOG.

- [`.github/`](https://github.com/spotify/backstage/tree/master/.github) -
  Standard GitHub folder. It contains - amongst other things - our workflow
  definitions and templates. Worth noting is the
  [styles](https://github.com/spotify/backstage/tree/master/.github/styles)
  folder which is used for a markdown spellchecker.

- [`.yarn/`](https://github.com/spotify/backstage/tree/master/.yarn) - Backstage
  ships with it's own `yarn` implementation. This allows us to have better
  control over our `yarn.lock` file and hopefully avoid problems due to yarn
  versioning differences.

- [`docker/`](https://github.com/spotify/backstage/tree/master/docker) - Files
  related to our root Dockerfile. We are planning to refactor this, so expect
  this folder to be moved in the future.

- [`contrib/`](https://github.com/spotify/backstage/tree/master/contrib) -
  Collection of examples or resources provided by the community. We really
  appreciate contributions in here and encourage them being kept up to date.

- [`docs/`](https://github.com/spotify/backstage/tree/master/docs) - This is
  where we keep all of our documentation markdown files. These ends up on
  http://backstage.io/docs. Just keep in mind that changes to
  [this](https://github.com/spotify/backstage/blob/master/microsite/sidebars.json)
  file also needs to be updated.

- [`.editorconfig`](https://github.com/spotify/backstage/tree/master/.editorconfig) -
  A configuration file used by most common code editors.

- [`.imgbotconfig`](https://github.com/spotify/backstage/tree/master/.imgbotconfig) -
  Configuration for a [bot](https://imgbot.net/)

## Monorepo packages

Every folder in both `packages/` and `plugins/` is within our monorepo setup, as
defined in
[`package.json`](https://github.com/spotify/backstage/blob/master/package.json):

```json
 "workspaces": {
    "packages": [
      "packages/*",
      "plugins/*"
    ]
  },
```

Let's look at them individually.

### `packages/`

These are all the packages that is we use within the project.
[Plugins](#plugins) are separated out into their own folder, see further down.

- [`app/`](https://github.com/spotify/backstage/tree/master/packages/app) - This
  is our take on how an App could look like, bringing together a set of packages
  and plugins into a working Backstage App. This is not a published package, and
  the main goals is to provide a demo of what an App could look like, and also
  enabling local development.

- [`backend/`](https://github.com/spotify/backstage/tree/master/packages/backend) -
  Every standalone backstage project will have both an `app` _and_ a `backend`
  package. The `backend` uses plugins to construct a working backend that the
  frontend (`app`) can use.

- [`backend-common/`](https://github.com/spotify/backstage/tree/master/packages/backend-common) -
  There are no "core" packages in the backend. Instead we have `backend-common`
  which contains helper middleware´s and other utils.

- [`catalog-model/`](https://github.com/spotify/backstage/tree/master/packages/catalog-model) -
  You can considers this to be a library for working with the catalog of sorts.
  It contains the definition of an
  [Entity](https://backstage.io/docs/features/software-catalog/references#docsNav),
  as well as validation an other logic related to it. This package can be used
  in both the frontend and the backend.

- [`cli/`](https://github.com/spotify/backstage/tree/master/packages/cli) - One
  of the biggest packages in our project, the `cli` is used to build, serve,
  diff, create-plugins and more. In the early days of this project, we started
  out with calling tools directly - such as `eslint` - through package.json. But
  as it was tricky to have a good development experience around that when we
  change named tooling, we opted for wrapping those in our own cli. That way
  everything looks the same in package.json. Much like
  [react-scripts](https://github.com/facebook/create-react-app/tree/master/packages/react-scripts).

- [`cli-common/`](https://github.com/spotify/backstage/tree/master/packages/cli-common) -
  This package mainly handles path resolving. It is a separate package to reduce
  bugs in [cli](https://github.com/spotify/backstage/tree/master/packages/cli).
  We also want as few dependencies as possible to reduce download time when
  running the cli which is another reason this is a separate package.

* [`config/`](https://github.com/spotify/backstage/tree/master/packages/config) -
  The way we read configuration data. This package can take a bunch of config
  objects and merge them together.
  [app-config.yaml](https://github.com/spotify/backstage/blob/master/app-config.yaml)
  is an example of an config object.

* [`config-loader/`](https://github.com/spotify/backstage/tree/master/packages/config-loader) -
  This package is used to read config objects. It does not know how to merge,
  this only reads files and passes them on to the config. As this part os only
  used by the backend, we chose to separate `config` and `config-loader` into
  two different packages.

- [`core/`](https://github.com/spotify/backstage/tree/master/packages/core) -
  This package contains our visual React components, some of which you can find
  [here](https://backstage.io/storybook/?path=/story/plugins-examples--plugin-with-data).
  Apart from that it re-exports everything from [`core-api`] so that users only
  need to rely on one package.

* [`core-api/`](https://github.com/spotify/backstage/tree/master/packages/core-api) -
  This package contains apis and definitions of such. It is it's own package
  because we needed to split our `test-utils` package. It's an implementation
  detail that we try to hide from our users, and no-one should have to depend on
  it directly.

* [`test-utils/`](https://github.com/spotify/backstage/tree/master/packages/test-utils) -
  This package contains specific testing facilities used when testing
  `core-api`.

* [`test-utils-core/`](https://github.com/spotify/backstage/tree/master/packages/test-utils-core) -
  This package contains more general purpose testing facilities for testing an
  Backstage App.

* [`create-app/`](https://github.com/spotify/backstage/tree/master/packages/create-app) -
  An CLI to specifically scaffold a new Backstage App. It does so by using a
  [template](https://github.com/spotify/backstage/tree/master/packages/create-app/templates/default-app).

- [`dev-utils/`](https://github.com/spotify/backstage/tree/master/packages/dev-utils) -
  Helps you setup a plugin for isolated development so that it can be served
  separately.

* [`docgen/`](https://github.com/spotify/backstage/tree/master/packages/docgen) -
  Uses the
  [Typescript Compiler API](https://github.com/Microsoft/TypeScript/wiki/Using-the-Compiler-API)
  to read out definitions and generate documentation for it.

* [`e2e-test/`](https://github.com/spotify/backstage/tree/master/packages/e2e-test) -
  Another CLI that can be run to try out what would happen if you built all the
  packages, publish them, created a new app, and the run it. CI uses this for
  e2e-tests.

* [`storybook/`](https://github.com/spotify/backstage/tree/master/packages/storybook) -
  This folder contains only the storybook config. Stories are within the core
  package. The Backstage Storybook is found
  [here](https://backstage.io/storybook)

* [`techdocs-cli/`](https://github.com/spotify/backstage/tree/master/packages/techdocs-cli) -
  Used for verifying TechDocs locally.

* [`techdocs-container/`](https://github.com/spotify/backstage/tree/master/packages/techdocs-container) -
  Used by the `techdocs-cli`

* [`test-utils-core/`](https://github.com/spotify/backstage/tree/master/packages/test-utils-core)

* [`test-utils/`](https://github.com/spotify/backstage/tree/master/packages/test-utils)

* [`theme/`](https://github.com/spotify/backstage/tree/master/packages/theme) -
  Holds the Backstage Theme.

### `plugins/`

Most of the functionality of an Backstage App comes from plugins. Even core
features can be plugins, take the
[catalog](https://github.com/spotify/backstage/tree/master/plugins/catalog) as
an example.

We can categorize plugins into three different types; **Frontend**, **Backend**
and **GraphQL**. We differentiate these types of plugins when we name them, with
a dash-suffix. `-backend` means it’s a backend plugin and so on.

One reason for splitting a plugin is because of to it's dependencies. Another
reason is for clear separation of concerns.

Take a look at our [Plugin Gallery](https://backstage.io/plugins) or browse
through the
[`plugins/`](https://github.com/spotify/backstage/tree/master/plugins) folder.

## Packages outside of the monorepo

For convenience we include packages in our project that is not part of our
monorepo setup.

- [`microsite/`](https://github.com/spotify/backstage/blob/master/microsite) -
  This folder contains the source code for backstage.io. It is built with
  [Docusaurus](https://docusaurus.io/). This folder is not part of the monorepo
  due to dependency reasons. Look at the
  [README](https://github.com/spotify/backstage/blob/master/microsite/README.md)
  for instructions on how to run it locally.

## Root files specifically used by the `app`

These files are kept in the root of the project mostly by historical reasons.
Some of these files may be subject to be moved out of the root sometime in the
future.

- [`.npmrc`](https://github.com/spotify/backstage/tree/master/.npmrc) - It's
  common for companies to have their own npm registry, this files makes sure
  that this folder use the public registry.

- [`.vale.ini`](https://github.com/spotify/backstage/tree/master/.vale.ini) -
  [Spell checker](https://github.com/errata-ai/vale) for markdown files

- [`.yarnrc`](https://github.com/spotify/backstage/tree/master/.yarnrc) -
  Enforces "our" version of yarn.

- [`app-config.yaml`](https://github.com/spotify/backstage/tree/master/app-config.yaml) -
  Configuration for the app, both frontend and backend

- [`app-config.development.yaml`](https://github.com/spotify/backstage/tree/master/app-config.development.yaml) -
  Used for overriding configuration when developing locally.

- [`catalog-info.yaml`](https://github.com/spotify/backstage/tree/master/catalog-info.yaml) -
  Description of Backstage in the Backstage Entity format.

- [`lerna.json`](https://github.com/spotify/backstage/tree/master/lerna.json) -
  [lerna](https://github.com/lerna/lerna) monorepo config. We are using
  `yarn workspaces`, so this will only be used for executing scripts.
