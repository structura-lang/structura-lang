# Structura Language Specification

# 1. Language Structure

Structura is a structured-data programming language.

A Structura program is a function. A function consists of inputs, variables, operations, and an output expression.

Structura has three fundamental constructs:

* **Values** represent data.
* **Expressions** evaluate to values.
* **Operations** modify local execution state or control flow.

Expressions use the following structure:

```json
{
  "type": "<expression-type>",
  "data": <expression-data>
}
```

Operations use the following structure:

```json
{
  "type": "<operation-type>",
  "arguments": <operation-arguments>
}
```

JSON is the reference serialization format for Structura.

---

# 2. Values

## 2.1 Supported Values

Structura supports the following value types:

* `null`
* boolean
* number
* string
* array
* object
* function

## 2.2 Null

`null` is a normal value.

It does not implicitly convert to another value or type.

## 2.3 Boolean

A boolean is either `true` or `false`.

## 2.4 Number

A number represents a numeric value.

The exact numeric representation and implementation limits are defined by the applicable Structura runtime.

## 2.5 String

A string is a sequence of characters.

## 2.6 Array

An array is an ordered sequence of values.

Array indices are zero-based.

For example:

```json
["a", "b", "c"]
```

contains:

```text
index 0 → "a"
index 1 → "b"
index 2 → "c"
```

Arrays are immutable.

## 2.7 Object

An object is a collection of string keys mapped to values.

Example:

```json
{
  "name": "Karl",
  "age": 25
}
```

Objects are immutable.

## 2.8 Function

A function is a first-class value.

A function can be:

* stored in a variable;
* passed as an input;
* returned from another function;
* contained in an array or object;
* obtained from the runtime;
* retrieved using a `get` expression;
* invoked using a `call` expression.

---

# 3. Functions

A function has the following structure:

```json
{
  "inputs": [],
  "variables": {},
  "operations": [],
  "output": {}
}
```

All four members are required.

## 3.1 Inputs

`inputs` is an array containing the names of the function's inputs.

Input names:

* are strings;
* are case-sensitive;
* must be unique;
* have no default values;
* are immutable.

Example:

```json
{
  "inputs": ["name", "age"],
  "variables": {},
  "operations": [],
  "output": {
    "type": "input",
    "data": "name"
  }
}
```

The runtime supplies the values of the inputs when the function is invoked.

A function may have zero inputs:

```json
{
  "inputs": [],
  "variables": {},
  "operations": [],
  "output": {
    "type": "literal",
    "data": null
  }
}
```

A function invocation with missing or unexpected inputs is a runtime error.

## 3.2 Variables

`variables` is an object mapping variable names to initialization expressions.

Every variable MUST have an expression as its initializer.

Example:

```json
{
  "variables": {
    "counter": {
      "type": "literal",
      "data": 0
    },
    "name": {
      "type": "input",
      "data": "user_name"
    }
  }
}
```

Variable names are case-sensitive.

A variable MUST be declared before it can be modified.

Variables are mutable bindings and may contain values of different types during execution.

## 3.3 Variable Initialization

Variable initializers are evaluated when a function invocation begins, before its operations are executed.

A variable initializer MUST NOT contain a `variable` expression.

For example, the following is invalid:

```json
{
  "variables": {
    "a": {
      "type": "variable",
      "data": "b"
    },
    "b": {
      "type": "literal",
      "data": 10
    }
  }
}
```

If a variable depends on another local variable, the dependency must instead be evaluated by an operation after initialization.

## 3.4 Operations

`operations` is an array containing operations.

Operations are executed sequentially.

Control-flow operations may cause later operations to be skipped.

## 3.5 Output

`output` is an expression.

When normal execution reaches the end of the operation sequence, the `output` expression is evaluated.

The resulting value is the function's return value.

If a `return` operation is executed, execution immediately transfers to evaluation of `output`.

Every function returns exactly one value.

Multiple logical values may be returned by placing them in an object or array.

## 3.6 Function Purity

Functions are pure.

A function MUST NOT:

