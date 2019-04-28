# Validation

This chapter will demonstrate how you can apply different validation strategies for the [@marblejs/middleware-io](../api-reference/middleware-io.md) validator using [io-ts](https://github.com/gcanti/io-ts) schemas.

[io-ts](https://github.com/gcanti/io-ts) is a nifty library that validates and checks at runtime that incoming data has the shape that you expect. This powerful library can extract a static type from the validator that is guaranteed to match any values that pass validation.

## String literals

You can create string literals using combination of `t.union` and `t.literal` codecs. Lets say that we would like to validate users that can contain different roles from the given set: `'ADMIN' | 'GUEST'`.

```typescript
const userSchema = t.type({
  id: t.string,
  name: t.string,
  roles: t.array(t.union([
    t.literal('ADMIN'),
    t.literal('GUEST'),
  ])),
});

type User = t.TypeOf<typeof userSchema>;

👇

type User = {
  id: string;
  name: string;
  roles: ('ADMIN' | 'GUEST')[];
};
```

## Branded types

[io-ts](https://github.com/gcanti/io-ts) allows you to create a custom codec validators that must match to given predicate. There are ton of use cases where you can use this mechanism, eg. you would like to validate users which are adult \(age is bigger or equal 18\).

```typescript
interface AgeAdultBrand {
  readonly AgeAdult: unique symbol;
}

type AdultAge = t.brand(
  t.number,
  (age): age is t.Branded<number, AgeAdultBrand> => age >= 18,
  'AgeAdult'
);

const userSchema = t.type({
  id: t.string,
  name: t.string,
  age: AdultAge,
}),

type User = t.TypeOf<typeof userSchema>;

👇

type User = {
  id: string;
  name: string;
  age: t.Branded<number, NumberAdultBrand>;
};
```

As you can see, [io-ts](https://github.com/gcanti/io-ts) requires some boilerplate in order to have it properly typed, but the benefits are invaluable.

## Optional properties

According to [io-ts](https://github.com/gcanti/io-ts) documentation you can define a validator with optional values as an intersection of optional and required properties.

```typescript
const StorySchema = t.intersection([
  t.type({
    type: t.literal('story'),
    commentsTotal: t.number,
  }),
  t.partial({
    description: t.string,
    url: t.string,
  })
]);

type Story = t.TypeOf<typeof StorySchema>;

👇

type Story = {
  type: 'story';
  commentsTotal: number;
} & {
  description?: string | undefined;
  url?: string | undefined;
}
```

As you can see in the example above, the generated `Story` type is not as clean as it might be. [Jasse Hallett](https://www.olioapps.com/blog/checking-types-real-world-typescript/) in his article proposed a slightly different approach to defining optional values in validator schemas. Lets define a handy `optional` combinator!

```typescript
export const optional = <T extends t.Any>(
  type: T,
  name = `${type.name} | undefined`
): t.UnionType<
  [T, t.UndefinedType],
  t.TypeOf<T> | undefined,
  t.OutputOf<T> | undefined,
  t.InputOf<T> | undefined
> =>
  t.union<[T, t.UndefinedType]>([type, t.undefined], name);
```

> Technically this implies that we expect the given property to be present in every case, but that the value might be `undefined`. In practice, that distinction often does not matter, and io-ts will validate an object that is missing a required property if the type of that property is allowed to be `undefined`.

Using the introduced combinator our `StorySchema` definition can be much more cleaner and readable.

```typescript
const StorySchema = t.type({
  type: t.literal('story'),
  commentsTotal: t.number,
  description: optional(t.string),
  url: optional(t.string),
});

type Story = t.TypeOf<typeof StorySchema>;

👇

type Story = {
  type: 'story';
  commentsTotal: number;
  description: string | undefined;
  url: string | undefined;
}
```

