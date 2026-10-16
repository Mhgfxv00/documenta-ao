# Callbacks do Showdown × peças do projeto

| | |
|---|---|
| **O que este documento responde** | Para cada tipo de "gancho" que o Showdown usa nos dados de golpes, habilidades, itens e condições, qual peça nossa faz o mesmo papel, e quais ainda não têm peça |
| **Quando abrir** | Antes de uma leva de conteúdo de batalha (para saber se falta peça antes de gerar assets) e ao ler uma divergência do teste de conformidade |
| **Fonte** | Showdown no commit `3065d24d`: `data/moves.ts`, `abilities.ts`, `items.ts`, `conditions.ts`. **159 tipos de callback** distintos (contagem por ocorrência entre parênteses). Levantamento de 10/10/2026 (item 8 da fase 1 em [`../Projeto/PENDENCIAS.md`](../Projeto/PENDENCIAS.md)) |
| **Nossas peças** | Os 65 ganchos de `RegraDeBatalha` (`Data/Batalha/RegrasDeBatalha.cs`), os eventos de `GatilhoDeEfeito` (`EventoBatalha`) e os `EfeitoDeBatalha`. Mapa do código em [`REFERENCIA_DE_SISTEMAS.md`](REFERENCIA_DE_SISTEMAS.md) |
| **Como ler** | ✅ há peça com o mesmo papel · 🟡 há peça, mas o momento ou o alcance é diferente (conferir com a camada B do teste de conformidade) · ❌ sem peça |

Este é um cruzamento **por papel**, feito lendo os nomes e o uso; não foi provado caso a caso. A prova é a camada B do [teste de conformidade](../Planos/TESTE_DE_CONFORMIDADE.md).

---

## 1. Dano, poder, atributos

| Showdown | Nosso | |
|---|---|---|
| `onBasePower` (106), `onSourceBasePower`, `onAllyBasePower`, `onAnyBasePower` | `ModificarPoderBase`; `FormulaDePoder` no golpe | ✅ (aliado/qualquer: 🟡, auras e Battery) |
| `onModifyDamage`, `onSourceModifyDamage` (34), `onAnyModifyDamage`, `onWeatherModifyDamage` | `ModificarDano`, `ModificarMultiplicadorDeClima` | ✅ |
| `onModifyAtk/SpA/Def/SpD/Spe` (25/22/8/6/23) e as variantes `Source`, `Ally`, `Any` | `ModificarAtributo`, `TrocaAtaqueEDefesa`, `TrocaDefesas`, `UsaOMaiorAtaque` | ✅ (`Ally`/`Any`: 🟡, Flower Gift, Plus/Minus, Ruínas) |
| `onModifySTAB` | `ModificarStab` | ✅ |
| `onModifyCritRatio` (11), `onCriticalHit` | `ModificarRazaoDeCritico`, `BloqueiaCritico` | ✅ |
| `onModifyAccuracy`, `onSourceModifyAccuracy`, `onAnyModifyAccuracy`, `onAccuracy`, `onSourceAccuracy`, `onAnyAccuracy` | `ModificarPrecisao`, `ConcedeNuncaErrar` | ✅ |
| `onModifyWeight` | `ModificarPeso` | ✅ |
| `onEffectiveness` (8), `onNegateImmunity` | `NeutralizaFraqueza`, `IgnoraImunidade` | 🟡 (Tera Shell, Freeze-Dry contra Água) |
| `onModifyType` (22), `onType` | `ModificarTipo`; tipo assumido do `Pokemon` | ✅ |
| `onModifyBoost`, `onAnyModifyBoost` | `IgnoraEstagiosDoOutroLado` | ✅ |
| `onDamage` (17), `onAnyDamage` | `AntesDeReceberDano` | ✅ |

## 2. Fluxo do golpe

