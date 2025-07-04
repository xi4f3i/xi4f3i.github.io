---
title: 字符串计算器 TypeScript 实现
date: 2021-07-25 21:44:58
tags: [TypeScript]
---

计算字符串中四则运算的结果

```typescript
/**
 * 计算字符串中算式的结果
 */
function calculate(str: string | Array<string>): number {
    // string 转为 array ，方便处理
    if (typeof str === 'string') str = str.split('');

    let sign = '+';
    let num = 0;
    let steps: Array<number> = [];

    while (sign.length > 0) {
        const c = str.pop();
        const c_is_digit = is_digit(c);

        if (c_is_digit) num = num * 10 + Number(c);

        if (c === '(') num = calculate(str);

        if ((!c_is_digit && c !== ' ') || sign.length === 0) {
            if (sign === '+') {
                steps.push(num);
            } else if (sign === '-') {
                steps.push(-num);
            } else if (sign === '*') {
                const prev = steps.pop();
                steps.push(prev * num);
            } else if (sign === '/') {
                const prev = steps.pop();
                steps.push(prev / num);
            }
            num = 0;
            sign = c;
        }

        if (c === ')') break;
    }

    return sum(steps);
}

/**
 * 计算值的合
 */
function sum(steps: Array<number>): number {
    return steps.reduce((prev, cur) => prev + cur, 0);
}

/**
 * 判断字符是否为数字
 */
function is_digit(c: string): boolean {
    const n = Number(c);
    return !isNaN(n);
}
```
