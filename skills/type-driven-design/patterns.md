# Type-Driven Design — Pattern Catalog

Each pattern below is shown in multiple languages. The language names are labels, not recommendations — pick the version that matches the code you are actually writing. The mental model is the same across all of them.

---

## 1. Domain Primitives (newtype / branded / value object)

**Problem:** A raw `String` or `Int` carries an invariant the compiler cannot see (`"must be a valid email"`, `"must be positive"`, `"must not be empty"`). Every function that touches the value is then either trusting an unwritten contract or re-validating.

**Solution:** Wrap the primitive in a dedicated type whose only legal constructor enforces the invariant. Once you hold the type, the invariant is proven.

### TypeScript — branded types

```typescript
type Email = string & { readonly __brand: unique symbol };

function parseEmail(raw: string): Email | null {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(raw) ? (raw as Email) : null;
}

// Downstream:
function sendWelcome(to: Email): void { /* ... */ }
// sendWelcome("not an email"); // ❌ compile error
```

### Rust — newtype

```rust
pub struct Email(String);

impl Email {
    pub fn parse(raw: &str) -> Option<Self> {
        if raw.contains('@') { Some(Email(raw.to_string())) } else { None }
    }
    pub fn as_str(&self) -> &str { &self.0 }
}
```

### Swift — failable initializer

```swift
struct Email {
    let rawValue: String
    init?(_ rawValue: String) {
        guard rawValue.isValidEmail else { return nil }
        self.rawValue = rawValue
    }
}
```

### C# — record struct with `Result`

```csharp
public readonly record struct Email {
    public string Value { get; }
    private Email(string value) { Value = value; }
    public static Result<Email> Create(string raw) =>
        raw.Contains('@')
            ? Result<Email>.Success(new Email(raw))
            : Result<Email>.Failure("invalid email");
}
```

### Haskell — smart constructor with hidden data constructor

```haskell
newtype Email = UnsafeEmail Text  -- hide this constructor at module boundary

mkEmail :: Text -> Maybe Email
mkEmail t | "@" `isInfixOf` t = Just (UnsafeEmail t)
          | otherwise         = Nothing
```

**Heuristic for when to wrap:** only when the value carries an invariant or crosses an API boundary. Don't wrap a string that is rendered once and never compared.

---

## 2. Sum Types (discriminated unions / ADTs)

**Problem:** Your code says "exactly one of these shapes" (a comment, a `Bool` flag gating other fields, or co-dependent optionals). The type admits combinations the logic rejects.

**Solution:** Use a sum type. Every legal shape becomes a named variant; illegal shapes cannot be constructed.

### TypeScript — discriminated union

```typescript
type PaymentMethod =
  | { kind: "card"; number: CardNumber; cvv: Cvv }
  | { kind: "giftCard"; code: GiftCode }
  | { kind: "bankTransfer"; iban: Iban };

function charge(method: PaymentMethod): void {
  switch (method.kind) {
    case "card":         return chargeCard(method.number, method.cvv);
    case "giftCard":     return redeemGift(method.code);
    case "bankTransfer": return initiateTransfer(method.iban);
    // Exhaustive: missing a case is a compile error with `strict` + `noImplicitReturns`.
  }
}
```

### Rust — enum with payload

```rust
enum PaymentMethod {
    Card { number: CardNumber, cvv: Cvv },
    GiftCard(GiftCode),
    BankTransfer(Iban),
}
```

### Swift — enum with associated values

```swift
enum PaymentMethod {
    case card(number: CardNumber, cvv: Cvv)
    case giftCard(GiftCode)
    case bankTransfer(Iban)
}
```

### Kotlin — sealed class

```kotlin
sealed interface PaymentMethod {
    data class Card(val number: CardNumber, val cvv: Cvv) : PaymentMethod
    data class GiftCard(val code: GiftCode) : PaymentMethod
    data class BankTransfer(val iban: Iban) : PaymentMethod
}
```

### Haskell — data with multiple constructors

```haskell
data PaymentMethod
  = Card CardNumber Cvv
  | GiftCard GiftCode
  | BankTransfer Iban
```

### C# — discriminated union via sealed interface (C# 9+)

```csharp
public interface IPaymentMethod {}
public record Card(CardNumber Number, Cvv Cvv) : IPaymentMethod;
public record GiftCard(GiftCode Code) : IPaymentMethod;
public record BankTransfer(Iban Iban) : IPaymentMethod;

string Describe(IPaymentMethod m) => m switch {
    Card c         => $"card ending {c.Number.Last4}",
    GiftCard g     => $"gift {g.Code}",
    BankTransfer b => $"bank {b.Iban}",
    _              => throw new InvalidOperationException("unreachable"),
};
```

---

## 3. Product Types (records / structs)

