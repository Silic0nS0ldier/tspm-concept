# TSPM Concept

Concept for a dependency/package manager with static module resolution. Runtime agonistic, dependable lockfile, simple dependency folder strucutre (no deep nesting), no rouge scripts executed immeditely after install.

## Static module resolution

One of the major problems with Node at the moment is how tightly coupled with NPM it is. When you `require` a file a lot happens behind the scenes to resolve the intended file. In particular, the extra reads of various `package.json` files. There are ways to avoid this by performing the work ahead of time (while we have the needed information already in memory).

One of the important considerations for this file-oriented approach is that releases on the same semver major that lack a file would need to be blocked.

Would be nice to have a minimal dependencies, with old duplicates being flagged with a warning (since they can be the source of odd issues).

### `tspm.ts`, `tspm.js`, etc dependency maps

One possible solution would be to store the critical information in a file understood by the consuming language. In the case of TypeScript, a `const enum` could be used to inline values for better performance. This would be comparable to a composer autoload in some ways.

This approach however has the greatest work required, and simply won't work for certain scenarios.

### Rewrite paths

One option that would provide a nearly global solution is to rewrite paths like `tspm/depname/x.x.x` to refer to the correct version.

This would translate to the most processing during install, however the amount of processing required could be limited with some extra processing during submission (identify affected files, replace with an easily searchable value and maybe even some special file-specific files if a template engine technique is used). Language conflicts would be considered with escape sequences.

CSS (and SASS/LESS) imports would benefit from this.

A consideration with this is that different languages support different escape codes, etc. Another thing to consider is that dependency paths might be split across variables (e.g. a constant pointing to the dependencies folder). Customisation options sound like the best bet here.

To ensure consistent performance, replace locations should be identified during publish.

#### Static module paths via flattened dependency tree

An extension on path rewriting, we could enforce a flat dependency tree requirement which would permit path rewriting to happen once (during submission). This could obviously present challenges if we decide to allow multiple versions of dependencies, however there is no reason we couldn't support both scenarios (defaulting to flat for performance).

## Directory structure

Depending on the route we go with version range collisions, there are 4 viable structures.

Support for alternate tools could be implemented by having fixed semver ranges being recommended. Whatever static resolution logic is in place could still be applied to dependencies themselves.

If the solution is for tooling to work around the version tree (not the reverse), could provide a helper to inform what the direct dependency is, or even a source update command.

### Single dependency version

```
. (project root)
+-- tspm
|   +-- dep-name-1
|   |   +-- (dependency files)
|   +-- dep-name-2
|       +-- (dependency files)
+-- tspm.json
+-- tspm-lock.json
```

