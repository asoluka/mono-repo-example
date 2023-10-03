### Mono-repo

A mono repo means a single repository which contains more than one packages. The term package actually refers to a project.

It is called a mono repo because all these different packages/projects will exist in the same repository.

Normally, we are used to creating different git repositories for different projects but with the concept of mono-repos, we can create just one repository and host many projects in it.

### External package management.

When using mono-repos, we technically have just one node-modules folder. This is where all packages used by all the individual projects in the mono-repo will be stored.

The projects in the repository might have a node-modules folder inside them but it won't actually contain the external packages/modules they use. Instead, the packages they use will be stored in the node-modules directory shared by all the projects, while each individual project/package will have a node-modules folder where links to the packages they use will be stored.

It's like variables and scoping in Javascript. When we refer to a package (module) in any of the individual projects, nodejs looks for that package in the immediate node-modules file and if it isn't stored there, it follows the link to the parent node-modules folder (which is shared by all projects in the repository) and then if it's found there, it is used.

### Code sharing

When using a monorepo, code from one package (project) can be used in the other project (package).

You'd simply import it as you would import any package.

For example;

> import code from "/other-package"

Just like that, you'd be able to share functionality between projects. This is so cool.

### Yarn workspaces

If you don't already have yarn installed, run the code below;

> npm install yard -g

Having installed yarn, let's add a package.json file to the root of our mono-repository.

We are going to add configurations to it. The first configuration we'd add is `private`

```
{
  "private": true,
}
```

It is important that our mono-repository itself is private. This doesn't stop us from making the individual projects (packages) within it public.

Next, we'll add a name for the repository;

```
{
  "private": true,
  "name": "@mono-repo",
}
```

We'll go ahead to specify the workspaces in this mono-repo. These are the individual packages present in the repository. The workspaces key could accept an array of paths to the packages or an object with a `packages` key whose value could be the path to the packages (projects) in the mono-repo.

```
{
  "private": true,
  "name": "@mono-repo",
  "workspaces": [
    "guestclient",
    "hostclient"
  ]
}
```

Now, we can do into the packages and initialize them with yarn. This will allow us create individual package.json files for them. Which means within those packages, we can install other packages from npm or yarn package manager as we would normally do.

> cd guestclient
> yarn init -y
> cd ../hostclient
> yarn init -y
> cd ..

Next, let's create a dummy module in one of our packages in the mono-repo, and then try to use it in the other package.

We will create an index.js file in `hostclient``. The content of the file will be an exported function.

> cd hostclient
> touch index.js

The content of index.js should be;

```
module.exports = () => {
  console.log("Module content from hostclient")
}
```

Next, we'll try to use the code from our `hostclient` package in our `guestclient` package.

In the `guestclient` directory, we'll add a dependency (in the package.json file) to the project like so;

```
{
  "dependencies: {
    "hostclient": "1.0.0"
  }
}
```

Haven done this, we need to run `yarn install`

This will host our packages in a `node_modules` folder at the root of the repository. Now, they could be used within any project in the mono-repo.

Next, let's create an `index.js` file, require that dependency in it and then attempt to use it.

```
const hostCode = require("hostclient");

console.log(hostCode());
```

Still in the `guestclient` directory, If we go ahead to run our ./index.js file with node, we'd see that it actually works.

As we mentioned before, we can configure our workspaces to accept an object with a packages key which accepts an array of paths to packages used. Let's adopt that for our setup but this time, we are going to use a wildcard to specify that we want to include all sub-folders of the packages folder as packages in our mono-repo

```
{
  "private": true,
  "name": "@mono-repo",
  "workspaces": {
    packages: ["./packages/*"]
  }
}
```

Now, we have moved both the hostclient and guestclient packages into the packages folder. We need to run yarn again to re-install and update our node-modules folder.

From now on, if we install a dependency in any of our packages (projects) within the mono-repo, that dependency will be added to our global node_moduels file while the package.json file of the package where the external dependency was installed will be updated to reflect that that package depends on the installed dependency.

Next, let's learn how to effectively manage mono-repos using a tool called `lerna`

### Managing mono-repos with lerna
