# Zerofy's Structured Data Language
ZSDL aims to be a readable and efficient language for structured data, such as configuration files, offering greater 
flexibility by introducing data types missing in other languages and new ones as well.

## Example
```zsdl
# This is a ZSDL file.

= Pancakes ===================================  # Start of section.
desc : 'Simple, delicious pancakes!'
servings : 4
total_time : '20 min'

ingredients : List  # start of list
  # Indentation is allowed, but not required.
  - '1 cup flour'
  - '2 tablespoons sugar'
  - '2 teaspoons baking powder'
  - '0.5 teaspoon salt'
  - '1 cup milk'
  - '2 tablespoons oil'
  - '1 beaten egg'
---------------------------------  # end of list

toppings : Dict
  recommended : List | 'cream', 'chocolate', 'banana'  # in-line list
  optional : List
    - 'honey'
    - 'strawberries'
    - 'peanut butter'
    ------------------
  usual: None
------------------------------------------------------  # end of dict
```

## Table of Contents
- [Starting Notes](#starting-notes)
- [Comments & Whitespaces](#comments--whitespaces)
- [Sections & Key-Value Pairs](#sections--key-value-pairs)
- [Supported Data Types](#supported-data-types)
- [Strings](#strings)
- [Numbers](#numbers)
- [Booleans](#booleans)
- [None Types](#none-types)
- [Lists & Tuples](#lists--tuples)
- [Dictionaries](#dictionaries)
- [Nesting Rules](#nesting-rules)
- [References](#references)
- [Executables](#executables)

## Starting Notes
This document explains the syntax rules of ZSDL and provides examples for all the supported data types.

---

**Preliminaries:**
- ZSDL is case-sensitive.
- Indentation (tabs and spaces) are allowed, but not required.
- Parsing a ZSDL file results in a dictionary of dictionaries (sections) containing key-value pairs.

## Comments & Whitespaces
Comments are marked with a hash symbol, making the parser ignore everything until the end of the line. There are no
multi-line comments. Hash symbols within strings do not start comments.

```zsdl
# This is a comment.
ex1 : 'value'  # This is also a comment.
ex2 : '# I am not a comment!'
```

Tabs and spaces are ignored, unless inside strings. This is to allow for indentations without making them mandatory.

## Sections & Key-Value Pairs
All data within a ZSDL document is organized in sections and key-value pairs.

### Sections
The main dictionary created when parsing a ZSDL file contains sub-dictionaries called sections. Each of these sections
holds data in key-value pairs.

**Defining a Section:**
- Sections are declared using equal signs ( `=` ) surrounding the section name.
- There must be at least one `=` before and after the section name.
- The section name must be separated from the equal signs by at least one space ( ` ` ) on each side.
- All sections must have a unique name, which cannot be empty.
- Section names can contain letters ( `abc...ABC...` ), numbers ( `123...` ), underscores ( `_` ), and hyphens ( `-` ).
  Spaces ( ` ` ) are also allowed, but not at the beginning or end of the section name.
  - Sections can be declared as single-line strings ( `'sect name'` or `"sect name"` ) to allow for special characters.
- There must be a new line or EOF character after defining a section.

**❗ Internal Representation of Section Names:**
Regardless if the section name is declared as a string or not, the parser interprets it as a string and that is also how
they are stored internally within the base dictionary. The parser ignores whitespaces around the equal signs, unless the
section name is represented as a string. For non-string section names, the parser only sees allowed characters, ignoring
whitespaces at the beginning and end of the section name, and raising an exception for other illegal characters. 
Non-string sections with only numbers in the name are also stored as strings. 

---

**✅ Valid Section Examples:**
```zsdl
= Simple Section =
=== Advanced Settings ===
= User Preferences ======
======== Offset Section =
= "String Defined Section" =
# This is actually a valid ZSDL document since sections don't have to contain any data.
```

---

**❌ Invalid Section Examples:**
```zsdl
=Sect One =    # Missing space before the section name.
= Sect Two=    # Missing space after the section name.
=Sect Three=   # Missing spaces before and after the section name.

Sect Four =    # Missing equals sign before the section name.
= Sect Five    # Missing equals sign after the section name.

= Sect. Six =  # Contains special characters without being represented as a string.

= '''Sect. Seven''' =  # Multi-line string representation is not allowed.

== Sect Eight A == = Sect Eight B =  # No new line or EOF character after section definition.

= Original Name =
= Original Name =  # Section name isn't unique.
```

### Keys & Values
Within each section, data is stored as key-value pairs.

**Defining a Key-Value Pair:**
- Key-value pairs are declared using a colon ( `:` ) separating the key (on the left) and value (on the right).
- Empty keys are not allowed. All keys must have a value, even if it's `None`.
- Keys must be unique within their section or dictionary, but can be repeated across different sections or dictionaries.
- Keys can contain letters ( `abc...ABC...` ), numbers ( `123...` ), underscores ( `_` ), and hyphens ( `-` ).
  - Keys can be declared as single-line strings ( `'key name'` or `"key name"` ) to allow for special characters.
- There must be a new line or EOF character after defining a key-value pair, except for values that can or must be
  represented over multiple lines.

**❗ Internal Representation of Keys:**
Regardless if the key is declared as a string or not, the parser interprets it as a string and that is also how they are
stored internally within sections. The parser ignores whitespaces around the colon, unless the key is represented as a
string. For non-string keys, the parser only sees allowed characters, ignoring whitespaces and raising an exception for
other illegal characters. Non-string, numerical keys are also stored as strings.

---

**✅ Valid Key-Value Examples**
```zsdl
keyone    : 'value'
key_two   :'value'
_key_three:'value'
key-four  : 'value'
-key-five : 'value'
'key six' : 'value'

key-one : 'value'
key--one : 'value'  # Key names are unique.
```

---

**❌ Invalid Key-Value Examples**
```zsdl
key one : 'value'  # Contains spaces without being represented as a string.
key.two : 'value'  # Contains special characters without being represented as a string.

key-three :        # Missing value.
: 'value'          # Missing key name.
'' : 'value'       # Empty key name.

'''key four''' : 'value'  # Multi-line string representation is not allowed.

key-five : 'value'  key-six : 'value'  # No new line or EOF character after section definition.

key-one : 'value'
key-one : 'value'  # Key name isn't unique.
```

## Supported Data Types
ZSDL supports the following data types, which are explored in-depth later in this document:
- **Strings**, both single and multi line.
- **Numbers**, integers, floats, binary, and hexadecimal.
- **Booleans**
- **None**
- **Lists**, both in-line and multi-line.
- **Tuples**, both in-line and multi-line.
- **Dictionaries**, multi-line only.
- **References**, custom data type that allows copying already defined values.
- **Executables**, custom data type that allows for writing and executing code.

## Strings
... continue here ...

## Numbers
## Booleans
## None Types
## Lists & Tuples
## Dictionaries
## Nesting Rules

## References
Coming soon...

The idea is to make it so values can be copied from other already defined values by specifying the path.

```zsdl
= ReferenceExample =
key1 : 'cats are cool'
key2 : @ReferenceExample.key1  # this key would get the value 'cats are cool'
                               # when being accessed or parsed, depending on
                               # how I choose to implement this feature.
```

## Executables
Coming soon...

The idea is to make a new data type that can define executable code.
Pairing this with References would allow for making dynamic values that change depending on other predefined values.

```zsdl
= ExecutableExample =
key1 : True
key2 : Exec
  return 'cats are cool' if @ExecutableExample.key1 else 'cats are not cool'
  # this key would get the value 'cats are cool' if key1 is True, otherwise
  # it would get the value 'cats are not cool' when being accessed or parsed
  # depending on how I choose to implement this feature.
```
