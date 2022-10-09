# 4. Linuxの起動

まず、asmラベル`startup_32:`から実行されるCコードからLinux
カーネルを起動します。

```
|startup_32:
   |start_kernel
      |lock_kernel
      |trap_init
      |init_IRQ
      |sched_init
      |softirq_init
      |time_init
      |console_init
      |#ifdef CONFIG_MODULES
         |init_modules
      |#endif
      |kmem_cache_init
      |sti
      |calibrate_delay
      |mem_init
      |kmem_cache_sizes_init
      |pgtable_cache_init
      |fork_init
      |proc_caches_init
      |vfs_caches_init
      |buffer_init
      |page_cache_init
      |signals_init
      |#ifdef CONFIG_PROC_FS
        |proc_root_init
      |#endif
      |#if defined(CONFIG_SYSVIPC)
         |ipc_init
      |#endif
      |check_bugs
      |smp_init
      |rest_init
         |kernel_thread
         |unlock_kernel
         |cpu_idle
```

- startup_32 [arch/i386/kernel/head.S]
- start_kernel [init/main.c]
- lock_kernel [include/asm/smplock.h]
- trap_init [arch/i386/kernel/traps.c]
- init_IRQ [arch/i386/kernel/i8259.c]
- sched_init [kernel/sched.c]
- softirq_init [kernel/softirq.c]
- time_init [arch/i386/kernel/time.c]
- console_init [drivers/char/tty_io.c]
- init_modules [kernel/module.c]
- kmem_cache_init [mm/slab.c]
- sti [include/asm/system.h]
- calibrate_delay [init/main.c]
- mem_init [arch/i386/mm/init.c]
- kmem_cache_sizes_init [mm/slab.c]
- pgtable_cache_init [arch/i386/mm/init.c]
- fork_init [kernel/fork.c]
- proc_caches_init
- vfs_caches_init [fs/dcache.c]
- buffer_init [fs/buffer.c]
- page_cache_init [mm/filemap.c]
- signals_init [kernel/signal.c]
- proc_root_init [fs/proc/root.c]
- ipc_init [ipc/util.c]
- check_bugs [include/asm/bugs.h]
- smp_init [init/main.c]
- rest_init
- kernel_thread [arch/i386/kernel/process.c]
- unlock_kernel [include/asm/smplock.h]
- cpu_idle [arch/i386/kernel/process.c]

最後の関数`rest_init`は次のことを行います。

1. カーネルスレッド`init`の起動
2. `unlock_kernel`の呼び出し
3. カーネルは`cpu_idle`ルーチンを実行し、スケジュールされるタスクが
   ない時にアイドルループが実行されるようになります。

実際、`start_kernel`プロシージャは終了することはなく、`cpu_idle`
ルーチンを永遠に実行します。

最初のカーネルスレッドである`init`は次のように実行します。

```
|init
   |lock_kernel
   |do_basic_setup
      |mtrr_init
      |sysctl_init
      |pci_init
      |sock_init
      |start_context_thread
      |do_init_calls
         |(*call())-> kswapd_init
   |prepare_namespace
   |free_initmem
   |unlock_kernel
   |execve
```
