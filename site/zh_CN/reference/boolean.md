---
id: boolean.md
summary: 了解 Milvus 中的布尔表达式规则。
title: 标量过滤规则
---
# 标量过滤规则

## 概述

谓词表达式输出布尔值。Milvus 通过使用谓词进行标量过滤。谓词表达式在求值时会返回 TRUE 或 FALSE。查看 [Python SDK API 参考](/api-reference/pymilvus/v{{var.milvus_python_sdk_version}}/Collection/query().md) 以获取有关使用谓词表达式的指导。

[EBNF](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_form) 语法规则描述了布尔表达式规则：

```
Expr = LogicalExpr | NIL
LogicalExpr = LogicalExpr BinaryLogicalOp LogicalExpr 
              | UnaryLogicalOp LogicalExpr
              | "(" LogicalExpr ")"
              | SingleExpr;
BinaryLogicalOp = "&&" | "and" | "||" | "or";
UnaryLogicalOp = "not";
SingleExpr = TermExpr | CompareExpr;
TermExpr = IDENTIFIER "in" ConstantArray;
Constant = INTEGER | FLOAT
ConstantExpr = Constant
               | ConstantExpr BinaryArithOp ConstantExpr
               | UnaryArithOp ConstantExpr;
                                                          
ConstantArray = "[" ConstantExpr { "," ConstantExpr } "]";
UnaryArithOp = "+" | "-"
BinaryArithOp = "+" | "-" | "*" | "/" | "%" | "**";
CompareExpr = IDENTIFIER CmpOp IDENTIFIER
              | IDENTIFIER CmpOp ConstantExpr
              | ConstantExpr CmpOp IDENTIFIER
              | ConstantExpr CmpOpRestricted IDENTIFIER CmpOpRestricted ConstantExpr;
CmpOpRestricted = "<" | "<=";
CmpOp = ">" | ">=" | "<" | "<=" | "=="| "!=";
MatchOp = "like" | "LIKE";
JsonArrayOps = JsonDefs "(" IDENTIFIER "," JsonExpr | JsonArray ")";
JsonArrayDefs = "json_contains" | "JSON_CONTAINS" 
           | "json_contains_all" | "JSON_CONTAINS_ALL" 
           | "json_contains_any" | "JSON_CONTAINS_ANY";
JsonExpr =  Constant | ConstantArray | STRING | BOOLEAN;
JsonArray = "[" JsonExpr { "," JsonExpr } "]";
ArrayOps = ArrayDefs "(" IDENTIFIER "," ArrayExpr | Array ")";
ArrayDefs = "array_contains" | "ARRAY_CONTAINS" 
           | "array_contains_all" | "ARRAY_CONTAINS_ALL" 
           | "array_contains_any" | "ARRAY_CONTAINS_ANY"
           | "array_length"       | "ARRAY_LENGTH";
ArrayExpr =  Constant | ConstantArray | STRING | BOOLEAN;
Array = "[" ArrayExpr { "," ArrayExpr } ]";
```

以下表格列出了上述布尔表达式规则中提到的每个符号的描述。

| 符号      | 描述 |
| ----------- | ----------- |
| =      | 定义。       |
| ,      | 连接。       |
| ;      | 终止。        |
| \|      | 交替。       |
| {...}   | 重复。        |
| (...)      | 分组。       |
| NIL   | 空。表达式可以是空字符串。        |
| INTEGER      | 整数，如 1、2、3。       |
| FLOAT   | 浮点数，如 1.0、2.0。        |
| CONST      | 整数或浮点数。       |
| 标识符   | 标识符。在 Milvus 中，标识符代表字段名。        |
| 逻辑运算符      | 逻辑运算符支持在一个比较中结合多个关系运算。逻辑运算符的返回值为 TRUE（1）或 FALSE（0）。逻辑运算符有两种类型，包括二元逻辑运算符和一元逻辑运算符。    |
| 一元逻辑运算符   | 一元逻辑运算符指的是一元逻辑运算符“非”。        |
| 二元逻辑运算符   | 二元逻辑运算符对两个操作数执行操作。在一个包含两个或多个操作数的复杂表达式中，求值顺序取决于优先规则。       |
| 算术运算符   | 算术运算符执行诸如加法和减法等数学运算的算术操作。         |
| 一元算术运算符      | 一元算术运算符是对单个操作数执行操作的算术运算符。负一元算术运算符将正表达式变为负表达式，反之亦然。      |
| 二元算术运算符   | 二元算术运算符对两个操作数执行操作。在一个包含两个或多个操作数的复杂表达式中，求值顺序取决于优先规则。        |
| 比较运算符   | 比较运算符对两个操作数执行操作。        |
| 限制比较运算符      | 限制比较运算符仅限于“小于”和“等于”。       |
| 常量表达式   | 常量表达式可以是常量或两个常量表达式上的二元算术运算符，或单个常量表达式上的一元算术运算符。它被递归定义。        |
| 常量数组      | 常量数组用方括号括起来，方括号中可以重复出现常量表达式。常量数组必须至少包含一个常量表达式。       |
| 项表达式   | 项表达式用于检查标识符的值是否出现在常量数组中。项表达式用“in”表示。        |
| 比较表达式      | 比较表达式可以是两个标识符之间的关系运算，或一个标识符和一个常量表达式之间的关系运算，或两个常量表达式和一个标识符之间的三元运算。       |
| 单一表达式   | 单一表达式可以是项表达式或比较表达式。      |
| 逻辑表达式      | 逻辑表达式可以是两个逻辑表达式之间的二元逻辑运算符，或一个逻辑表达式上的一元逻辑运算符，或括号内的逻辑表达式，或单一表达式。逻辑表达式被递归定义。    |
| 表达式   | 表达式，即 expression 的缩写，可以是逻辑表达式或 NIL。 |
| 匹配运算符   | 匹配运算符比较一个字符串与一个字符串常量或字符串前缀、中缀或后缀常量。 |
| Json 数组运算符 | Json 操作符，即 JSON 运算符，检查指定标识符是否包含指定元素。 |
| 数组运算符 | 数组操作符，即数组运算符，检查指定标识符是否包含指定元素。 |

