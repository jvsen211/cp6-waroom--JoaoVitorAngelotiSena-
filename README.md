## RODADA 1: O saldo que virou negativo (19h12)
**Tipo:** rodada · **Voto:** C
**Justificativa:** O `catch` engolia a `SaldoInsuficienteException` e o `debitar()` rodava sempre, então a regra de saldo nunca era aplicada. Escolhemos corrigir a causa e escrever o `assertThrows` antes de subir: demora 40 min a mais (8 clientes afetados), mas o teste protege a invariante e impede que a quebra de regra volte sem ninguém perceber.
**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 35 mil · 🧹 1

## RODADA 2: A injeção que derreteu o banco (21h02)
**Tipo:** rodada · **Voto:** A
**Justificativa:** A concatenação de String misturava dado do usuário com código SQL, e por isso `' OR '1'='1` virava parte da query. O `PreparedStatement` usa parâmetros vinculados, que tratam a entrada sempre como valor e nunca como SQL, e corrige a causa em ~40 min. O trade-off é manter o acesso manual via JDBC, que exige cuidado com recursos (`close()`), enquanto a derived query (D) evitaria isso.
**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 35 mil · 🧹 1

## ⚡ EVENTO RELÂMPAGO B: As conexões que não fecham (22h40)
**Tipo:** relâmpago · **Voto:** C
**Justificativa:** O pool esgotou porque Connection, Statement e ResultSet eram abertos e nunca fechados. Votamos C (delay entre requisições) para dar tempo ao banco de processar, mas o delay só alivia o sintoma e aumenta a latência na Black Friday, sem corrigir o vazamento. A correção real seria o `try-with-resources` (A), que garante o `close()` mesmo com exceção.
**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 35 mil · 🧹 2

## RODADA 3: A noite das contas gêmeas (22h50)
**Tipo:** rodada · **Voto:** C
**Justificativa:** O `getInstancia()` criava um objeto novo a cada chamada, então o contador recomeçava em 1001 e gerava números duplicados. Como o número da conta é identificador único perante o Bacen, escolhemos auditar as duplicatas antes de mexer no código, para saber quantos clientes já estão afetados e se há extratos cruzados. O custo é que o fix só entra às 04h, mantendo o risco de novas duplicatas até lá.
**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 45 mil · 🧹 

## RODADA 4: O comprovante que aponta pra pessoa errada (00h47)
**Tipo:** rodada · **Voto:** B
**Justificativa:** Os saldos estavam certos, mas o `new Transferencia(...)` recebia origem e destino trocados, então só o registro do extrato saía invertido. Corrigir a gravação e reprocessar o histórico leva ~2h, mas elimina a dívida de dados e entrega comprovantes corretos para auditoria. A opção A deixaria extratos antigos errados, e a C sobrecarregaria o SAC a madrugada toda.
**Placar do grupo após esta decisão:** 🔥 9 · 💰 R$ 70 mil · 🧹 1


## RODADA 5: A fábrica que devolve null (01h30)
**Tipo:** rodada · **Voto:** A
**Justificativa:** O `default -> null` da factory deixava o erro estourar longe da causa, no `save()`, com uma mensagem que não cita o problema. Fail-fast, com uma exceção clara na própria factory ("Tipo de conta inválido"), faz o erro nascer e morrer no mesmo lugar. A validação no controller (B) ficaria espalhada, e devolver CORRENTE por padrão (C) criaria contas erradas silenciosamente.
**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 70 mil · 🧹 1