* perform I/O;
* access the filesystem;
* access a network;
* access a database;
* modify external state;
* access mutable global state;
* access variables belonging to another function invocation;
* access the current time;
* access randomness.

All information required by a function must be supplied through its inputs or immutable runtime values.

## 3.7 Function Scope

A function can access only:

* its own variables;
* its own inputs;
* active loop values;
* immutable runtime values.

A function cannot access variables belonging to its caller.

Functions do not implicitly capture variables from their surrounding scope.

## 3.8 Argument Passing

Values are passed by value.

Structura has no pointer or reference semantics.

Because objects and arrays are immutable, passing them to a function cannot allow the called function to modify the caller's value.

## 3.9 Recursion

Functions may invoke themselves recursively.

---

# 4. Expressions

Expressions produce exactly one value.

Every expression has the following structure:

```json
{
  "type": "<expression-type>",
  "data": <expression-data>
}
```

The following expression types are defined:

* `literal`
* `variable`
* `input`
* `loop`
* `runtime`
* `get`
* `call`

## 4.1 Literal Expression

A `literal` expression evaluates directly to the value stored in `data`.

Syntax:

```json
{
  "type": "literal",
  "data": <value>
}
```

Examples:

```json
{
  "type": "literal",
  "data": 42
}
```

```json
{
  "type": "literal",
  "data": "hello"
}
```

```json
{
  "type": "literal",
  "data": [1, 2, 3]
}
```

```json
{
  "type": "literal",
  "data": {
    "name": "Karl"
  }
}
```

The contents of `data` are literal data and are not recursively interpreted as expressions.

### 4.1.1 Function Literal

A function definition is represented by a `literal` expression.

There is no separate `function` expression type.

Example:

```json
{
  "type": "literal",
  "data": {
    "inputs": ["x"],
    "variables": {},
    "operations": [],
    "output": {
      "type": "input",
      "data": "x"
    }
  }
}
```

The expression evaluates to a function value.

A function literal does not capture variables from its surrounding scope.

## 4.2 Variable Expression

A `variable` expression evaluates to the current value of a local variable.

Syntax:

```json
{
  "type": "variable",
  "data": "<variable-name>"
}
```

Example:

```json
{
  "type": "variable",
  "data": "result"
}
```

The referenced variable MUST be declared in the current function.

A `variable` expression MUST NOT be used in a variable initializer.

## 4.3 Input Expression

An `input` expression evaluates to the value of an input of the current function.

Syntax:

```json
{
  "type": "input",
  "data": "<input-name>"
}
```

Example:

```json
{
  "type": "input",
  "data": "name"
}
```

Inputs are immutable.

Referencing an input that does not exist is a runtime error.

## 4.4 Loop Expression

A `loop` expression accesses a value provided by the currently executing `for_each` loop.

Syntax:

```json
{
  "type": "loop",
  "data": "item"
}
```

or:

```json
{
  "type": "loop",
  "data": "index"
}
```

Every `for_each` loop provides both values:

* `item` — the current element;
* `index` — the zero-based index of the current element.

Both values are read-only.

A `loop` expression is valid only while the corresponding loop is executing.

When loops are nested, the innermost loop's `item` and `index` values are used.

Using a `loop` expression outside a valid loop context is a runtime error.

## 4.5 Runtime Expression

A `runtime` expression retrieves a value exposed by the Structura runtime.

Syntax:

```json
{
  "type": "runtime",
  "data": "<runtime-name>"
}
```

Example:

```json
{
  "type": "runtime",
  "data": "add"
}
```

The runtime may expose immutable values and pure functions.

Runtime-exposed values are immutable.

Runtime-exposed functions MUST obey Structura's function purity rules.

The names and behavior of runtime-provided values and functions are defined by the applicable runtime specification.

## 4.6 Get Expression

A `get` expression retrieves a value from an object or array.

Object syntax:

```json
{
  "type": "get",
  "data": {
    "from": <expression>,
    "path": <expression>
  }
}
```

Array syntax:

```json
{
  "type": "get",
  "data": {
    "from": <expression>,
    "index": <expression>
  }
}
```

`from` MUST be an expression.

Exactly one of `path` and `index` MUST be present.