## 运算符
### 逻辑运算符

逻辑运算符执行两个表达式之间的比较。

| 符号 | 操作 | 示例 | 描述 |
| ---- | ---- | ---- | ---- |
| 'and' && | 与 | expr1 && expr2 | 如果expr1和expr2都为真，则为真。 |
| 'or' \|\| | 或 | expr1 \|\| expr2 | 如果expr1或expr2为真，则为真。 |

### 二进制算术运算符

二进制算术运算符包含两个操作数，可以执行基本算术运算并返回相应的结果。

| 符号 | 操作 | 示例 | 描述 |
| ---- | ---- | ---- | ---- |
| + | 加法 | a + b | 将两个操作数相加。 |
| - | 减法 | a - b | 从第一个操作数中减去第二个操作数。 |
| * | 乘法 | a * b | 将两个操作数相乘。 |
| / | 除法 | a / b | 将第一个操作数除以第二个操作数。 |
| ** | 幂 | a ** b | 将第一个操作数提升到第二个操作数的幂。 |
| % | 取模 | a % b | 将第一个操作数除以第二个操作数，并返回余数部分。 |

### 关系运算符

关系运算符使用符号来检查两个表达式之间的相等性、不等性或相对顺序。

| 符号 | 操作 | 示例 | 描述 |
| ---- | ---- | ---- | ---- |
| < | 小于 | a < b | 如果a小于b，则为真。 |
| > | 大于 | a > b | 如果a大于b，则为真。 |
| == | 等于 | a == b | 如果a等于b，则为真。 |
| != | 不等于 | a != b | 如果a不等于b，则为真。 |
| <= | 小于等于 | a <= b | 如果a小于或等于b，则为真。 |
| >= | 大于等于 | a >= b | 如果a大于或等于b，则为真。 |

## 运算符优先级和结合性

以下表格列出了运算符的优先级和结合性。运算符按优先级从高到低的顺序列出。

| 优先级 | 运算符 | 描述 | 结合性 |
|--------|--------|-----|--------|
| 1 | + - | 一元算术运算符 | 从左到右 |
| 2 | not | 一元逻辑运算符 | 从右到左 |
| 3 | ** | 二元算术运算符 | 从左到右 |
| 4 | * / % | 二元算术运算符 | 从左到右 |
| 5 | + - | 二元算术运算符 | 从左到右 |
| 6 | < <= > >= | 比较运算符 | 从左到右 |
| 7          | == !=                                 | CmpOp         | 从左到右   |
| 8          | like LIKE                             | MatchOp       | 从左到右   |
| 9          | json_contains JSON_CONTAINS           | JsonArrayOp   | 从左到右   |
| 9          | array_contains ARRAY_CONTAINS         | ArrayOp       | 从左到右   |
| 10         | json_contains_all JSON_CONTAINS_ALL   | JsonArrayOp   | 从左到右   |
| 10         | array_contains_all ARRAY_CONTAINS_ALL | ArrayOp       | 从左到右   |
| 11         | json_contains_any JSON_CONTAINS_ANY   | JsonArrayOp   | 从左到右   |
| 11         | array_contains_any ARRAY_CONTAINS_ANY | ArrayOp       | 从左到右   |
| 12         | array_length  ARRAY_LENGTH            | ArrayOp       | 从左到右   |
| 13         | && and                                | BinaryLogicOp | 从左到右   |
| 14         | \|\| or                               | BinaryLogicOp | 从左到右   |

表达式通常从左到右进行评估。复杂表达式逐个进行评估。表达式的评估顺序由使用的运算符的优先级确定。

如果一个表达式包含两个或多个具有相同优先级的运算符，则首先评估左侧的运算符。

<div class="alert note">

例如，10 / 2 * 5 将被评估为 (10 / 2)，然后乘以 5。

</div>

当应该先处理优先级较低的操作时，应将其括在括号内。

<div class="alert note">

例如，30 / 2 + 8。通常会被评估为 30 除以 2，然后加上 8 的结果。如果要除以 2 + 8，则应编写为 30 / (2 + 8)。

</div>

