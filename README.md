# Typescript Snippets

Stuff I find useful in typescript, and some experiments.

## Deep assign object

```ts
type ShallowMerge<T, U> = {
    [K in keyof (T & U)]: K extends keyof U
        ? U[K]
        : K extends keyof T
        ? T[K]
        : never;
};

type Primitive =
    | number
    | string
    | boolean
    | bigint
    | void
    | ((...args: any[]) => any);

type DeepMerge<T, U> = {
    deep: {
        [K in keyof (T & U)]: K extends keyof T & keyof U
            ? U[K] & T[K] extends Primitive
                ? U[K]
                : DeepMerge<T[K], U[K]>
            : K extends keyof U
            ? U[K]
            : K extends keyof T
            ? T[K]
            : never;
    };
    shallow: ShallowMerge<T, U>;
}[T[keyof T] | U[keyof U] extends Primitive ? "shallow" : "deep"];

type Shifted<T extends any[]> = T extends [any, ...infer Tail] ? Tail : never;

type AssignDeep<T extends any[], Res = {}> = {
    final: Res;
    recur: AssignDeep<Shifted<T>, DeepMerge<Res, T[0]>>;
}[T["length"] extends 0 ? "final" : "recur"];

type a = AssignDeep<
    [{ a: number; b: string; c: { foo: number; bar: boolean } }, { a: string }]
>;

type tests = [
    AssignDeep<
        [
            { a: number; b: string; c: { foo: number; bar: boolean } },
            { a: string; c: { foo: string } },
            { c: { bar: any[] } }
        ]
    >["c"]
];
```

