# TypeScript Fundamentals

## Basics:
#### (1) `x` is a string, because we've initialized it
```typescript
let x = "hello world";
```

#### (2) re-assignment is fine
```typescript
let x = "hello world";
```

#### (3) but if we try to change type
```typescript
x = 42; // 🚨 ERROR -> Type '42' is not assignable to type 'string'.
```

#### (4) let's look at const. The type is literally 'hello world'
```typescript
const y = "hello world";

/*
  This is called a 'string literal type'. y can never be re-assigned since it's a const,
  so we can regard it as only ever holding a value that's literally the string 'hello world'
  and no other possible value.
*/
```

#### (5 && 6) sometimes we need to declare a variable w/o initializing it
```typescript
let z;
z = 41;
z = "abc"; // this isn't good

/*
  If we look at the type of z, its `any`.
  This is the most flexible type in TypeScript.
*/
```

#### (7) we could improve this situation by providing a type annotation when we declare our variable
```typescript
let zz: number;
zz = 41;
zz = "abc"; // 🚨 ERROR -> Type '"abc"' is not assignable to type 'number'.
```


### Simple Arrays
#### (8) simple array types can types can be expressed using [ ]
```typescript
let aa: number[] = [];
aa.push(33);
aa.push("abc"); // 🚨 ERROR -> Argument of type "abc" is not assignable to parameter of type 'number';
```

#### (9) we can even define a tuple, which has a fixed length
```typescript
let bb: [number, string, string number] = [
  123,
  "Fake Street",
  "Nowhere, USA",
  10010
];

bb = [1, 2, 3]; // 🚨 ERROR -> Type 'number' is not assignable to type 'string'.
```

#### (10) Tuple values often require type annotations (  : [number, number] )
```typescript
const xx = [32, 31]; // number[];
const yy: [number, number] = [32, 31];
```


### Objects
#### (11) object types can be expressed using {} and property names
```typescript
let cc: { houseNumber: number; streetName: string };

cc = {
  streetName: "Fake Street",
  houseNumber: 123
};

cc = {
  houseNumber: 33
};
/* 🚨 Property 'streetname'
   🚨 is missing in type '{ houseNumber: number; }'
   🚨 but required in type '{ houseNumber: number; streetName: string; }'
*/
```

#### (12) You can use the optional operator (?) to indicate that something may or may not be there
```typescript
let dd: { houseNumber: number; streetName?: string};

dd = {
  houseNumber: 33
}
```

#### (13) If we want to re-use this type, we can create an interface
```typescript
interface Address {
  houseAddress: number;
  streetName?: string;
}
// and refer to it by name

let ee: Address = { houseNumber: 33 };
```


### Union & Intersection
#### (14) Union types
Sometimes we have a type that can be on of several things
```typescript
export interface HasPhoneNumber {
  name: string;
  phone: number;
}

export interface HasEmail {
  name: string;
  email: string;
}

let contactInfo: HasEmail | HasPhoneNumber =
  Math.random() > 0.5
    ? {
        // we can assign it to a HasPhoneNumber
        name: "Mike",
        phone: 3215551212
      }
    : {
        // or a HasEmail
        name: "Mike",
        email: "mike@example.com"
      };

contactInfo.name; // NOTE: we can only access the .name property  (the stuff HasPhoneNumber and HasEmail have in common)
```

#### (15) Intersection types
```typescript
let otherContactInfo: HasEmail & HasPhoneNumber = {
  // we _must_ initialize it to a shape that's asssignable to HasEmail _and_ HasPhoneNumber
  name: "Mike",
  email: "mike@example.com",
  phone: 3215551212
};

otherContactInfo.name; // NOTE: we can access anything on _either_ type
otherContactInfo.email;
otherContactInfo.phone;
const zzz: any = {} as never;
```
___

## Function Basics:
#### (1) function arguments and return values can have type annotations
```typescript
function sendEmail(to: HasEmail): { recipient: string; body: string } {
  return {
    recipient: `${to.name} <${to.email}>`, // Mike <mike@example.com>
    body: "You're pre-qualified for a loan!"
  }
}
```

#### (2) or the arrow-function variant
```typescript
const sendTextMessage = (
  to: HasPhoneNumber
): { recipient: string; body: string; } => {
  return {
    recipient: `${to.name} <${to.phone}>`,
    body: "You're pre-qualified for a loan!"
  }
}
```