**Problem:** You need "all of these together".

**Solution:** Record / struct / data class / case class. This is the uncontroversial half of algebraic data types — most languages do it well. The lever is *what* goes in the struct:

- Prefer **domain primitives** over raw strings and ints.
- Prefer **sum types** over fields that are co-dependent.
- Prefer **non-null fields with meaningful types** over nullable fields with boolean flags.

```rust
struct Credentials {
    email: Email,         // not String
    password: Password,   // not String
}
```

---

## 4. Make Illegal States Unrepresentable

**Problem:** A type permits combinations the business logic rejects. This is the "algebraic distribution" diagnostic from `SKILL.md` applied in anger.

**Solution:** Expand the product across the sum until every shape in the type is legal.

### Swift — ContactMethod

```swift
// ❌ Admits (nil, nil) and (some, some) — both illegal
struct ContactMethod {
    let email: Email?
    let phone: PhoneNumber?
}

// ✅ Only legal shapes exist
enum ContactMethod {
    case email(Email)
    case phone(PhoneNumber)
    case both(email: Email, phone: PhoneNumber)
}
```

### TypeScript — subscription status

```typescript
// ❌ premiumExpiry may be set with isPremium=false, or vice versa
interface User {
  id: UserId;
  isPremium: boolean;
  premiumExpiry: Date | null;
  premiumTier: "silver" | "gold" | null;
}

// ✅ One shape per legal state
type User = { id: UserId } & (
  | { subscription: "free" }
  | { subscription: "premium"; tier: "silver" | "gold"; expiresAt: Date }
);
```

### Rust — API response

```rust
// ❌ Admits (Some, Some) which should be impossible
struct ApiResponse<T> {
    data: Option<T>,
    error: Option<String>,
}

// ✅
enum ApiResponse<T> {
    Ok(T),
    Err(String),
}
```

---

## 5. Option / Maybe over null

**Problem:** `null` / `undefined` makes every value of type `T` secretly a value of type `T | absence`, and the compiler does not force you to check.

**Solution:** Encode absence in the type. The compiler then forces you to handle it at every access.

### Rust

```rust
fn find_user(id: UserId) -> Option<User> { /* ... */ }

match find_user(id) {
    Some(user) => greet(user),
    None => redirect_to_signup(),
}
```

### Swift — Optional + guard/if-let

```swift
func findUser(_ id: UserID) -> User?

if let user = findUser(id) { greet(user) }
else { redirectToSignup() }
```

### Haskell — Maybe

```haskell
findUser :: UserId -> IO (Maybe User)

case user of
  Just u  -> greet u
  Nothing -> redirectToSignup
```

### TypeScript — explicit `T | null` with `strictNullChecks`

```typescript
function findUser(id: UserId): User | null { /* ... */ }

const user = findUser(id);
if (user === null) { redirectToSignup(); return; }
greet(user);
```

**Antipattern:** returning `null` or throwing when both "no result" and "lookup failed" are possible. Use `Result<Option<T>, Error>` instead of overloading `null`.

---

## 6. Result / Either over exceptions

**Problem:** A function's signature says it returns `User`, but it can also throw. The failure path is invisible to the caller and to the compiler.

**Solution:** Return a sum type whose variants are "success with value" and "failure with error". The compiler then forces the caller to consider both.

### Rust

```rust
enum CreateUserError { InvalidEmail, EmailTaken, Database(DbError) }

fn create_user(req: CreateUserRequest) -> Result<User, CreateUserError> { /* ... */ }

match create_user(req) {
    Ok(user) => respond_created(user),
    Err(CreateUserError::EmailTaken) => respond_409(),
    Err(e) => respond_500(e),
}
```

### Swift

```swift
enum CreateUserError: Error { case invalidEmail, emailTaken, database(Error) }

func createUser(_ req: CreateUserRequest) async -> Result<User, CreateUserError>
```

### C# — Codespoti's Result pattern

```csharp
public readonly record struct Result<TValue> {
    public TValue? Value { get; }
    public string? Error { get; }
    public bool IsSuccess { get; }
    public static Result<TValue> Success(TValue v) => new(v, null, true);
    public static Result<TValue> Failure(string e) => new(default, e, false);
    private Result(TValue? v, string? e, bool ok) { Value = v; Error = e; IsSuccess = ok; }
}
```

### Haskell — Either

```haskell
createUser :: CreateUserRequest -> IO (Either CreateUserError User)
```

**Rule of thumb:** *Exceptions should be reserved for truly exceptional cases.* Invalid email during signup is not exceptional. Network timeout on a retry is borderline. A null pointer deep in a library you imported is exceptional.

---

## 7. Parse, Don't Validate

