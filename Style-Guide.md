This style guide is currently under development. In time, and with input from Laurent, the goal is for this to become SFML's official style guide which contributors to SFML may use to know what programming style they must adopt in order for their code to be homogenous with SFML's code.

# Naming Conventions

## Camel Case

SFML uses CamelCase, where identifiers are formed from combining words, and the individually distinguished by their capitalization.

### `struct`s and `class`es

Capitalize the first letter of the first word, and for every word thereafter, capitalize the first letter of the word also.
**Example:**
```cpp
class InputStream;
class Event;
struct KeyEvent;
```

### Member Functions

Member functions have the first letter of the first word lower case. This applies to both static and non-static member functions.
**Example:**
```cpp

```

### Non-member Functions

Non-member functions follow the same conventions as member functions: the first letter of the first word is lower case. This applies to both publicly accessible non-member functions, as well as functions within an unnamed namespace.
**Example:**
```cpp

```

### Enumerations

The enumeration type name is capitalized just like `struct`s and `class`es are. In some cases, the enumeration may be unnamed/anonymous. The values in the enumerator list are also capitalized just like `struct`s and `class`es.
**Example:**
```cpp
enum
{
    Count       = 8,
    ButtonCount = 32,
    AxisCount   = 8
};

enum Axis
{
    X,
    Y,
    Z,
    R,
    U,
    V,
    PovX,
    PovY
};
```

### Acronyms and Abbreviations

Acronyms and Abbreviations are treated as a single word, and as such all letters are lower case (except the first letter, which may be upper case or lower case depending on the identifier using the acronym or abbreviation). Example:
```cpp
enum Axis
{
    // ... from above
    PovX, // Pov, not POV
    PovY
};

class TcpSocket; // Tcp, not TCP
class IpAddress; // Ip, not IP
```

## Non-camel Case

### Macros

Macros are written ALL_CAPS with words separated by underscores.
**Example:**
```cpp
#ifndef SFML_CONFIG_HPP
#define SFML_CONFIG_HPP

#define SFML_VERSION_MAJOR 2
#define SFML_VERSION_MINOR 0

#endif
```

### Namespaces

Namespaces are lower case and are as short as possible while still being obvious what they stand for.

**Example:**
```cpp
namespace sf // Namespace containing all of SFML
{
namespace priv // Namespace containing private implementation functionality for SFML
{
}
}
```

# Comments

## Doxygen

## In-code Documentation

# Indentation and Bracing

## Bracing

Braces are always on their own line.

## Indentation

### Spaces, not Tabs

SFML uses 4 spaces for indentation, no tabs.

### General Indentation

### Aligning Statements and Expressions

Align statements and expressions that form logical groups.
**Example:**
```cpp
enum
{
    Count       = 8, // Aligned at =
    ButtonCount = 32,
    AxisCount   = 8
};
```

### Inside of a Namespace

### Inside of a Macro `#if`/`#elif`/`#else#`/`#endif`

### Switches

### Access Modifiers

Access modifiers (`public`, `protected`, and `private`) are dedented. *Note that there is a space between the access modifier and the colon*.
**Example:**
```cpp

```

# Misc

## Trailing Whitespace

Remove trailing whitespace. This includes whitespace appearing on a blank line.