`path` and `index` are mutually exclusive.

### 4.6.1 Object Get

Object access uses `path`.

`path` MUST be an expression that evaluates to an array of strings.

Example:

```json
{
  "type": "get",
  "data": {
    "from": {
      "type": "input",
      "data": "data"
    },
    "path": {
      "type": "literal",
      "data": ["user", "name"]
    }
  }
}
```

Given:

```json
{
  "user": {
    "name": "Karl"
  }
}
```

the expression evaluates to:

```text
"Karl"
```

Every path element MUST be a string.

Each path element identifies one object field.

Missing path elements are a runtime error.

Using `path` with a non-object value is a runtime error.

### 4.6.2 Array Get

Array access uses `index`.

`index` MUST be an expression that evaluates to an integer.

Example:

```json
{
  "type": "get",
  "data": {
    "from": {
      "type": "variable",
      "data": "items"
    },
    "index": {
      "type": "literal",
      "data": 2
    }
  }
}
```

The index MUST be:

* an integer;
* non-negative;
* within the array bounds.

An invalid index is a runtime error.

Using `index` with a non-array value is a runtime error.

## 4.7 Call Expression

A `call` expression invokes a function.

Syntax:

```json
{
  "type": "call",
  "data": {
    "function": <expression>,
    "inputs_from": {
      "<input-name>": <expression>
    }
  }
}
```

`function` MUST be an expression that evaluates to a function.

`inputs_from` maps function input names to expressions.

Each argument expression is evaluated to produce the corresponding input value.

Example:

```json
{
  "type": "call",
  "data": {
    "function": {
      "type": "runtime",
      "data": "add"
    },
    "inputs_from": {
      "a": {
        "type": "literal",
        "data": 10
      },
      "b": {
        "type": "literal",
        "data": 20
      }
    }
  }
}
```

Calling a value that is not a function is a runtime error.

The order in which argument expressions are evaluated is unspecified.

---

# 5. Operations

Operations modify local execution state or control flow.

Every operation has the following structure:

```json
{
  "type": "<operation-type>",
  "arguments": <operation-arguments>
}
```

The following operation types are defined:

* `store`
* `set`
* `append`
* `remove`
* `if`
* `for_each`
* `while`
* `break`
* `continue`
* `return`

## 5.1 Store Operation

The `store` operation evaluates an expression and assigns its result to a variable.

Syntax:

```json
{
  "type": "store",
  "arguments": {
    "value": <expression>,
    "target_var": <expression>
  }
}
```

`value` MUST be an expression.

`target_var` MUST be an expression that evaluates to a string.

The resulting string identifies the local variable to modify.

Example:

```json
{
  "type": "store",
  "arguments": {
    "value": {
      "type": "literal",
      "data": 42
    },
    "target_var": {
      "type": "literal",
      "data": "result"
    }
  }
}
```

The target variable MUST already be declared.

A non-string `target_var` result is a runtime error.

## 5.2 Set Operation

The `set` operation replaces a value inside an object or array and stores the resulting collection in a variable.

Object syntax:

```json
{
  "type": "set",
  "arguments": {
    "target_var": <expression>,
    "path": <expression>,
    "value": <expression>
  }
}
```

Array syntax:

```json
{
  "type": "set",
  "arguments": {
    "target_var": <expression>,
    "index": <expression>,
    "value": <expression>
  }
}
```

Exactly one of `path` and `index` MUST be present.

`target_var` MUST evaluate to the name of a declared variable.

### 5.2.1 Object Set

`path` MUST be an expression evaluating to an array of strings.

`value` MUST be an expression.

Example:

```json
{
  "type": "set",
  "arguments": {
    "target_var": {
      "type": "literal",
      "data": "user"
    },
    "path": {
      "type": "literal",
      "data": ["profile", "name"]
    },
    "value": {
      "type": "literal",
      "data": "Karl"
    }
  }
}
```

The target variable MUST contain an object.

The final field may be created if it does not exist.

Missing intermediate fields are a runtime error.

The original object remains unchanged. The resulting object replaces the value of the target variable.

### 5.2.2 Array Set