[Playground Link](https://www.typescriptlang.org/play?#code/C4TwDgpgBAygFgQwDZIPYHcCyEBOBzCAHgBUAaKAVQD4oBeKAbwCgpWoBtAaSgEsA7KAGsIIVADMoACmJQAZJQCUAXQBcUbhAAewCHwAmAZyEjxlFmwsB+SlyXmLrNRu27Dx0ROL2H14re8WanwQAG64ANxMAL6RTKCQUAAKODwAtjzAPGF03gA+UHwArqkARrh5UAbAKXx4FSWoqEgQCHz1PHj8wBUhqDx6FZKSAHSjCPgGaq0g7EoKdDTTCrHx0AAiEBBg2PhEZJQ09MwWeptgascOrFy8AsIeUjLyFMpOUFo6+kb3pk-uphQAlcoNYKLY5FA-JwlO8XF8kil0pkwkDgWxQf40Wi1BstjsCCRbOQwdCqKiHG8Pq5viYJICseibNDyYF1LDPm4fp4WYyoXYGY4CqEIt4Yt4DIgUBg1PBkGgsLgCftqJEouw-FzITD8mDNRQYVT4ck0hkstBrAAiCVyjAWqBqC2nLYWpQrcDQeA8MQ6PQkdnUqDTWaHSH++HsabkUbDfhiXCQhA8JAw3yJpD2oVhHBuhIAQQMBg6fFxYD9hrcQaU5AAShAjEcoiHLqwxPxkGpawZIhYcBAAMaFHBqfOFvDFs6ET3eiC+4hUcgl-FETvkPwABiUVCoqvV7AtzVqwDgLrDbjXIKgFtbfGQdodvYHOBdOegCDoUBHRZLhG87AYgaCYoymzKASjUKoajwcIoD7C4oDERpANKCJQPGNQGiaFoBCiKAonIf8EHA6p+DwXC7G3Jg4ndKAdCqesOG8T8x2-cl2B5VgCKQ4DoLAypiNqaDYMYeDEIKICUJKNDQMaZpWlw3DSHY4TCL4yDBLghDUCIyD5LwpT-yE-9JKHQM+BmGEcKicl+TYKg9z7F0mFdJggA)

## JS kwargs (WIP)

```ts
type ShalloMerge<T, U> = {
    [K in keyof (T & U)]: K extends keyof U
        ? U[K]
        : K extends keyof T
        ? T[K]
        : never;
};

const KWARGS = Symbol("KWARGS");

class _kwargs<T extends Record<keyof any, unknown>> {
    protected _kwargs: T;
    public [KWARGS] = true;

    constructor(kwargs: T) {
        this._kwargs = kwargs;
    }

    public get kwargs(): T {
        return this._kwargs;
    }

    public merged<U extends Record<keyof any, unknown>>(
        kwargs: _kwargs<U>
    ): _kwargs<ShalloMerge<T, U>> {
        return new _kwargs(
            Object.assign({}, this._kwargs, kwargs.kwargs) as ShalloMerge<T, U>
        );
    }
}

interface KwargsConstructor {
    new <T extends Record<keyof any, unknown>>(kwargs: T): _kwargs<T>;
    <T extends Record<keyof any, unknown>>(kwargs: T): _kwargs<T>;
}

function __kwargs<T extends Record<keyof any, unknown>>(
    this: any,
    kwargs: T
): _kwargs<T> {
    return new _kwargs(kwargs);
}

const kwargs: KwargsConstructor = __kwargs as KwargsConstructor;

type Shifted<T extends any[]> = T extends [any, ...infer Tail] ? Tail : never;

type ExtractKwargs<T extends any[], Res = {}> = T["length"] extends 0
    ? Res
    : T[0] extends _kwargs<any>
    ? ExtractKwargs<Shifted<T>, ShalloMerge<Res, T[0]["kwargs"]>>
    : ExtractKwargs<Shifted<T>, Res>;

type tests = [ExtractKwargs<[1, 2, _kwargs<{ foo: 1 }>, _kwargs<{ bar: 2 }>]>];
```

[Playground Link](https://www.typescriptlang.org/play?#code/C4TwDgpgBAygFgQwDZIPYFkICcDmEA8AKgDRQCqAfFALxQDeAUFM1ANoDSUAlgHZQDWEEKgBmUABSEoAMnIBKALoAuKExbrOEAB7AIPACYBnAUNHk16y1AD85DgotWWKzTr1GTwsYUdPLtwntfP2coHggAN2wAbgYAXwYGAGNUHkNgKHYAdQBBACUAcRgaWBAAWwAjVCRxAHJs-KLauVjkpARDYwB9fgB3BFxDIihtXQNjPIgUrH18QS8oBB4QUgBXHn4eVF6eCipGdTAsVF0k3X0oHv7BlUJYw9WKpC4ktgbCmAUS4CxViFb1Ck0j9VmdUFgJH0BjhDLc5PRgsxgHAuIYAHRXaHGWhQwb3FgJRxgR7PV54DK4mHiOS3BF+LAQYCrLB8ZGojGUwz45iEh5PF5QMrYPCzMgjNzjKCTaazeZmJYrKDrTbbXYUcSclSYwb4Sg0y6c-DwZBoTC4Agkch7OlOBlMllhCC9A3XKkAeQqACspsA0R1DFwcDxxHQ4qQ2ejtTDSJy0Zz4R1YIgUBhhRbSHruVAEoTeLosCIEEloOxXYYAMKpdK-MEQg4scLO4ajdwTKbg2WmMQKtYbLY7PYastwrWGwgULPNiUeaUduZdxbLXsqgfqzVQQj6qNDcexQkidZnLipS7bqdjGftmbzhY9pV91WDiMqO-rzejstEfaOO3MviNl0sSHLEWniRIgXSARh0yMtK2BGtgHBEoum3RZjFLLE4OrUFEKwVpQEgJMuBEc5z1bRcQFYBQqGoRwpBbSVWDvNEWN4ERsA3BAuCQBw-ACLikERKAVHCKIsESAjoAAUR0LAi2ADCdXo6djAVKjSEmbF6DiGi6NYWokD0HBkVqL4GI8AAGITbE0oTblYCyzJUwCdQVCghPUWwZJ+eTFJhI0URIiBZnHUhjRTM08HwTTSECRz9M5Uy9g80JvLks4-KGeBiNI0KpQgQx3IYSSoF0dItNYRw0t8z9WAARigYgACZSDPOgRFQVAVDqnTWsNOgKgGFQmp06iGAcIA)

## Merge

```ts
export type Merge<A, B> = {
    [K in keyof (A & B)]: K extends keyof B
        ? B[K]
        : K extends keyof A
        ? A[K]
        : never;
};
```

## Type equality

[Github Link](https://github.com/Microsoft/TypeScript/issues/27024#issuecomment-421529650)
[calls `isTypeIdenticalTo`](https://github.com/Microsoft/TypeScript/issues/27024#issuecomment-510924206) under the hood.
