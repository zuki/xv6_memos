
```c
void smp_local_timer_interrupt(struct pt_regs *regs)
{
	int cpu = smp_processor_id();

	profile_tick(CPU_PROFILING, regs);
	if (--per_cpu(prof_counter, cpu) <= 0) {
		per_cpu(prof_counter, cpu) = per_cpu(prof_multiplier, cpu);
		if (per_cpu(prof_counter, cpu) != 
		    per_cpu(prof_old_multiplier, cpu)) {
			__setup_APIC_LVTT(calibration_result/
					per_cpu(prof_counter, cpu));
			per_cpu(prof_old_multiplier, cpu) =
				per_cpu(prof_counter, cpu);
		}

#ifdef CONFIG_SMP
		update_process_times(user_mode(regs));
#endif
	}
```

```c
in: kernel/timer.c

void update_process_times(int user_tick)
{
    struct task_struct *p = current;
    int cpu = smp_processor_id();

    /* Note: this timer irq context must be accounted for as well. */
    if (user_tick)
        account_user_time(p, jiffies_to_cputime(1));
    else
        account_system_time(p, HARDIRQ_OFFSET, jiffies_to_cputime(1));
    run_local_timers();
    if (rcu_pending(cpu))
        rcu_check_callbacks(cpu, user_tick);
    scheduler_tick();
     run_posix_cpu_timers(p);
}

include/asm-generic/cputime.h:

typedef unsigned long cputime_t;
typedef u64 cputime64_t;

#define jiffies_to_cputime(__hz)	(__hz)
#define cputime_add(__a, __b)       ((__a) +  (__b))
#define cputime64_add(__a, __b)     ((__a) + (__b))

#define cputime_to_secs(jif)        ((jif) / HZ)
#define secs_to_cputime(sec)        ((sec) * HZ)
#define cputime_to_cputime64(__ct)	((u64) __ct)

include/linux/hardirq.h

/*
 * PREEMPT_MASK: 0x000000ff
 * SOFTIRQ_MASK: 0x0000ff00
 * HARDIRQ_MASK: 0x0fff0000
*/

#define HARDIRQ_OFFSET	(1UL << HARDIRQ_SHIFT)	// HARIRQ_SHIFT = 16

in: kernel/sched.c

/*
 * Convert user-nice values [ -20 ... 0 ... 19 ]
 * to static priority [ MAX_RT_PRIO..MAX_PRIO-1 ] ([100..139]),
 * and back.
 */
#define NICE_TO_PRIO(nice)	(MAX_RT_PRIO + (nice) + 20)
#define PRIO_TO_NICE(prio)	((prio) - MAX_RT_PRIO - 20)
#define TASK_NICE(p)		PRIO_TO_NICE((p)->static_prio)

void account_user_time(struct task_struct *p, cputime_t cputime)
{
	struct cpu_usage_stat *cpustat = &kstat_this_cpu.cpustat;
	cputime64_t tmp;

	p->utime = cputime_add(p->utime, cputime);	// ここだけが重要
						// p->utime += cputime 
						// p->uitme += 1
	/* Add user time to cpustat. */
	tmp = cputime_to_cputime64(cputime);
	if (TASK_NICE(p) > 0)
		cpustat->nice = cputime64_add(cpustat->nice, tmp);
	else
		cpustat->user = cputime64_add(cpustat->user, tmp);
}

oid account_system_time(struct task_struct *p, int hardirq_offset,
			 cputime_t cputime)
{
	struct cpu_usage_stat *cpustat = &kstat_this_cpu.cpustat;
	runqueue_t *rq = this_rq();
	cputime64_t tmp;

	p->stime = cputime_add(p->stime, cputime);	// ここだけが重要

	/* Add system time to cpustat. */
	tmp = cputime_to_cputime64(cputime);
	if (hardirq_count() - hardirq_offset)
		cpustat->irq = cputime64_add(cpustat->irq, tmp);
	else if (softirq_count())
		cpustat->softirq = cputime64_add(cpustat->softirq, tmp);
	else if (p != rq->idle)
		cpustat->system = cputime64_add(cpustat->system, tmp);
	else if (atomic_read(&rq->nr_iowait) > 0)
		cpustat->iowait = cputime64_add(cpustat->iowait, tmp);
	else
		cpustat->idle = cputime64_add(cpustat->idle, tmp);
	/* Account for system time used */
	acct_update_integrals(p);
}


```

# linux-5.15

include/asm/ptrace.h

#define user_mode(regs) \
        (((regs)->pstate & PSR_MODE_MASK) == PSR_MODE_EL0t)

uapi/asm/ptrace.h

#define PSR_MODE_EL0t   0x00000000
#define PSR_MODE_EL1t   0x00000004
#define PSR_MODE_EL1h   0x00000005
#define PSR_MODE_MASK   0x0000000f

xv6ではtrap frameにspsr_el1を保存しているのでこれが使える

    mrs     x9, spsr_el1
    stp     x9, x10, [sp, #-16]!
    mov     x0, sp
    bl      trap

spsr_el1: M[3:0]
    0b0000    EL0t
    0b0100  EL1t
    ob0101  EL1h