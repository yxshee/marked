<!-- claude-projects-attribution: {"github_login":"yxshee"} -->
_Requested by **Sir** · [project thread](https://claude.ai/code/project/chan_01FGhb5Fba7ZosnqFBVKeY7t?thread=cmsg_01FGhb5Fba7ZosnqFBVKeY7tB5HtdF6YvT4RNDg9dYg6Fi)_

## Pull Request Checklist

- [x] Have you added new tests to prevent regressions?
- [ ] If a documentation update is necessary, have you opened a PR to [the documentation repository](https://github.com/sequelize/website/)? Not needed, this makes the documented `sql` usage work.
- [x] Did you update the typescript typings accordingly (if applicable)? N/A
- [x] Does the description below contain a link to an existing issue (Closes #[issue]) or a description of the issue you are solving?
- [x] Does the name of your PR follow [our conventions](https://github.com/sequelize/sequelize/blob/main/CONTRIBUTING.md#6-commit-your-modifications)?

## Description of Changes

When a `sql` literal is the value of an attribute in `where`, the values interpolated into it are escaped with the attribute's type. They are not values of that column, so this either throws or changes them:

```ts
// User.age is DataTypes.INTEGER
User.findAll({ where: { age: { [Op.gt]: sql`LENGTH(${'hello'})` } } });
// Error: Invalid value received for the "where" option.
//   cause: 'hello' is not a valid integer

queryGenerator.whereQuery({ age: sql`CAST(${'5'} AS INTEGER)` }, { model: User });
// WHERE "age" = CAST(5 AS INTEGER)   <- the string '5' was turned into a number

// User.name is DataTypes.STRING
queryGenerator.whereQuery({ name: sql`CONCAT(${'a'}, ${1})` }, { model: User });
// cause: 1 is not a valid string
```

(Reproduced with `@sequelize/core` and `@sequelize/postgres` 7.0.0-alpha.48.)

`formatFn` and `formatDialectAwareFn` already drop `type` from the escape options for their arguments, with the comment "arguments of a function can be anything, it's not necessarily the type of the attribute". `formatLiteral` passed the options through unchanged. This PR does the same thing there: the pieces of a literal are formatted without the attribute's `type`, and every other escape option (replacements, model, bind collector, ...) is kept.

Tests added in `test/unit/sql/where.test.ts`, next to the existing `{ intAttr1: literal('literal') }` case: a string inside a literal compared to an integer attribute (with `=` and with `Op.gt`), and a number inside a literal compared to a string attribute.

## List of Breaking Changes

None. Values in a literal used to be escaped with the attribute's type; now they are escaped by their JS type, like `fn()` arguments. Inputs that used to throw now work.
