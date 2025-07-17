---
title: "hello world"
date: 2024-05-25
lastmod: 2024-05-30
draft: false
description: "a description"
categories: ["test"]
tags: ["welcome", "example", "test"]
series: ["test"]
series_order: 1
---
{{< katex >}}
{{< typeit
  tag=h2
  loop=true
>}}
Hello world!
{{< /typeit >}}

 an example to get you started

## 这是一个标题

### 这是一个副标题

#### 这是一个副标题

##### 这是一个子标题

~~This is a paragraph~~ with **bold** and *italic* text.
`Check` more at [Blowfish documentation](https://blowfish.page/)
undefined

- [ ] test1
- [x] test2
  - [ ] test22
  - [x] test23

1. item1
2. item2
3. item3
   1. item3a
   2. item3b

> We're living the future so
> the present is our past.

---

## code

```c {.line-numbers}
#include <stdio.h>

int main() {
    printf("Hello, world!");
    return 0;
}
```

```python
print('hello')
```

## table

| a   | b   |
| --- | --- |
| one | two |
| 3   | 4   |

## math

$$
\sum_{i=2}^ni=\frac{n(n+1)}{2}-1
$$
