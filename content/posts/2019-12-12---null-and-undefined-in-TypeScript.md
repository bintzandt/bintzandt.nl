---
title: "null and undefined in TypeScript"
date: "2019-12-12T12:50:00.169Z"
template: "post"
draft: false
slug: "/posts/null-and-undefined-in-ts/"
category: "Development"
tags:
  - "TypeScript"
description: "
null and undefined lead to some unexpected behaviour in TypeScript.
This post explains one of the problems that may occur.
"
socialImage: "/media/ts.png"
---
![header](/media/ts.png)
At Yoast, we currently use [NestJS](https://nestjs.com/) for our server.
This is a NodeJS server framework written in [TypeScript](https://www.typescriptlang.org/).
From the homepage of TypeScript: "TypeScript is a typed superset of JavaScript that compiles to plain JavaScript."
The main advantage of using TypeScript instead of plain JavaScript is that TypeScript does type-checking at compile time.
This leads to more robust code and faster development since you can be sure about the type of data you are working with.

That is what we thought, at least.

## The casus
We were tasked with creating a function that takes an array of objects and executes a function on each object.
For this example, lets call the type `Cheese` and the function that needs to be executed `slice()`, which turns a Cheese in a CheeseSlice.

We started with declaring the function.
```typescript
function sliceCheeses( cheeses: Cheese[] ): CheeseSlice[] {}
```
So far, so good, so lets add some functionality to this function.
```typescript
function sliceCheeses( cheeses: Cheese[] ): CheeseSlice[] {
    cheeses.map( cheese => cheese.slice() );
}
```
Looks good right? The array of cheeses are now getting sliced into smaller pieces.

## Testing
Since we are good developers, we decide to implement some unit tests for this function.
We start with a basic test, that simply checks whether our function works as expected.
We use [jest](https://jestjs.io/) for testing but the framework should not matter.
```typescript
it( "slices one cheese correctly", () => {
	const cheese = new Cheese();
    const expected = [ cheese.slice() ];
    const result = sliceCheeses( [ cheese ] );

    expect( result ).toEqual( expected );
} )
```
This test works as expected.
We decide to add some more tests to cover some other cases.
```typescript
it( "slices multiple cheeses", () => {
    const cheese1 = new Cheese();
    const cheese2 = new Cheese();
    
    const expected = [ cheese1.slice(), cheese2.slice() ];
    const result = sliceCheeses( [ cheese1, cheese2 ] );

    expect( result ).toEqual( expected );
} )

it( "handles an empty array", () => {
    const result = sliceCheeses( [] );

    expect( result ).toEqual( [] );
} )
``` 
All our tests are still succeeding.
We become so convinced of the correctness of our function that we decide to test some really weird cases.
```typescript
it( "handles invalid input in the array", () => {
    const cheese1 = new Cheese();
    
    const expected = [ cheese1.slice() ];
    const result = sliceCheeses( [ null, null, cheese1 ] );

    expect( result ).toEqual( expected );
} )

it( "handles invalid input", () => {
    const result = sliceCheeses( null );

    expect( result ).toEqual( [] );
} )
```
Now something odd happens.
I expected that TypeScript would at least complain about the second call since `null` does not resemble `Cheese[]` in my opinion.

But TypeScript does not do any of that.
It happily compiles the test and lets the test crash because `map()` does not exist on `null`.

Something similar happens in the other test.
It compiles fine but crashes because `slice()` does not exist on `null`.
This was not what we expected to happen.

## So what happens here?
It turns out that TypeScript has two special types that do not adhere to the normal type-checking. 
These types are `Null` and `Undefined`, which have the values `null` and `undefined` respectively.
In TypeScript, `null` and `undefined` are assignable to _every_ type.

## Conclusion
We do not want our application to crash when, for whatever reason, an incorrect input gets passed to this function.
This might happen when our database cannot find any results and returns a `null`.

Of course, it is better to immediately check for those cases where they occur and do the error handling there, but we all know that we often forget about that while developing.

In the end, we decided to rewrite the function so that it can handle more cases.
But doing this for every function where a `null` or `undefined` might show up, is tedious.
```typescript
function sliceCheeses( cheeses: Cheese[] ): CheeseSlice[] {
    if ( ! cheeses ){
        return [];
    }
    
    cheeses
        .filter( cheese => cheese )
        .map( cheese => cheese.slice() );
}
```
A compiler flag (`--strictNullChecks`) [has been added](https://github.com/Microsoft/TypeScript/pull/7140) to TypeScript that enforces stricter typing and that would have complained about this.
But for some reason, this flag is not on by default.
I suspect it has to do with backwards-compatibility but this lenient checking created some confusion on our end.

It would have been cool if TypeScript would have complained about it from the beginning.
In their [documentation](https://www.typescriptlang.org/docs/handbook/basic-types.html#null-and-undefined), they write that turning on the flag helps to avoid _many_ common errors.
Three lines below, they again "encourage the use of `--strictNullChecks` when possible".

Maybe the time has come to turn on this flag by default? 
