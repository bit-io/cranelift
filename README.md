# cranelift
Cranelift bindings for bit.io.

Bindingi [Cranelift](https://cranelift.dev) (kompilator JIT) dla
HackerScript. Cały shim jest napisany w HackerScript (`lib/mod.hcs`):
funkcje `fun` z blokami Rust, bez osobnego crate'a, staticlib ani warstwy C.

## Model

Stan Cranelift trzymają uchwyty (`Int`): `jit` (moduł JIT) i `f` (budowana
funkcja). Instrukcje są zapisywane i odtwarzane przez `FunctionBuilder`
dopiero w `cl_finish`. Wartości SSA i bloki to zwykłe `Int`-y; błąd
instrukcji to `-1` (powód: `cl_fn_error`).

Typy w sygnaturach (jeden znak): `i` = i64, `f` = f64, `w` = i32, `b` = i8.

```
let jit = cl_jit_new()
let f = cl_fn_new("add", "ii", "i")
cl_return(f, cl_iadd(f, cl_param(f, 0), cl_param(f, 1)))
let error = cl_finish(jit, f)
log(cl_call_i(jit, "add", [2, 3]))
```

## API (`lib/mod.hcs`)

| Grupa | Funkcje |
|---|---|
| Moduł | `cl_jit_new`, `cl_jit_free`, `cl_jit_triple`, `cl_last_error`, `cl_has`, `cl_signature`, `cl_clif`, `cl_declare` |
| Funkcja | `cl_fn_new`, `cl_fn_free`, `cl_fn_error`, `cl_finish`, `cl_define_clif` |
| Bloki | `cl_block`, `cl_block_param`, `cl_switch`, `cl_param` |
| Stałe | `cl_iconst`, `cl_fconst` |
| Arytmetyka | `cl_iadd` `cl_isub` `cl_imul` `cl_sdiv` `cl_udiv` `cl_srem` `cl_urem` `cl_band` `cl_bor` `cl_bxor` `cl_ishl` `cl_sshr` `cl_ushr` `cl_fadd` `cl_fsub` `cl_fmul` `cl_fdiv` |
| Jednoargumentowe | `cl_ineg` `cl_bnot` `cl_fneg` `cl_fsqrt` `cl_fabs` `cl_i2f` `cl_f2i` `cl_b2i` |
| Porównania | `cl_icmp` (`eq ne slt sle sgt sge ult ule ugt uge`), `cl_fcmp` (`eq ne lt le gt ge`), `cl_select` |
| Sterowanie | `cl_jump`, `cl_brif`, `cl_return`, `cl_return_void`, `cl_call` |
| Wywołanie | `cl_call_i` (tylko i64, do 6 arg.), `cl_call_f` (tylko f64, do 4 arg.) |

Własny tekst CLIF: `cl_define_clif(jit, "nazwa", tekst)`.
Pełny przykład: `examples/jit` (jego `cmd/cranelift.hcs` to kopia `lib/mod.hcs`).

## Wersje i wymagania

Zależności: `cranelift-{codegen,frontend,module,jit,native,reader}` w wersji
**0.130**. Kod Rust z bloków `native` został skompilowany i przetestowany z
tą wersją (Rust 1.91). Nowsze wydania Cranelift (np. 0.136) wymagają
Rusta 1.96+ i nie były sprawdzane - zmiana wersji w liniach `get <crates:...>`
na własne ryzyko.

## Status

Przetestowane: kod Rust z bloków `native` (wyciągnięty 1:1 z `lib/mod.hcs`)
zbudowany jako zwykła biblioteka Rust; test buduje i wywołuje funkcje JIT
(add, silnia z pętlą i parametrami bloków, hypot na f64, wywołania między
funkcjami, select, CLIF z tekstu, błędy i weryfikator IR).

Nieprzetestowane: sam przejazd przez `hackerc`/`virus` (kompilator
HackerScript nie był dostępny) - czyli dokładna składnia opakowań w
HackerScript i generowany kod. Pierwszy `virus build` może wymagać drobnych
poprawek.