Advantages;
- Dependency paths are known prior to installation.
- Path resolution solution (as defined in [Static module resolution](#static-module-resolution)) can be determined once.

Disadvantages;
- Exact dependency version won't be visible in stack traces making it harder debugging experience compared to [Multiple dependency versions](#multiple-dependency-versions).
- Unresolvable version conflicts.

### Multiple dependency versions

```
. (project root)
+-- tspm
|   +-- dep-name-1
|   |   +-- 3.2.7
|   |   |   +-- (dependency files)
|   |   +-- 0.2.7
|   |       +-- (dependency files)
|   +-- dep-name-2
|       +-- 1.1.0-rc.1
|           +-- (dependency files)
+-- tspm.json
+-- tspm-lock.json
```

Advantages;
- Dependency version clearly visible in stacktraces.
- Highly effective dependency de-duplication.
- Possibility to implement a 'force-latest' or similiar option that grabs the newest possible versions for each dependency by sacrificing on de-duplication effectiveness.

Disadvantages;
- Tooling for LESS, SASS, etc would need to consider variable paths.

### Multiple dependency versions with direct named

```
. (project root)
+-- tspm
|   +-- dep-name-1
|   |   +-- direct (3.2.7)
|   |   |   +-- (dependency files)
|   |   +-- 0.2.7
|   |       +-- (dependency files)
|   +-- dep-name-2
|       +-- 1.1.0-rc.1
|           +-- (dependency files)
+-- tspm.json
+-- tspm-lock.json
```

Advantages;
- Tooling for LESS, SASS, etc will "just work"™.
- Highly effective dependency de-duplication.
- Possibility to implement a 'force-latest' or similiar option that grabs the newest possible versions for each dependency by sacrificing on de-duplication effectiveness.

Disadvantages;
- Dependency version not known many stacktraces.

### Multiple dependency versions with prefixed direct dependencies

```
. (project root)
+-- tspm
|   +-- dep-name-1
|   |   +-- direct-3.2.7
|   |   |   +-- (dependency files)
|   |   +-- 0.2.7
|   |       +-- (dependency files)
|   +-- dep-name-2
|       +-- 1.1.0-rc.1
|           +-- (dependency files)
+-- tspm.json
+-- tspm-lock.json
```

Advantages;
- Same as [Multiple dependency versions](#multiple-dependency-versions).
- Tooling for LESS, SASS, etc will have an easier time as direct dependencies can be more easily found.

Disadvanages;
- Tooling for LESS, SASS, etc would still need to consider variable paths.

## Plugins

Many of the problems faced in dealing with specific tools could be handled with install plugins, but would we want to allow it? The potential extensibility options are limitless (could host any source, integrate with any system), however they will drastically increase complexity and make it much harder to introduce major changes.

Best to hold off till after a practical prototype of TSPM exists, however the potential here is immeserable particularly if TSPM were to grab the extensions on its own. Could go as far as being THE dependency manager, however there are security concerns.

## Installation performance

The concept of basing on git for everything is fantastic, but certainly comes with notable drawbacks most feel during installation. The speed installation in composer is its achillies heel and a major source of productivity loss. NPM on the other hand is extremely fast despite how many dependencies you end up having, which demonstrates the effectiveness of using infrustructure designed with dependency management in mind.

There are a few ideas around how best to achieve optimal performance;
- Calculate the lockfile first, have it act as the source of truth and let the client decide what it wants.
- Use dedicated infrastructure.
- Use streams to minimise the eliminate TCP trip overhead.
- Cache versions (and vulnerability data) locally.
- Utilise the last updated header and/or a resource age API.
- Integrity hashes to validate local and recieved data.

## Business dependencies

Business often have need to use private dependency sources, ensuring that this is addressed early is essential.

Considerations include;
- Dependencies with the same name.
- Using public and private dependencies together.
- Preventing packages depending on private dependencies being publish to public site.

## Non-package dependencies

Consider composer and npm. Each allows you to require that certain criteria be met to even consider certain dependencies outside of semver compatibility. Generally speaking packages shouldn't be OS tied but will end up being need, runtime ties are likely. Clearly we will need to consider dependencies outside of the service. The question is, how far do we go with this?

## Lockfile

This serves a few purposes;
- Acts as a means to easily audit for vulnerabilities.
- Instructs TSPM on what dependencies to install, if it exists.
- Ensures reproducible builds which help with scenarios like ensuring that production code is what was tested.
- In version control systems changes to contents show what has updated.
- Provide a means to audit the data integrity.

An existing issue that would be nice to address with this implementation is different environments wanting different dependencies. This should not happen, the lockfile should already have these taken into consideration. In the same stroke, dependencies not needed should not be downloaded.

## Mechanism of operation

From a performance perspective, and getting running as quickly as possible perspective, the best bet for this is having a native binary of some kind.

.NET Core can be compiled like this with the runtime bundled up, which makes it a strong candidate. Its also got a reasonably low barrier of entry, simple multithreading, and surprisingly small binary size.

If we wanted to squeeze even more performance, Rust looks like a solid choice. Particularly due to how much harder it is to cause memory leaks vs. C++.
