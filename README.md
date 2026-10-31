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

## ⚡ EVENTO RELÂMPAGO C: O cadastro com metade dos campos (02h20)
**Tipo:** relâmpago · **Voto:** B
**Justificativa:** O construtor exige todos os campos, e um terceiro construtor sobrecarregado aumentaria o "construtor telescópico". O Builder concentra os valores padrão e a validação no `build()`, deixando a criação legível e consistente. O custo é uma classe a mais, mas evita `null` espalhado (C).
**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 70 mil · 🧹 1

## ⚡ EVENTO RELÂMPAGO D: O teste verde que não testa nada (03h20)
**Tipo:** relâmpago · **Voto:** A
**Justificativa:** O teste executava `depositar()` mas não tinha nenhum `assert`, então ficava verde mesmo se o depósito quebrasse (falsa segurança). Adicionar o `assertEquals` leva ~10 min e faz o teste verificar o comportamento de verdade. Apagá-lo (C) derrubaria a cobertura, e deixá-lo assim (B) esconderia regressões.
**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 70 mil · 🧹 0

## RODADA 6: O teste vermelho às 03h45
**Tipo:** rodada · **Voto:** B
**Justificativa:** O teste vermelho era o alarme certo: o contrato diz TED = R$ 5,00 e o código cobrava R$ 50,00 por causa de um número mágico sem constante. Desabilitar (A), ignorar (C) ou ajustar o teste para 50.0 (D) manteriam a cobrança errada no ar. Investigar custa a janela de deploy das 04h, mas corrige a causa (extraindo uma constante) e evita cobrar 10x a mais dos clientes na Black Friday.
**Placar do grupo após esta decisão:** 🔥 7 · 💰 R$ 100 mil · 🧹 0

## ⚡ EVENTO RELÂMPAGO E: "A taxa é 5 ou 50?" (05h58)
Não afetado: a R6 foi B (suíte saudável), então o grupo já tinha investigado e sabia que a taxa correta era R$ 5. Nenhuma decisão registrada.

## RODADA 7: Go/No-Go da Black Friday (06h15)
**Tipo:** rodada · **Voto:** A
**Justificativa:** Como a suíte está verde de verdade (o `assertThrows` da R1, o assert do ⚡D e a taxa corrigida na R6), ela segura o pico e não há bug conhecido bloqueando a abertura. Abrir 100% no horário preserva o faturamento da campanha, sem o atraso de 2h da opção B nem a perda da BF das opções C e D. O custo é a dívida técnica: o `TransferenciaService` de 80 linhas continua concentrando tudo, e a refatoração precisa ser agendada para não virar "a correção de segunda nunca chega".
**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 100 mil · 🧹 1

## 🏁 Epílogo: placar final congelado
🔥 8 · 💰 R$ 100 mil · 🧹 1
**Perfil da noite:** A Noite dos Heróis (🔥 7 ou mais e 🧹 até 2)

---
# 🩺 Pós-mortem: relatório de incidente da madrugada

## Linha do tempo
| Hora | Incidente | Mecanismo | Nossa escolha |
|---|---|---|---|
| 19h12 | R1: saldo negativo | exceção lançada e engolida pelo `catch` | C |
| 21h02 | R2: injeção de SQL | concatenação de SQL | A |
| 22h40 | ⚡B: conexões que não fecham | vazamento sem `close()` | C |
| 22h50 | R3: contas gêmeas | Singleton sem instância guardada | C |
| 00h47 | R4: comprovante invertido | origem/destino trocados na entidade gravada | B |
| 01h30 | R5: factory devolve null | `return null` explodindo longe da causa | A |
| 02h20 | ⚡C: cadastro com metade dos campos | telescópio de construtores | B |
| 03h20 | ⚡D: teste sem assert | teste verde que não verifica nada | A |
| 03h45 | R6: teste vermelho | contrato (R$ 5) contra código (R$ 50) | B |
| 06h15 | R7: Go/No-Go | | A |

## O caminho do grupo
Corrigimos a causa em vez de mascarar na maioria das rodadas (R1, R4, R5, R6), aceitando atrasos em troca de integridade. As escolhas mais fracas foram o ⚡B (delay no lugar do `try-with-resources`) e a R3 (auditar antes de corrigir). O ⚡B só existiu porque escolhemos o `PreparedStatement` manual na R2, e uma escolha da noite voltou para nos assombrar. O placar subiu de 🔥 7 para 8 e fechou com 🧹 1 e R$ 100 mil de prejuízo.

## Causa raiz de 2 incidentes
**R1 (Aula 11, exceções):** o `SaldoInsuficienteException` era lançado dentro de um `try` e capturado por um `catch` vazio no próprio método. Como o `debitar()` ficava fora do `try`, ele rodava sempre, e a regra de negócio era anulada pela exceção que deveria protegê-la.

**R2 (Aulas 12 e 13, JDBC/JPA):** o SQL era montado por concatenação de String, então a entrada do usuário (`' OR '1'='1`) virava parte do código SQL. Parâmetros vinculados, no `PreparedStatement` ou na derived query, tratam a entrada sempre como valor, nunca como SQL.

## O que faríamos diferente
1. **R2:** votaríamos D (derived query `findByTitular`). Ela resolve a injeção usando a abstração que já existia e evita o acesso manual via JDBC que gerou o vazamento do ⚡B.
2. **R3:** votaríamos A ou D, corrigindo o Singleton ou delegando a numeração ao banco (sequence), e auditaríamos as duplicatas já existentes em paralelo, para não deixar o fix só para as 04h.

## Uma palavra
**Aprendizado**