`index` MUST be an expression evaluating to an integer.

The target variable MUST contain an array.

The index MUST be:

* an integer;
* non-negative;
* within the existing array bounds.

`set` MUST NOT expand an array.

Example:

```json
{
  "type": "set",
  "arguments": {
    "target_var": {
      "type": "literal",
      "data": "items"
    },
    "index": {
      "type": "literal",
      "data": 2
    },
    "value": {
      "type": "literal",
      "data": "Karl"
    }
  }
}
```

The original array remains unchanged. The resulting array replaces the value of the target variable.

## 5.3 Append Operation

The `append` operation adds one value to the end of an array.

Syntax:

```json
{
  "type": "append",
  "arguments": {
    "target_var": <expression>,
    "value": <expression>
  }
}
```

`target_var` MUST evaluate to the name of a declared variable.

`value` MUST be an expression.

The target variable MUST contain an array.

Example:

```json
{
  "type": "append",
  "arguments": {
    "target_var": {
      "type": "literal",
      "data": "items"
    },
    "value": {
      "type": "literal",
      "data": "hello"
    }
  }
}
```

The resulting array contains the previous elements followed by the new value.

The original array remains unchanged.

## 5.4 Remove Operation

The `remove` operation removes a field from an object or an element from an array.

Object syntax:

```json
{
  "type": "remove",
  "arguments": {
    "target_var": <expression>,
    "path": <expression>
  }
}
```

Array syntax:

```json
{
  "type": "remove",
  "arguments": {
    "target_var": <expression>,
    "index": <expression>
  }
}
```

Exactly one of `path` and `index` MUST be present.

`target_var` MUST evaluate to the name of a declared variable.

### 5.4.1 Object Remove

`path` MUST be an expression evaluating to an array of strings.

The target variable MUST contain an object.

The field identified by the final path element is removed.

Missing intermediate path elements are a runtime error.

The final field MUST exist. Removing a nonexistent field is a runtime error.

Example:

```json
{
  "type": "remove",
  "arguments": {
    "target_var": {
      "type": "literal",
      "data": "user"
    },
    "path": {
      "type": "literal",
      "data": ["profile", "nickname"]
    }
  }
}
```

The original object remains unchanged. The resulting object replaces the value of the target variable.

### 5.4.2 Array Remove

`index` MUST be an expression evaluating to an integer.

The target variable MUST contain an array.

The index MUST be:

* an integer;
* non-negative;
* within the array bounds.

The element at the specified index is removed.

Elements after the removed element shift one position toward the beginning of the array.

Example:

```json
{
  "type": "remove",
  "arguments": {
    "target_var": {
      "type": "literal",
      "data": "items"
    },
    "index": {
      "type": "literal",
      "data": 2
    }
  }
}
```

The original array remains unchanged. The resulting array replaces the value of the target variable.

## 5.5 If Operation

The `if` operation conditionally executes an operation sequence.

Syntax:

```json
{
  "type": "if",
  "arguments": {
    "condition": <expression>,
    "then": [],
    "else": []
  }
}
```

`condition` MUST evaluate to a boolean.

If `condition` is `true`, `then` is executed.

If `condition` is `false`, `else` is executed.

Only the selected branch is executed.

The expressions and operations in the unselected branch are not evaluated.

The `else` sequence may be empty.

Nested `if` operations may be used to implement else-if chains.

## 5.6 For-Each Operation

The `for_each` operation executes an operation sequence once for every element of a collection.

Syntax:

```json
{
  "type": "for_each",
  "arguments": {
    "in": <expression>,
    "operations": []
  }
}
```

`in` MUST be an expression.

The expression is evaluated when the loop begins.

Every iteration provides the following loop values:

* `item` — the current element;
* `index` — the zero-based index of the current element.

There is no `as` member.

Example:

```json
{
  "type": "for_each",
  "arguments": {
    "in": {
      "type": "variable",
      "data": "items"
    },
    "operations": [
      {
        "type": "store",
        "arguments": {
          "value": {
            "type": "loop",
            "data": "item"
          },
          "target_var": {
            "type": "literal",
            "data": "current"
          }
        }
      }
    ]
  }
}
```

