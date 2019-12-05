author: @yucj

## Interface

### Function Types

```typescript
interface SearchFunc {
    (source: string, subString: string): boolean;
}
```

###  Indexable Types

```typescript
interface StringArray {
    [index: number]: string;
}
```

```typescript
interface NumberDictionary {
    [key: string]: number;
}
```

## Generic

### *type variable*

```typescript
function identity<T>(arg: T): T {
    return arg;
}
```

Explain:

- `<T>`: Declare *type variable* `T`
- `(arg: T)`: Assign the type of `arg` to `T`
- `function (): T`: Assign the value of `T` to the return type of the function


Multiple type variables: Separated with comma

```typescript
function identities<T, U> (arg1: T, arg2: U): [T, U] {
    return [arg1, arg2];
}
```

*tuple*: `[T, U]`

### *type of generic function*

`<U>(arg: U) => U` means a type of function that takes and returns variables with the same type

```ts
// Declare a generic function `identity`
function identity<T>(arg: T): T {
    return arg;
}

// Assign the 'type of generic function' to `myIdentity`. Then assign `identity` to `myIdentity`.
let myIdentity: <U>(arg: U) => U = identity;
```

As a *call signature* of an object literal type:

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: { <U>(arg: U): U } = identity;
```

Explained:

- https://stackoverflow.com/a/48967389
- https://www.typescriptlang.org/docs/handbook/interfaces.html#function-types


### *generic interfaces*

```typescript
interface Identities<V, W> {
    id1: V,
    id2: W
}

function identities<T, U> (arg1: T, arg2: U): Identities<T, U> {
    let identities: Identities<T, U> = {
        id1: arg1,
        id2: arg2
    };
    return identities;
}
```

### *generic class*

```typescript
type Props = {
   className?: string
   ...
};

type State = {
   submitted?: bool
   ...
};

class Component<U, T> {
    public id1: U
    public id2: T
    constructor(u: U, t: T) {
        this.id1 = u
        this.id2 = t
    }
    ...
}

class MyComponent extends Component<Props, State> {
   ...
}
```

**Caution: static members can not use the class’s type parameter**

> a class has two sides to its type: the static side and the instance side. Generic classes are only generic over their instance side rather than their static side, so when working with classes, static members can not use the class’s type parameter
> https://www.typescriptlang.org/docs/handbook/generics.html#generic-classes

### Reference

https://medium.com/@rossbulat/typescript-generics-explained-15c6493b510f


## `extends` a type variable (generic constraints)

Works as a constraint:

```typescript
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

`<T extends Lengthwise>`:

Means that the type variable `T` must _be a child of_ (_inherit_, _have the same shape with_) the interface `Lengthwise`, or the compiler will report an error.

## `keyof` type variable (index type query operator)

> For any type T, keyof T is the union of known, **public property names** of T

```typescript
interface Car {
    manufacturer: string;
    model: string;
    year: number;
}

let carProps: keyof Car; // the union of ('manufacturer' | 'model' | 'year')
```

Comined with `extends`:

```typescript
function getProperty<T, K extends keyof T>(o: T, propertyName: K): T[K] {
    return o[propertyName]; // o[propertyName] is of type T[K]
}
```

Means that the type variable `K` must be the type of the union of public property names of `T`.

## `T in Unions` (mapped types)

```typescript
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}
type Partial<T> = {
    [P in keyof T]?: T[P];
}
```

## `T extends U ? X : Y` (conditional types)

## Utility Types 

https://www.typescriptlang.org/docs/handbook/utility-types.html

```typescript
/**
 * Make all properties in T optional
 */
type Partial<T> = {
    [P in keyof T]?: T[P];
};

/**
 * Make all properties in T required
 */
type Required<T> = {
    [P in keyof T]-?: T[P];
};

/**
 * Make all properties in T readonly
 */
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};

/**
 * From T, pick a set of properties whose keys are in the union K
 */
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};

/**
 * Construct a type with a set of properties K of type T
 */
type Record<K extends keyof any, T> = {
    [P in K]: T;
};

/**
 * Exclude from T those types that are assignable to U
 */
type Exclude<T, U> = T extends U ? never : T;

/**
 * Extract from T those types that are assignable to U
 */
type Extract<T, U> = T extends U ? T : never;

/**
 * Construct a type with the properties of T except for those in type K.
 */
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;

/**
 * Exclude null and undefined from T
 */
type NonNullable<T> = T extends null | undefined ? never : T;

/**
 * Obtain the parameters of a function type in a tuple
 */
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;

/**
 * Obtain the parameters of a constructor function type in a tuple
 */
type ConstructorParameters<T extends new (...args: any) => any> = T extends new (...args: infer P) => any ? P : never;

/**
 * Obtain the return type of a function type
 */
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;

/**
 * Obtain the return type of a constructor function type
 */
type InstanceType<T extends new (...args: any) => any> = T extends new (...args: any) => infer R ? R : any;

/**
 * Marker for contextual 'this' type
 */
interface ThisType<T> { }
```
