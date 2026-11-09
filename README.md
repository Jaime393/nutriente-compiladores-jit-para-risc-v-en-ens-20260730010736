# nutriente-compiladores-jit-para-risc-v-en-ens-20260730010736
Cazado en Senegal. IA exótica 2026

# Compiladores JIT para RISC-V

Un compilador JIT (just-in-time) genera código máquina en tiempo de ejecución, mientras el programa corre, en vez de compilarlo todo antes. Observa qué código se ejecuta de verdad y lo optimiza con esa información. En la práctica: reserva memoria escribible (`mmap` con `PROT_READ|PROT_WRITE`), escribe los bytes de las instrucciones, la vuelve ejecutable (`mprotect` con `PROT_READ|PROT_EXEC`) y la llama. Así funcionan V8, HotSpot, LLVM ORC y los runtimes de WebAssembly.

RISC-V es una ISA abierta y modular: el mismo nombre cubre desde microcontroladores de 32 bits hasta servidores de 64 bits con vector. Eso hace que "JIT para RISC-V" no sea una sola cosa: cada JIT apunta a una configuración concreta (RV32 o RV64, con qué extensiones) y el estado del soporte varía mucho de un runtime a otro. Esta pieza cuenta qué hay de verdad, con fuentes, y qué falta.

## 1. Qué tiene de específico RISC-V para un JIT

**Las instrucciones no miden todas lo mismo.** La base es de 32 bits, pero la extensión C añade instrucciones comprimidas de 16 bits que se entremezclan libremente con las de 32 bits (alineación de 16 bits). El manual de la ISA dice que típicamente el 50–60 % de las instrucciones de un programa se pueden reemplazar por versiones comprimidas, con una reducción del tamaño del código del 25–30 %; Waterman (2011) midió que se leen un 25–30 % menos de bits de instrucción, lo que reduce los fallos de caché de instrucciones un 20–25 %, más o menos lo mismo que duplicar la caché de instrucciones. Para un JIT eso es doble filo: el código generado ocupa menos (la caché de código rinde más), pero el emisor ya no puede suponer que cada instrucción mide 4 bytes. El bug 238008 de WebKit lo dice claro: en el JIT de JavaScriptCore el tamaño fijo de 32 bits se daba por sentado al enlazar y reparchear el código generado, y desacoplar esa suposición es trabajo.

**Muchos registros.** El backend de OpenJDK para RISC-V define 32 registros de propósito general (x0–x31), 32 de punto flotante (f0–f31) y 32 vectoriales (v0–v31 cuando RVV está activo). Un JIT tiene dónde asignar, pero cada backend debe describir sus clases de registros y sus patrones de instrucción.

**Las extensiones son el objetivo, no el adorno.** Un JIT para RISC-V compila para una configuración: RV64GC es la base habitual (G = IMAFD, C = comprimidas), y luego V (vector) y B (manipulación de bits: Zba, Zbb) si el hardware las tiene. El JIT de RandomX, por ejemplo, optimiza para Zba (generación de direcciones) y Zbb (bits), y como la mayoría de procesadores RISC-V no tienen aceleración AES por hardware, implementa AES por software. En Wasmtime, el soporte SIMD de WebAssembly en riscv64 exige la extensión V (ajuste `has_v` en Cranelift); sin ella hay que desactivar esas propuestas o el motor no arranca.

**NO SÉ (no verificado en las fuentes):** cómo debe un JIT emitir barreras de memoria (`fence`) bajo el modelo de memoria débil de RISC-V (RVWMO). Es un punto real del diseño de JITs en esta ISA, pero no se encontró en las fuentes consultadas y no se afirma aquí.

## 2. Las implementaciones reales

