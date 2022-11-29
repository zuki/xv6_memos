# Linuxにおけるclean_and_invalidate_data_cache_range関連コード

## arch/arm64/kernel/cpu_errta.c

```
const struct arm64_cpu_capabilities arm64_errata[] = {
    {
        .desc = "Mismatched cache type (CTR_EL0)",
        .capability = ARM64_MISMATCHED_CACHE_TYPE,
        .matches = has_mismatched_cache_type,
        .type = ARM64_CPUCAP_LOCAL_CPU_ERRATUM,
        .cpu_enable = cpu_enable_trap_ctr_access,
    },
}
```

## arch/arm64/include/asm/alternative-macros.h

```
/*
 * Begin an alternative code sequence.
 */
.macro alternative_if_not cap
    .set .Lasm_alt_mode, 0
    .pushsection .altinstructions, "a"
    altinstruction_entry 661f, 663f, \cap, 662f-661f, 664f-663f
    .popsection
661:
.endm

/*
 * Provide the other half of the alternative code sequence.
 */
.macro alternative_else
662:
    .if .Lasm_alt_mode==0
    .subsection 1
    .else
    .previous
    .endif
663:
.endm
```

## arch/arm64/include/assembler.h

```
/*
 * read_ctr - read CTR_EL0. If the system has mismatched register fields,
 * provide the system wide safe value from arm64_ftr_reg_ctrel0.sys_val
 */
    .macro  read_ctr, reg
#ifndef __KVM_NVHE_HYPERVISOR__
alternative_if_not ARM64_MISMATCHED_CACHE_TYPE
    mrs \reg, ctr_el0           // read CTR
    nop
alternative_else
    ldr_l   \reg, arm64_ftr_reg_ctrel0 + ARM64_FTR_SYSVAL
alternative_endif
#else
alternative_if_not ARM64_KVM_PROTECTED_MODE
    ASM_BUG()
alternative_else_nop_endif
alternative_cb kvm_compute_final_ctr_el0
    movz    \reg, #0
    movk    \reg, #0, lsl #16
    movk    \reg, #0, lsl #32
    movk    \reg, #0, lsl #48
alternative_cb_end
#endif
    .endm

/*
 * dcache_line_size - get the safe D-cache line size across all CPUs
 */
    .macro  dcache_line_size, reg, tmp
    read_ctr    \tmp
    ubfm        \tmp, \tmp, #16, #19    // cache line size encoding
    mov     \reg, #4        // bytes per word
    lsl     \reg, \reg, \tmp    // actual cache line size
    .endm

/*
 * Macro to perform a data cache maintenance for the interval
 * [start, end)
 *
 *  op:     operation passed to dc instruction
 *  domain:     domain used in dsb instruciton
 *  start:          starting virtual address of the region
 *  end:            end virtual address of the region
 *  fixup:      optional label to branch to on user fault
 *  Corrupts:       start, end, tmp1, tmp2
 */
    .macro dcache_by_line_op op, domain, start, end, tmp1, tmp2, fixup
    dcache_line_size \tmp1, \tmp2
    sub \tmp2, \tmp1, #1
    bic \start, \start, \tmp2
.Ldcache_op\@:
    .ifc    \op, cvau
    __dcache_op_workaround_clean_cache \op, \start
    .else
    .ifc    \op, cvac
    __dcache_op_workaround_clean_cache \op, \start
    .else
    .ifc    \op, cvap
    sys 3, c7, c12, 1, \start   // dc cvap
    .else
    .ifc    \op, cvadp
    sys 3, c7, c13, 1, \start   // dc cvadp
    .else
    dc  \op, \start
    .endif
    .endif
    .endif
    .endif
    add \start, \start, \tmp1
    cmp \start, \end
    b.lo    .Ldcache_op\@
    dsb \domain

    _cond_extable .Ldcache_op\@, \fixup
    .endm
```

## arch/arm64/mm/cache.S

```
/*
 *  dcache_clean_inval_poc(start, end)
 *
 *  Ensure that any D-cache lines for the interval [start, end)
 *  are cleaned and invalidated to the PoC.
 *
 *  - start   - virtual start address of region
 *  - end     - virtual end address of region
 */
SYM_FUNC_START_PI(dcache_clean_inval_poc)
    dcache_by_line_op civac, sy, x0, x1, x2, x3
    ret
SYM_FUNC_END_PI(dcache_clean_inval_poc)
```