**Problem:** A validation function returns `bool` (or `void` and throws) and discards the proof that validation happened. Every downstream function either re-validates or relies on a comment.

**Solution:** Replace the validator with a **parser**: a function that consumes less-structured input and produces more-structured output, preserving the proof in the type.

### Haskell — Alexis King's NonEmpty example

```haskell
-- Validation: discards information
validateNonEmpty :: [a] -> IO ()
validateNonEmpty (_:_) = pure ()
validateNonEmpty [] = throwIO (userError "list cannot be empty")

-- Parsing: preserves the proof
parseNonEmpty :: [a] -> IO (NonEmpty a)
parseNonEmpty (x:xs) = pure (x :| xs)
parseNonEmpty [] = throwIO (userError "list cannot be empty")

-- Consumers now see `NonEmpty a`, not `[a]`, so no re-check is needed.
```

### TypeScript — parseEmail returns a branded type

```typescript
type Email = string & { readonly __brand: unique symbol };

function parseEmail(raw: string): Email | null {
  return isValidEmail(raw) ? (raw as Email) : null;
}

// Downstream functions take Email, not string.
function sendWelcome(to: Email) { /* ... */ }
```

### Swift — smart constructor on a struct

```swift
struct Password {
    let rawValue: String
    init?(_ rawValue: String) {
        guard rawValue.count >= 8 else { return nil }
        self.rawValue = rawValue
    }
}
```

**The aphorism:** *Write functions on the data representation you wish you had, not the data representation you are given.*

---

## 8. Phantom Types & Typed Identities

**Problem:** Two IDs of different entities have the same underlying type (`String` / `Int64` / `UUID`). Nothing prevents passing a `UserId` where an `OrderId` is expected.

**Solution:** Tag the ID with a phantom type parameter. The tag has no runtime cost but is different for each entity.

### Rust — PhantomData

```rust
use std::marker::PhantomData;

struct Id<T> {
    raw: u64,
    _phantom: PhantomData<T>,
}

struct User;
struct Order;

fn ship(order: Id<Order>) { /* ... */ }

let uid: Id<User>  = Id { raw: 1, _phantom: PhantomData };
// ship(uid); // ❌ compile error
```

### Swift — generic ID type

```swift
struct ID<Owner>: Hashable {
    let rawValue: String
}

enum User {}
enum Song {}

func play(_ song: ID<Song>) { /* ... */ }
let userID: ID<User> = ID(rawValue: "u_123")
// play(userID) // ❌ compile error
```

### TypeScript — branded generic

```typescript
type Id<Owner> = string & { readonly __owner: Owner };

declare const user: Id<"User">;
declare const order: Id<"Order">;

function ship(o: Id<"Order">) { /* ... */ }
ship(user); // ❌ compile error
```

### Haskell — phantom type parameter

```haskell
newtype Id a = Id Int64
data User
data Order

ship :: Id Order -> IO ()
```

**Extension — Ghosts of Departed Proofs.** Matt Noonan's GDP pattern generalises this: use phantom types to carry *proofs* (e.g. "this list is sorted", "this resource is initialized") through the type system with zero runtime cost. The proof vanishes at runtime, leaving only the type-level shadow that gates which operations are available.

---

## 9. Typestate (state machines in types)

**Problem:** An object has distinct states (`Closed`, `Open`, `Authenticated`, `Disposed`). Some methods are only legal in some states. Calling the wrong method in the wrong state is a runtime error.

**Solution:** Encode the state as a type parameter. Methods that change the state consume `self` and return a value of the new type. The wrong transition is a compile error.

### Rust — Connection typestate

```rust
use std::marker::PhantomData;

struct Closed;
struct Open;
struct Authenticated;

struct Connection<State> {
    socket: TcpStream,
    _state: PhantomData<State>,
}

impl Connection<Closed> {
    fn open(self) -> Connection<Open> { /* ... */ }
}

impl Connection<Open> {
    fn authenticate(self, creds: Credentials) -> Result<Connection<Authenticated>, Self> { /* ... */ }
}

impl Connection<Authenticated> {
    fn send(&mut self, msg: Message) { /* ... */ }
}

// conn.send(msg); // ❌ compile error unless conn is Connection<Authenticated>
```

### TypeScript — phantom state via branding

```typescript
type Closed = { readonly __state: "closed" };
type Open = { readonly __state: "open" };
type Authed = { readonly __state: "authed" };

type Connection<S> = { socket: Socket } & S;

declare function open(c: Connection<Closed>): Connection<Open>;
declare function auth(c: Connection<Open>, creds: Credentials): Connection<Authed>;
declare function send(c: Connection<Authed>, msg: Message): void;
```

### Swift — generic phantom type

