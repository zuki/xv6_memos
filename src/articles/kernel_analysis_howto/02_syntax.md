# 2. この文書で使用する構文

## 2.1 関数構文

When speaking about a function, we write:

```
"function_name  [ file location . extension ]"
```

For example:

```
"schedule [kernel/sched.c]"
```

tells us that we talk about

"schedule"

function retrievable from file

[ kernel/sched.c ]

Note: We also assume /usr/src/linux as the starting directory.

## 2.2 インデント

Indentation in source code is 3 blank characters.

## 2.3 InterCallings Analysis

### 概要

We use the"InterCallings Analysis "(ICA) to see (in an indented fashion) how kernel functions call each other.

For example, the sleep_on command is described in ICA below:

```
|sleep_on
|init_waitqueue_entry      --
|__add_wait_queue            |   enqueuing request
   |list_add                 |
      |__list_add          --
   |schedule              ---     waiting for request to be executed
      |__remove_wait_queue --
      |list_del              |   dequeuing request
         |__list_del       --

                          sleep_on ICA
```

The indented ICA is followed by functions' locations:

- sleep_on [kernel/sched.c]
- init_waitqueue_entry [include/linux/wait.h]
- __add_wait_queue
- list_add [include/linux/list.h]
- __list_add
- schedule [kernel/sched.c]
- __remove_wait_queue [include/linux/wait.h]
- list_del [include/linux/list.h]
- __list_del

Note: We don't specify anymore file location, if specified just before.

### 詳細

In an ICA a line like looks like the following

```
 function1 -> function2
```

means that < function1 > is a generic pointer to another function. In this case < function1 > points to < function2 >.

When we write:

```
  function:
```

it means that < function > is not a real function. It is a label (typically assembler label).

In many sections we may report a ''C'' code or a ''pseudo-code''. In real source files, you could use ''assembler'' or ''not structured'' code. This difference is for learning purposes.

### PROs of using ICA

The advantages of using ICA (InterCallings Analysis) are many:

- You get an overview of what happens when you call a kernel function
- Function locations are indicated after the function, so ICA could also be considered as a little ''function reference''
- InterCallings Analysis (ICA) is useful in sleep/awake mechanisms, where we can view what we do before sleeping, the proper sleeping action, and what we'll do after waking up (after schedule).

### CONTROs of using ICA

- Some of the disadvantages of using ICA are listed below:

As all theoretical models, we simplify reality avoiding many details, such as real source code and special conditions.

- Additional diagrams should be added to better represent stack conditions, data values, and so on.