#### (3) return types can alsmost always be inferred
```typescript
function getNameParts(contact: { name: string }) {
  const parts = contact.name.split(/\s/g); // split @ whitespace
  if (parts.length < 2) {
    throw new Error(`Can't calculate name parts from name "${contact.name}"`);
  }
  return {
    first: parts[0],
    middle: parts.length === 2
              ? undefined
              : // everything except first and last
              parts.slice(1, parts.length - 2).join(" "),
    last: parts[parts.length - 1]
  };
}
```

#### (4) rest params work just as you'd think. Type must be array-ish
```typescript
const sum = (...vals: number[]) => vals.reduce((sum, x) => sum + x, 0);
console.log(sum(3, 4, 6)); // 13
```

#### (5) we can even provide multiple function signatures
```typescript
// "overload signatures"
function contactPeople(method: "email", ...people: HasEmail[]): void;
function contactPeople(method: "phone", ...people: HasPhoneNumber[]): void;

// "function implementation"
function contactPeople(
  method: "email" | "phone",
  ...people: (HasEmail | HasPhoneNumber)[]
): void {
  if (method === "email") {
    (people as HasEmail[]).forEach(sendEmail);
  } else {
    (people as HasPhoneNumber[]).forEach(sendTextMessage);
  }
}

// ✅ email works
contactPeople("email", { name: "foo", email: "" });

// ✅ phone works
contactPeople("phone", { name: "foo", phone: 12345678 });

// 🚨 mixing does not work
contactPeople("email", { name: "foo", phone: 12345678 });
```

#### (6) the lexical scope (this) of a function is part of it's signature
```typescript
function sendMessage(
  this: HasEmail & HasPhoneNumber,
  preferredMethod: "phone" | "email"
) {
  if (preferredMethod === "email") {
    console.log("sendEmail");
    sendEmail(this);
  }
}

const c = { name: "Mike", phone: 3215551212, email: "mike@example.com" };

function invokeSoon(cb: () => any, timeout: number) {
  setTimeout(() => cb.call(null), timeout);
}

// 🚨 this is not satisfied
invokeSoon(() => sendMessage("email"), 500);
// The 'this' context of type 'void' is not assignable to method's 'this' of type
// 'HasEmail & HasPhoneNumber'. Type 'void' is not assignable to type 'HasEmail'


// ✅ creating a bound function is one solution
const bound = sendMessage.bind(c, "email");
invokeSoon(() => bound(), 500);

// ✅ call/apply works as well
invokeSoon(() => sendMessage.apply(c, ["phone"]), 500);
```
---

## Interface Type Basics

### Type Alias:
#### (1) Type aliases allow us to give a type a name
```typescript
type StringOrNumber = string | number;

// this is the ONLY time you'll see a type on the RHS of assignment
type HasName = { name: string };

// 🚨 self-referencing types don't work! (we'll get there!)
type NumVal = 1 | 2 | 3 | NumArr; // 🚨 ERROR -> Type alias 'NumVal' circularly references itself.
type NumArr = NumVal[]; // 🚨 ERROR -> Type alias 'NumArr' circularly references itself.
```

## Interface
#### (2) Interfaces can extend from other interfaces
```typescript
export interface HasInternationalPhoneNumber extends HasPhoneNumber {
  countryCode: string;
}
```

#### (3) they can also be used to describe call signatures

```typescript
interface ContactMessenger1 {
  (contact: HasEmail | HasPhoneNumber, message: string): void;
}

type ContactMessenger2 = (
  contact: HasEmail | HasPhoneNumber,
  message: string
) => void;

// NOTE: we don't need type annotations for contact or message
const emailer: ContactMessenger1 = (_contact, _message) => {
  /** ... */
};
```

#### (4) construct signatures can be described as well

```typescript
interface ContactConstructor {
  new (...args: any[]): HasEmail | HasPhoneNumber;
}
```

#### (5) index signatures describe how a type will respond to property access

```typescript
/**
 * @example
 * {
 *    iPhone: { areaCode: 123, num: 4567890 },
 *    home:   { areaCode: 123, num: 8904567 },
 * }
 */

interface PhoneNumberDict {
  // arr[0],  foo['myProp']
  [numberName: string]:
    | undefined
    | {
        areaCode: number;
        num: number;
      };
}

const phoneDict: PhoneNumberDict = {
  office: { areaCode: 321, num: 5551212 },
  home: { areaCode: 321, num: 5550010 } // try editing me
};
```