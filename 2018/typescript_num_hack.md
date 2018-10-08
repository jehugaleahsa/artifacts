## TypeScript: Compile-time vs Runtime
TypeScript's type-safety is fantastic and getting better all the time. On rare occasions, TypeScript's type-safety just gets in the way. I try as hard as possible to avoid using `any` in my code, but today I ran across an issue where it seemed unavoidable.

In my Angular project, I have type-safe interfaces defined for all of my API models. I was working with a model like this:

```typescript
export interface PaymentModel {
    paymentAmount: number | null;
    // ... other properties
}
```

I was binding this payment amount to an `<input type="text" />` so users could enter in how much they wanted to pay. For whatever reason, I could not use `<input type="number" />`. The challenge is that, at runtime, Angular is going to stick a `string` in that property. Now, I have a problem: if TypeScript already thinks the property is a `number`, how do I check that the value is a valid number at runtime?

Consider this code to determine if a valid payment amount is entered:

```typescript
if (model.paymentAmount == null) {
    return false;
}
if (typeof model.paymentAmount === "string" && model.paymentAmount.trim() === "") {
    return false;
}
// ... more
```

The call to `trim` above will result in a compile-time error. Since TypeScript "knows" `paymentAmount` is a `number`, the only way the code on the right-hand side of the condition would execute is if the property was a `number` *and a `string` at the same time*. TypeScript is smart enough to know that's impossible so it switches the type of the property to `never`.

The `never` type exists to indicate an impossible situation. As such, there are no legal operations allowed on a variable of type `never`. More frequently, `never` is used as the return type for methods with infinite loops or that always throw exceptions. You can read more about never [here](https://www.typescriptlang.org/docs/handbook/basic-types.html#never).

### The challenge
So, at runtime we need to validate we have a `string` that can be parsed as a `number`. The challenge is we can't call any operations specific to `string`s. One alternative is to forego type-safety and use `any`.

```typescript
const paymentAmount: any = model.paymentAmount;
if (typeof paymentAmount === "string" && paymentAmount.trim() === "") {  // OK
    return false;
}
```

## A hack solves everything
But what if the user enters in `sdkfljsldfjks`? We need to handle more than just empty strings! Fortunately, in JavaScript there's a nifty trick to convert `string`s to `number`s: the `+` operator.

If you apply `+` to a `string` *representing a number*, you get a `number`. If you have `sdkfljsldfjks`, you get `NaN`. So you can test for invalid `string`s with this short expression:

```typescript
if (isNaN(+paymentAmount)) {
    return false;
}
```

I hope you already see what's coming. You can just as equally apply `+` to a `number` and get back a `number`. So we don't need to create an intermediate `any` variable anymore. We can work directly with `model.paymentAmount`. At compile-time, TypeScript is happy because you are applying `+` to a `number`, so the type of the expression is a `number`. At runtime, JavaScript is happy because you are applying `+` to a `string`, so the type of the expression is a `number`. So the TypeScript code ends up just looking like this:

```typescript
if (model.paymentAmount == null || isNaN(+model.paymentAmount)) {
    return false;
}
```

## Why not use `parseFloat`?
You might wonder why I prefer `+` here, instead of [`parseFloat`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseFloat). Actually, `parseFloat` is more lenient about parsing than `+`. In other words, `parseFloat` will gladly return `123` when it is passed `123abc`, but `+` will return `NaN`. Unless you are willing to accept trailing garbage, `+` is the way to go.

## Handling integers (`~~`)
When you are dealing with integers, you can use a similar hack. Rather using `+`, you can use `~~` instead. This will convert a runtime `string` into a truncated `number`. If you do not want to allow decimal values, you have to be a little more careful. In that case you might need to add an additional condition:

```typescript
if (model.paymentAmount == null 
    || isNaN(+model.paymentAmount) 
    || +model.paymentAmount !== ~~model.paymentAmount) {
        return false;
}
```

Of course, if you know a little bit about how `NaN` works, you also know `NaN === NaN` always evaluates to `false`. So you can eliminate the call to `isNaN` entirely:

```typescript
if (model.paymentAmount == null || +model.paymentAmount !== ~~model.paymentAmount) {
        return false;
}
```

Here's a truth table if you want to see for yourself:

```
 ----------------------------------------------------
|   pa   | +pa  | ~~pa | +pa === ~~pa | +pa !== ~~pa |
|--------|------|------|--------------|--------------|
| "abc"  | NaN  | NaN  | false        | true         |
|--------|------|------|--------------|--------------|
| "3.14" | 3.14 | 3    | false        | true         |
|--------|------|------|--------------|--------------|
| "3"    | 3    | 3    | true         | false        |
|--------|------|------|--------------|--------------|
| 3.14   | 3.14 | 3    | false        | true         |
|--------|------|------|--------------|--------------|
| 3      | 3    | 3    | true         | false        |
 ----------------------------------------------------
```

## (Update) Handling whitespace
Shortly after posting this article, I discovered there was an edge-case I missed... empty strings and all whitespace! It turns out `+""` and `+"     "` both evaluate to `0` (zero). So now what? We're back to needing to call `string` operations on `number`s in TypeScript. So, time to think of another workaround!

We can simply add an empty string (`""`) to our `number` or `string` to convert it to a `string` first. Then we can call `trim` and see if it's empty:

```typescript
("" + model.paymentAmount).trim() === ""
```

So our final check for valid numbers becomes:

```typescript
if (model.paymentAmount == null
    || ("" + model.paymentAmount).trim() === ""
    || isNaN(+model.paymentAmount)) {
    return false;
}
```

## Hacky but all right...
Needless to say, this is all a bit hacky, but it is wonderful when you can avoid relying on `any` whenever possible. Obviously, you'll want to document this hack so others know what you're doing. I figured I would share this hack even if it's not immediately useful for folks, simply because it highlights a lot of nifty JavaScript techniques.