| Runtime | JIT en RISC-V | Estado verificado |
|---|---|---|
| OpenJDK HotSpot (Java) | C1 y C2 | Integrado en JDK 19 (JEP 422, septiembre 2022). Apunta a RV64GV; incluye intérprete, C1, C2 y todos los recolectores, incluidos ZGC y Shenandoah. Autor del port: Huawei. |
| V8 (JavaScript/WebAssembly) | Sparkplug, Maglev, TurboFan | Integrado en el árbol principal desde el primer commit RISC-V (2021-02-09). 913 commits RISC-V. Cerca de la paridad con x86_64/ARM64; la suite completa de JetStream pasa (informe RISE, diciembre 2025). RV32 en desuso desde finales de 2025. |
| SpiderMonkey (Firefox) | Baseline JIT + JIT optimizador + Wasm | Backend completo en `js/src/jit/riscv64/`, en el árbol desde 2023-01-23 (Bug 1800431). JIT activado por defecto en riscv64 desde Firefox 143 (Bug 1982266). |
| LLVM ORC / JITLink | ORC (LLJIT) | JITLink con ELF maduro en RISC-V (junto a x86-64, arm64, LoongArch, PowerPC 64). El backend RISC-V de LLVM está completo en mainline: RVV 1.0, Zb*, Zk*, GlobalISel, BOLT, LLD, LLDB. |
| Wasmtime / Cranelift (WebAssembly) | Cranelift | Backend riscv64 en Wasmtime desde v2.0.0 (2022-10-20). Nivel 3 (no listo para producción): le falta un mantenedor a tiempo completo. Cranelift 0.127.0 (22 dic 2025); Wasmtime 47.0.0 (20 jul 2026) añadió Zvbb. |
| LuaJIT (Lua) | — | Sin backend RISC-V en el upstream. El port usable es el fork de PLCT Lab, experimental: solo 64 bits, solo little-endian, ABI de punto flotante duro, GC64 siempre. OpenResty lo lista como "TBA" (RVA22+). |
| RandomX (prueba de trabajo) | JIT propio (RV64) | `JitCompilerRV64` traduce el bytecode de RandomX a RISC-V nativo en ejecución. Límite: máximo 494 literales IMUL_RCP. Sin AES por hardware. |
| AtomVM (Erlang/BEAM) | JIT opcional | Soporta riscv32 (ESP32Cx, ESP32Hx, ESP32P4) y riscv64 (Linux). El JIT en tiempo de carga exige Erlang/OTP 28 o posterior. |
| TornadoVM + OCK (Java) | JIT en dos etapas | TornadoVM compila bytecode Java a OpenCL/SPIR-V; OCK lo baja a binario RISC-V con vector. Aceleraciones de hasta 33x frente a Java de un hilo y 4,6x frente a Java multihilo (Spacemit K1, RISC-V Summit Europe 2025). |
| AdaptiveCPP / SYCL | SSCP JIT | JIT de un solo paso que despacha a GPU y a RISC-V/OCK con SPIR-V. En SYCLOPS superó a NVCC y a DPC++ en un 30 % / 23 % en media geométrica (entregable D4.2, sep 2025). |
| PyVedas / Tiny-Vedas (IA) | JIT de modelos | Compilador JIT que toma un grafo de PyTorch y genera C que compila el GCC estándar de RISC-V. Probado con YOLOv3-Tiny int8 en FPGA Xilinx Alveo U280. Sentido distinto de JIT: compila modelos, no un lenguaje. |

## 3. Rendimiento y límites, con números

- **Java en RISC-V ya no es interpretado.** Antes de JEP 422, Java en RISC-V corría sobre el intérprete de cero (Zero) o con JITs externos. El documento de PLCT Lab registra que portar Firefox/SpiderMonkey con JIT a RV64GCV aceleró las aplicaciones Java más de 100x frente a OpenJDK/Zero. Esa cifra es del propio proyecto (PLCT), no de una medición independiente.
- **La vectorización paga.** TornadoVM + OCK llega a 33x frente a Java de un hilo y 4,6x frente a Java multihilo en multiplicación de matrices, en un Spacemit K1 de 8 núcleos a 1,6 GHz con 4 GB de RAM. Pero para matrices de hasta 64x64 el Java secuencial sigue ganando: el JIT no compensa el coste de lanzar el trabajo.
- **RVC paga en densidad, no en velocidad directa.** 25–30 % menos de código, 20–25 % menos fallos de I-caché (Waterman 2011). Es un efecto de caché, no de cómputo.
- **El soporte no es uniforme.** Wasmtime en riscv64 es Nivel 3 y sin mantenedor a tiempo completo; V8 en RV32 se abandona; LuaJIT no existe en upstream. El que busque "JIT para RISC-V" debe preguntar primero por el runtime concreto y por su nivel de soporte, no por la ISA en general.
- **El hardware manda.** RandomX en RISC-V no tiene AES por hardware y lo hace por software; TornadoVM necesita RVV 1.0; Wasmtime necesita V para SIMD. Un JIT que no conoce las extensiones del chip genera código genérico y lento.

## 4. Cómo se arma la pila

1. **Elegir el runtime por su nivel real.** Java: OpenJDK 19 o posterior (JEP 422). JavaScript: V8 o SpiderMonkey, ambos en el árbol principal. WebAssembly: Wasmtime/Cranelift, sabiendo que es Nivel 3. Lua: el fork de PLCT Lab, no el upstream. Lenguajes nuevos: LLVM ORC/JITLink sobre el backend RISC-V.
2. **Fijar la configuración de ISA.** RV64GC como base; añadir V y B solo si el hardware las tiene y el JIT las usa. Un binario RV64GV no corre en un chip sin V.
3. **Medir en el hardware real.** Los números de esta pieza vienen de un Spacemit K1 (TornadoVM) y de la suite JetStream (V8). No hay una cifra única de "el JIT en RISC-V es Nx más rápido": depende del runtime, del chip y de la carga.

## 5. Qué queda sin saber (NO SÉ)

- Qué significa "en-ens" en el nombre de este repo. No se interpreta.
- Qué significa "Cazado en Senegal. IA exótica 2026". Se conserva tal cual, sin interpretar.
- Si existe un JIT de .NET (RyuJIT) o de Go para RISC-V. No se verificó en fuentes primarias y no se afirma.
- Cuánto cuesta compilar en JIT frente a AOT en placas RISC-V pequeñas (tiempo de calentamiento, memoria de la caché de código). No se encontró una medición con fuente.
- Si V8 en RISC-V iguala las puntuaciones de JetStream de x86_64/ARM64. Verificado: la suite completa pasa; no verificadas: las puntuaciones.
- El coste de las barreras de memoria (`fence`) en JITs bajo RVWMO (ver §1).

---

*Solo SÉ con fuente. Lo demás queda marcado NO SÉ, no se rellena.*
