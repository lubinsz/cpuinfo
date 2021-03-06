# CPU INFOrmation library

cpuinfo is a library to detect essential for performance optimization information about host CPU.

## Examples

Detect if target is a 32-bit or 64-bit ARM system:

```c
#if CPUINFO_ARCH_ARM || CPUINFO_ARCH_ARM64
    /* 32-bit ARM-specific code here */
#endif
```

Check if the host CPU support ARM NEON
```c
cpuinfo_initialize();
if (cpuinfo_has_arm_neon()) {
    neon_implementation(arguments);
}
```

Check if the host CPU supports x86 AVX
```c
cpuinfo_initialize();
if (cpuinfo_has_x86_avx()) {
    avx_implementation(arguments);
}
```

Check if the thread runs on a Cortex-A53 core
```c
cpuinfo_initialize();
switch (cpuinfo_get_current_core()->uarch) {
    case cpuinfo_uarch_cortex_a53:
        cortex_a53_implementation(arguments);
        break;
    default:
        generic_implementation(arguments);
        break;
}
```

Get the size of level 1 data cache on the fastest core in the processor (e.g. big core in big.LITTLE ARM systems):
```c
cpuinfo_initialize();
const size_t l1_size = cpuinfo_get_processor(0)->l1d->size;
```

Pin thread to cores sharing L2 cache with the current core (Linux or Android)
```c
cpuinfo_initialize();
cpu_set_t cpu_set;
CPU_ZERO(&cpu_set);
const struct cpuinfo_cache* current_l2 = cpuinfo_get_current_processor()->l2;
for (uint32_t i = 0; i < current_l2->processor_count; i++) {
    CPU_SET(cpuinfo_get_processor(current_l2->processor_start + i).linux_id, &cpu_set);
}
pthread_setaffinity_np(pthread_self(), sizeof(cpu_set_t), &cpu_set);
```

## Exposed information
- [x] Processor (SoC) name
  - [x] Integrated GPU name (Android only)
- [x] Microarchitecture
- [x] Usable instruction sets
- [ ] CPU frequency
- [x] Cache
  - [x] Size
  - [x] Associativity
  - [x] Line size
  - [x] Number of partitions
  - [x] Flags (unified, inclusive, complex hash function)
  - [x] Topology (logical processors that share this cache level)
- [ ] TLB
  - [ ] Number of entries
  - [ ] Associativity
  - [ ] Covered page types (instruction, data)
  - [ ] Covered page sizes
- [x] Topology information
  - [x] Logical processors
  - [x] Cores
  - [x] Packages (sockets)

## Supported environments:
- [x] Android
  - [x] x86 ABI
  - [x] x86_64 ABI
  - [x] armeabi ABI
  - [x] armeabiv7-a ABI
  - [x] arm64-v8a ABI
  - [ ] mips ABI
  - [ ] mips64 ABI
- [x] Linux
  - [x] x86
  - [x] x86-64
  - [x] 32-bit ARM (ARMv5T and later)
  - [x] ARM64
  - [ ] PowerPC64
- [ ] iOS
  - [ ] ARMv7
  - [ ] ARM64
- [x] OS X
  - [x] x86
  - [x] x86-64
- [ ] Windows
  - [ ] x86
  - [ ] x86-64
- [ ] Native Client
  - [ ] x86
  - [x] x86-64
  - [ ] ARMv7-A
- [ ] Portable Native Client
- [ ] Emscripten

## Features

- Processor (SoC) name detection
  - [x] Using CPUID leaves 0x80000002–0x80000004 on x86/x86-64
  - [x] Using `/proc/cpuinfo` on ARM
  - [ ] Using kernel log (`dmesg`) on ARM
  - [x] Using `ro.chipname`, `ro.board.platform`, `ro.product.board` properties (Android)
- Vendor and microarchitecture detection
  - [x] Intel-designed x86/x86-64 cores (up to Kaby Lake, Airmont, and Knights Mill)
  - [x] AMD-designed x86/x86-64 cores (up to Puma/Jaguar and Zen)
  - [ ] VIA-designed x86/x86-64 cores
  - [ ] Other x86 cores (DM&P, RDC, Transmeta, Cyrix, Rise)
  - [x] ARM-designed ARM cores (up to Cortex-A17, Cortex-75)
  - [x] Qualcomm-designed ARM cores (up to Kryo and Kryo-280)
  - [x] nVidia-designed ARM cores (Denver)
  - [x] Samsung-designed ARM cores (Mongoose)
  - [x] Intel-designed ARM cores (XScale up to 3rd-gen)
  - [ ] Apple-designed ARM cores (up to Hurricane)
  - [x] Cavium-designed ARM cores (ThunderX)
  - [ ] AppliedMicro-designed ARM cores
- Instruction set detection
  - [x] Using CPUID (x86/x86-64)
  - [x] Using dynamic code generation validator (Native Client/x86-64)
  - [x] Using `/proc/cpuinfo` on 32-bit ARM EABI (Linux)
  - [x] Using microarchitecture heuristics on (32-bit ARM)
  - [x] Using `FPSID` and `WCID` registers (32-bit ARM)
  - [ ] Using `getauxval` or `/proc/self/auxv` (Linux)
  - [ ] Using instruction probing on ARM (Linux)
  - [ ] Using CPUID registers on ARM64 (Linux)
- Cache detection
  - [x] Using CPUID leaf 0x00000002 (x86/x86-64)
  - [x] Using CPUID leaf 0x00000004 (non-AMD x86/x86-64)
  - [ ] Using CPUID leaves 0x80000005-0x80000006 (AMD x86/x86-64)
  - [x] Using CPUID leaf 0x8000001D (AMD x86/x86-64)
  - [x] Using `/proc/cpuinfo` (Linux/pre-ARMv7)
  - [x] Using microarchitecture heuristics (ARM)
  - [x] Using chipset name (ARM)
  - [x] Using `sysctlbyname` (Mach)
  - [x] Using sysfs `typology` directories (ARM/Linux)
  - [ ] Using sysfs `cache` directories (Linux)
  - [ ] Using `clGetDeviceInfo` with `CL_DEVICE_GLOBAL_MEM_CACHE_SIZE`/`CL_DEVICE_GLOBAL_MEM_CACHELINE_SIZE` parameters (Android)
- TLB detection
  - [x] Using CPUID leaf 0x00000002 (x86/x86-64)
  - [ ] Using CPUID leaves 0x80000005-0x80000006 and 0x80000019 (AMD x86/x86-64)
  - [x] Using microarchitecture heuristics (ARM)
- Topology detection
  - [x] Using CPUID leaf 0x00000001 on x86/x86-64 (legacy APIC ID)
  - [x] Using CPUID leaf 0x0000000B on x86/x86-64 (Intel APIC ID)
  - [ ] Using CPUID leaf 0x8000001E on x86/x86-64 (AMD APIC ID)
  - [x] Using `host_info` (Mach)
  - [x] Using sysfs (Linux)
  - [x] Using chipset name (ARM/Linux)
