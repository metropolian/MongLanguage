# Mong Grammar

This document describes the syntax grammar for Mong v0.1.

Mong is a type-stable language with first-assignment type inference, optional explicit type annotations, vectors, arrays, maps, groups, multi-return functions, and scope-based error handling.

## 1. Lexical Structure

### 1.1 Identifiers

```ebnf
identifier = letter , { letter | digit | "_" } ;
letter     = "A" | ... | "Z" | "a" | ... | "z" | "_" ;
digit      = "0" | ... | "9" ;
```

### 1.2 Keywords

```text
var, group, if, else, when, while, repeat, break, out, none, true, false, and, or
```

If Thai aliases are enabled, they map to the same tokens:

```text
ตัวแปร, ถ้า, มิฉะนั้น, เมื่อ, ทำ, ซ้ำ, หยุด, ออก, ว่าง, จริง, เท็จ, และ, หรือ
```

### 1.3 Comments

```text
# single line comment
// single line comment
/* multi-line comment */
```

### 1.4 Literals

```ebnf
number = digit , { digit } , [ "." , digit , { digit } ] ;
string = '"' , { stringChar } , '"' ;
char   = "'" , charBody , "'" ;
```

## 2. Program

```ebnf
program = { declaration | statement } , EOF ;
```

## 3. Declarations

```ebnf
declaration = variableDecl
            | groupDecl
            | functionDecl ;
```

### 3.1 Variable Declaration

A variable declaration may contain a comma-separated list of entries.

```ebnf
variableDecl = "var" , varItem , { "," , varItem } ;
varItem      = identifier , [ ":" , type ] , [ "=" , expression ] ;
```

Examples:

```mong
var hp = 100
var a = 1, b = 2, c = 3
var name: string = "Metro"
var x: int = 10, y: float = 2.5, z: string = "hi"
```

### 3.2 Group Declaration

Groups are data-only user-defined types.

```ebnf
groupDecl = "group" , identifier , "{" , groupField , { "," , groupField } , "}" ;
groupField = identifier , ":" , type ;
```

Example:

```mong
group Family {
    firstName: string,
    lastName: string,
    age: int,
    height: float32
}
```

### 3.3 Function Declaration

Functions are defined with `@`.

```ebnf
functionDecl = "@" , identifier , "(" , [ parameters ] , ")" ,
               [ ":" , returnTypes ] ,
               block ;

parameters   = parameter , { "," , parameter } ;
parameter    = identifier , ":" , type ;
returnTypes   = type , { "," , type } ;
```

Example:

```mong
@ DoPower(x: float, y: float64): float64 {
    out math.pow(x, y)
}
```

## 4. Types

```ebnf
type = simpleType
     | arrayType
     | mapType
     | vectorType
     | groupType ;

simpleType = "none"
           | "byte"
           | "bool"
           | "word"
           | "int"
           | "int32"
           | "uint32"
           | "float"
           | "float32"
           | "float64"
           | "char"
           | "string"
           | "function" ;

arrayType  = "array" , "<" , type , ">" ;
mapType    = "map" , "<" , type , "," , type , ">" ;
vectorType = ( "byte" | "word" | "int" | "float" | "double" ) , "#" , vectorSize ;
vectorSize = "2" | "3" | "4" ;
groupType  = identifier ;
```

## 5. Statements

```ebnf
statement = block
          | ifStmt
          | whenStmt
          | whileStmt
          | repeatStmt
          | breakStmt
          | errorHandler
          | exprStmt ;
```

### 5.1 Block

```ebnf
block = "{" , { declaration | statement } , "}" ;
```

### 5.2 If Statement

```ebnf
ifStmt = "if" , "(" , expression , ")" , statement , [ "else" , statement ] ;
```

### 5.3 When Statement

```ebnf
whenStmt = "when" , "(" , expression , ")" , "{" , { whenCase } , [ whenElse ] , "}" ;
whenCase = expression , block ;
whenElse = "else" , block ;
```

### 5.4 While Statement

```ebnf
whileStmt = "while" , "(" , expression , ")" , statement
          | "while" , block , "(" , expression , ")" ;
```

### 5.5 Repeat Statement

```ebnf
repeatStmt = "repeat" , repeatSpec , block ;
repeatSpec = expression
           | identifier , "=" , expression , "to" , expression ;
```

