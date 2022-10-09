# 13. ユーティリティ関数

## 13.1 list_entry [include/linux/list.h]

定義:

```c
#define list_entry(ptr, type, member) \
((type *)((char *)(ptr)-(unsigned long)(&((type *)0)->member)))
```

意味:

"list_entry"は、内部構造体ポインタの1つを使って、親の構造体ポインタを取得する
ためのマクロです。

例:

```c
struct __wait_queue {
   unsigned int flags;
   struct task_struct * task;
   struct list_head task_list;
};
struct list_head {
   struct list_head *next, *prev;
};

// and with type definition:
typedef struct __wait_queue wait_queue_t;

// we'll have
wait_queue_t *out list_entry(tmp, wait_queue_t, task_list);

// where tmp point to list_head
```

そこで、この例では`*tmp`ポインタ[list_head]から`*out`ポインタ [wait_queue_t]を
取得しています。

```
 ____________ <---- *out [we calculate that]
|flags       |             /|\
|task *-->   |              |
|task_list   |<----    list_entry
|  prev * -->|    |         |
|  next * -->|    |         |
|____________|    ----- *tmp [we have this]
 ```

## 13.2 スリープ

### スリープのコード

ファイル:

- kernel/sched.c
- include/linux/sched.h
- include/linux/wait.h
- include/linux/list.h

関数:

- interruptible_sleep_on
- interruptible_sleep_on_timeout
- sleep_on
- sleep_on_timeout

呼ばれる関数:

- init_waitqueue_entry
- __add_wait_queue
- list_add
- __list_add
- __remove_wait_queue

InterCallings Analysis:

```
|sleep_on
   |init_waitqueue_entry  --
   |__add_wait_queue        |   enqueuing request to resource list
      |list_add              |
         |__list_add        --
   |schedule              ---     waiting for request to be executed
      |__remove_wait_queue --
      |list_del              |   dequeuing request from resource list
         |__list_del        --
```

説明:

Linuxでは各リソース（多くのユーザやプロセスで共有される観点的なオブジェクト）は
自身を要求するすべてのタスクを管理するキューを持っています。

このキューは「ウェイトキュー」と呼ばれており、「ウェイトキュー要素」と腰部多くの
項目で構成されています。

```c
/* wait queue structure [include/linux/wait.h] */

struct __wait_queue {
   unsigned int flags;
   struct task_struct * task;
   struct list_head task_list;
}
struct list_head {
   struct list_head *next, *prev;
};
```

動作の図解:

```
        ***  wait queue element  ***

                             /|\
                              |
       <--[prev *, flags, task *, next *]-->




                 ***  wait queue list ***

          /|\           /|\           /|\                /|\
           |             |             |                  |
--> <--[task1]--> <--[task2]--> <--[task3]--> .... <--[taskN]--> <--
|                                                                  |
|__________________________________________________________________|

              ***   wait queue head ***

       task1 <--[prev *, lock, next *]--> taskN
```

"wait queue head"は"wait queue list"の先頭 (`next *`) と末尾 (`prev *`) の
要素を指します。

新しい要素を追加する場合は`__add_wait_queue` [include/linux/wait.h] が呼ばれ、
その後、汎用ルーチン`list_add` [include/linux/wait.h] が実行されます。

```c
/* function list_add [include/linux/list.h] */

// classic double link list insert
static __inline__ void __list_add (struct list_head * new,  \
                                   struct list_head * prev, \
                                   struct list_head * next) {
   next->prev = new;
   new->next = next;
   new->prev = prev;
   prev->next = new;
}
```

最後に、`remove_wait_queue` [include/linux/wait.h] 内の `list_del`
[include/linux/list.h] から呼び出される関数`__list_del` [include/linux/list.h]を
見て説明を終えます。

```c
/*  function list_del [include/linux/list.h]  */


// classic double link list delete
static __inline__ void __list_del (struct list_head * prev, struct list_head * next) {
   next->prev = prev;
   prev->next = next;
}
```

### スタックの考察

通常、リスト（やキュー）はヒープに割り当てて管理するのが普通です（ヒープとスタックの
定義と変数を割り当てる場所については10章を参照ください）。しかし、ここではウェイト
キューのデータをローカル変数(Stack)として静的に確保しています。スケジューリングで
関数が中断されると、最終的には、（スケジューリングから復帰した際に）ローカル変数を
消去しています。

```
  new task <----|          task1 <------|          task2 <------|
                |                       |                       |
                |                       |                       |
|..........|    |       |..........|    |       |..........|    |
|wait.flags|    |       |wait.flags|    |       |wait.flags|    |
|wait.task_|____|       |wait.task_|____|       |wait.task_|____|
|wait.prev |-->         |wait.prev |-->         |wait.prev |-->
|wait.next |-->         |wait.next |-->         |wait.next |-->
|..        |            |..        |            |..        |
|schedule()|            |schedule()|            |schedule()|
|..........|            |..........|            |..........|
|__________|            |__________|            |__________|

   Stack                   Stack                   Stack
```
