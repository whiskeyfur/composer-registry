# composer-registry

The private Composer registry for [jcw.wyrmfyre.com](https://jcw.wyrmfyre.com). A
[Satis](https://github.com/composer/satis) build, served by GitHub Pages at
**https://whiskeyfur.github.io/composer-registry**.

The site's core repository declares it as a Composer repository:

```json
"repositories": [
    { "type": "composer", "url": "https://whiskeyfur.github.io/composer-registry" }
]
```

## This repository is public. The packages are not.

What is published here is **metadata only** &mdash; package names, versions, dependencies,
and the git URL each package lives at. The code itself stays in private repositories and
is fetched over SSH by whoever runs `composer install`. Someone without access to those
repositories can read the index and download nothing.

**`satis.json` has no `archive` block, and must never get one.** With `archive`, Satis
builds zip files of each package into the output directory, and GitHub Pages would then
serve the private source code to the world. There is a CI check that fails if an archive
ever appears under `docs/`, because this is the one mistake here that cannot be undone:
once Pages has served a zip, it has been published.

Note that Satis still writes a `dist` URL pointing at GitHub's zipball API for each
version. That is not a leak &mdash; downloading it from a private repository needs
credentials &mdash; but it does mean this index names the repositories and their commit
hashes. If that ever becomes unwanted, the registry has to move somewhere authenticated.

## Rebuilding

From a checkout of the site, which has the SSH key and the `gh` login:

```sh
bin/satis-rebuild
```

That pulls this repository, rebuilds `docs/`, refuses to continue if any archive appeared,
commits and pushes. It is also reachable as `php artisan jcw:registry:rebuild`, and runs
nightly from the scheduler.

Rebuild after tagging a package &mdash; a version that is not in this index cannot be
installed, which is deliberate: it makes it impossible to put code live that exists in no
repository.

## Adding a package

Add its repository to `satis.json` and rebuild:

```json
{ "type": "vcs", "url": "git@github.com:whiskeyfur/<name>.git" }
```

A repository with no `composer.json` is simply skipped, so an app repository can be listed
before it has been turned into a package.