| Showdown | Nosso | |
|---|---|---|
| `onTryHit` (79), `onTry` (41), `onTryMove` (25), `onPrepareHit` (23), `onTryPrimaryHit`, `onSourceTryPrimaryHit` (19), `onAnyTryPrimaryHit`, `onAnyTryMove`, `onFoeTryMove` | `FalhaAoUsar`, `PermiteGolpe`, `PermiteGolpeRecebido`, `AbsorveGolpe`, `RefleteGolpeDeStatus`, `RoubaGolpe` | ✅ |
| `onHit` (175), `onAfterHit`, `onAfterMoveSecondary`, `onAfterMoveSecondarySelf`, `onAfterMove`, `onAnyAfterMove` | os `EfeitoDeBatalha` do golpe; evento `UsouGolpe` | ✅ |
| `onDamagingHit` (44), `onSourceDamagingHit` | eventos `RecebeuDano`, `CausouDano` | ✅ |
| `onModifyMove` (61) | não é um gancho só: cada uso do Showdown virou peça própria (tipo, categoria, alvo, contato, efeitos) | 🟡 conferir caso a caso |
| `onModifySecondaries`, `onSourceModifySecondaries` | `ModificarChanceDeEfeito` | ✅ |
| `onModifyTarget`, `onRedirectTarget`, `onFoeRedirectTarget`, `onAnyRedirectTarget` | `AtraiGolpeDoOponente`, `IgnoraChamariz` | ✅ |
| `onMoveFail`, `onMoveAborted` | evento `GolpeFalhou` | ✅ |
| `onBeforeMove` (21), `onFoeBeforeMove`, `onFlinch` | status que impede agir (sono, paralisia, hesitar) | 🟡 (`onFlinch`: Steadfast) |
| `onLockMove`, `onChargeMove`, `onOverrideAction` | `TravaDeAcao`, `DispensaCarga` | ✅ |
| `onDisableMove` (12), `onFoeDisableMove` | `PermiteGolpe` (Encore, Taunt, Disable, Imprison) | ✅ |
| `onDeductPP` | `CustoExtraDePpParaOponente` | ✅ |
| `onStallMove`, `onHitProtect` | proteção (Protect e famílias); `IgnoraProtecaoDoOponente` | ✅ |
| `onInvulnerability`, `onAnyInvulnerability`, `onSourceInvulnerability` | semi-invulnerável (Fly, Dig) | 🟡 conferir No Guard e Lock-On |
| `onUseMoveMessage` | narração | — (não é regra) |

## 3. Status, voláteis, prisão

| Showdown | Nosso | |
|---|---|---|
| `onStart` (169), `onEnd` (55), `onRestart` (15) | duração e início/fim de `StatusData` e de condição de campo; `ModificarDuracaoDeStatus` | ✅ |
| `onResidual` (54), `onFieldResidual` (8) | evento `FimDoTurno` | ✅ (a ordem dos resíduos tem teste próprio: `OrdemDeResiduosTests`) |
| `onSetStatus` (15), `onAllySetStatus`, `onAnySetStatus`, `onAttract` | `PermiteStatus`, `ModificarStatusImposto`, `IgnoraImunidadeDeStatus` | ✅ |
| `onAfterSetStatus`, `onAnyAfterSetStatus` | evento `StatusAplicado` (Synchronize) | ✅ |
| `onTryAddVolatile` (12), `onAllyTryAddVolatile` | `PermiteStatus` (os voláteis também são `StatusData`) | 🟡 (Aroma Veil no aliado) |
| `onImmunity` (15), `onTryImmunity` (10) | `IgnoraImunidade`, imunidade de tipo e de status | ✅ |
| `onTrapPokemon` (9), `onMaybeTrapPokemon`, `onFoeTrapPokemon`, `onFoeMaybeTrapPokemon` | `PrendeOponente`, `ImuneAArmadilhas`, `SempreConsegueSair`, `ImpedeSaida` | ✅ |
| `onDragOut`, `onAnyDragOut` | `ImuneATrocaForcada` | ✅ |
| `onTryBoost` (16), `onAllyTryBoost`, `onChangeBoost` | `PermiteQuedaDeAtributo`, `RefleteQuedaDeAtributo`, `ModificarMudancaDeEstagio` | ✅ |
| `onAfterBoost`, `onAfterEachBoost`, `onFoeAfterBoost` | reação a mudança de estágio (Defiant, Competitive, Rattled, Mirror Herb, Opportunist) | 🟡 conferir se há evento para "estágio caiu" |
| `onAfterSubDamage` (12) | substituto; `IgnoraSubstitutoDoOponente` | ✅ |

## 4. Itens e bagas

