# npm-lockdown

Put your dependencies on lockdown.

![lockdown](https://github.com/lloyd/npm-lockdown/raw/master/npm-lockdown.png)

## What's this?

NPM Lockdown is a tool that locks your node.js app to
specific versions of dependencies... So that you can:

  1. know that the code you develop against is what you test and deploy
  2. `npm install` and get the same code, every time.
  3. not have to copy all of your dependencies into your project
  4. not have to stand up a private npm repository to solve this problem.

## Who is this for?

Node.JS application developers, but not library authors.  Stuff published
in npm as libraries probably wouldn't be interested.

## Why Care?

Even if you express verbatim versions in your package.json file, you're still
vulnerable to your code breaking at any time.  This can happen if a dependency
of a project you depend on with a specific version *itself* depends on another
packages with a version range.

How can other people accidentally or intentionally break your node.js app?
Well, they might...

  * ... push a new version that no longer supports your preferred version of node.js.
  * ... fix a subtle bug that you actually depend on.
  * ... accidentally introduce a subtle bug.
  * ... be having a bad day.

And, any author at any time can overwrite the package version they have published
so one under-thought `npm publish -f` can mean a subtle bug that steals days
of your week.

## Usage!

`npm-lockdown` is easy to get started with.  It generates a single file that lists
the versions and check-sums of the software you depend on, so any time something
changes out from under you, `npm install` will fail and tell you what package has
changed.

To get started:

  1. npm install the specific dependencies of your app
  2. npm install the version of lockdown you want: `npm install --save lockdown@0.0.1`
  3. copy the bootstrap file into your repository: `cp -L node_modules/.bin/lockdown lockdown`
  4. add a line to your package.json file: `"scripts": { "preinstall": "./lockdown" }`
  5. generate a lockdown.json: `node_modules/.bin/lockdown-relock`
  6. commit: `git add package.json lockdown lockdown.json && git commit -m "be safe"`

Notes:

  * You can put the lockdown script anywhere you like, but the `preinstall` line must be changed
  * You should use the latest stable version of lockdown, find it from the [npm registry](https://npmjs.org/package/lockdown)

## Installing dependencies once locked down

    npm install

## Changing dependencies once locked down

You update your dependencies explicitly, relock, and commit:

    npm install --save foo@1.2.3
    node_modules/.bin/lockdown-relock
    git add lockdown.json package.json
    git commit -m "move to foo v1.2.3"

done!

## Related Tools

**[npm shrinkwrap][]** - NPM itself has a feature called "shrinkwrap" that

> locks down the versions of a package's dependencies so that you can control exactly which
> versions of each dependency will be used when your package is installed.

At present (as of npm v1.1.33), the implementation of shrinkwrap has a couple flaws
which make it unusable for certain applications:

  1. No checksums!  NPM shrinkwrap does not guarantee bit-wise equality of the installed
     dependencies, so if an upstream server or author decides to change the contents of
     version 1.2.3 of `foo`, you'll install something different than you intended without
     knowing.
  2. Does not play nice with `optionalDependencies` - If you "shrinkwrap" your app and you
     have an installed dep that is optional, the dependency is no longer optional.  This might
     not be what you want.

  [npm shrinkwrap]: https://npmjs.org/doc/shrinkwrap.html

*NOTE:* you can combine lockdown with shrinkwrap just fine.  If all you care about is #1 above.

The path forward is to build checksums into shrinkwrap and kick lockdown to the curb, but until
then, lockdown solves some problems.  (@izs is [interested in patches][]).

  [interested in patches]: https://twitter.com/izs/status/234330784931143682

**[npm-seal][]** - Solves the same problem as lockdown in a very different way.  Because seal
is built to be used in concert with shrinkwrap, it suffers from the `optionalDependencies` issue
described above.

  [npm-seal]: https://github.com/zaach/npm-seal
