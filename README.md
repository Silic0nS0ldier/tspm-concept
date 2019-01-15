## Static module resolution

- Use typescript inlined constants like `enum`s (I think) to resolve dependencies.
- Constant names need to be hashed out to retain usefullness of type completion in own source.
- This avoids IDEs having to put an excess of work into module resolution for type completion.
- Comparable to a composer autoload but more efficient as unused source isn't parsed.
- Permits compatibility with any TS (and JS after compilation) runtime.
- What should the associated value be? Should each dependency have a few constants to describe it? e.g. root directory and a 'main' file?
- If a main file is not a concept supported, then we might need to enforce a file state freeze for non-major releases.
- Do we want to enforce one module only in the first release? Keeping the dependency tree to a minimum is desirable.
- Maybe consider it a warning (since it could serve as a source for errors), however TS's type system could help with preventing errors provided library authors have defined types well.

## Basic directory structure

`tspm/depname/x.x.x/`

- This addresses an issue node has with depduplicating certain dependencies, plus dependency versions can be seen in stack traces.
- Mentioned somewhere else on list, should direct dependencies be located in a more consistent path?

## Non-TS resources

- Need to figure out a way for this to fit into everything, because it likely will end up happening as it has with composer, nuget, npm, etc.
- Dependency paths can differ unlike with npm, which may present an issue for tools like SASS. How can we improve this?
- Maybe main dependencies can live above the version range? This or something like it could be achieveable.

## Installation performance

- Don't even touch github in the first release, nor git. This is the achillies heel of composer in terms of installation speed.
