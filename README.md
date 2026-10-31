## RODADA 1: O saldo que virou negativo (19h12)
**Tipo:** rodada · **Voto:** C
**Justificativa:** O `catch` engolia a `SaldoInsuficienteException` e o `debitar()` rodava sempre, então a regra de saldo nunca era aplicada. Escolhemos corrigir a causa e escrever o `assertThrows` antes de subir: demora 40 min a mais (8 clientes afetados), mas o teste protege a invariante e impede que a quebra de regra volte sem ninguém perceber.
**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 35 mil · 🧹 1

## RODADA 2: A injeção que derreteu o banco (21h02)
**Tipo:** rodada · **Voto:** A
**Justificativa:** A concatenação de String misturava dado do usuário com código SQL, e por isso `' OR '1'='1` virava parte da query. O `PreparedStatement` usa parâmetros vinculados, que tratam a entrada sempre como valor e nunca como SQL, e corrige a causa em ~40 min. O trade-off é manter o acesso manual via JDBC, que exige cuidado com recursos (`close()`), enquanto a derived query (D) evitaria isso.
**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 35 mil · 🧹 1