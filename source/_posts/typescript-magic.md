title: Into the Chamber of Secret, Break through the limits of TypeScript
date: 2023-04-30 22:26:14
tags: TypeScript
---


> This is a rewrite of the [zhihu article](https://zhuanlan.zhihu.com/p/613266541). Permission has been granted.
_The original author of this article is finding a job, [find him on GitHub](https://github.com/fightingcat)._

# What's Type level programming, a.k.a type gymnastics?

TypeScript is a powerful and expressive language that allows you to write complex yet elegant code. Some TypeScript users love to explore the possibilities of the type system and love to encode logic at type level.
This practice, as we have a slang for it, is known as _type gymnastics_. Type gymnastics can help you to perform various tasks such as parsing, validating, transforming, or computing values based on types. Type gymnastics often requires the users mastering a wide range of advanced TypeScript features such as conditional types, recursive types, template literal types, mapped types and more. The community also helps users to learn type gymnastics by creating fun and challenges such as [type challenges](https://tsch.js.org/).

You can proudly call yourself a TypeScript magician if you can solve most of the type challenges!

The type-level programming community has been creating all kinds of amazing tricks ever since the introduction of template literal type in version 4.1. Some enthusiasts have pushed this hobby to the extreme, and even tried to reimplement TypeScript at compile-time using pure types!

However, there are still three major obstacles that prevent TypeScript becoming a true [turing complete machine (#14833)](https://github.com/microsoft/TypeScript/issues/14833). They are:
* tail recursion count: the maximum number of recursive calls in a conditional type.
* type instantiation depth: the maximum depth when we instantiate a generic type alias.
* type instantiation count: the maximum number of type instantiations.

These limits are not arbitrary. The compiler has set a maximum limit for these three values and will throw an error [TS2589](https://github.com/microsoft/TypeScript/blob/1416053b9e85ca2344a7a6aa10456d633ea1cd65/src/compiler/diagnosticMessages.json#L2718) if we exceed them to prevent the compiler from hanging or crashing.
They are the walls that stop us from breaking through the limits of TypeScript compiler. But, there is actually a hidden layer behind the wall, if we explore the compiler source code carefully.

> Doing type level programming is like casting magic, but breaking through the wall of type gymnastics is like finding the hidden chamber of secrets that contains the most powerful and dangerous type of all.

## Some digression for background...

Back then three years ago, I wrote [an article](https://zhuanlan.zhihu.com/p/86008532) about how to implement a recursive descent parser. At that time, there was no template literal type, and a lot of type system limitations have been lifted since then. Crafting a code interpreter using recursive descent parsing would easily exceed the compiler's limit.

So I decided to switch to a bottom-up parsing method to more easily bypass the limit, with some tricks :). Later, as TypeScript gradually evolves, newly added features opened unprecedented possibilities of type gymnastics. This finally gave me a chance to implement a [type gymnastics parser generator for LALR(1) grammar](https://github.com/fightingcat/sits) to generate a parser for a subset of JavaScript syntax.
I designed a special purpose _"bytecode"_ for the ease of type level execution and directly compiled the input code into the bytecode during the parsing process(I still have some unfinished work left, procrastinating now...). My ultimate goal is to implement a type level scripting language that can execute sufficiently complex code.

To achieve this goal, I need to solve all the three limitations mentioned above to run lunatic code patterns, to some extent, at compile time!

![Type Level Programming nowadays can represent monsters like this...](https://pic1.zhimg.com/v2-7fac8439827a5a3e618f7faba7c5dfb8_r.jpg)

# Tail Recursion Count

Let’s look at the first limitation, the type tail recursion count (tailCount). Type level programming often uses recursive types. Before the previous versions supported recursive types, we usually [implemented recursion](https://github.com/microsoft/TypeScript/issues/14833) by combining mapped types, recursive type definitions and index types. Ever since we had [recursive types](https://github.com/microsoft/TypeScript/pull/33050) and [tail recursion of type instantiation](https://github.com/microsoft/TypeScript/pull/45711), this kind of type gymnastics became much more elegant, but at the same time we also got this tail recursion count limitation.

Searching for `tailCount` in the TypeScript source code, we can see that the implementation is related to computing conditional types. Let’s simplify the code a bit and take a look (sorry folks, I cannot get a link to GitHub source code due to checker.ts is too large):

```typescript
// transitively used in instantiateType
function getConditionalType(root: ConditionalRoot) {
    let result;
    let tailCount = 0;

    while (true) {
        if (tailCount === 1000) {
            error(2589);
            result = errorType;
            break;
        }
        // false branch
        if (!isTypeAssignableTo(checkType, extendsType)) {
            const falseType = getTypeFromTypeNode(root.node.falseType);

            if (canTailRecurse(falseType)) {
                // root = falseType;
                continue;
            }
            // tail call terminates, creating new type
            result = instantiateType(falseType);
            break;
        }
        // true branch
        if (isTypeAssignableTo(checkType, extendsType)) {
            const trueType = getTypeFromTypeNode(root.node.trueType);

            if (canTailRecurse(trueType)) {
                // root = trueType;
                continue;
            }
            // end of tail recursion, instantiate new type
            result = instantiateType(trueType);
            break;
        }
    }
    return result;

    function canTailRecurse(newType: Type) {
        if (newType.flags & TypeFlags.Conditional) {
            const newRoot = (newType as ConditionalType).root;
            const newCheckType = newRoot.isDistributive
                ? newRoot.checkType
                : undefined;

            if (
                !newCheckType ||
                !(newCheckType.flags & (TypeFlags.Union | TypeFlags.Never))
            ) {
                root = newRoot;

                if (newRoot.aliasSymbol) {
                    tailCount++;
                }
                return true;
            }
        }
    }
}
```

We can see that as long as the branch of the conditional type that matches is another conditional type, and the new conditional type is neither a distributive conditional type nor a union type or never, then the new conditional type will be substituted into the next round of computation.

The only problem is that it will trigger an error TS2589 when the loop reaches 1000 times, [TS Playground](https://www.typescriptlang.org/play?#code/C4TwDgpgBAWhBOB7AYgSwDboDwCgr6gHkBXYMUqCAD2AgDsATAZygAYBtAXQBo8CBhRMTrBKNesyh1iAWwBGCHAD4oAXiKlywdgCJ09AObAAFjs5jajFoOGiA-BrIUAXLAQoM2dgDpfJJ8DcbDxQNiJKANw4OKCQUADKqABeEAwAnBlqbkhomFhcQRlpkVAA9KVEANYAhiDeMeDQiSkMAIwA0llwOZ75Ia2sgyXlRIhgTEEAKvEATACsABxp9UA):

```typescript
type ZeroFill<
    Output extends 0[],
    Count extends number
> = Output["length"] extends Count ? Output : ZeroFill<[...Output, 0], Count>;

type Sized999 = ZeroFill<[], 999>; // Okay.
type Sized1K = ZeroFill<[], 1000>; // Oops, TS2589.
```

To avoid reaching the 1000 times recursion limit, we can deliberately break the tail recursion after a certain number of recursions (using an extra parameter to count), as long as we return a type that does not satisfy the above conditions, and does not affect the result of the type we return, such as returning an intersection type ([Playground Link](https://www.typescriptlang.org/play?#code/PTAEAkEMCcBMFoDGB7WBTUlQDsCuBbAIzWk2mkgE8AaMiy0AS23QA9QVy0BnAB2RbdQAF2SgAbpAA2uDLxlCAjAChhlXhgCS2RKQC8oANrLQpxbVAAmCwGYLAFgsBWCwDYLAdgsAOCwE5aRQAGQPNQRWtwu3DHcJdw93CvcN9wgKsQkyswy0jLaMtYy3jLRMtky1TLdJsQ0Bswm0ibaJtYm3ibRJtkm1SbdPtM03sw+0j7aPtY+3j7RPtk+1T7dKc6pzCnSKdop1ineKdEp2SnVKd012HQVzDXSNdo11jXeNdE12TXVNd0jzqHjCHkiHmiHliHniHkSHmSHlSHnS3hu3jC3ki3mi3li3ni3kS3mS3lS3nSfjqfjCfkifmiflifnifkSfmSflSfnSwRumQAugBuZSqdQYABaJGQADFGFIpAAeLKmADyuGEvDVoDQrGEaEEoCChj51CVoAAwshcNhhFqdXrYEI8EQSCbTKZNLqKMJGAIhNrdfrtLpDE7iNA+aADEFlAA+SOgD0kSDe322gMO8JBaNu0AAflAIHNAAtINgAOYYYRFjDQNCIXDQbiMcSV0UiMSlpjWkjcOsp7AiUW0Qi1yAAaxE1ZEkFloFr9cbPoHpdgc54aBtVYwjE9yaXptMEug0tlCtV6rVtAtVuEtCCcYAZKAAN4AX1NAC4C2AAEp1htNgIIgkPgzDJjwprnhqwiGAARFIepllWsERv69pCNe1qmvmhYytg0igH+3C4FIwgHqAUFqp+34Jjotb4Hqm5TigN6gAAZsgpBoJAiBFmuC6Adg5FHiecryoYAB0UmUbeBrGualrWrQQbQIYiZeku3B8jGQoihooAAMqMAAXmgsA2AA0vGIkymJRq0LUWY6TRypjlQEl6RgRmmbA9hWQYNmnuJ8lDE5AouW5NCgIQmpVowQjxTgyBMRg9qgMgbGgNwojQJQUkeUAA)):

```typescript
// Hard-code a number array, array index corresponds to value plus 1
type Incr = [
   1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
  21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40,
  41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60,
  61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80,
  81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100,
  0,
];

type ZeroFill<
    Output extends 0[],
    Count extends number,
    Iterations extends Incr[number] = 0
> = Iterations extends 100
    ? // Change the recursive type to an intersection type, break the tail recursion and reset the iteration
      ZeroFill<Output, Count, 0> & {}
    : // Recursion terminates
    Output["length"] extends Count
    ? // Final Result
      Output
    : // Increment the count for each recursion
      ZeroFill<[...Output, 0], Count, Incr[Iterations]>;

type Sized3K = ZeroFill<[], 3000>; // Okay.
type Sized4K = ZeroFill<[], 4000>; // Okay, but this is not the end of story...
```

# Type Instantiation Depth

The type above has now exceeded the limit of 1000 recursions, but if you comment out `Sized3K`, you will see that `Sized4K` will trigger TS2589. This is related to the second limitation, which is the [type instantiation depth](https://github.com/microsoft/TypeScript/pull/45025) (`instantiationDepth`). The reason why commenting out has an effect is because TypeScript will cache all instantiated types, which we will discuss later. Again, we can search for `instantiationDepth` in the TS source code. You can see that it is implemented with all types that have type mapping. There are several kinds of type mapping. For example, types that use generic parameters will have type mapping to map parameters to actual types. The relevant code is simplified as follows:

```typescript
function instantiateType(type?: Type, mapper?: TypeMapper): Type | undefined {
    return type && mapper ? instantiateTypeWithAlias(type, mapper) : type;
}

function instantiateTypeWithAlias(
    type: Type,
    mapper: TypeMapper,
    aliasSymbol?: Symbol,
    aliasTypeArguments?: readonly Type[]
) {
    if (instantiationDepth === 100 || instantiationCount >= 5000000) {
        error(2589);
        return errorType;
    }
    totalInstantiationCount++;
    instantiationCount++; // note: `count` only increments
    instantiationDepth++;
    const result = instantiateTypeWorker(
        type,
        mapper,
        aliasSymbol,
        aliasTypeArguments
    );
    instantiationDepth--; // note: only depth got decremented
    return result;
}
```

We also see the `instantiationCount` mentioned at the beginning, but we will first focus on the `instantiationDepth` for now. We can see that before and after each type alias instantiation, the instantiation depth will increase and decrease, and any call to instantiateType in the middle process will indirectly call the above type, which will make the depth count continue to increase.
To avoid reaching the maximum depth, one is to use tail recursion as much as possible, and the other is to reduce the dependence on intermediate types in the recursive process if you use the technique of breaking tail recursion before.
Here is an example code (to reduce interference we simplify it to only have two generic parameters, only keep the recursive logic itself, and use intersection type to prevent tail recursion instantiation):

```typescript
// Simulate a type that keeps calculating values and sets the finish state based on the results
type BadWay<State, Value> = State extends `finish`
    ? Value
    : // Use conditional type and infer type to save the new result
    Calculate<Value> extends infer NewValue
    ? // Use conditional type and infer type to save the new state
      CheckValue<NewValue> extends infer NewState
        ? // Compared to direct recursion, this adds two levels of type instantiation depth
          BadWay<NewState, NewValue>
        : never
    : never;

// In fact, repeating “calling” type aliases does not lose anything, TS will always cache all types instances.
type GoodWay<State, Value> = State extends `finish`
    ? Value
    : // Direct recursion reduces the instantiation depth as much as possible
      GoodWay<
          // The intermediate result is instantiated when passing parameters, which can be instantiated before recursion, releasing the increased depth.
          CheckValue<Calculate<Value>>,
          // The repeated alias calls will only be truly instantiated once due to the caching mechanism, so you don’t have to worry about various restrictions and overheads.
          Calculate<Value>
      > & {};
```

In the example above, `BadWay` increases the depth by two levels more than `GoodWay` for each recursion, which will reach the recursion depth limit faster.

With the instantiation level limit, we should **start recursion as early as possible**, putting all kinds of intermediate type calculations into parameters. And let each iteration type complete instantiation as soon as possible, releasing the increased depth.

Another way to release the depth count in advance is to manually **split the iteration process into multiple parts**, storing intermediate result as a type alias.
Let's take the first type as an example:


```typescript
type Sized3K = ZeroFill<[], 3000>; // Okay.
type Sized4K = ZeroFill<Sized3K, 4000>; // Okay.
type Sized5K = ZeroFill<Sized4K, 5000>; // Okay.
```

One thing to keep in mind is that as the type becomes more complex, the tsc compiler may handle it fine, but the editor may not give any feedback due to the long response time. This is because they use different methods and caches for type checking. For example, in the code above, if we comment out `Sized4K` (which has a cache effect), and directly generate `Sized5K` from `Sized3K`, we will get a `TS2589` error again. See [the playground link](https://www.typescriptlang.org/play?#code/PTAEAkEMCcBMFoDGB7WBTUlQDsCuBbAIzWk2mkgE8AaMiy0AS23QA9QVy0BnAB2RbdQAF2SgAbpAA2uDLxlCAjAChhlXhgCS2RKQC8oANrLQpxbVAAmCwGYLAFgsBWCwDYLAdgsAOCwE5aRQAGQPNQRWtwu3DHcJdw93CvcN9wgKsQkyswy0jLaMtYy3jLRMtky1TLdJsQ0Bswm0ibaJtYm3ibRJtkm1SbdPtM03sw+0j7aPtY+3j7RPtk+1T7dKc6pzCnSKdop1ineKdEp2SnVKd012HQVzDXSNdo11jXeNdE12TXVNd0jzqHjCHkiHmiHliHniHkSHmSHlSHnS3hu3jC3ki3mi3li3ni3kS3mS3lS3nSfjqfjCfkifmiflifnifkSfmSflSfnSwRumQAugBuZSqdQYABaJGQADFGFIpAAeLKmADyuGEvDVoDQrGEaEEoCChj51CVoAAwshcNhhFqdXrYEI8EQSCbTKZNLqKMJGAIhNrdfrtLpDE7iNA+aADEFlAA+SOgD0kSDe322gMO8JBaNu0AAflAIHNAAtINgAOYYYRFjDQNCIXDQbiMcSV0UiMSlpjWkjcOsp7AiUW0Qi1yAAaxE1ZEkFloFr9cbPoHpdgc54aBtVYwjE9yaXptMEug0tlCtV6rVtAtVuEtCCcYAZKAAN4AX1NAC4C2AAEp1htNgIIgkPgzDJjwprnhqwiGAARFIepllWsERv69pCNe1qmvmhYytg0igH+3C4FIwgHqAUFqp+34Jjotb4Hqm5TigN6gAAZsgpBoJAiBFmuC6Adg5FHiecryoYAB0UmUbeBrGualrWrQQbQIYiZeku3B8jGQoihooAAMqMAAXmgsA2AA0vGIkymJRq0LUWY6TRypjlQEnKIWaj6UZpmwPYVkGDZp7yr5ZmWbQQxOQKLluZQHneRgYWwE4gWgMFYnJRFoDrNFsXucoQA).This is not caused by the two limitations we mentioned before, because this time we only have 2000 iterations, much less than `Sized3K`. The actual cause of this compilation error leads us to the third limitation: the type instantiation count (`instantiationCount`).


# Type Instantiation Count

We have encountered the variable `instantiationCount` before in the function `instantiateTypeWithAlias`. It has a current limit of `5,000,000`, which is incremented together with `instantiationDepth` every time a type is instantiated. Unlike `instantiationDepth`, it does not decrease when the instantiation is done because it represents the number of type instances that are created in one instantiation process. If we continue to search the source code, we can see that it is set to 0 when the compiler checks some types of syntax nodes:

```typescript
function checkSourceElement(node) {
    // ...
    instantiationCount = 0;
    // ...
}

function checkDeferredNode(node) {
    // ...
    instantiationCount = 0;
    // ...
}

function checkExpression(node, checkMode, forceTuple) {
    // ...
    instantiationCount = 0;
    // ...
}

```

If we debug tsc and set a breakpoint after the condition `instantiationCount >= 5000000`, we will see in the call stack that the error is thrown when instantiating an object type with more than 3700 members. This object type is actually the array type that we generated. We will not analyze step by step why it produces so many (5 million) type instances here. We just need to know that this array is created by many generic parameters, which leads to a large number of type instances. Even if our type gymnastics did not generate such a large array, it would slow down the speed when producing so many type instances. So how can we avoid this situation?

The direct conclusion is to use other types to achieve the purpose according to the needs. For example, if you use an array type to simulate a stack structure, and only need to access the last one or a few elements, then you can define a List object type, which is concatenated, but the structure of the object is fixed:

```typescript
type List<T> = { data: T; prev: List<T> };

type Append<P extends List<unknown>, T> = { data: T; prev: P };

type Stack = Append<Append<Append<never, 1>, 2>, 3>; // [1, 2, 3]

type Popped = Stack["prev"]; // [1, 2]

type Last = Stack["data"]; // 3

type SecondToLast = Stack["prev"]["data"]; // 2
```

Alternatively we can encode the logic in string by constructing template string of a specific format.

```typescript
type Append<S extends string, T extends string> = `${T},${S}`;

type Stack = Append<Append<Append<"", "alpha">, "beta">, "gamma">;

type Popped = Stack extends `${string},${infer S}` ? S : never;

type Last = Stack extends `${infer T},${string}` ? T : never;

```

[You can even use union types](https://github.com/fightingcat/sits/blob/fd958753c1f7514c29314fbfabab86d97075acec/lib/helpers.d.ts#L77). However, you need to pay attention that when union types are used in distributive conditional types, they will generate the same number of type instances as the number of elements, but they are still a better choice in many cases.

Usual type level programming will not trigger this limitation, so we will not describe it too much. To optimize the number of instantiations, you inevitably need to read the compiler source code, or you can add the `--diagnostics` parameter when calling `tsc`, and observe the `Instantiations` in the output, which can show how many type instances are generated in total.

# Conclusion

One caveat is that all tricks in the article are probably the byproduct of a specific compiler implementation and are not officially supported by the TypeScript team. And they may even be broken in the future!
Actually, most of these dark magics are manipulating type instantiation to make time-space trade-offs. We are nudging compiler to forget intermediate type caches ([Obliviate!](https://harrypotter.fandom.com/wiki/Memory_Charm#cite_note-COS16-1)), prompting it to compute more types ([Imperio!](https://harrypotter.fandom.com/wiki/Imperius_Curse)) and forcing it to work longer for a compilation ([Incarcerous!](https://harrypotter.fandom.com/wiki/Incarcerous_Spell)). All the grueling rituals for the holy grail of turing completeness!

However, they are still very useful in some cases and help us break the limitations of TypeScript! The snake is in the chamber of secret, so use these lost spells wisely, my great mage!


If you find this article useful, I would be more than happy if you can [treat me some coffee](https://github.com/sponsors/HerringtonDarkholme/).

{%img /images/chamber.png%}

-----

_Translation, rewrite and title image generation are all assisted by MicroSoft Bing Chat._