### 5.6 Break Statement

```ebnf
breakStmt = "break" , ";"? ;
```

### 5.7 Error Handling

Error handling is scope-based and does not require a `try` block.

```ebnf
errorHandler = "catch" , identifier , [ catchFilter ] , block ;
catchFilter  = "if" , "(" , expression , ")"
             | "when" , "(" , expression , ")" , "{" , { catchCase } , [ catchElse ] , "}" ;
catchCase    = expression , block ;
catchElse    = "else" , block ;
```

Examples:

```mong
catch error {
    statements
}

catch error if (error == "IOError") {
    statements
}

catch error when (error) {
    "IOError" { statements }
    "GPError" { statements }
    else { statements }
}
```

### 5.8 Expression Statement

```ebnf
exprStmt = expression , ";"? ;
```

## 6. Expressions

```ebnf
expression = assignment ;

assignment = logicOr , [ "=" , assignment ] ;
logicOr    = logicAnd , { ( "||" | "or" ) , logicAnd } ;
logicAnd   = equality , { ( "&&" | "and" ) , equality } ;
equality   = comparison , { ( "==" | "!=" ) , comparison } ;
comparison = term , { ( "<" | "<=" | ">" | ">=" ) , term } ;
term       = factor , { ( "+" | "-" ) , factor } ;
factor     = unary , { ( "*" | "/" | "%" ) , unary } ;
unary      = ( "!" | "-" ) , unary
           | call ;
```

## 7. Calls and Access

```ebnf
call = primary , { callSuffix } ;
callSuffix = "(" , [ arguments ] , ")"
           | "[" , expression , "]"
           | "." , identifier ;

arguments = expression , { "," , expression } ;
```

## 8. Primary Expressions

```ebnf
primary = literal
        | identifier
        | "(" , expression , ")"
        | arrayLiteral
        | mapLiteral
        | vectorLiteral ;
```

## 9. Literals

### 9.1 Basic Literals

```ebnf
literal = number | string | char | "true" | "false" | "none" ;
```

### 9.2 Array Literals

```ebnf
arrayLiteral = "[" , [ expression , { "," , expression } ] , "]" ;
```

Examples:

```mong
["a", "b", "c"]
[3][4][5]
[, ][3]
```

### 9.3 Map Literals

```ebnf
mapLiteral = "[" , [ mapEntry , { "," , mapEntry } ] , "]" ;
mapEntry   = expression , "=" , expression ;
```

Examples:

```mong
[1 = "p1", 2 = "p2", 3 = "p3"]
["a" = "A1", "b" = "B2"]
```

### 9.4 Vector Literals

```ebnf
vectorLiteral = type , "(" , [ expression , { "," , expression } ] , ")" ;
```

Examples:

```mong
float#3(0.1, 0.2, 0.3)
float#4(0, 0, 0, 1)
```

## 10. Built-in Calls

```ebnf
builtinCall = "std" , "." , identifier , "(" , [ arguments ] , ")" ;
```

Examples:

```mong
std.out("Hello World")
name = std.in()
```

## 11. Operator Precedence

From highest to lowest:

1. Unary `!`, `-`
2. `*`, `/`, `%`
3. `+`, `-`
4. `<`, `<=`, `>`, `>=`
5. `==`, `!=`
6. `&&`, `and`, `และ`
7. `||`, `or`, `หรือ`
8. `=`

## 12. Semantic Rules

These are not grammar rules, but they belong in the language spec:

- First assignment fixes the variable type.
- Typed variables reject incompatible assignments.
- Safe numeric widening may be allowed.
- `none` is only allowed where the type permits it.
- `count(x)` returns logical length.
- `size(x)` returns storage size in bytes.
- Truthiness follows the language truthiness rules.

## 13. Example Program

```mong
var hp = 100
var power: float64 = float64(hp)
var name: string = "Metro"
var pos: float#3 = float#3(1, 2, 3)
var friends = ["A", "B", "C"]

catch error if (error == "IOError") {
    std.out("file error")
}

@ PrintPlayer(playerName: string, playerHp: int) {
    std.out("Player: " + playerName)
    std.out("HP: " + string(playerHp))
}

if (hp > 0) {
    PrintPlayer(name, hp)
}
```

## Status

This document describes a **draft v0.1 grammar** for Mong.