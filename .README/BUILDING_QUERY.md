## Building query

### `sql.valueList`

```js
(values: $ReadOnlyArray<PrimitiveValueExpressionType>) => ValueListSqlTokenType;

```

Creates a list of values, e.g.

```js
await connection.query(sql`
  SELECT (${sql.valueList([1, 2, 3])})
`);

```

Produces:

```js
{
  sql: 'SELECT ($1, $2, $3)',
  values: [
    1,
    2,
    3
  ]
}

```

### `sql.tuple`

```js
(values: $ReadOnlyArray<PrimitiveValueExpressionType>) => TupleSqlTokenType;

```

Creates a tuple (typed row construct), e.g.

```js
await connection.query(sql`
  INSERT INTO (foo, bar, baz)
  VALUES ${sql.tuple([1, 2, 3])}
`);

```

Produces:

```js
{
  sql: 'INSERT INTO (foo, bar, baz) VALUES ($1, $2, $3)',
  values: [
    1,
    2,
    3
  ]
}

```

### `sql.tupleList`

```js
(tuples: $ReadOnlyArray<$ReadOnlyArray<PrimitiveValueExpressionType>>) => TupleListSqlTokenType;

```

Creates a list of tuples (typed row constructs), e.g.

```js
await connection.query(sql`
  INSERT INTO (foo, bar, baz)
  VALUES ${sql.tupleList([
    [1, 2, 3],
    [4, 5, 6]
  ])}
`);

```

Produces:

```js
{
  sql: 'INSERT INTO (foo, bar, baz) VALUES ($1, $2, $3), ($4, $5, $6)',
  values: [
    1,
    2,
    3,
    4,
    5,
    6
  ]
}

```

### `sql.unnset`

```js
(
  tuples: $ReadOnlyArray<$ReadOnlyArray<PrimitiveValueExpressionType>>,
  columnTypes: $ReadOnlyArray<string>
): UnnestSqlTokenType;

```

Creates an `unnset` expressions, e.g.

```js
await connection.query(sql`
  SELECT bar, baz
  FROM ${sql.unnest(
    [
      [1, 'foo'],
      [2, 'bar']
    ],
    [
      'int4',
      'text'
    ]
  )} AS foo(bar, baz)
`);

```

Produces:

```js
{
  sql: 'SELECT bar, baz FROM unnest($1::int4[], $2::text[]) AS foo(bar, baz)',
  values: [
    [
      1,
      2
    ],
    [
      'foo',
      'bar'
    ]
  ]
}

```

### `sql.identifier`

```js
(names: $ReadOnlyArray<string>) => IdentifierTokenType;

```

[Delimited identifiers](https://www.postgresql.org/docs/current/static/sql-syntax-lexical.html#SQL-SYNTAX-IDENTIFIERS) are created by enclosing an arbitrary sequence of characters in double-quotes ("). To create create a delimited identifier, create an `sql` tag function placeholder value using `sql.identifier`, e.g.

```js
sql`
  SELECT 1
  FROM ${sql.identifier(['bar', 'baz'])}
`;

```

Produces:

```js
{
  sql: 'SELECT 1 FROM "bar"."bar"',
  values: []
}

```

### `sql.identifierList`

```js
(identifiers: $ReadOnlyArray<$ReadOnlyArray<string>>) => IdentifierListTokenType;

```

Creates a list of identifiers, e.g.

```js
sql`
  SELECT 1
  FROM ${sql.identifierList([
    ['bar', 'baz'],
    ['qux', 'quux']
  ])}
`;

```

Produces:

```js
{
  sql: 'SELECT 1 FROM "bar"."baz", "qux"."quux"',
  values: []
}

```

#### Identifier aliases

A member of the identifier list can be aliased:

```js
sql`
  SELECT 1
  FROM ${sql.identifierList([
    {
      alias: 'qux',
      identifier: ['bar', 'baz']
    },
    {
      alias: 'corge',
      identifier: ['quux', 'quuz']
    }
  ])}
`;

```

Produces:

```js
{
  sql: 'SELECT 1 FROM "bar"."baz" "qux", "quux"."quuz" "corge"',
  values: []
}

```

### `sql.raw`

```js
(rawSql: string, values?: $ReadOnlyArray<PrimitiveValueExpressionType>) => RawSqlTokenType;

```

Raw/ dynamic SQL can be inlined using `sql.raw`, e.g.

```js
sql`
  SELECT 1
  FROM ${sql.raw('"bar"')}
`;

```

Produces:

```js
{
  sql: 'SELECT 1 FROM "bar"',
  values: []
}

```

The second parameter of the `sql.raw` can be used to bind [positional parameter](https://www.postgresql.org/docs/current/sql-expressions.html#SQL-EXPRESSIONS-PARAMETERS-POSITIONAL) values, e.g.

```js
sql`
  SELECT ${sql.raw('$1', [1])}
`;

```

Produces:

```js
{
  sql: 'SELECT $1',
  values: [
    1
  ]
}

```

#### Building dynamic queries

If you require to build a query based on a _dynamic_ condition, then consider using an SQL builder for that specific query, e.g. [Sqorn](https://sqorn.org/).

```js
const query = sq
  .return({
    authorId: 'a.id',
    name: 'a.last_name'
  })
  .distinct
  .from({
    b: 'book'
    })
  .leftJoin({
    a: 'author'
  })
  .on`b.author_id = a.id`
  .where({
    title: 'Oathbringer',
    genre: 'fantasy'
  })
  .query;

sql`${sql.raw(query.text, query.args)}`

```

#### Named parameters

`sql.raw` supports named parameters, e.g.

```js
sql`
  SELECT ${sql.raw(':foo, :bar', {bar: 'BAR', foo: 'FOO'})}
`;

```

Produces:

```js
{
  sql: 'SELECT $1, $2',
  values: [
    'FOO',
    'BAR'
  ]
}

```

Named parameters are matched using `/[\s,(]:([a-z_]+)/g` regex.