The loop values are read-only.

Variables declared by the containing function may be modified inside the loop.

The collection types supported by `for_each` and their iteration order are defined by the collection semantics.

## 5.7 While Operation

The `while` operation repeatedly executes an operation sequence while its condition evaluates to `true`.

Syntax:

```json
{
  "type": "while",
  "arguments": {
    "condition": <expression>,
    "operations": []
  }
}
```

The condition is evaluated before every iteration.

The condition MUST evaluate to a boolean.

If the condition evaluates to `false`, the loop terminates.

## 5.8 Break Operation

The `break` operation exits the nearest enclosing loop.

Syntax:

```json
{
  "type": "break",
  "arguments": {}
}
```

Execution continues with the operation following the terminated loop.

Using `break` outside a loop is a runtime error.

## 5.9 Continue Operation

The `continue` operation terminates the current iteration of the nearest enclosing loop.

Syntax:

```json
{
  "type": "continue",
  "arguments": {}
}
```

For a `for_each` loop, execution continues with the next element.

For a `while` loop, execution continues with the next condition evaluation.

Using `continue` outside a loop is a runtime error.

## 5.10 Return Operation

The `return` operation immediately terminates the current function.

Syntax:

```json
{
  "type": "return",
  "arguments": {}
}
```

When executed:

1. The current operation sequence is terminated.
2. No subsequent operations are executed.
3. Any enclosing control-flow sequences are exited.
4. The function's `output` expression is evaluated.
5. The resulting value becomes the function's return value.

`return` does not contain a return value.

Example:

```json
{
  "inputs": ["x"],
  "variables": {
    "result": {
      "type": "literal",
      "data": null
    }
  },
  "operations": [
    {
      "type": "if",
      "arguments": {
        "condition": {
          "type": "literal",
          "data": true
        },
        "then": [
          {
            "type": "store",
            "arguments": {
              "value": {
                "type": "literal",
                "data": "early"
              },
              "target_var": {
                "type": "literal",
                "data": "result"
              }
            }
          },
          {
            "type": "return",
            "arguments": {}
          }
        ],
        "else": []
      }
    },
    {
      "type": "store",
      "arguments": {
        "value": {
          "type": "literal",
          "data": "later"
        },
        "target_var": {
          "type": "literal",
          "data": "result"
        }
      }
    }
  ],
  "output": {
    "type": "variable",
    "data": "result"
  }
}
```

The second `store` is never executed.

---

# 6. Variable Targets

Operations that modify variables use `target_var`.

The following operations have a `target_var`:

* `store`
* `set`
* `append`
* `remove`

## 6.1 Target Variable Expression

`target_var` MUST be an expression.

The expression MUST evaluate to a string.

The resulting string identifies a local variable in the current function.

Example:

```json
{
  "type": "store",
  "arguments": {
    "value": {
      "type": "literal",
      "data": 42
    },
    "target_var": {
      "type": "input",
      "data": "destination"
    }
  }
}
```

If `destination` contains `"result"`, the operation targets the variable named `result`.

The target variable MUST already be declared.

`target_var` MUST NOT create a new variable.

A non-string result is a runtime error.

An undeclared variable name is a runtime error.

---

# 7. Evaluation

## 7.1 Function Invocation

When a function is invoked:

1. Its input values are established.
2. Its variables are initialized.
3. Its operations are executed sequentially.
4. If execution reaches the end of the operations, `output` is evaluated.
5. If `return` is executed, remaining operations are skipped and `output` is evaluated immediately.

## 7.2 Expression Evaluation

Expressions are evaluated when execution reaches them.

An expression is not evaluated merely because it appears in the program.

## 7.3 Sequential Evaluation

Operations are executed in their listed order.

An operation that changes control flow may prevent subsequent operations from being executed.

## 7.4 Conditional Evaluation

Only the selected branch of an `if` operation is evaluated.

## 7.5 Loop Evaluation

A `for_each` body is evaluated once per collection element.

A `while` condition is evaluated before every iteration.

## 7.6 Unreachable Code

