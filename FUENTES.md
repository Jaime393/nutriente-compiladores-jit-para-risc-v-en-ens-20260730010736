# Fuentes — Compiladores JIT para RISC-V

Cada fuente con su URL y su fecha, y qué afirma de lo que dice el README. Todas verificadas en primario o en informe con verificación cruzada declarada. Fecha de consulta: 2026-11-09.

## 1. Manual de la ISA RISC-V, Vol. I — Extensión C (instrucciones comprimidas)

- URL: https://www.five-embeddev.com/riscv-user-isa-manual/IMFDQC-Ratification-20190305/c
- Qué afirma: la extensión C añade codificaciones de 16 bits para operaciones comunes; típicamente el 50–60 % de las instrucciones de un programa se pueden reemplazar, con una reducción del tamaño del código del 25–30 %. Waterman (2011): se leen un 25–30 % menos de bits de instrucción, lo que reduce los fallos de caché de instrucciones un 20–25 %, más o menos lo mismo que duplicar la caché de instrucciones. Las instrucciones de 16 y 32 bits se entremezclan libremente (IALIGN=16).

## 2. JEP 422: Linux/RISC-V Port (OpenJDK 19)

- URL: https://bell-sw.com/announcements/2022/05/18/what-is-risc-v-and-when-is-the-java-port-coming/
- Fecha del artículo: 18 may 2022 (actualizado 18 ene 2024).
- Qué afirma: el port a Linux/RISC-V se integró en JDK 19 (JEP 422); soporta la configuración RV64GV; incluye el intérprete, los compiladores JIT C1 y C2 y todos los recolectores principales, incluidos ZGC y Shenandoah. Autor del port: Huawei. Confirmado también en las notas de JDK 19 (JDK-8276799, implementación de JEP 422).

## 3. V8 en RISC-V — informe RISE (sw-ecosystem)

- URL: https://github.com/riseproject-dev/sw-ecosystem/blob/HEAD/project-reports/v8.md
- Fecha del informe: 30 sep 2026.
- Qué afirma: primer commit RISC-V en el árbol principal de V8: `ffd9e82d`, 2021-02-09. 913 commits RISC-V. El backend lo mantiene la comunidad (ISCAS/PLCT Lab), no Google. Cerca de la paridad con x86_64/ARM64; la suite completa de JetStream pasa (blog RISE, 9 dic 2025). Soporte RV32 en desuso desde finales de 2025, mantenimiento hasta mayo 2026.

## 4. SpiderMonkey en RISC-V — informe RISE (sw-ecosystem)

- URL: https://github.com/riseproject-dev/sw-ecosystem/blob/HEAD/project-reports/spidermonkey.md
- Fecha del informe: 20 jul 2026.
- Qué afirma: backend JIT riscv64 completo (`js/src/jit/riscv64/`), en el árbol desde 2023-01-23 (Bug 1800431, tres commits). JIT activado por defecto en riscv64 desde Firefox 143 (Bug 1982266). Port del backend RISC-V de V8. Sin cobertura de CI de Mozilla; el port lo mantienen PLCT Lab y contribuidores.

## 5. LLVM — JITLink y ORC

- URL: https://llvm.org/docs/JITLink.html
- Qué afirma: JITLink es la librería de enlazado JIT de ORC; el backend ELF es maduro y cubre x86-64, arm64, RISC-V, LoongArch, PowerPC 64, arm32, SystemZ y Hexagon (estado a mayo 2026).
- Complemento: informe RISE sobre LLVM (10 sep 2026), https://github.com/riseproject-dev/sw-ecosystem/blob/HEAD/project-reports/llvm.md — el backend RISC-V vive completo en mainline: ~150 archivos, RVV 1.0, Zb*, Zk*, GlobalISel, BOLT, LLD, LLDB, JITLink, compiler-rt.

## 6. Wasmtime / Cranelift en RISC-V — informe RISE (sw-ecosystem)