括号可以嵌套在表达式中。最内层的括号表达式首先进行评估。

## 用法

Milvus 中所有可用布尔表达式的示例如下所示（`int64` 表示包含 INT64 类型数据的标量字段，`float` 表示包含浮点类型数据的标量字段，`VARCHAR` 表示包含 VARCHAR 类型数据的标量字段）：

1. CmpOp

```
"int64 > 0"
```

```
"0 < int64 < 400"
```

```
"500 <= int64 < 1000"
```

```
VARCHAR > "str1"
```

2. BinaryLogicalOp 和括号

```
"(int64 > 0 && int64 < 400) or (int64 > 500 && int64 < 1000)"
```

3. TermExpr 和 UnaryLogicOp

<div class="alert note">

Milvus 仅支持通过明确定义的主键删除实体，这仅可通过术语表达式 <code>in</code> 实现。

</div>

```
"int64 not in [1, 2, 3]"
```

```
VARCHAR not in ["str1", "str2"]
```


4. TermExpr、BinaryLogicalOp 和 CmpOp（在不同字段上）

```
"int64 in [1, 2, 3] and float != 2"
```

5. BinaryLogicalOp 和 CmpOp

```
"int64 == 0 || int64 == 1 || int64 == 2"
```

6. CmpOp 和 UnaryArithOp 或 BinaryArithOp

```
"200+300 < int64 <= 500+500"
```

7. MatchOp
```
VARCHAR 类似于 "前缀%"
VARCHAR 类似于 "%后缀"
VARCHAR 类似于 "%中间%"
VARCHAR 类似于 "_后缀"
```

8. JsonArrayOp

- `JSON_CONTAINS(identifier, JsonExpr)`

    如果`JSON_CONTAINS`语句中的 JSON 表达式（第二个参数）是一个列表，则标识符（第一个参数）应该是一个列表的列表。否则，该语句始终计算为 False。

    ```python
    # {"x": [1,2,3]}
    json_contains(x, 1) # ==> true
    json_contains(x, "a") # ==> false
        
    # {"x": [[1,2,3], [4,5,6], [7,8,9]]}
    json_contains(x, [1,2,3]) # ==> true
    json_contains(x, [3,2,1]) # ==> false
    ```

- `JSON_CONTAINS_ALL(identifier, JsonExpr)`

    `JSON_CONTAINS_ALL`语句中的 JSON 表达式应始终是一个列表。

    ```python
    # {"x": [1,2,3,4,5,7,8]}
    json_contains_all(x, [1,2,8]) # ==> true
    json_contains_all(x, [4,5,6]) # ==> false 6 is not exists
    ```

- `JSON_CONTAINS_ANY(identifier, JsonExpr)`

    `JSON_CONTAINS_ANY`语句中的 JSON 表达式应始终是一个列表。否则，它的行为与 `JSON_CONTAINS` 相同。

    ```python
    # {"x": [1,2,3,4,5,7,8]}
    json_contains_any(x, [1,2,8]) # ==> true
    json_contains_any(x, [4,5,6]) # ==> true
    json_contains_any(x, [6,9]) # ==> false
    ```

9. ArrayOp

- `ARRAY_CONTAINS(identifier, ArrayExpr)`

    如果`ARRAY_CONTAINS`语句中的数组表达式（第二个参数）是一个列表，则标识符（第一个参数）应该是一个列表的列表。否则，该语句始终计算为 False。

    ```python
    # 'int_array': [1,2,3]
    array_contains(int_array, 1) # ==> true
    array_contains(int_array, "a") # ==> false
    ```

- `ARRAY_CONTAINS_ALL(identifier, ArrayExpr)`

    `ARRAY_CONTAINS_ALL`语句中的数组表达式应始终是一个列表。

    ```python
    # "int_array": [1,2,3,4,5,7,8]
    array_contains_all(int_array, [1,2,8]) # ==> true
    array_contains_all(int_array, [4,5,6]) # ==> false 6 is not exists
    ```

- `ARRAY_CONTAINS_ANY(identifier, ArrayExpr)`

    `ARRAY_CONTAINS_ANY`语句中的数组表达式应始终是一个列表。否则，它的行为与 `ARRAY_CONTAINS` 相同。

    ```python
    # "int_array": [1,2,3,4,5,7,8]
    array_contains_any(int_array, [1,2,8]) # ==> true
    array_contains_any(int_array, [4,5,6]) # ==> true
    array_contains_any(int_array, [6,9]) # ==> false
    ```

- `ARRAY_LENGTH(identifier)`

    检查数组中元素的数量。

    ```python
    # "int_array": [1,2,3,4,5,7,8]
    array_length(int_array) # ==> 7
    ```

## 接下来做什么

现在您已经了解了 Milvus 中位集是如何工作的，您可能还想：

- 学习如何进行[多向量搜索](multi-vector-search.md)。
- 学习如何[使用字符串来过滤](https://milvus.io/blog/2022-08-08-How-to-use-string-data-to-empower-your-similarity-search-applications.md)您的搜索结果。
- 学习如何[在构建布尔表达式中使用动态字段](enable-dynamic-field.md)。