Unreachable operations and expressions are not evaluated.

This includes:

* unselected `if` branches;
* operations following `return`;
* operations skipped by `break`;
* operations skipped by `continue`;
* loop bodies that are never entered.

## 7.7 Function Argument Evaluation Order

The order in which function argument expressions are evaluated is unspecified.

Programs MUST NOT depend on a particular argument evaluation order.

---

# 8. Scope

## 8.1 Function Scope

Each function invocation has its own variables.

A function cannot access variables belonging to its caller.

## 8.2 Variable Scope

Variables declared by a function are accessible throughout that function's operation execution.

Conditional and loop operation sequences do not create separate ordinary variable scopes.

## 8.3 Loop Scope

`item` and `index` belong to the loop namespace.

They are available only while their corresponding `for_each` loop is executing.

Nested loops provide their own `item` and `index` values.

The innermost active loop determines the values returned by `loop` expressions.

---

# 9. Immutability

## 9.1 Inputs

Function inputs are immutable.

## 9.2 Loop Values

`item` and `index` are read-only.

## 9.3 Objects

Objects are immutable values.

`set` and `remove` produce new objects rather than modifying existing objects.

## 9.4 Arrays

Arrays are immutable values.

`set`, `append`, and `remove` produce new arrays rather than modifying existing arrays.

## 9.5 Variable Bindings

Variables themselves are mutable bindings.

Replacing a variable's value does not mutate any previous value stored in that variable.

---

# 10. Type System

Structura is dynamically typed.

Variables do not have fixed declared types.

A variable may contain values of different types at different points during execution.

## 10.1 Expression Type Requirements

The following requirements apply:

| Expression or field | Required value   |
| ------------------- | ---------------- |
| `get.path`          | array of strings |
| `get.index`         | integer          |
| `call.function`     | function         |

## 10.2 Operation Type Requirements

| Operation field     | Required value   |
| ------------------- | ---------------- |
| `store.target_var`  | string           |
| `set.target_var`    | string           |
| `set.path`          | array of strings |
| `set.index`         | integer          |
| `append.target_var` | string           |
| `remove.target_var` | string           |
| `remove.path`       | array of strings |
| `remove.index`      | integer          |
| `if.condition`      | boolean          |
| `while.condition`   | boolean          |

## 10.3 Collection Requirements

| Operation       | Required target |
| --------------- | --------------- |
| object `get`    | object          |
| array `get`     | array           |
| object `set`    | object          |
| array `set`     | array           |
| object `remove` | object          |
| array `remove`  | array           |
| `append`        | array           |

---

# 11. Type Conversion

Structura does not perform implicit type coercion.

Values are not automatically converted when another type is required.

For example:

* `"1"` is not implicitly converted to `1`;
* `1` is not implicitly converted to `true`;
* `null` is not implicitly converted to `false`;
* an empty string is not implicitly converted to `false`.

Explicit conversion functions may be provided by the runtime.

---

# 12. Runtime Interface

The runtime may expose immutable values and pure functions.

Runtime values are accessed using the `runtime` expression.

Example:

```json
{
  "type": "runtime",
  "data": "add"
}
```

Runtime functions MUST obey the function rules defined by this specification.

The runtime interface defines the names and behavior of runtime-provided functions and values.

---

# 13. Errors

Structura does not define language-level exception handling.

Runtime errors are reported by the runtime.

The following conditions are errors:

## 13.1 Function Errors

* duplicate input names;
* missing function inputs;
* unexpected function inputs;
* invalid function definitions;
* calling a non-function value.

## 13.2 Variable Errors

* referencing an undeclared variable;
* using a `variable` expression in a variable initializer;
* targeting an undeclared variable;
* using a non-string `target_var`;
* attempting to modify an input.

## 13.3 Loop Errors

* evaluating a `loop` expression outside a valid loop;
* using `break` outside a loop;
* using `continue` outside a loop.

## 13.4 Get Errors

* providing both `path` and `index`;
* providing neither `path` nor `index`;
* using `path` with a non-object;
* using `index` with a non-array;
* using a path containing a non-string element;
* accessing a missing object path;
* using an invalid array index.

