# Batalha: barramento único e determinismo — o desenho proposto

| | |
|---|---|
| **O que este documento responde** | Qual é o desenho de longo prazo proposto para o motor de regras (etapa 8 do plano de batalha) e o que o projeto decidiu sobre **determinismo** (semente, replay, rede) |
| **Quando abrir** | Antes de mexer em `MesaDeRegras`, em quem sorteia na batalha, ou de propor "trocar por handlers como o Showdown" |
| **Estado** | O **determinismo** está implementado e vale (GDD §5, Referência "O acaso"). O **barramento único** é **proposta não feita** (risco alto; ver PENDENCIAS M1). Do que a etapa 8 pedia, duas partes já estão feitas (13/10): o **PRNG no contexto** (`ContextoGolpe` nasce com o acaso da batalha de quem ataca) e **mais de uma batalha por processo** (cada Pokémon carrega a sua batalha; provado por `DuasBatalhasAoMesmoTempo_*`). Falta o barramento em si (`MesaDeRegras` como fachada, ordem numérica dos gatilhos = D14) |
| **Origem** | Seções 5 e 6 do guia de implementação ([`../Historico/GUIA_DE_IMPLEMENTACAO_2026-09.md`](../Historico/GUIA_DE_IMPLEMENTACAO_2026-09.md)), movidas sem edição. "Higor" é o autor e dono do motor de regras |

---

## 5. Plano incremental de batalha e desenho híbrido

Cada etapa: compila, passa `MesaDeRegras/DesempateDeRegras/ProibicaoDeGolpe/AcasoDaBatalha/AcasoGlobal/Sorteio/SistemaBatalhaPlayMode/BatalhaEmRede` sem editar asserção.

| # | Etapa | Toca | Risco | Teste novo |
|---|---|---|---|---|
| 0 | Toda batalha semeada + replay | `SistemaBatalha.Campo`, `BatalhaEmRedeTests` | baixo | `ReplayDeBatalhaTests` |
| 1 | Imunidade/dano 0 como evento | `CalculoDano.Resolver`, `AplicarGolpeEm`, `EventosDeBatalha` | baixo | `ImunidadeDisparaGatilhoTests` |
| 2 | `EstadoDeEfeito` em `StatusAtivo`/`EfeitoDeCampoAtivo` | `StatusData`, `EfeitoDeCampoData`, save/foto | médio (save) | round-trip |
| 3 | Status como fonte (origem no fim da precedência) | `StatusData`, `MesaDeRegras` | médio | `StatusComRegraTests` (Taunt) |
| 4 | ~~Evento `AntesDeMover`~~ **não fazer sem consumidor (29/09)**: `PerdeuAVez` e `GolpeEscolhido` já cobrem o que ela destravaria | `SistemaBatalha.Golpes/Acoes` | médio — consumo do PRNG | — |
| 5 | Crítico consultável (24/8/2/1) | `CalculoDano`, `RegrasDeBatalha` | muda sequência: commit isolado | `CriticoTests` |
| 6 | Poder base, nunca erra, STAB, filtros | `CalculoDano`, `CondicaoGolpe` | baixo | Technician/NoGuard/Filter |
| 7 | Multi-hit | `GolpeData`, `AplicarGolpeEm` | médio (narração) | `MultiHitTests` 35/35/15/15 |
| 8 | Barramento único; `MesaDeRegras` vira fachada; PRNG no contexto | novo `BarramentoDeBatalha` | **alto** | `OrdemDeResiduosTests` |
| 9 | Protect + fila consultável | `Golpes`, `Acoes` | médio | `ProtectTests` |
| 10 | ~~Clima/terreno como fonte (mundo mantém enum)~~ **✅ 26/09** (fonte + 3 climas; ver §4) | `Campo`, `CondicaoDeCampo` | médio | `ClimaComoFonteTests` |
| 11 | Eventos de oponente, estágios, troca forçada — **eventos de oponente já existiam; troca forçada ✅ 26/09** (falta Pursuit/Baton Pass) | `EventosDeBatalha`, fila | médio/alto | `TrocaForcadaPlayModeTests` |

**Ordem: 0 → 1 → 3 → 4 → 5 → 6 → 7 → 2 → 9 → 8 → 10 → 11.**

### Desenho híbrido (não handlers por string)

- **Mantém** peças tipadas como conteúdo (Inspector, geradores, "regra não muta" são os ativos).
- **Barramento único** com eventos tipados (struct por evento), prioridade **por evento**, velocidade entre donos, empate pelo PRNG.
- **Tudo que reage é `IFonteDeEfeito`**: habilidade, item, golpe, status, volatile, lado, clima, terreno.
- **`EstadoDeEfeito`** serializável (turnos, camadas, contador, ordem de instalação); sem `Dictionary`.
- **Escape**: `RegraPorCodigo` com `[SerializeReference] IComportamentoEspecial`.
- **Consultar é puro, Disparar muta** (a IA depende disso).

```csharp
public enum Resultado { Segue, FalhaComMensagem, FalhaSilenciosa }
public interface IFonteDeEfeito { string Id { get; } OrigemDaRegra Origem { get; }
    IReadOnlyList<RegraDeBatalha> Regras { get; } IReadOnlyList<GatilhoDeEfeito> Gatilhos { get; } }
public interface IBarramentoDeBatalha {
    T Consultar<T>(T evento, Pokemon alvo, Pokemon origem = null) where T : struct, IEventoDeBatalha; // puro
    void Disparar(EventoBatalha e, Pokemon dono, Pokemon outro, ContextoGolpe ctx, List<string> falas); // muta
}
```

---

## 6. Determinismo (mantido)

Decisão: **manter** — para teste, replay de bug e rede; não vira mecânica de jogador. Revoga a nota "esquece semente".
Argumentos ao Higor: flake de 5% reproduzível; anexo de bug vira teste; assertiva de consumo do PRNG do plano dele (1.3); rede sem reenvio de resultado.

| Manter | Simplificar | Criar |
|---|---|---|
| `IAleatorio`, `AleatorioSemeado.Consumos`, `SorteioDaBatalha/SemearProximaBatalha` | Batalha sem semente (`AleatorioDoUnity`, `consumos=-1`) → toda batalha semeada | `RegistroDeEntrada`: versão, hash do catálogo, semente, estado inicial, ações, respostas de pendência |
| `AcasoGlobalTests`, `AcasoDaBatalhaTests`, `SorteioTests`, `MesmaSemente_MesmoResultado_ComRitmosDiferentes` | `System.Random` + contagem → PRNG próprio com estado serializável (PCG32/xoshiro) | `ReexecutorDeBatalha` com `RitmoInstantaneo`, sem UI |
| `IRitmoDeBatalha`, `AcaoDeRede`, `IArbitroDeBatalha`, `RegistroDaBatalha` | `SorteioEmCurso` estático → PRNG no contexto (etapa 8) | Testes: mesma entrada = mesma foto; consumo idêntico; retomar do meio; hash mudou recusa |
| Ordem total da `MesaDeRegras`, ponto fixo 4096 | Helpers de teste com habilidade/IV fixos → semente de "mundo de teste" no SetUp | Menu de depuração que salva `RegistroDeEntrada` em `Logs/` |
| `Sorteio.ZerarAoComecarAJogar`; IA com `estado.Sorteio` | `float` no dano → inteiros só se houver PvP entre plataformas | — |
| `EstadoDeBatalhaSerializado` (congelado, não apagar) | Desempate da fila: padronizar `Embaralhar` | — |

Cuidado: etapas 4 e 5 mudam a ordem de consumo — atualizar valores dourados em commit isolado.
