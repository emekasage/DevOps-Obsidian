Creation date: Sunday, March 8th 2026, 8:43:58 am

**Goal**: Learn to organize code with functions and store lists of data in arrays.

#### **Functions**

```
greet() {
    echo "Hello, $1!"
}

greet "World"
# Hello, World!
```

##### Function Arguments

Functions use the same special variables as scripts:

```
show_info() {
    echo "First arg: $1"
    echo "All args: $@"
    echo "Number of args: $#"
}

show_info one two three
```

---
#### **Local Variables**

By default, variables are global:

```
my_function() {
    name="Changed"
}

name="Original"
my_function
echo "$name"
# Changed (oops!)
```

The function changed our variable! Use `local` to keep variables inside the function:

```
my_function() {
    local name="Changed"
    echo "Inside: $name"
}

name="Original"
my_function
echo "Outside: $name"
# Still "Original"
```

**Rule**: Always use `local` for variables inside functions.

---
#### **Return Values**

##### Exit Codes

Functions return exit codes just like commands:

```
is_even() {
    local num="$1"
    (( num % 2 == 0 ))
}

if is_even 4; then
    echo "4 is even"
fi
```

##### Returning Data

To return actual data, use `echo` and capture with `$()`:

```
get_extension() {
    local filename="$1"
    echo "${filename##*.}"
}

ext=$(get_extension "document.txt")
echo "$ext"
# txt
```

---
#### **Arrays**

Arrays store lists of values:

```
fruits=("apple" "banana" "orange")

# Single element (zero-indexed)
echo "${fruits[0]}"
# apple

# All elements
echo "${fruits[@]}"
# apple banana orange

# Length
echo "${#fruits[@]}"
# 3

# Add element
fruits+=("grape")
```

#### Looping Over Arrays

```
for fruit in "${fruits[@]}"; do
    echo "I like $fruit"
done
```

Use `"${arr[@]}"` (with quotes) to handle elements with spaces correctly.

---
#### **Putting It Together**

Here’s a function that works with an array:

```
print_numbered() {
    local items=("$@")
    local i=1
    for item in "${items[@]}"; do
        echo "$i. $item"
        ((i++))
    done
}

tasks=("Buy milk" "Walk dog" "Write code")
print_numbered "${tasks[@]}"
# 1. Buy milk
# 2. Walk dog
# 3. Write code
```

---
***Previous***: [[Loops]]                                                                ***Next***: [[Unix Filters and Editor Integration]]