```swift
enum Closed {}
enum Open {}
enum Authenticated {}

struct Connection<State> {
    private let socket: Socket
    fileprivate init(_ socket: Socket) { self.socket = socket }
}

extension Connection where State == Closed {
    func open() -> Connection<Open> { /* ... */ }
}
extension Connection where State == Open {
    func authenticate(_ creds: Credentials) -> Connection<Authenticated>? { /* ... */ }
}
extension Connection where State == Authenticated {
    func send(_ msg: Message) { /* ... */ }
}
```

---

## 10. NonEmpty and Refined Collections

**Problem:** A function requires a list / string / set to have at least one element, and checks it at every entry point.

**Solution:** Use a type that structurally guarantees non-emptiness. No more `if xs.is_empty()`.

### Haskell — `Data.List.NonEmpty`

```haskell
data NonEmpty a = a :| [a]

send :: NonEmpty Event -> IO ()  -- no isEmpty check needed
```

### Rust — `nonempty` crate or hand-rolled

```rust
struct NonEmpty<T> {
    head: T,
    tail: Vec<T>,
}

impl<T> NonEmpty<T> {
    fn new(head: T, tail: Vec<T>) -> Self { Self { head, tail } }
    fn first(&self) -> &T { &self.head }
}
```

### Swift — hand-rolled NonEmptyArray

```swift
struct NonEmptyArray<Element> {
    let first: Element
    let rest: [Element]
    var all: [Element] { [first] + rest }
}
```

**Similar refinements worth knowing:** `PositiveInt`, `NonEmptyString`, `SortedList<T>` (GDP-style), `UniqueList<T>`.

---

## 11. Parse at the Boundary

**Problem:** External data (JSON, database rows, URL parameters, deep links, cross-module calls) enters your program as raw strings / maps / dynamic values. If you let it flow untyped into business logic, every downstream function is a potential parser.

**Solution:** Parse once at the boundary into your domain model. Reject illegal combinations at that single point. Everything downstream operates on parsed types.

### Swift — hand-written Decodable that rejects illegal combinations

```swift
enum PaymentMethod: Decodable {
    case creditCard(CreditCard)
    case giftCard(GiftCard)

    init(from decoder: Decoder) throws {
        let c = try decoder.container(keyedBy: CodingKeys.self)
        let card = try c.decodeIfPresent(CreditCard.self, forKey: .creditCard)
        let gift = try c.decodeIfPresent(GiftCard.self, forKey: .giftCard)
        switch (card, gift) {
        case let (c?, nil): self = .creditCard(c)
        case let (nil, g?): self = .giftCard(g)
        case (.some, .some), (nil, nil):
            throw DecodingError.dataCorruptedError(
                forKey: .creditCard, in: c,
                debugDescription: "exactly one payment method required"
            )
        }
    }
}
```

### TypeScript — zod (or equivalent) at the network boundary

```typescript
import { z } from "zod";

const PaymentMethodSchema = z.discriminatedUnion("kind", [
  z.object({ kind: z.literal("card"),     number: CardNumberSchema, cvv: CvvSchema }),
  z.object({ kind: z.literal("giftCard"), code: GiftCodeSchema }),
]);

type PaymentMethod = z.infer<typeof PaymentMethodSchema>;

async function fetchOrder(id: OrderId): Promise<Order> {
  const raw = await http.get(`/orders/${id}`);
  return OrderSchema.parse(raw); // throws on illegal shape; returns typed Order on success
}
```

### Rust — `serde` with a custom deserializer

```rust
#[derive(Deserialize)]
#[serde(tag = "kind", rename_all = "camelCase")]
enum PaymentMethod {
    Card { number: CardNumber, cvv: Cvv },
    GiftCard { code: GiftCode },
}
```

**The aphorism:** *Push the burden of proof upward as far as possible, but no further.* The outer layer of your program is where you pay the validation tax; every layer below runs on parsed types.

---

## Cross-cutting summary

| Pattern | Languages where it's idiomatic |
|---|---|
| Domain primitives | TS, Rust, Swift, C#, Haskell, Kotlin, Scala |
| Sum types | Haskell, Rust, Swift, Kotlin, Scala, TS, F#, C# (9+) |
| Option / Maybe | Rust, Swift, Haskell, Scala, Kotlin, TS (`T \| null` + strict) |
| Result / Either | Rust, Swift, Haskell, Scala, F#, C# (with library), TS (with library) |
| Parse don't validate | Universal |
| Phantom types | Haskell, Rust, Swift, TS, Scala |
| Typestate | Rust (strongest), Swift, TS, Scala |
| NonEmpty | Haskell, Rust, Swift, Scala |
| Parse at the boundary | Universal |

If your language is not on a row above, the pattern probably still exists — it's just awkward. Evaluate the cost/benefit honestly before forcing it.