| Showdown | Nosso | |
|---|---|---|
| `onTakeItem` (181) | `ProtegeHeldItem`; `HeldItemData.PodeMudarDeMao` (item que força forma) | ✅ |
| `onEat` (77), `onTryEatItem` (13), `onFoeTryEatItem`, `onEatItem` | `MesaDeRegras.ComerBaga`, `ImpedeBagasDoOponente`, `DobraBagas`, `ModificarLimiarDeBaga` | ✅ |
| `onUpdate` (54) | gatilhos com `CondicaoDeCampo` em eventos fixos (`RecebeuDano`, `FimDoTurno`, `StatusAplicado`) | 🟡 o Showdown confere **depois de toda ação**; nós, nos eventos. Baga no meio do turno pode sair em momento diferente |
| `onPlate` (34), `onMemory` (17), `onDrive` | `HeldItemData.tipoDePlaca`, `formaForcada` | ✅ |
| `onUseItem`, `onAfterUseItem`, `onAllyAfterUseItem` | Symbiosis, Cud Chew depois de usar | 🟡 |
| `onTryHeal`, `onSourceTryHeal` | `ModificarCuraPorDreno`, `DrenoMachucaQuemDrena` | 🟡 (Heal Block como bloqueio geral de cura) |
| `onEmergencyExit` | Emergency Exit, Wimp Out | ❌ conferir |

## 5. Campo, clima, terreno

| Showdown | Nosso | |
|---|---|---|
| `onSideStart` (23), `onSideEnd` (14), `onSideRestart`, `onSideConditionStart` | condições de lado (telas, Espinhos, Vento a Favor) | ✅ |
| `onFieldStart` (21), `onFieldEnd` (18), `onFieldRestart` | clima, terreno e campo; `ModificarDuracaoDeClima/Terreno/Campo` | ✅ |
| `onWeather`, `onWeatherChange`, `onAnySetWeather`, `onTerrainChange`, `onAnyPseudoWeatherChange` | reação a mudança de clima ou terreno (Forecast, Mimicry, Protosynthesis, Quark Drive) | 🟡 conferir se há evento "clima mudou" dentro da batalha |
| `onHitField`, `onHitSide`, `onAllyTryHitSide` | golpes de campo e de lado | ✅ |

## 6. Entrada, saída, desmaio, mecânicas de formato

| Showdown | Nosso | |
|---|---|---|
| `onSwitchIn` (17), `onAnySwitchIn`, `onBeforeSwitchIn` | evento `EntrouEmCampo` | ✅ |
| `onSwitchOut`, `onBeforeSwitchOut`, `onFoeBeforeSwitchOut` | evento `SaiuDeCampo`; Pursuit | 🟡 (`onFoeBeforeSwitchOut` é o Pursuit pegando quem sai) |
| `onFaint`, `onAllyFaint`, `onAnyFaint`, `onSourceAfterFaint` (8) | evento `Desmaiou` | ✅ |
| `onBattleStart`, `onBeforeTurn` | eventos `InicioDaBatalha`, `InicioDoTurno` | ✅ |
| `onSwap` | troca de posto em dupla (Ally Switch) | 🟡 |
| `onCopy`, `onSetAbility` | `RestricaoDeHabilidade`; `ProtegeHabilidade` | ✅ |
| `onCheckShow` | Illusion | ❌ (DECISOES P11) |
| `onAnyAfterMega`, `onAfterTerastallization`, `onAnyAfterTerastallization` | evento `Terastalizou` (10/10/2026; Embody Aspect da Ogerpon). Reagir à Mega de outro continua sem peça | 🟡 |

---

## 7. O que sai daqui

**Sem peça (❌):** Emergency Exit/Wimp Out, Illusion, reagir a Mega/Tera. **A conferir (🟡), em ordem de impacto:**

1. `onUpdate`: o momento em que bagas e itens reagem. É o que mais muda o resultado de uma batalha real.
2. `onAfterBoost`: Defiant, Competitive e Mirror Herb.
3. `onWeatherChange` e `onTerrainChange`: as habilidades que mudam com o clima.
4. `onFoeBeforeSwitchOut`: o Pursuit.
5. Os ganchos `Ally`/`Any` de atributo e poder em dupla.

Cada um vira um cenário da camada B do teste de conformidade; o que divergir vira item em [`BATALHA_ATE_O_OFICIAL.md`](BATALHA_ATE_O_OFICIAL.md) §4.