- URL: https://github.com/riseproject-dev/sw-ecosystem/blob/HEAD/project-reports/wasmtime.md
- Fecha del informe: 10 sep 2026.
- Qué afirma: backend riscv64 de Cranelift en Wasmtime desde v2.0.0 (2022-10-20, PR #4271). Nivel 3 (no listo para producción): a `riscv64gc-unknown-linux-gnu` le falta un mantenedor a tiempo completo. Confirmado en la política de niveles de Wasmtime: https://docs.wasmtime.dev/stability-tiers.html
- Versiones: Cranelift 0.127.0 (22 dic 2025); Wasmtime 47.0.0 (20 jul 2026) añadió soporte de Zvbb en Cranelift.

## 7. TornadoVM + OCK en RISC-V — RISC-V Summit Europe 2025

- URL: https://pure.manchester.ac.uk/ws/portalfiles/portal/361946410/main.pdf
- Fecha: 12–15 may 2025, París. Autores: Fumero Alfonso et al.
- Qué afirma: cadena de dos etapas (TornadoVM JIT: bytecode Java → OpenCL/SPIR-V; OCK: → binario RISC-V con vector). Aceleraciones de hasta 33x frente a Java de un hilo y 4,6x frente a Java multihilo. Hardware: Spacemit K1 de 8 núcleos a 1,6 GHz, 4 GB de RAM, Bianbu OS 1.0.5. Software: TornadoVM 1.0.10-dev, OCK, LLVM 19.1.5, GCC 13.2, OpenJDK 21.0.5. Para matrices de hasta 64x64 el Java secuencial sigue ganando.

## 8. RandomX — JIT para RISC-V (DeepWiki)

- URL: https://deepwiki.com/tevador/RandomX/4.4-jit-compiled-vm-for-risc-v
- Qué afirma: `JitCompilerRV64` traduce el bytecode de RandomX a instrucciones nativas RISC-V de 64 bits en ejecución. Optimizaciones para Zba (generación de direcciones) y Zbb (manipulación de bits). Sin AES por hardware en la mayoría de procesadores RISC-V: implementación por software. Límite: máximo 494 literales IMUL_RCP.

## 9. LuaJIT en RISC-V — fork PLCT Lab (experimental) y ausencia en upstream

- URL: https://deepwiki.com/plctlab/LuaJIT/2.2-architecture-and-platform-support
- Qué afirma: el soporte RISC-V64 en el fork de PLCT Lab es experimental: solo 64 bits, solo little-endian, ABI de punto flotante duro, GC64 siempre activado. Archivo `vm_riscv64.dasc` con instrucciones comprimidas.
- Ausencia en upstream: OpenResty LuaJIT2, https://openresty-luajit2.jetzou.com/doc/tag:v2.1-20220915/install.html — RISC-V 64 (RVA22+) listado como "TBA". Neovim, issue #29817, https://github.com/neovim/neovim/issues/29817 — ppc64el, riscv64 y s390x no soportan LuaJIT en upstream.

## 10. AtomVM — compilación JIT

- URL: https://doc.atomvm.org/main/jit.html
- Qué afirma: AtomVM incluye un JIT opcional que traduce bytecodes BEAM a código nativo en tiempo de carga. Arquitecturas soportadas: riscv32 (ESP32Cx, ESP32Hx, ESP32P4) y riscv64 (Linux). El JIT en tiempo de carga exige Erlang/OTP 28 o posterior.

## 11. WebKit, bug 238008 — RVC y el JIT de JavaScriptCore

- URL: https://bugs.webkit.org/show_bug.cgi?id=238008
- Qué afirma: el soporte inicial de JavaScriptCore para RISC-V se basaba en instrucciones de 32 bits; el tamaño fijo de 32 bits se daba por sentado al enlazar y reparchear el código generado por el JIT, así que desacoplar esa suposición para la extensión C es trabajo. Cita la cifra del manual: 50–60 % reemplazable, 25–30 % menos de código.

## 12. SYCLOPS, entregable D4.2 — Compilador con autovectorización

- URL: https://www.syclops.org/static/files/deliverables/SYCLOPS-D4.2_CompilerWithAutovectorization_20251006.pdf
- Fecha: presentación 30 sep 2025, oficial 6 oct 2025.
- Qué afirma: DPC++ y oneAPI Construction Kit adaptados a RISC-V con RVV 1.0. AdaptiveCPP con JIT SSCP (single-pass) unificado: el único SYCL con esa capacidad. Con optimizaciones en tiempo de JIT, AdaptiveCpp superó a NVCC y a DPC++ en un 30 % / 23 % en media geométrica.

## 13. Tiny-Vedas / PyVedas — JIT de modelos para RISC-V

- URL: https://siliscale.com/papers/tiny-vedas.pdf
- Qué afirma: PyVedas es un compilador JIT que toma un modelo de PyTorch, extrae su grafo y genera código C que compila el GCC estándar de RISC-V. Probado con YOLOv3-Tiny int8 en FPGA Xilinx Alveo U280. El JIT aquí compila modelos, no un lenguaje.

## 14. Qué es un JIT — glosario

- URL: https://datafield.dev/assembly-language/glossary.html
- Qué afirma: el JIT genera código máquina en tiempo de ejecución; requiere reservar memoria escribible (`mmap` con `PROT_READ|PROT_WRITE`), escribir los bytes, volverla ejecutable (`mprotect` con `PROT_READ|PROT_EXEC`) y llamar al código generado. Usado en V8, HotSpot, LLVM ORC y runtimes de WebAssembly.

## 15. OpenJDK — backend RISC-V (estructura)

- URL: https://deepwiki.com/openjdk/jdk21u/3.3.3-risc-v-backend
- Qué afirma: el backend apunta a RV64GC más comprimidas opcionales, con vector y manipulación de bits cuando están disponibles. Define 32 registros generales (x0–x31), 32 de punto flotante (f0–f31) y 32 vectoriales (v0–v31 con RVV). Archivos clave: `assembler_riscv.cpp`, `riscv.ad` (C2), `riscv_v.ad` (vector).
