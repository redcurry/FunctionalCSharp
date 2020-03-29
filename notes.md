Use Parallel LINQ (PLINQ) to run expressions in parallel:

    // Sequential
    list.Select(ToSentenceCase).ToList();

    // Parallel
    list.AsParallel().Select(ToSentencCase).ToList();

Combine elements from "parallel" lists with Zip:

    Enumerable.Zip(
        new[] {1, 2, 3},
        new[] {"ichi", "ni", "san"},
        (number, name) => $"In Japanese, {number} is {name}");

Instead of using the IoC approach of injecting an interface
to provide a service, one could inject a function with less ceremony.

If you need to constrain the inputs of your functions,
it's usually better to define a custom type (e.g., Age),
including any necessary operators.

Model objects in a way that gives you fine control over the range of inputs
that your functions will need to handle (e.g., using custom types).

You may need to use a custom-defined `Unit` type instead of using `void`.

## Option

Use `Option<T>` instead of accepting `null` values.

You can adapt a dictionary to use `Option` with an extension method (`Lookup`):

    dictionary.Lookup("blue");  // returns an Option<T>

Create a "smart" constructor (static method that returns `Option<T>`)
when creating a custom type that performs validation:

    public static Option<Age> Of(int age) =>
        IsValid(age) ? Some(new Age(age)) : None;

Never write function that returns `null`.

Definition of `Map` for `Option`:

    public static Option<R> Map<T, R>(
        this Option<T> optT, Func<T, R> f) =>
        optT.Match(
            () => None,
            (t) => Some(f(t)));

Perform side effects with `ForEach` (after defining it for `Option`),
although the name is a bit counterintuitive:

    opt.Map(name => $"Hello {name}")
       .ForEach(WriteLine);

Bind (for `Option`) takes an `Option`-returning function and applies it
to what's inside the given `Option`:

    Option.Bind : (Option<T>, (T -> Option<R>)) -> Option<R>

Bind for `IEnumerable` is similar to `SelectMany`.

You can also define `Bind` such that it flattens a list of Options
into a list of the inner types. You would be able to say:

    var ages = Population.Bind(p => p.Age);
    // returns [ Age(33), Age(37) ] instead of
    // [ Some(Age(33)), None, Some(Age(37)) ] when using Map

Choosing between `Map` and `Bind`:
`Map` takes an A<T> and returns A<R>
by applying a normal function (T -> R).
`Bind` takes an A<T> and returns A<R>
by applying an upward-crossing function (T -> A<R>).

## Composition

Use method chaining via extension methods to achieve function composition.

Instead of writing imperatively, like:

    if (validator.IsValid(transfer))
        Book(transfer)

write functionally:

    Some(transfer)               // lift to Option
    .Where(validator.IsValid)    // filter within Option
    .ForEach(Book);              // process non-None values

## Either

Use `Left` to represent failure and `Right` to represent success.

Create an `Error` parent class to represent errors in `Left`,
and subclass `Error` for each kind of error in the domain. (6.3.1)

(Me: Consider renaming `Either` as `Result`.)

Use `Bind` for `Either` to chain together operations,
any of which that may fail.

At the edge of an onion architecture, use `Match`
to expose the result (error or data) that the outer layer expects.

`Either`s with different error types can be adapted into one,
but it's best to always use one kind of error type.

You can be more expressive by creating specialized versions of `Either`,
such as `Validation` and `Exceptional`, and combining them to generate
something like `Validation<Exceptional<T>>`.

## Partial application

Use custom `Apply` to create partial application:

    public static Func<T2, R> Apply<T1, T2, R>(
        this Func<T1, T2, R> f, T1 t1) =>
        t2 => f(t1, t2);

    // Note: doesn't work with a method (must be a Func)
    Func<string, string, string> greet =
        (gr, name) => $"{gr}, {name}";

    var greetInformally = greet.Apply("Hey");

Example above uses a field (`greet`), but it could also be a property
or a method that returns `Func`. The advantage of using a method
is that it can be generic if necessary.

You can also convert any function to one ready for partial application
by currying:

    public static Func<T1, Func<T2, R>> Curry<T1, T2, R>(
        this Func<T1, T2, R> f) =>
        t1 => t2 => f(t1, t2);

    var greetWith = greet.Curry();             // gets it ready for currying
    var greetInformally = greenWith("Hey");

## Dependency injection

Instead of inject an interface, you could inject a function,
and use partial application to store it inside another function:

    public static Validator<BookTransfer> DateNotPast(Func<DateTime> clock)
        => cmd                           // returns function that takes a cmd
        => cmd.Date.Date < clock().Date  // calls the dependency
           ? Errors.TransferDateIsPast
           : Valid(cmd);

This means that you need to inject each function that's needed,
as opposed to an interface that may declare several methods.
But the functional approach adheres closer to "interface segregation".

## Aggregation (fold)

In the case of validation, you may want to take a list of validators
and combine them (using `Aggregate`) to get a single validator. (7.6.2)

You could combine them in various ways:
fail fast to return the first validation that fails,
or havrest errors to keep a list of validation failures.

## Misc

To attach some meaning to a type, like `string`, use:

    using Name = System.String;
    using Greeting = System.String;

Define custom types using implicit operators to make transformations easy.
(7.4.1)


