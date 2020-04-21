---
title: Java 警告抑制
date: 2020-04-21 16:39:36
tags: SuppressWarnings
cover: static/bgpic/20140719164339_iHFB2.jpeg
---





##### @SuppressWarnings 注解目标为类、字段、函数、函数入参、构造函数和函数的局部变量。

#####  而家建议注解应声明在最接近警告发生的位置。

| **关键字**               | **用途**                                                     |
| ------------------------ | ------------------------------------------------------------ |
| all                      | to suppress all warnings                                     |
| boxing                   | to suppress warnings relative to boxing/unboxing operations  |
| cast                     | to suppress warnings relative to cast operations             |
| dep-ann                  | to suppress warnings relative to deprecated annotation       |
| deprecation              | to suppress warnings relative to deprecation                 |
| fallthrough              | to suppress warnings relative to missing breaks in switch statements |
| finally                  | to suppress warnings relative to finally block that don’t return |
| hiding                   | to suppress warnings relative to locals that hide variable   |
| incomplete-switch        | to suppress warnings relative to missing entries in a switch statement (enum case) |
| nls                      | to suppress warnings relative to non-nls string literals     |
| null                     | to suppress warnings relative to null analysis               |
| rawtypes                 | to suppress warnings relative to un-specific types when using generics on class params |
| restriction              | to suppress warnings relative to usage of discouraged or forbidden references |
| serial                   | to suppress warnings relative to missing serialVersionUID field for a serializable class |
| static-access            | o suppress warnings relative to incorrect static access      |
| synthetic-access         | to suppress warnings relative to unoptimized access from inner classes |
| unchecked                | to suppress warnings relative to unchecked operations        |
| unqualified-field-access | to suppress warnings relative to field access unqualified    |
| unused                   | to suppress warnings relative to unused code                 |