# Co-presença no mundo — a base local para vários jogadores (A10)

| | |
|---|---|
| **O que este documento responde** | O que dá para fazer **hoje, sem escolher pacote de rede**, para tirar do código a suposição de "existe exatamente um jogador", e em que ordem |
| **Quando abrir** | Antes de começar qualquer trabalho de rede no mundo (não na batalha) |
| **Estado** | **Nada disto foi começado** (conferido em 09/10/2026: não há `AvatarRemoto.cs` nem `DadosJogador.Local`). Plano escrito em 15/09/2026, sobre o código de `mapa/blocos` daquela data |
| **Antes de executar** | Conferir as linhas citadas (`VisaoDeTreinador:28`, `PlayerInteracao:80,88`…): o código andou desde 15/09. Contagem de `FindAnyObjectByType<DadosJogador>()` em 09/10/2026: **15** fora dos testes (48 com eles); o plano dizia ~25 |
| **Fora deste documento** | Rede de **batalha** (ver [`REDE_E_MULTIPLAYER.md`](REDE_E_MULTIPLAYER.md)) e streaming de áreas (ver [`../Mundo/KANTO_REVISAO_E_MULTICENA.md`](../Mundo/KANTO_REVISAO_E_MULTICENA.md)) |

Origem: seção 9 do guia de implementação ([`../Historico/GUIA_DE_IMPLEMENTACAO_2026-09.md`](../Historico/GUIA_DE_IMPLEMENTACAO_2026-09.md)), movida sem edição de conteúdo. "Pra outra máquina executar" significava: pode ser feito por outra pessoa do grupo sem conflito com o resto.

---

**Por que existe:** o alvo declarado (GDD §1) é mundo com **2 a 15–18 jogadores em rede**,
mas o overworld hoje é 100% single-player — zero código de rede, zero avatar remoto, e vários
pontos assumindo "existe exatamente um jogador". Isso aqui é só a parte que dá pra fazer **sem
escolher pacote de rede**: tirar as suposições de "um jogador só" do código local, pra que
quando o transporte for escolhido (NGO/Mirror/Photon) ligar a rede seja aditivo, não mais um
refactor. **Rede de batalha é frente separada** (`IArbitroDeBatalha` e cia. já existem e são
testados) e não entra aqui.

**O que NÃO precisa mudar** (verificado por leitura direta em `mapa/blocos`): `PlayerMovement`,
`CameraRotate`, `PlayerInteracao` já são por-instância, sem estático — 10 instâncias já
funcionariam. `EntradaJogador`/`EscutaDeInput` já resolvem dono por hierarquia
(`Para(Component)`), é o padrão a copiar. O problema está em **quem amarra** essas peças.

**Ordem obrigatória — do mais seguro pro mais arriscado:**

| # | O quê | Como | Por quê nessa ordem |
|---|---|---|---|
| **D** | `Assets/Scripts/World/AvatarRemoto.cs` (novo) | `struct EstadoDeAvatarRemoto { posicao, anguloHorizontal, idAnimacao }` + `DefinirAlvo(estado)`; `Update()` interpola até o alvo e repassa `idAnimacao` pro `Animator`. **Nunca** recebe `EntradaJogador`/`EscutaDeInput`/`PlayerMovement`/`PlayerInteracao`/`CameraRotate` (não tem input local) e **nunca** se registra como jogador local. Sem chave de `ObjetoPersistente` por ora | Arquivo novo, zero chamador, risco zero. Fica pronto pra ser alimentado por qualquer transporte depois, e testável hoje com dado falso |
| **C1** | `DadosJogador.Local` em `Assets/Scripts/Data/DadosJogador.cs` | Lista estática + `OnEnable`/`OnDisable` + guard de domain-reload — **mesmo idioma que `EntradaJogador.Principal` já usa**. `Local => ativos.Count > 0 ? ativos[0] : null` | Puramente aditivo, ninguém chama ainda. Precisa existir antes de C2 |
| **C2** | Migrar ~25 `FindAnyObjectByType<DadosJogador>()` → `DadosJogador.Local` | 7 commits temáticos, menor raio primeiro: (1) `ItemColetavel`; (2) UI: `Menu`, `MenuPrincipalUIToolkit`, `ModoDesenvolvedor`, `OmegaModoDev`; (3) sistemas: `ContadorDeAmbiente`, `SistemaEncontros`, `SistemaInventario`, `SistemaLoja`; (4) `SistemaSave` (commit isolado); (5) roteiro: `AtorDeRoteiro`, `ExecutorDeRoteiro`, `MundoDoRoteiroNaCena`; (6) `SistemaTransicao` (só essa linha); (7) testes — adiar, não precisa | Troca semanticamente idêntica enquanto só existir 1 jogador. Também **melhora o single-player hoje**: tira varredura de cena, alinha com a regra "sem `GameObject.Find`" da §2 |
| **A** | `GerenciadorEstados` deixa de ser `SingletonDeCena` e vira componente por jogador | Copia o padrão de `EntradaJogador`: lista estática + `Principal` (pra UI global) + `Para(Component)` via `GetComponentInParent`. `Estado`/`Mudar`/`OnEstadoMudou`/`AplicarControles` continuam iguais, só a descoberta muda. 6 pontos: `VisaoDeTreinador:28` e `PlayerInteracao:80,88` (já têm o próprio `DadosJogador`, passam a usar `Para(dados)`); `ExecutorDeRoteiro:219` (já recebe `dados` por parâmetro, para de cachear global — **é correção real mesmo sem multiplayer**); `SistemaTransicao:69` e `SistemaDialogo` (viram `.Principal`, seguem globais de propósito) | **Risco médio** — é o primeiro que toca o gate real de controle (`podeMover`/`podeRotacionar`/`podeInteragir`). Fazer em 4 commits separados, rodar PlayMode inteiro (batalha, diálogo, roteiro, transição) |
| **B** | `ObjetoPersistente` aceita mais de uma instância por chave | Campo `subChave` opcional; chave do dicionário vira `chave + ":" + subChave` quando preenchida. Com `subChave` vazio (padrão em tudo hoje) o comportamento é **byte a byte idêntico** | Por último: relaxar isso só é **seguro** depois de C e A, senão um segundo objeto `"jogador"` corromperia todo `FindAnyObjectByType`/`Instancia` ainda não migrado |

**Testes novos, na ordem de chegada:** `AvatarRemotoTests` (D) → `DadosJogadorLocalTests` (C1)
→ `GerenciadorEstadosPorJogadorTests` (A, duas instâncias não vazam estado uma pra outra) →
extensão de `ObjetoPersistenteTests` (B). Depois de A+B juntos: `CoPresencaLocalTests` (PlayMode)
com duas hierarquias de jogador na mesma cena — é a capacidade que este item inteiro destrava, e
hoje é estruturalmente impossível de testar.

**Regra que vale em todas as etapas:** nenhum teste existente muda de expectativa (§2). Cada
fase preserva o single-player por construção — com 1 jogador, `Local` ≡ `FindAnyObjectByType`,
`Principal` ≡ `Para`, e `subChave` vazio ≡ chave antiga.

**Fora de escopo, registrado:** `SistemaTransicao` faz `LoadSceneMode.Single` (descarrega tudo)
em torno de um jogador cacheado — streaming aditivo de múltiplas áreas é rework separado e
maior. Este item cobre co-presença **dentro de uma cena já carregada**.
