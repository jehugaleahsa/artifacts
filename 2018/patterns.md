## Intialization
### Limited scope and one-time use
Variables should always be declared as close to where they're used as possible. Not reusing variables for different purposes will help.

### Conditional Initialization
Instead of:

```csharp
int i;
if (condition) 
{
    i = 1;
}
else 
{
    i = 2;
}
```

Prefer:

```csharp
int i = condition ? 1 : 2;
```

Or:

```csharp
int i = GetInitialValue(condition);
// ...
int GetInitialValue(bool condition)
{
    if (condition)
    {
        return 1;
    }
    else
    {
        return 2;
    }
}
```

### Case Initialization
If there are several initial values, instead of:

```csharp
string day;
switch (dayOfWeek)
{
    case DayOfWeek.Sunday: day = "Sunday";
    case DayOfWeek.Monday: day = "Monday";
    case DayOfWeek.Tuesday: day = "Tuesday";
    case DayOfWeek.Wednesday: day = "Wednesday";
    // ...
    case DayOfWeek.Saturday: day = "Saturday";
    default: throw new ArgumentException("Unknown day of week.", nameof(dayOfWeek));
}
```

Prefer:

```csharp
string day = ToString(dayOfWeek);
// ...
string ToString(DayOfWeek dayOfWeek)
{
    switch (dayOfWeek)
    {
        case DayOfWeek.Sunday: return "Sunday";
        case DayOfWeek.Monday: return "Monday";
        case DayOfWeek.Tuesday: return "Tuesday";
        case DayOfWeek.Wednesday: return "Wednesday";
        // ...
        case DayOfWeek.Saturday: return "Saturday";
        default: throw new ArgumentException("Unknown day of week.", nameof(dayOfWeek));
    }
}
```


### Conditional Augmentation
When initialization involves duplicated code, instead of:

```csharp
decimal rate = customer.IsPreferred ? customer.DiscountRate + 10 : customer.DiscountRate;
```

Prefer:

```csharp
decimal rate = customer.DiscountRate;
if (customer.IsPreferred)
{
    rate += 10;
}
```

Or, if you prefer immutability:

```csharp
decimal rate = customer.DiscountRate + (customer.IsPreferred ? 10 : 0);
```

Or, provide a more descriptive name:

```csharp
decimal rate = customer.DiscountRate + GetPreferredRate(customer);
// ...
decimal GetPreferredRate(Customer customer)
{
    return customer.IsPreferred ? 10 : 0;
}
```

## Comparison

## Iteration
## Going Backwards
You can go backward through an array like this:

```csharp
for (int i = items.Length - 1; i != -1; --i)
{
    // ...
}
```

This works fine for `int` since `-1` is a valid value. Here's an equivalent loop that doesn't assume `-1` is possible:

```csharp
for (int i = items.Length; i != 0; )
{
    --i;
    // ...
}
```

## Meeting in the middle
You can move two indexes from the start and end of a list and stop in the middle like this:

```csharp
for (int start = 0, end = items.Length; start != end; )
{
    --end;
    // ...
}
```

It doesn't matter if the array has an even or odd number of items.

## Methods
### Use guard clauses
