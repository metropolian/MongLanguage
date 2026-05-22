# The Mong Language

อยากมีภาษาคอมพิวเตอร์เป็นของตัวเอง
เลยอยากลองพัฒนาให้มันง่ายขึ้นจากภาษาอื่นๆที่เคยเขียนตั้งแต่เริ่มแรกได้เรียน Pascal

ภาษาชื่อ มั่ง (Mong) ชื่อนี่เป็นเรื่องราวแฟนตาซีแต่ไม่สำคัญเท่ากับว่าตอนนี้เราจะลองสร้างมาเป็นภาษาใช้งานจริงๆดู อยากให้พัฒนาเกมได้
ภาษาเป็นแนวที่ประเภทตัวแปรจะผูกไว้กับการประการครั้งแรก ดังนั้นหากขี้เกียจก็ไม่จำเป็นจะต้องประกาศก็ให้ตั้งค่าไปเลยได้

## Design goals

- อ่านง่าย เขียนง่าย มีลักษณะเหมือนภาษาที่คุนเคย
- มีคำสั่งตัวช่วยลดการเขียนจากภาษาที่คุ้นเคย ลงบ้าง
- พอที่จะมีการดักความผิดพลาดได้
- อยากให้เป็นภาษาที่คอมพายเป็น Executable ได้เลย
- support for vectors, arrays, maps, groups (struct), and multi-return functions.

## Typing model

การกำหนดประเภทตัวแปรของ Mong 

- `var name = value` ประเภทจะถูกตั้งจากค่าที่กำหนด
- `var name: type = value` ประกาศค่าแบบเจาะจงไปเลย
- หากมีประเภทกำกับแล้ว ไปเปลี่ยนทีหลังจะปลิวแน่ๆ
- การแปลงค่าประเภทคล้ายกันแบบ int <-> float ไม่เป็นอะไรหรอก แต่ถ้า int -> string ปลิวแน่

Example:

```mong
var hp = 100
hp = 120          // ok
hp = "full"       // error

var mp: float = hp // ok
var sp: int64 = hp // ok
var pos: float#3 = hp // error
```

## Core types

| Type | Size | Default |
|------|------|---------|
| `none` | - | `none` |
| `byte` | 8-bit | `0` |
| `bool` | 8-bit | `false` |
| `word` | 16-bit | `0` |
| `int` / `int32` | 32-bit | `0` |
| `uint32` | 32-bit | `0` |
| `uint64` | 64-bit | `0` |
| `float` / `float32` | 32-bit | `0` |
| `float64` | 64-bit | `0` |
| `char` | UTF character | `'\0'` |
| `string` | dynamic | `""` |
| `array<T>` | dynamic | `[]` |
| `map<K,V>` | dynamic | `{}` |
| `function` | callable | implementation-defined |

## `none` rules

`none` ค่า NULL นั้นละ เหมือนกล่องเปล่าไม่มีอะไร

Recommended rules:

- `var x` ค่าเริ่มต้นเป็น `none`.
- `string`, `int`, or `float#3` ต้องไม่เป็น `none`.

Example:
```mong
var unknown
unknown = "hello"
unknown = none

var name: string = none // error
```

## SIMD / vector types

สำหรับสายคำนวณและทำเกม

Supported:
- `byte#2` to `byte#4`
- `word#2` to `word#4`
- `int#2` to `int#4`
- `float#2` to `float#4`
- `double#2` to `double#4`

Example:

```mong
var pos: float#3 = float#3(0.1, 0.2, 0.3)
var rotation: float#4 = float#4(0, 0, 0, 1)

var vectorA = float#4(1.0, 1, 2, 0)
var vectorB = float#4(1.0, 0, 1.2, 0.0)
var vectorSum = vectorA + vectorB

pos = pos * 1.2
std.out(pos[0])
```

## Comments

คอมเม้น Mong 3 styles:

```mong
# single line comment
// single line comment
/* multi-line
   comment */
```

## Variables and literals

Examples:

```mong
var unknown
var hp: int = 123456
var power: float64 = 13.43511
var name: string = "Metro"
var alive: bool = false
var gender: char = 'M'
```

multiple var:

```mong
var mhp = 100, msp = 50, my_name = "Metro"
var xx: int = 1, yy: float = 2.5, zz: string = "hi"
```

### Strings

- Strings ต่อกันด้วย `+`.
- String[x] ได้ `char`.
- การต่อคำใช้รูปแบบ `$"..."` ได้นะ.

```mong
var hello = "hello"
var world: string = "world"
var helloWorld = hello + world
std.out(helloWorld)

std.out(hello[0])

var message = $"{hello} {world}"
```

## Collections

### Arrays

```mong
var friends: array<string> = ["a", "b", "c"]
var toys = ["t", "x", "z"]
var ages = [1, 2, 3]
var weights = [55.5, 66, 67.2]
var image: array<array<int>> = [[0, 0], [1, 0], [1, 0]]
```

คำสั่ง array:

```mong
friends.add("d")
friends.remove("b")
friends.count()
friends[0]
friends.contains("a")
friends.range(1, 2)

var classroom: array<string>
classroom.from(friends)
classroom.sort()
```

### Maps

Map/Hashmap/Dictionary

```mong
var players: map<int, string> = [1 = "p1", 2 = "p2", 3 = "p3"]
var callsigns: map<string, string> = ["a" = "A1", "b" = "B2", "c" = "C3"]

players[1] == "p1"
players.set(4, "p4")
players.remove(1)
players.contains(2)
players.keys()
players.values()
```

## Groups

Groups are user-defined structured data types.

