---
title: 计算机如何表示小数
date: 2024-02-12 22:00:00
categories:
- Idea
tags:
- 突发奇想

---


在计算机中，表示小数有两种主要方法：定点数和浮点数。每种方法有其独特的表示方式和应用场景。本文将详细介绍这两种表示方法及其内存中的存储形式。

## 定点数表示法

定点数表示法是将小数固定在某个位置，分为定点整数部分和定点小数部分。我们将通过两个例子来详细解释定点数的表示方法。

### 例子 1：表示 3.25

1. **定点整数部分**：
    - 数字：+3
    - 二进制表示：11
    - 内存中存储（补足 8 位）：0000 0011

2. **定点小数部分**：
    - 数字：+0.25
    - 二进制表示：0.01
    - 内存中存储（补足 8 位）：0010 0000

### 例子 2：表示 -0.25

1. **整数部分**：
    - 数字：0
    - 内存中存储：0000 0000

2. **小数部分**：
    - 数字：-0.25
    - 二进制表示：-0.01
    - 内存中存储：
        - 原码取反：1111 1110
        - 加 1 得到补码：1110 0000

总结：定点数表示法通过将整数和小数分别处理并填充到指定长度，简单且高效，但由于小数点位置固定，表示范围有限。

## 浮点数表示法

浮点数表示法更为灵活，可以表示更大的范围和精度。浮点数分为三部分：符号位、指数和尾数。以下是具体步骤及例子。

### 例子 1：表示 3.25

1. **转换为二进制**：
    - 3.25 = 11.01（二进制）

2. **标准化**：
    - 11.01 = 1.101 × 2¹

3. **符号位**：
    - 正数，符号位为 0

4. **指数**：
    - 指数为 1，加上偏移量 127 得到 128
    - 二进制表示：1000 0000

5. **尾数**：
    - 标准化后尾数为 1.101，隐藏开头的 1，只存储 101
    - 内存中存储：10100000000000000000000（补足 23 位）

### 例子 2：表示 -0.25

1. **转换为二进制**：
    - -0.25 = -0.01（二进制）

2. **标准化**：
    - -0.01 = 1.0 × 2⁻²

3. **符号位**：
    - 负数，符号位为 1

4. **指数**：
    - 指数为 -2，加上偏移量 127 得到 125
    - 二进制表示：0111 1101

5. **尾数**：
    - 标准化后尾数为 1.0，只存储 0
    - 内存中存储：00000000000000000000000（补足 23 位）

总结：浮点数表示法通过科学计数法，使用符号位、指数和尾数来灵活表示数值范围，但实现复杂，可能会有精度损失。

## 结论

定点数和浮点数在计算机中各有用途。定点数适用于要求高效和简单的应用场景，而浮点数则适用于需要更大范围和精度的场景。了解这两种表示方法对于计算机科学及其应用至关重要。通过具体例子和内存存储方式的说明，希望大家对计算机如何表示小数有更深入的理解。