## 13.5 Set Errors

* providing both `path` and `index`;
* providing neither `path` nor `index`;
* using `path` with a non-object target;
* using `index` with a non-array target;
* using a path containing a non-string element;
* missing intermediate object path;
* using an invalid array index.

## 13.6 Remove Errors

* providing both `path` and `index`;
* providing neither `path` nor `index`;
* using `path` with a non-object target;
* using `index` with a non-array target;
* using a path containing a non-string element;
* missing intermediate object path;
* removing a nonexistent object field;
* using an invalid array index.

## 13.7 Other Operation Errors

* appending to a non-array;
* using a non-boolean `if` condition;
* using a non-boolean `while` condition.

---

# 14. Serialization

JSON is the reference serialization format for Structura.

The language semantics are independent of JSON.

A different structured serialization format may represent a Structura program provided that the resulting representation preserves the semantics defined by this specification.

---

# 15. Complete Syntax Reference

## 15.1 Function

```json
{
  "inputs": [],
  "variables": {},
  "operations": [],
  "output": {}
}
```

## 15.2 Expression

```json
{
  "type": "<expression-type>",
  "data": <expression-data>
}
```

## 15.2.1 Literal

```json
{
  "type": "literal",
  "data": <value>
}
```

## 15.2.2 Variable

```json
{
  "type": "variable",
  "data": "<variable-name>"
}
```

## 15.2.3 Input

```json
{
  "type": "input",
  "data": "<input-name>"
}
```

## 15.2.4 Loop

```json
{
  "type": "loop",
  "data": "item"
}
```

or:

```json
{
  "type": "loop",
  "data": "index"
}
```

## 15.2.5 Runtime

```json
{
  "type": "runtime",
  "data": "<runtime-name>"
}
```

## 15.2.6 Get

Object:

```json
{
  "type": "get",
  "data": {
    "from": <expression>,
    "path": <expression>
  }
}
```

Array:

```json
{
  "type": "get",
  "data": {
    "from": <expression>,
    "index": <expression>
  }
}
```

## 15.2.7 Call

```json
{
  "type": "call",
  "data": {
    "function": <expression>,
    "inputs_from": {
      "<input-name>": <expression>
    }
  }
}
```

## 15.3 Operation

```json
{
  "type": "<operation-type>",
  "arguments": <operation-arguments>
}
```

## 15.3.1 Store

```json
{
  "type": "store",
  "arguments": {
    "value": <expression>,
    "target_var": <expression>
  }
}
```

## 15.3.2 Set

Object:

```json
{
  "type": "set",
  "arguments": {
    "target_var": <expression>,
    "path": <expression>,
    "value": <expression>
  }
}
```

Array:

```json
{
  "type": "set",
  "arguments": {
    "target_var": <expression>,
    "index": <expression>,
    "value": <expression>
  }
}
```

## 15.3.3 Append

```json
{
  "type": "append",
  "arguments": {
    "target_var": <expression>,
    "value": <expression>
  }
}
```

## 15.3.4 Remove

Object:

```json
{
  "type": "remove",
  "arguments": {
    "target_var": <expression>,
    "path": <expression>
  }
}
```

Array:

```json
{
  "type": "remove",
  "arguments": {
    "target_var": <expression>,
    "index": <expression>
  }
}
```

## 15.3.5 If

```json
{
  "type": "if",
  "arguments": {
    "condition": <expression>,
    "then": [],
    "else": []
  }
}
```

## 15.3.6 For-Each

```json
{
  "type": "for_each",
  "arguments": {
    "in": <expression>,
    "operations": []
  }
}
```

The loop provides `item` and `index`.

## 15.3.7 While

```json
{
  "type": "while",
  "arguments": {
    "condition": <expression>,
    "operations": []
  }
}
```

## 15.3.8 Break

```json
{
  "type": "break",
  "arguments": {}
}
```

## 15.3.9 Continue

```json
{
  "type": "continue",
  "arguments": {}
}
```

## 15.3.10 Return

```json
{
  "type": "return",
  "arguments": {}
}
```