```mong
group Family {
    firstName: string,
    lastName: string,
    age: int,
    height: float32
}

var family: Family = Family(
    firstName = "John",
    lastName = "Doh",
    age = 30,
    height = 1.6
)

var secondFam: Family
secondFam = family

std.out(family.firstName)
```

## Built-in functions

```mong
type(unknown) == "none"
type(pos) == "float#3"
type(rotation) == "float#4"
type(hello[0]) == "char"
type(hello) == "string"

count(hello) == 5
count(friends) == 3
count(pos) == 3
count(rotation) == 4

size(hello) == 5
size(friends) == 3
size(pos) == 12
size(rotation) == 16
```

Standard I/O:

```mong
std.out("Hello World")
name = std.in()
```

## Functions

Mong functions เขียน `@` กำกับไว้ด้านหน้าและ out ได้หลายค่านะ

```mong
@ function_name(param1: type, param2: type): type1, type2, type3 {
    statement
    statement
    out return1, return2, return3
}
```

Example:

```mong
@ DoPower(x: float, y: float64): float64 {
    out math.pow(x, y)
}

@ IsChildAge(age: int): bool {
    out age < 20
}

var checkAgeFunction: function = IsChildAge
var result = checkAgeFunction(12)
std.out(result)
```

Return binding rules:

```mong
var r1, r2, r3 = function_name()
var s1, s2 = function_name()
var t1 = function_name()
```

- การส่งค่ากลับ หากมีตัวแปรรับไม่หมด ก็รับไปตามลำดับนะ มีเกินไม่เป็นไร

## Control flow

### If / else

```mong
if (condition) {
    statements
} else {
    statements
}
```

### When

`when` ก็เหมือน switch ไง

```mong
when (value) {
    1 { std.out("one") }
    2 { std.out("two") }
    3 { std.out("three") }
    else { std.out("other") }
}

when (room) {
    "101" { out 1 }
    "102" { out 2 }
    "103" { out 3 }
    else { out 4 }
}
```

### While

เช็คเงือนไขก่อนทำ

```mong
while (true_condition) {
    statements
}
```

เช็คเงื่อนไขหลังทำ

```mong
while {
    statements
} (true_condition)
```

### Repeat

ลูปตามจำนวน

```mong
repeat times { statements }
repeat value = 0 to times { statements }
repeat value = 1 to 10 { statements }
repeat value = 10 to 1 { statements }
repeat value = arrays { statements }
```
## Break Loop

```mong
repeat 10 { 
	break;
}

while {
    break;
} ( true )
```



## Operators

การทำงานไล่ตามชั้นก่อนหลัง

1. Unary: `!`, `-`
2. Multiply/divide/mod: `*`, `/`, `%`
3. Add/subtract: `+`, `-`, `++`, `--`,
4. Comparison: `<`, `<=`, `>`, `>=`
5. Equality: `==`, `!=`
6. Logical and: `&&` `and` `และ`
7. Logical or: `||` `or` `หรือ`
8. Assignment: `=`

## Boolean Operators
1. Logical and: `&&` `and` `และ`
2. Logical or: `||` `or` `หรือ`

## Thai Language supports

var = var / ตัวแปร
if = if / ถ้า
else = else / มิฉะนั้น
while = while / ทำ
repeat = repeat / ซ้ำ
group = group / กลุ่ม
when = when / เมื่อ
out = out / ออก
none = none / ว่าง
true = true / จริง
false = false / เท็จ
break = break / หยุด


## Truthiness:

ความจริงมีเพียงหนึ่งเดียว

- bool: true = truthy, false = falsey
- int/byte: 0 = falsey, non-zero = truthy
- float: 0.0 = falsey, non-zero = truthy
- char: '\0' = falsey, other chars = truthy
- string: "" = falsey, otherwise truthy
- vector: all components zero = falsey, otherwise truthy
- array: empty = falsey, otherwise truthy
- map: empty = falsey, otherwise truthy
- none: falsey

## Error handling

Mong การทำงานจะขึ้นกับสโคปของโปรแกรม

`trigger(error)` จะสร้าง error ส่งไปในสโคป และ function
หากมี `catch` จะดักจับ error นั้นในสโคปทันที จะเขียนบนหรือล่างก็ได้ 
การเขียน catch อย่างเดียว ไม่ต้องมี try ... นะ

```mong
catch error {
    statements
}
```

### Catch with "if"

```mong
catch error if (error == "IOError") {
    statements
}
```

### Catch with "when"

```mong
catch error when (error) {
    "IOError" { statements }
    "GPError" { statements }
    else { statements }
}
```

### Error rules

- `trigger(error)` จะส่งค่าไปหา `catch` ที่ใกล้สุดก่อน
- `catch` เขียนอยู่บนหรือล่างก็ได้ในสโคปน้นๆ
- ถ้าในสโคปนั้นไม่มี ก็จะขึ้นไปหาสโคปก่อนหน้านี้
- The error value จะเป็นอะไรก็ค่อยว่ากันอีกที

Example:

```mong
catch error when (error) {
    "IOError" {
        std.out("file error")
    }
    "GPError" {
        std.out("gpu error")
    }
    else {
        std.out("unknown error")
    }
}
```

## Example program

```mong
var hp = 100
var power: float64 = float64(hp)
var name: string = "Metro"
var pos: float#3 = float#3(1, 2, 3)
var friends = ["A", "B", "C"]

@ PrintPlayer(playerName: string, playerHp: int) {
    std.out("Player: " + playerName)
    std.out("HP: " + string(playerHp))
}

if (hp > 0) {
    PrintPlayer(name, hp)
}
```

## Status

This README describes a **draft v0.1 design** for Mong. 


## Contact
iMakeProject
Kobchaipuk Kemapirom
Nutt: imp.metropolian@gmail.com
