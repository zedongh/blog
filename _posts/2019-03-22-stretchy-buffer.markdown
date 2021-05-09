---
layout: post
title:  "Stretchy Buffer"
categories: c programming-language
date: 2019-03-22 11:39:31
---

---

## I. Memory Structure

---

先来看两个类似的`struct`的定义
1. 
    ```c
    typedef struct Buf {
        size_t len;
        char *buf; // using pointer
    } Buf;
    ```
2. 
    ```c
    typedef struct Buf {
        size_t len;
        char buf[0]; // using array
    }
    ```

两种定义的差别在于`buf`用指针还是数组。在C语言中数组和指针是两个相近的概念，《C与指针》中指出一定程度上数组的指针的语法糖，当然两者的并非等价，使用方式也天差地别，这里不做详细讨论，先看上述两种`struct`在内存的布局情况：
<figure class="image">
  <img src="{{site.baseurl}}/images/struct1.svg" alt="struct definition 1">
  <figcaption>第一种struct内存布局</figcaption>
</figure>
<figure class="image">
  <img src="{{site.baseurl}}/images/struct2.svg" alt="struct definition 2">
  <figcaption>第二种struct内存布局</figcaption>
</figure>
第一种定义的`buf`与`struct`本身处于内存的两个部分，第二种定义`struct`的`buf`乍看没有占据内存，试图访问`buf`实际上是访问`struct`占据的内存之后的地方，可以说是越界访问，这里有个`tricky`，可以通过内存分配时，给`struct`分配的额外的内存存储数据，继而通过利用`buf`越界访问，这就是`stretchy buffer`或者[`flexible array member`](https://en.wikipedia.org/wiki/Flexible_array_member)<sup>1</sup>。比较下可以发现，后者的定义比前者的定义内存上更加集中紧凑，同时也可以发现`stretchy buffer`的`buf`如果不在`strcut`的最后一个`field`，那么就没有意义了。

---

## II. Usage

---

两种`struct`内存结构的不同，导致使用上的些许差别，这里不考虑`struct`本身分配在`stack`上的情况，只考虑在动态内存分配下的使用，毕竟`stretchy buffer`在`stack`上是无法体现优势的。
1. 
    ```c 
    // for buffer initialize
    void BufferInit(Buf* buf, size_t len) {
        // first allocate memory to Buf
        buf = (Buf *)malloc(sizeof(Buf)); 
        // allocate memory to buf field
        buf->buf = (char *)malloc(len);   
        buf->len = len;
    }

    // for buffer element access
    char BufferGet(Buf* buf, size_t index) {
        if (index >= buf->len) {
            exit(1); // error
        } 
        return buf->buf[index];
    }

    // for buffer clear
    void BufferClear(Buf* buf) {
        if (buf && buf->buf) {
            free(buf->buf); // first free buf filed
        }
        if (buf) {
            free(buf);      // clear Buf itself
        }
    }
    ```
2. 
    ```c 
    // for buffer initialize
    void BufferInit(Buf* buf, size_t len) {
        // only get Buf size is not enough
        buf = (Buf *)malloc(sizeof(Buf) + len); 
        buf->len = len;
    }

    // for buffer element access
    char BufferGet(Buf* buf, size_t index) {
        if (index >= buf->len) {
            exit(1); // error
        } 
        return buf->buf[index];
    }

    void BufferClear(Buf* buf) {
        if (buf) {
            free(buf); // only need to free is buf itself
        }
    }

    ```

通过上述的例子，可以看出几乎完全相同的元素访问代码，但是`stretchy buffer`的内存管理工作比普通方式下定义的`struct`简单很多。


---

## III. Type & Generic

---

当Buffer中存储的数据类型越来越复杂，可以简单做如下的修改：
```c
typedef struct Buffer {
    size_t len;
    Type *buf;
} Buffer;
```
由于C语言缺乏Generic，为了保持代码的通用性，可以借助宏来避免繁琐的重复定义实现：
```c
#define DEFINE_BUFFER(Type) \
    typedef struct Type##Buffer { \
        size_t len; \
        Type* buf; \
    } Type##Buffer; \
    \
    void Type##BufferInit(Type##Buffer *buf, size_t len) { \
        buf = (Type *)malloc(sizeof(Type##Buffer)); \
        buf->buf = (Type *)malloc(sizeof(Type) * len); \
        buf->len = len; \
    } \
    \
    Type Type##BufferGet(Type##Buffer *buf, size_t index) { \
        if (index >= buf->len) { \
            exit(1); \ 
        } \
        return buf->buf[index]; \
    } \
    \
    void Type##BufferClear(Type##Buffer *buf) { \
        if (buf && buf->buf) { \
            free(buf->buf); \
        } \
        if (buf) { \
            free(buf); \
        } \
    } \

// when use
DEFINE_BUFFER(char)
DEFINE_BUFFER(uint32);
```
当然我们也可以利用相同的方式定义`stretchy buffer`，但是这里不再描述，而是介绍一种不同的实现方式。

---

## IV. A Difference Implementation

---

目标聚集在`Buffer`的`buf`字段，内存分配完毕后，并不直接返回整个`Buffer`的指针，仅仅返回`buf`的地址，即`(char *) malloc(...) + offset_size_to_buf`。而类型信息通过外部传入，即`Type *buf = (Type *)((char *) malloc(...) + offset_size_to_buf)`。这样想来，一个清晰的实现方式就出来了，这里给出了一个较为完整的实现：

```c
#include<stdlib.h>
#include<stdio.h>
#include <zconf.h>
#include <assert.h>

#define DEFAULT_CAPACITY 8
#define MAX(a, b) (a) >= (b) ? (a) : (b)
// test if buffer need expand capacity
#define buf_fits(buf, n) (buf_len(buf) + (n) <= buf_cap(buf))
// expand capacity if necessary
#define buf_fit(buf, n) (buf_fits(buf, n) ? 0 : \ 
    ((buf) = buf_grow(buf, MAX(2 * buf_cap(buf), buf_cap(buf) + (n)), sizeof(*(buf)))))
// access buffer header address
#define buf_hdr(buf) ((BufHdr *)((char *)(buf) - offsetof(BufHdr, buf)))
// access buffer len field
#define buf_len(buf) ((buf) ? buf_hdr(buf)->len : 0)
// access buffer last elem
#define buf_end(buf) ((buf) ? ((buf) + buf_len(buf)) : 0)
// access buffer cap field
#define buf_cap(buf) ((buf) ? buf_hdr(buf)->cap : 0)
// add a elem into buffer
#define buf_add(buf, elem) (buf_fit(buf, 1), buf[buf_hdr(buf)->len++]=(elem))

#define buf_free(buf) ((buf) ? free(buf_hdr(buf)), (buf) = NULL: NULL)

typedef struct BufHdr {
    size_t len;
    size_t cap;
    char buf[0];
} BufHdr;

void *buf_grow(const void *buf, size_t new_cap, size_t elem_size) {
    BufHdr *bufHdr;
    if (new_cap < DEFAULT_CAPACITY) {
        new_cap = DEFAULT_CAPACITY;
    }
    size_t newSize = new_cap * elem_size + offsetof(BufHdr, buf);
    if (buf == NULL) {
        bufHdr = (BufHdr *) malloc(newSize);
        bufHdr->len = 0;
    } else {
        bufHdr = (BufHdr *) realloc(buf_hdr(buf), newSize);
    }
    bufHdr->cap = new_cap;
    return bufHdr->buf;
}

// usage
int main() {
    int *buf = NULL;
    assert(buf_len(buf) == 0);
#define TEST_SIZE 1025
    for (int i = 0; i < TEST_SIZE; ++i) {
        buf_add(buf, i);
    }
    assert(buf_len(buf) == TEST_SIZE);
    for (int j = 0; j < TEST_SIZE; ++j) {
        assert(buf[j] == j);
    }
    printf("%lu", buf_cap(buf));
    buf_free(buf);
    assert(buf_len(buf) == 0);
    assert(buf_cap(buf) == 0);
}
```

## V. Reference
[1] [https://en.wikipedia.org/wiki/Flexible_array_member](https://en.wikipedia.org/wiki/Flexible_array_member)