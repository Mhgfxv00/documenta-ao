# Referência de sistemas — onde mora cada coisa

| | |
|---|---|
| **O que este documento responde** | Onde cada coisa mora no código, o que cada arquivo faz, e como **criar conteúdo novo** (regra, efeito, evento) sem quebrar o resto |
| **Quando abrir** | Ao procurar "onde está isso?"; antes de criar uma peça de batalha; antes de mexer num sistema |
| **O que ele NÃO é** | O desenho do jogo ([`GDD.md`](GDD.md)), um sistema explicado de ponta a ponta ([`MODULOS_DO_JOGO.md`](MODULOS_DO_JOGO.md)) nem o estado do projeto |
| **Conferido em** | 09/10/2026. As tabelas de arquivo listam o que existe em `Assets/Scripts`; **para um arquivo novo, acrescente a linha aqui** |

## Mapa do código

```
Assets/
├── Scripts/                      assembly NelOrd.Runtime — o jogo
│   ├── Data/                     ScriptableObjects (estáticos) + estado de partida     (52 arquivos)
│   │   └── Batalha/              regras, efeitos, gatilhos, eventos, campo, clima      (14, já contados acima)
│   ├── Systems/                  regras do jogo: batalha, IA, encontros, captura, save (44)
│   │   └── IA/                   CerebroDeIA, PerfilDeIA, avaliadores                   (7)
│   ├── World/                    jogador, câmera, input, NPCs, obstáculos, transição   (29)
│   ├── UI/                       menus uGUI, tela de batalha, caixa de diálogo         (18)
│   ├── Roteiro/                  história: roteiros, passos, condições, gatilhos       (12)
│   ├── Interacao/                protótipo de interação por cards                      (5)
│   ├── Editor/                   assembly NelOrd.Editor — menus "Pokemon/…" (36 ferramentas)
│   │   └── Tests/                assembly NelOrd.Tests.EditMode
│   └── Tests/PlayMode/           assembly NelOrd.Tests.PlayMode
├── Omega/                        assembly NelOrd.Omega — o OUTRO LADO do grupo (depende do Runtime; o Runtime não o enxerga)
│   ├── Game/Runtime/             UI Toolkit, criador de personagem, jogador Omega, carregador de áreas, culling
│   ├── Game/Editor, Editor/      assembly NelOrd.Omega.Editor — menus "Omega/…"
│   ├── Visual/                   pacote visual (água, céu, grama, toon, vegetação)
│   ├── Resources/               só o que carrega por caminho: sprites de batalha (`Batalha`) e retratos (`Retratos`)
│   └── Arte/, Dados/, Prefabs/   arte importada, JSONs do extrator, prefabs
├── Data/                         os assets de conteúdo (espécies, golpes, itens, treinadores…)
├── Scenes/                       Bootstrap, DevScene, Kanto*, Casa…
├── Prefabs/                      `Bootstrap/Player.prefab` (o jogador), `Personagens/` (corpos de NPC), `Campo/`…
└── Art/, Animations/            (quase vazios; ver ANIMACAO.md)
```

**Regra de dependência:** `NelOrd.Runtime` não referencia `NelOrd.Omega`. Para o save guardar dado do lado Omega, o Omega se registra em `IParticipanteDoSave`. A assembly de testes de PlayMode enxerga as duas. O Omega traz o **segundo conjunto** de várias coisas (UI Toolkit, controlador de jogador, `GameDatabase` lendo TSV): ver [`../Projeto/PENDENCIAS.md`](../Projeto/PENDENCIAS.md) D9 antes de criar algo em Equipe, Mochila, Pokédex, movimento ou dados.

**Quem escreve o quê:** [`../Projeto/COMO_TRABALHAR.md`](../Projeto/COMO_TRABALHAR.md) §1.

---

## Data

| Arquivo | O que é |
|---|---|
| `CategoriaGolpe.cs` | enum: Físico / Especial / Status — **derivado**, não gravado no asset: poder 0 → Status, senão o atributo ofensivo decide. Só leitura (UI e filtro de regra) |
| `AtributosDeGolpe.cs` | enums `AtributoOfensivo` (Ataque/Ataque Esp.) e `AtributoDefensivo` (Defesa/Defesa Esp.) — o que o golpe realmente usa no cálculo de dano |
| `TipoData.cs` | SO de um tipo elemental. **Tipo novo = asset novo**, sem tocar em código |
| `CaracteristicaGolpe.cs` | SO de etiqueta de golpe ("cortante", "faz contato"). Habilidades filtram por elas |
| `TabelaTipos.cs` | SO da efetividade (TipoData atacante + defensor → multiplicador). **Esparsa**: só entram as relações que mudam algo; neutra é a ausência |
| `EspecieData.cs` | SO da espécie: tipos, 6 atributos base, **habilidades com peso e marca "oculta"**, XP base, taxa de captura, golpes por nível, evoluções (10 condições) e `formaDe` (ver "Forma em batalha" abaixo) |
| `GolpeData.cs` | SO do golpe: tipo, categoria, atributo ofensivo + defensivo escolhidos no asset, poder, precisão, PP, prioridade, características, `IgnoraHabilidadeDefensiva` e efeitos secundários (inclusive marca de carga/repetição forçada — ver "Golpes de vários turnos") |
| `HabilidadeData.cs` | SO da habilidade: uma lista de **regras** (passivo) + uma de **gatilhos** (reativo) |
| `Batalha/RegrasDeBatalha.cs` | `RegraDeBatalha` (consultada, não muda estado) + as 10 regras concretas. Os ganchos de VETO são dois: `PermiteStatus` e `PermiteGolpe` (este via `RegraDeProibicaoDeGolpe` — Taunt, Disable) |
| `Batalha/EfeitosDeBatalha.cs` | `EfeitoDeBatalha` (aplicado, muda estado) + os efeitos concretos — **compartilhados** por golpe, habilidade e held item; inclui `EfeitoIniciarCarga`/`EfeitoIniciarRepeticaoForcada` (golpe multi-turno) e o trio de forma/tipo `EfeitoAssumirForma` / `EfeitoReverterForma` / `EfeitoAssumirTipo` (Protean). `EfeitoContar` alimenta os contadores de nome livre que a evolução lê |
| `Batalha/GatilhoDeEfeito.cs` | a ponte: evento → condição → efeitos |
| `Batalha/EventosDeBatalha.cs` | enum `EventoBatalha` (os momentos que um gatilho pode escutar) |
| `Batalha/CondicaoDeCampo.cs` | filtro de **estado do dono**: HP, status, clima, período |
| `Batalha/HeldItemData.cs` | SO do item segurado (herda `ItemData`: mora na mochila/loja, usar = equipar) |
| `CondicaoGolpe.cs` | o filtro "quais golpes contam": tipos, características, categoria |
| `ContextoGolpe.cs` | tudo de um golpe num objeto só; também serve aos gatilhos sem golpe (`Golpe` vem null) |
| `ItemData.cs` | SO abstrato + `ContextoUsoItem`. `ItemCura`, `ItemPokebola`, `ItemEvolucao` e `ItemAtributo` herdam dele |
| `Inventario.cs` | `SlotItem` + `Inventario` (só estrutura de dados, sempre a mochila do **jogador**) |
| `TabelaEncontros.cs` | SO: quem aparece, com que peso e em que faixa de nível |
| `Pokemon.cs` | **estado individual**: espécie + nível, XP, HP, `GolpeConhecido` (PP gasto), `TravaDeAcao` (golpe multi-turno em andamento) e a forma de batalha assumida, se houver |
| `DadosJogador.cs` | estado da partida: equipe (6), caixa, inventário, dinheiro, flags, pokédex |
| `CatalogoJogo.cs` | SO com todas as espécies/golpes/itens/status, usado pelo save para resolver ids |
| `EstadoPokedex.cs` | enum Desconhecido / Encontrado / Capturado (GDD 26) |
| `Ambiente.cs` | enums `Clima` e `PeriodoDoDia` ("Qualquer" = sem exigência, só em condição) |
| `ContextoDeEncontro.cs` | tabela de encontros + condições de clima/período |
| `AtributosDePokemon.cs` | `AtributoPokemon` (os 6, para IV/EV) e `AtributoBatalha` (estágios) |
| `StatusData.cs` | SO de uma condição (veneno, sono...) + `StatusAtivo` (a instância com turnos) |
| `ItemEvolucao.cs` / `ItemAtributo.cs` | pedra de evolução; vitamina (EV) e doce raro (nível) |

A separação que o GDD pede está em `EspecieData` (compartilhado) × `Pokemon` (por indivíduo).

### Outros arquivos de `Data`

| Arquivo | O que é |
|---|---|
| `Natureza.cs`, `Genero.cs` | As 25 naturezas (+10%/−10%; HP nunca entra; as 5 que favorecem e prejudicam o mesmo atributo são neutras). Gênero é sorteado **uma vez** ao nascer pela razão da espécie (`EspecieData.oitavosDeFemea`) e atravessa evolução e forma. Gravados como número: a ordem dos valores **nunca muda** (valor novo vai no fim) |
| `ItemCura.cs`, `CuraDeItem.cs` | Item de cura da mochila; `CuraDeItem` é a cura compartilhada (HP, status ou os dois; reviver) com a `Baga` |
| `Baga.cs` | Ao mesmo tempo item **segurado** (regras e gatilhos de `HeldItemData`: a Sitrus cura sozinha na metade do HP) e item de **mochila** (usar cura como o `ItemCura`) |
| `ItemChave.cs` | Item de história (encomenda, carta, chave): fica na mochila, sem efeito, não é vendido; quem o lê é o roteiro (`SeTemItem`) |
| `ItemEnsinaGolpe.cs` | MT e MO: o item que **ensina** um golpe (`Hm01..Hm07` e as MTs) |
| `ItemDeHabilidade.cs` | Concede **para sempre** qualquer habilidade ao Pokémon em que é usado. **Um asset só** (o molde `CapsulaDeHabilidade`); a habilidade vai no id composto `CapsulaDeHabilidade:<id>`, e o `CatalogoJogo.Item(id)` monta a instância |
| `ItemTrocaDeHabilidade.cs` | O **Ability Capsule** e o **Ability Patch** dos jogos: escolhem entre os slots da PRÓPRIA espécie. Capsule = outra habilidade comum (não funciona na oculta); Patch = a oculta. Fora de batalha; fica no indivíduo e a evolução não muda (`DefinirHabilidadePorItem`) |
| `ItemTravaDeHabilidade.cs` | Consumível que só **marca** a habilidade como "definida por item": a evolução deixa de mexer nela |
| `LojaData.cs` | O sortimento de **uma** loja. Preço e "pode vender" ficam no `ItemData`; **item de preço 0 nunca é vendido** |
| `TreinadorData.cs` | Um treinador como dado: classe, nome, time, prêmio, perfil de IA, `dupla`, `OpcaoDeEquipe` por condição (a mesma `CondicaoDeRoteiro`; 1ª que casa, senão o time padrão; nunca sorteio) |
| `TerrenoData.cs` | Efeito de piso de uma arena (grama, elétrico, nebuloso, psíquico). Só vale dentro da batalha e some quando ela termina |
| `CondicoesDeEvolucao.cs` | `CondicaoDeEvolucao` (as condições, como objetos) e o contexto de "por que a evolução está sendo avaliada agora" |
| `MecanicasDeFormato.cs` | Tipos de apoio do Z-move, Dynamax e Tera (`EfeitoZ`, `BonusDeAtributo`, `MecanicaDeFormato`) |
| `Batalha/ClimaData.cs` | O **comportamento** de um clima **dentro da batalha** (o mundo guarda o enum `Clima`). Um por valor do enum, achado pelo `CatalogoJogo`. **Não duplique** o ×1,5/×0,5 de Fogo/Água, que já vive na tabela do `SistemaMundo` |
| `Batalha/EfeitoDeCampoData.cs` | Efeito que vale para um **lado** do campo, não para um Pokémon (Reflect, Espinhos, Vento a Favor, Trick Room, Gravidade); entra na `MesaDeRegras` como a quarta fonte de regra |
| `Batalha/EstadoDeBatalhaDoPokemon.cs` | Tudo o que um `Pokemon` carrega **só na luta** e nunca vai para o save (estágios, proteção, substituto, trava de golpe, forma e tipo temporários, habilidade copiada, item desativado, memória). **Regra:** campo novo "da luta" entra aqui; campo que atravessa o save entra no `Pokemon` com `[SerializeField]` |
| `Batalha/MemoriaDeBatalha.cs` | O que a batalha lembra de um Pokémon (último golpe, se falhou, quanto apanhou neste turno e de quem, se já agiu, último item consumido). Famílias inteiras do Showdown leem isto |
| `Batalha/FormulasDePoder.cs` | Golpe cujo poder não é o número do asset (peso do alvo, HP de quem usa, velocidade, status…): peça tipada, ~70 golpes |
| `Batalha/DanosFixos.cs` | Golpe cujo dano não sai da fórmula (Seismic Toss, Dragon Rage, Super Fang, Endeavor, Counter, os OHKO) |
| `Batalha/RegrasDeBatalhaFamilias.cs`, `Batalha/EfeitosDeBatalhaFamilias.cs` | As peças das famílias de habilidade e golpe do Showdown (bloquear golpe do oponente, prisão, reflexo, aliado em dupla, recarga…). Diário em [`../Historico/CONTEUDO_DO_SHOWDOWN_LEVAS.md`](../Historico/CONTEUDO_DO_SHOWDOWN_LEVAS.md) |

### Regra × efeito: a separação que organiza tudo

A distinção mais importante do sistema não é "golpe / habilidade / item" — é **regra** ×
**efeito**. As três fontes são só *lugares onde as mesmas duas coisas moram*.

| | **Regra** (`RegraDeBatalha`) | **Efeito** (`EfeitoDeBatalha`) |
|---|---|---|
| O que faz | muda **como** a batalha resolve algo | muda **o estado** da batalha |
| Como age | é *consultada*, responde uma pergunta | é *aplicada*, uma vez, num momento |
| Quando vale | enquanto a condição existir | no instante em que dispara |
| Exemplos | "golpes cortantes ×1.5", "imune a queimadura", "+1 de prioridade", "Ataque ×1.5 com HP baixo" | "envenena", "cura 25%", "−1 de Defesa", "toma 1/8", "assume outra forma" |
| Teste rápido | você escreveria *"enquanto Y, vale Z"* | você escreveria *"aplique X"* |

Isso resolve o caso que quebra quando tudo é tratado como efeito: **nem tudo que uma
habilidade faz é algo aplicado ao monstro**. "Sou imune a fogo" não é um estado que se
gruda e se desgruda — é uma etapa da resolução que passa a responder diferente. Modelar
isso como efeito obrigaria a aplicar/desaplicar estado a cada mudança de condição, que é
exatamente onde interações entre habilidades começam a dar errado.

As duas metades se encontram no `GatilhoDeEfeito`, que é como um efeito ganha um momento:

```
evento  ->  condição  ->  efeitos
```

E o ponto onde tudo isso é coletado, ordenado e consultado é a
[`MesaDeRegras`](../../Assets/Scripts/Systems/MesaDeRegras.cs). Nem `CalculoDano` nem
`SistemaBatalha` sabem que existe habilidade ou held item: eles perguntam à mesa.
**Fonte nova é um método de coleta a mais lá dentro**, não uma alteração na fórmula
nem na máquina de turnos.

### Como se configura (só Inspector)

Toda fonte — `GolpeData`, `HabilidadeData`, `HeldItemData` — tem as mesmas duas listas
`[SerializeReference]`: **regras** e (efeitos ou gatilhos). No Inspector você clica no **+**,
e cada elemento mostra um botão **"Tipo ▾"**
([EfeitoDrawers.cs](../../Assets/Scripts/Editor/EfeitoDrawers.cs)) para escolher a classe
concreta. O menu "Add Managed Reference" nativo do Unity não é confiável para isso
nesta versão, por isso esse botão existe. `CondicaoGolpe`, `CondicaoDeCampo` e
`GatilhoDeEfeito` aparecem como um foldout **fechado por padrão**, com um resumo de
uma linha ([FiltroDrawers.cs](../../Assets/Scripts/Editor/FiltroDrawers.cs)) — por exemplo
"quando — categoria=Especial · tipos:1" ou "FimDoTurno · 2 efeitos" — em vez de sempre
expandir os 4-8 campos internos.

Cada regra e cada gatilho carrega dois filtros independentes:

- `quando` (`CondicaoGolpe`) — **qual golpe**: tipo, características, categoria.
- `enquanto` (`CondicaoDeCampo`) — **em que estado o dono está**: HP, status, clima, período.

Exemplos que já dão para montar sem escrever código:

| Quero | Onde | Como |
|---|---|---|
| +10 de precisão nos meus golpes | habilidade → regras | `EfeitoPrecisao`, lado "quando eu uso", aditivo 10 |
| golpes cortantes ou de contato ×1.3 | habilidade → regras | `EfeitoDanoDeHabilidade`, `quando`.características com as duas (sem "exigir todas") |
| imune a fogo | habilidade → regras | `EfeitoDanoDeHabilidade`, lado "quando eu recebo", `quando`.tipo Fogo, multiplicador 0 |
| Ataque ×1.5 com HP ≤ 1/3 | habilidade → regras | `RegraDeAtributo`, `enquanto`.hpNoMáximo 0.33 |
| nunca fico paralisado | habilidade → regras | `RegraDeImunidadeAStatus` com Paralisia na lista |
| meus golpes têm o dobro de chance de efeito | habilidade → regras | `RegraDeChanceDeEfeito`, multiplicador 2 |
| golpe atravessa a habilidade defensiva do alvo | golpe (campo próprio) | `GolpeData.IgnoraHabilidadeDefensiva` ligado — held item e regras do próprio golpe continuam valendo |
| quem me acerta com contato toma 1/8 | habilidade → gatilhos | evento `RecebeuDano`, `quando`."faz contato", `EfeitoDanoFracaoDeHp` 0.125 |
| held item: +50% de velocidade | held item → regras | `RegraDeAtributo` Velocidade ×1.5 |
| held item: cura 25% no fim do turno com HP ≤ 50%, e se gasta | held item → gatilhos | evento `FimDoTurno`, `enquanto`.hpNoMáximo 0.5, `EfeitoCuraPropria` 0.25, "consome o held item" |
| held item: limpa status assim que ele gruda | held item → gatilhos | evento `StatusAplicado`, `EfeitoCurarStatus` (lista vazia = todos), "consome" |
| held item: Mega Evolução ao entrar em campo | held item → gatilhos | evento `EntrouEmCampo`, `usosPorEntradaEmCampo` 1, `EfeitoAssumirForma` (forma) |
| não posso usar golpes de status (Taunt) | held item ou habilidade → regras | `RegraDeProibicaoDeGolpe`, `quando`.categoria = Status |
| não posso usar UM golpe específico (Disable) | regras | `RegraDeProibicaoDeGolpe` com `golpesProibidos` preenchido |
| viro do tipo do golpe que uso (Protean) | habilidade → gatilhos | evento `GolpeEscolhido`, `EfeitoAssumirTipo` com "tipo do golpe" ligado |
| viro forma de ataque quando ataco, e volto quando apoio (Aegislash) | habilidade → gatilhos | **dois** gatilhos em `GolpeEscolhido`: um com `quando`.categoria Físico → `EfeitoAssumirForma`; outro com Status → `EfeitoReverterForma` |
| viro outra forma com HP baixo, e volto ao curar (Zen Mode) | habilidade → gatilhos | dois em `FimDoTurno`: `enquanto`.hpNoMáximo 0.5 → `EfeitoAssumirForma`; `enquanto`.hpNoMínimo 0.5 → `EfeitoReverterForma` |
| meu Ataque sobe quando qualquer atributo meu muda | habilidade → gatilhos | evento `AtributoMudou`, `EfeitoAlterarAtributo` +1, `usosPorEntradaEmCampo` para não virar laço |
| conto quantas vezes já enfrentei perigo (para evoluir depois) | habilidade → gatilhos | evento `AntesDoDano`, `EfeitoContar` com um nome; o **mesmo nome** vai no `contadorNecessario` da evolução |
| acelero ao me livrar de um status | held item → gatilhos | evento `StatusRemovido`, `EfeitoAlterarAtributo` Velocidade +1 |

### Criar uma peça nova (isso sim é código)

Escolher uma peça existente é trabalho de Inspector; **inventar uma nova é uma classe C#**.
Herde de `RegraDeBatalha` ou de `EfeitoDeBatalha`, marque `[System.Serializable]`, e nada
mais: ela aparece sozinha no botão "Tipo ▾", porque a lista vem de
`TypeCache.GetTypesDerivedFrom<T>()`.

Efeito (implemente `Aplicar` — o único método abstrato):

```csharp
[System.Serializable]
public class EfeitoConfundir : EfeitoDeBatalha
{
    public StatusData confusao;

    public EfeitoConfundir() => exigirDano = false;

    public override void Aplicar(ContextoGolpe contexto)
    {
        // Atacante = quem produziu o efeito (num gatilho, o dono dele).
        Pokemon alvo = Alvo(contexto, noProprioUsuario: false);
        alvo.AplicarStatus(confusao, out string msg);
        contexto.Narrar(msg);
    }
}
```

Regra (sobrescreva só o gancho que interessa; os outros já vêm vazios da base):

```csharp
[System.Serializable]
public class RegraSemCritico : RegraDeBatalha
{
    public override void ModificarDano(ContextoGolpe contexto, Pokemon dono, ref float multiplicador)
    {
        if (!Vale(contexto, dono)) return;          // confere lado + quando + enquanto
        if (contexto.Critico) multiplicador /= 1.5f;
    }
}
```

Os ganchos disponíveis numa regra: `ModificarPrecisao`, `ModificarDano`,
`ModificarChanceDeEfeito`, `ModificarPrioridade`, `ModificarAtributo`, `ModificarTipo`,
`AntesDeReceberDano`, e os dois de **veto** — `PermiteStatus` (o status não gruda) e
`PermiteGolpe` (o golpe não pode nem ser escolhido).
`Vale(contexto, dono)` filtra lado + `quando` + `enquanto` de uma vez; `ValeOEstado(dono)`
é a versão para os ganchos que não têm golpe nenhum (atributo, imunidade).

### Evento novo

`EventoBatalha` lista os momentos. Acrescentar um é: mais um valor **no fim do enum** (ele é serializado como número) e mais uma chamada de `DispararEvento(...)` / `DispararEmCampo(...)` no ponto certo do `SistemaBatalha` — nenhum golpe, habilidade ou item existente muda. Os **27** de hoje, nas duas modalidades (1v1 e dupla):

| Evento | Quando dispara |
|---|---|
| `InicioDaBatalha` | antes do primeiro turno, todo mundo já em campo |
| `InicioDoTurno` / `FimDoTurno` | começo (antes das escolhas executarem) e fim de cada turno (junto com veneno, queimadura e cura de terreno) |
| `EntrouEmCampo` / `SaiuDeCampo` | entrada (início ou troca) e saída (troca, desmaio, fim) |
| `GolpeEscolhido` | o golpe está decidido e **ainda não executou** — a única janela para reagir ao que **vem** (Protean muda o tipo aqui) |
| `UsouGolpe` | usou, tendo acertado ou não |
| `GolpeFalhou` | errou o alvo |
| `AntesDoDano` | dano calculado e **ainda não aplicado** |
| `CausouDano` / `RecebeuDano` | depois do dano, um evento por lado |
| `StatusAplicado` / `StatusRemovido` | um status grudou / saiu |
| `AtributoMudou` | um estágio subiu ou caiu (inclusive o que o próprio dono se impõe) |
| `Desmaiou` | chegou a 0 de HP |
| `GolpeSemEfeito` | um golpe de dano **acertou** o dono e não fez efeito (imunidade ou regra que zerou o dano): a absorção de Volt/Water Absorb e Flash Fire mora aqui |
| `AtributoCaiuPorOponente` | um estágio do dono caiu **por ação do outro lado** (Defiant, Competitive) |
| `AtributoCaiu` | um estágio do dono caiu por **qualquer** motivo, inclusive o que ele se impôs (Overheat). Eject Pack |
| `DerrubouOponente` | alguém do outro lado caiu e o dono estava em campo (Moxie e família; aproximação: a batalha não registra quem deu o golpe final) |
| `PerdeuAVez` | o dono perdeu a vez por causa de um status (Steadfast) |
| `BloqueadoPorProtecao` | um golpe foi bloqueado pela proteção do dono, **antes** de qualquer resolução |
| `RecebeuGolpeCritico` | o dono levou um crítico de verdade (não no substituto). Anger Point |
| `ConsumiuHeldItem` | o dono consumiu o próprio item segurado (baga, Pedra Focal). Unburden, Cheek Pouch, Cud Chew |
| `ClimaMudou` | o clima **da batalha** mudou; disparado para todos depois da ação que mudou. Forecast, Ice Face |
| `GolpeAbsorvido` | um golpe de dano foi absorvido inteiro por uma `RegraDeAbsorverGolpe` (Disguise, Ice Face) |
| `AplicouStatusNoOutro` | o **dono** acabou de aplicar um status em outro Pokémon. Poison Puppeteer |
| `CampoInstalado` | um efeito de campo foi instalado (Trick Room…). Room Service |

Os "antes" (`GolpeEscolhido`, `AntesDoDano`, `BloqueadoPorProtecao`) são os pontos em que dá para reagir ao que **vem** em vez do que já aconteceu — é onde Protean e escudos de última hora precisam morar. Um gatilho pode escutar o **aliado** (`escutaOAliado`) ou o **oponente** (`escutaOOponente`).

### Categoria do golpe: derivada, não digitada

`GolpeData` não guarda um campo `categoria`. Ele já declara `poder` e `atributoOfensivo`, e
são esses dois que o `CalculoDano` realmente usa — categoria seria uma terceira cópia da
mesma informação, e a única que poderia sair de sincronia (nada impediria um golpe marcado
"Especial" configurado para atacar com o Ataque físico). `Categoria` é uma propriedade
**calculada**: poder ≤ 0 → `Status`, senão o atributo ofensivo decide entre `Fisico` e
`Especial`.

Ela continua existindo porque tem dois usos, os dois de leitura: o rótulo/ícone mostrado ao
jogador, e o filtro `CondicaoGolpe.categoria` ("golpes especiais causam ×1.5") — que evita
ter de etiquetar golpe por golpe com uma `CaracteristicaGolpe` só para isso. No
`CalculoDano`, o corte de golpe de status acontece *depois* da precisão (golpe de status
também erra) e não impede os efeitos, que são aplicados em seguida com "exigir dano"
desligado.

### Cálculo de dano na ordem do cartucho (`CalculoDano.DanoNaOrdemDoCartucho`)

Desde 10/10/2026 o dano é calculado em **inteiros**, na ordem e com o arredondamento do Showdown (`getDamage` e `modifyDamage`): base = piso(piso(piso(2N/5+2) × poder × ataque / defesa) / 50) + 2; depois clima, crítico (×1,5 com piso), a rolagem (inteiro de 85 a 100, ×r/100 com piso), STAB, efetividade (dobra ou corta pela metade, com piso), o corte da queimadura no dano físico (Guts e Facade pulam), as regras de dano dos dois lados, e no mínimo 1. Cada fator fracionário passa por `CalculoDano.Modificar` (base 4096, metade arredondada para baixo, como o `modify` do Showdown). O terreno entra no **poder**; Thick Fat e afins entram no **atributo de ataque de quem bate** (`MesaDeRegras.MultiplicadorDoAtaqueRecebido`).

O empilhamento de multiplicadores do mesmo lado continua em `EncadearModificador` (o `chainModify`). `CalculoDano.FaixaDeDano` usa a mesma conta nos dois extremos da rolagem; `DanoEstimado` (a média que a IA usa) continua em ponto flutuante, por ser uma média.

**Medido:** 100% de igualdade com a calculadora oficial em 12.420 casos (teste `ConformidadeDeDanoTests`, [`../Planos/TESTE_DE_CONFORMIDADE.md`](../Planos/TESTE_DE_CONFORMIDADE.md)). Antes da mudança: 55,5%.

## Systems

| Arquivo | Papel |
|---|---|
| `SingletonDeCena.cs` | base `SingletonDeCena<T>` para "só um por cena": guard de instância única (segunda cópia se destrói sozinha, avisando no Console) + `AoInicializar`/`AoDestruir` no lugar de `Awake`/`OnDestroy` |
| `GerenciadorEstados.cs` | `EstadoJogo` (Exploracao / Batalha / Menu / Dialogo / Transicao). Liga e desliga movimento, câmera e interação conforme o estado |
| `SistemaEncontros.cs` | recebe uma `TabelaEncontros`, sorteia e manda a batalha começar |
| `SistemaBatalha.cs` | máquina de estados do combate por turnos (escolha → ordem → execução → verificação → próximo turno), 1v1 e dupla; também dono do inventário de batalha do ADVERSÁRIO (`inventarioAdversario`, cópia isolada de `NpcTreinador.InventarioDeBatalha` por luta) |
| `BatalhaTipos.cs` | enums da batalha + `AcaoBatalha` |
| `MesaDeRegras.cs` | junta regras e gatilhos de todas as fontes (habilidade, held item, golpe), ordena por prioridade e responde. É a única porta entre a batalha e o módulo de regras |
| `IA/CerebroDeIA.cs` | o tomador de decisão: ações legais (golpe, troca, item) → avaliação → estilo → imperfeição → escolha |
| `IA/PerfilDeIA.cs` | SO com capacidade, estilo, planejamento, imperfeição e a lista de critérios; `PesoDoEstilo` tem entrada explícita pra todo critério em todo estilo nomeado |
| `IA/AvaliadoresDeIA.cs` | os 11 critérios: dano, nocaute, tipo, status, segurança, cura, troca, preparação, apoio ao aliado (dupla), item e golpe que vai falhar |
| `IA/EstadoDeIA.cs` | percepção: o recorte da batalha que **aquela** IA pode usar, incluindo os itens de batalha disponíveis |
| `IA/HistoricoDeBatalha.cs` | o que já foi revelado nesta batalha (golpes vistos) |
| `IA/AcaoCandidata.cs` | uma ação legal (golpe, troca ou item) com dano estimado, nota e o detalhamento da nota |
| `CalculoDano.cs` | precisão, dano (ponto fixo), faixa de dano, STAB, crítico, XP concedida, chance de fuga |
| `SistemaCaptura.cs` | fórmula de captura da Gen III+ (ver abaixo), separada da bola e da batalha |
| `SistemaInventario.cs` | única porta para adicionar/remover/usar item **da mochila do jogador** |
| `SistemaDialogo.cs` | fila de falas + avanço pelo botão de interagir |
| `SistemaSave.cs` | JSON em `Application.persistentDataPath`, ids resolvidos pelo catálogo |
| `SistemaMundo.cs` | relógio (período do dia) e clima; avisa `OnAmbienteMudou` e responde o multiplicador de dano do clima |
| `SistemaAudio.cs` | o único lugar que toca som: música em laço com troca suave + efeitos, e três volumes (mestre/música/efeitos). Sem clipe atribuído é silencioso |
| `ContadorDeAmbiente.cs` | conta, por indivíduo da equipe, as viradas de clima/período — o produtor de contador que vive fora da batalha |
| `SistemaEvolucao.cs` | procura/aplica a evolução; sem estado, lê as condições da espécie e o ambiente |

### Outros arquivos de `Systems`

| Arquivo | Papel |
|---|---|
| `SistemaBatalha.*.cs` | `SistemaBatalha` é uma `partial class` em 14 arquivos por assunto: `.cs` (núcleo: campos, eventos, `IniciarBatalha*`, `RotinaBatalha`), `.Acoes` (escolha e registro), `.Golpes` (fila, prioridade, resolução), `.IA`, `.Verificacao`, `.FimDeTurno`, `.PosBatalha` (XP, evolução, golpe novo), `.Campo`, `.Posicao`, `.TrocaForcada`, `.AtaqueAtrasado` (Future Sight), `.Mecanicas` (Mega/Z/Dynamax/Tera), `.MecanicasParaUI` (o que a tela precisa saber de Mega/Z/Dynamax/Tera) e `.Rede` (foto, tradução de ação). `EstadoDeLutaSerializado.cs` é o formato da **foto da luta** (`EstadoDeBatalhaSerializado`, formato 2): `SistemaBatalha.Restaurar` a repõe no começo da escolha do turno, e o sorteio é refeito pela semente. Cada Pokémon carrega a **sua** batalha (`DoPokemon`), então há várias batalhas no mesmo processo |
| `SistemaLoja.cs` | Compra e venda. Só mexe em `DadosJogador` (dinheiro + inventário); nenhuma tela faz a conta sozinha. Comprar e vender de volta dá prejuízo, nunca lucro (teste) |
| `SistemaTransicao.cs` | Dono da troca de cena: **ninguém mais chama `SceneManager.LoadScene`**. Quem pede a viagem (a porta) guarda posição e ângulo de chegada. Troca a cena inteira com fade; o carregamento por proximidade da multicena é o `CarregadorDeAreas` (Omega) |
| `SistemaDeTutores.cs` | As regras do professor de golpe: quem pode aprender, quanto custa, se já foi ensinado, a cobrança. O `NpcEnsinaGolpe` só guarda o que ensina e conduz a conversa |
| `TradutorDePokemon.cs` | `Pokemon` ↔ `SavePokemon`, num lugar só (save **e** rede). Grava a `EspecieBase`, nunca a forma temporária |
| `IParticipanteDoSave.cs` | Ponto de extensão do `SistemaSave` para código que o Runtime não enxerga (o Omega). Inverte a dependência |
| `Sorteio.cs` | De onde vem o acaso: global e semeável. Ver "O acaso" abaixo |
| `BatalhaEmRede.cs`, `RegistroDaBatalha.cs`, `RitmoDeBatalha.cs` | As costuras de rede: ação como dado, árbitro, foto; registro reproduzível; ritmo separado da regra. [`../Planos/REDE_E_MULTIPLAYER.md`](../Planos/REDE_E_MULTIPLAYER.md) |
| `MedidorDeCarga.cs` | Mede o carregamento inicial **numa build**: rode o executável com `-medirCarga`; anota tempo e memória logo depois do bootstrap em `medicao_carga.txt` (`Application.persistentDataPath`) e fecha |
| `IA/DificuldadeDoJogo.cs` | `Dificuldade` (Facil/Normal/Dificil) escolhida pelo jogador; estática apoiada em `PlayerPrefs` (como os volumes) |

### Captura (`SistemaCaptura`)

Fórmula da **Geração VIII/IX** (conferida em 15/10/2026 na Dragonfly Cave, "Gen VIII/IX Capture Mechanics"; a Gen III/IV com duas raízes quadradas, que o projeto usou até 14/10, foi trocada): `X = (3×HPmax − 2×HPatual) / (3×HPmax) × taxa × bola × nívelBaixo × status`. Com `X ≥ 255` a captura é garantida; senão `Y = ⌊65536 / (255/X)^(3/16)⌋` e cada uma das 4 sacudidas passa com chance `Y/65536`. **Nível baixo (Gen IX):** até o nível 13, `(36 − 2×nível)/10`. **Status:** sono e congelamento ×2,5; paralisia, veneno e queimadura ×1,5. **Captura crítica:** `CC = ⌊min(255, X) × P × charme / 6⌋`, com P pelas espécies capturadas (mais de 600 = 2,5; 451 a 600 = 2; 301 a 450 = 1,5; 151 a 300 = 1; 31 a 150 = 0,5; até 30 = 0); sorteia 0 a 255 e, abaixo de CC, a captura é crítica e tem **uma** verificação só. **Ultra Ente:** toda bola que não é a Beast Ball (nem a Master) vale ×410/4096. Não entram o modificador de dificuldade D da Gen VIII e o charme de captura (o gancho `AmuletoDeCaptura` existe, sem item). `SistemaCaptura.Tentar` devolve um `ResultadoDeCaptura` (sacudidas, se foi crítica, a taxa).

**Revisão de 14/10 (S12).** `a` é calculado em `double` e tem piso 1 (improvável, nunca impossível: 47 espécies do APK têm taxa 1 ou 2). O `bônusBola` não é mais só o número do asset: `ItemPokebola.MultiplicadorPara(ContextoDeCaptura)` multiplica o fixo pelos `BonusDeBola` do próprio asset (`BonusDeBolaRapida`, `DoTempo`, `PorTipo`, `DeRepeticao`, `PorNivel`, `DeVelocidade`, `DeAmor`, `PorStatus`, `DoPeriodo`, `PorOrigem`, `DeEvolucaoPorItem`, `DePeso` e `DeUltraEnte`). A Heavy Ball não multiplica: `AjusteDaTaxa` soma na taxa da espécie (o piso continua 1). O contexto leva o alvo, o Pokémon de quem joga, o turno, a Pokédex, a hora e a **origem do encontro** (`OrigemDoEncontro`, guardada em `SistemaBatalha.Origem`: `AreaDeEncontro.origem`, a pesca e o roteiro a preenchem). Bola nova é um asset novo, sem tocar no código.

### Itens que restauram PP (`ItemRestauraPp`)

Ether (10 PP, um golpe), Max Ether (completo, um golpe), Elixir (10 PP, todos) e Max Elixir (completo, todos): um só item com `quantidade` (0 = completo) e `todosOsGolpes`. O golpe do Ether vem de `ContextoUsoItem.IndiceDoGolpe` (a tela, quando existir) ou é o mais vazio. Não gasta o item quando nada precisa de PP. Caso de borda real: `taxaCaptura` 1 com HP cheio e bola ×1/×2 dá `a = 0` (captura **impossível**). Exemplos de chance e a lição de método (a aproximação `chance⁴` anterior condenava dados que estavam certos): [`../Historico/CAPTURA_VALORES_DE_TESTE.md`](../Historico/CAPTURA_VALORES_DE_TESTE.md).

### O acaso: quem sorteia, e de onde

Uma regra só, e ela é a base de replay, de rede e de bug reproduzível: **o que acontece
dentro de uma batalha sorteia do acaso DA BATALHA** (`sorteioDaBatalha`, semeável por
`SemearProximaBatalha`); o `Sorteio` global é para o mundo — encontro, criação de
Pokémon, o que é de um jogador só.

Como cumprir, em ordem de preferência:

1. **Receber o `IAleatorio` por parâmetro.** É para isso que a interface existe, e é o que
   `ConseguiuFugir`, `Sacudidas` e `SistemaEvolucao.Procurar` fazem — com `null` caindo no
   global, que é o certo fora de batalha.
2. **Perguntar a `SistemaBatalha.SorteioEmCurso`**, quando a função está fundo demais na
   pilha para carregar mais um argumento (`CalculoDano.DanoEstimado`,
   `MesaDeRegras.Disparar`).
3. Nunca chamar `Sorteio.Inteiro/Fracao/Faixa` de dentro de um caminho de batalha.

**Esta regra já foi quebrada seis vezes**, em quatro passadas, e nas seis em silêncio:
`DanoEstimado` (achado por 1 de HP de diferença entre dois ritmos); fuga, captura e o
desempate de evolução (achados relendo o laço de turno); e `SortearPerdaDeTurno` e
`StatusData.SortearDuracao` (achados varrendo o projeto atrás dos outros). O último par é
o mais grave dos seis: **se o Pokémon age ou não neste turno**, e **quantos turnos o sono
dura**.

Duas coisas fecharam isso, e as duas importam:

- **Um ponto único.** A condicional escrita à mão em dois lugares foi o que permitiu ao
  terceiro esquecer — daí `SorteioEmCurso`.
- **Uma lista de combinados** (`AcasoGlobalTests`). Todo uso do `Sorteio` global no runtime
  tem que estar declarado com o motivo escrito; uso novo falha no dia em que é escrito, e o
  único jeito de passar é acrescentar a linha. É o mesmo padrão do
  `AuditorDeAlcancabilidade`, e vale mais que os testes de comportamento: "mesma semente,
  mesmo resultado" **passa** com a brecha aberta (o global também é semeável), e contar o
  consumo só pega o caminho que aquele teste percorre.

Os oito usos legítimos que restam são todos do **mundo**: encontro (se acontece, qual
espécie, que nível), clima, e o que o indivíduo sorteia ao **nascer** (IV, natureza, slot
de habilidade).

### Um único dono por sistema (`SingletonDeCena<T>`)

`GerenciadorEstados`, `SistemaBatalha`, `SistemaDialogo`, `SistemaInventario`,
`SistemaEncontros`, `SistemaSave`, `SistemaMundo`, `SistemaTransicao`, `SistemaLoja`,
`SistemaAudio`, `ExecutorDeRoteiro`, `EscolhaDeBatalhaMenu` e `Menu` (13 ao todo, conferido em 09/10) herdam de `SingletonDeCena<T>` em vez de implementar o próprio `Awake`/`OnDestroy`:
o guard (recusa a própria existência se `Instancia != null && != this`, avisa e se
destrói) e o `OnDestroy` (zera `Instancia`) moram só uma vez na base; cada sistema
sobrescreve `AoInicializar()`/`AoDestruir()` para a própria inicialização (os
`FindAnyObjectByType` de sempre), que só roda depois que o guard já decidiu que esta é a
instância de verdade.

Isso não troca o padrão de dependência (continua `[SerializeField]` explícito →
`FindAnyObjectByType` como fallback) — só fecha a porta de uma classe inteira de bug: uma
segunda cópia colada por engano, ou trazida por uma cena aditiva no futuro, faria dois
sistemas ouvirem o mesmo evento e reagirem em dobro. Foi exatamente esse tipo de
duplicidade (duas instâncias de `Inputs` ouvindo o mesmo botão) que causou o bug do
diálogo travado descrito em "Um único mapa de Input" mais abaixo.

---

## World

`Assets/Scripts/World`. Receitas de uso (como montar cada uma no mapa): [`../Mundo/MONTAGEM_DE_MAPAS.md`](../Mundo/MONTAGEM_DE_MAPAS.md).

| Peça | Para quê |
|---|---|
| `Interagivel` (base) / `IInteractable` | Tudo que o jogador aciona. **Exige um `Collider` Is Trigger** — sem ele o objeto não responde, sem erro. Não há tag (removidas em 14/10): `PlayerInteracao` acha o componente `Interagivel` no colisor ou no pai, e `ListavelParaInteracao` decide se ele aparece na lista (porta de gatilho e mato com encontro ao andar não aparecem) |
| `AreaDeEncontro` | Mato, caverna, água: lista de `ContextoDeEncontro` (tabela + clima + período). Sorteia ao interagir **e** ao andar dentro |
| `PontoTransicao`, `SpawnDaCena`, `ObjetoPersistente` | Porta/saída; onde o jogador nasce; estado de objeto que sobrevive à troca de cena (uma instância por chave) |
| `ItemColetavel` | Item no chão; `idNoMundo` gerado no `OnValidate` (um GUID por objeto; **copiar e esquecer de trocar** apaga o outro). Instanciar em runtime a partir de um prefab herda o id do prefab: sortear na hora do spawn |
| `Npc`, `NpcTreinador`, `NpcEnsinaGolpe`, `NpcTrocador`, `Vendedor`, `Curandeiro`, `PontoDeArmazenamento` (o PC), `PontoDeEvolucao`, `PontoDePesca` | Os NPCs e pontos de serviço. `NpcTreinador`: desafia com equipe própria, registra uma flag **só em vitória**, `permiteRevanche`, `mecanicasQueUsa`, um perfil de IA por dificuldade |
| `VisaoDeTreinador` | O treinador que enxerga o jogador e vem até ele |
| `ObstaculoDeCampo` → `ArvoreCortavel`, `PedraDeForca`, `RochaQuebravel`; `RegrasDeCampo` | Obstáculos de MO. A regra exige **a MO na mochila, a insígnia e um Pokémon da equipe que conhece o golpe** |
| `PlayerMovement`, `CameraRotate`, `PlayerInteracao` | Movimento (Rigidbody), câmera, interação. Pegam o input do `EntradaJogador` pai |
| `EntradaJogador`, `EscutaDeInput` | **Dono único do mapa de input por jogador** (`[DefaultExecutionOrder(-100)]`). Ninguém faz `new Inputs()`; quem precisa de input resolve o dono por hierarquia (`EscutaDeInput`) e **nunca guarda `entrada.Mapa`** |
| `Inicializacao` | A cena inicial (`Bootstrap`): o único lugar onde jogador, câmera, UI e sistemas nascem. `GarantirSistemas` traz os sistemas por cima quando se dá Play direto numa cena de jogo |
| `CullingDeHistoriaPorCelula` | Culling dos objetos da história por célula do mapa |

Detalhes que já causaram bug: o diálogo travado por duas instâncias de `Inputs` (o `SistemaDialogo.Encerrar()` só volta para a exploração se o estado ainda for `Dialogo`); o jogador carrega só o colisor físico + `Rigidbody`, numa layer própria `"Jogador"`, e o **trigger é de cada interagível** (o raio dele é o alcance); a troca de cena por nome (`cenaDestino`) quebra se o arquivo da cena for renomeado.

## Roteiro (a história)

`Assets/Scripts/Roteiro`. Um **roteiro** é um asset (`RoteiroData`) com uma lista de **passos**; um NPC, uma área ou a chegada numa cena o dispara. Os passos são objetos `[SerializeReference]`, pelo mesmo motivo dos efeitos de batalha e das condições de evolução: passo novo é uma classe, não um campo a mais num componente gigante, e o "se/senão" e a "escolha" existem aninhando listas sem caso especial no executor. A extração do FireRed gera roteiros nesse formato (`msgbox` → `PassoFala`, `goto_if_set` → `PassoSe` com `SeTemFlag`, `setflag` → `PassoLigarFlag`…): é assim que 2.026 roteiros entraram.

| Peça | Papel |
|---|---|
| `PassoDeRoteiro` e **22 passos** | `PassoFala`, `PassoSe`, `PassoEscolha`, `PassoLigarFlag`/`PassoDesligarFlag`, `PassoDarItem`/`PassoTirarItem`, `PassoDarPokemon`, `PassoDarDinheiro`, `PassoDarInsignia`, `PassoBatalha`/`PassoBatalhaSelvagem`, `PassoCurarEquipe`, `PassoMoverNpc`/`PassoMoverJogador`/`PassoPosicionarAtor`/`PassoVirarPara`/`PassoMostrarEsconderNpc`/`PassoTeleportar`, `PassoEsperar`, `PassoChamarRoteiro`, `PassoDefinirNome`. Passos são **dado**: nada de estado de execução em campo (o mesmo passo pode rodar duas vezes). **Campo de flag escrita termina em `Escrita`** (`flagEscrita`, `flagDeVitoriaEscrita`…) — é por esse nome que o validador acha quem liga cada flag |
| `CondicaoDeRoteiro` e **8 condições** | `SeTemFlag`, `SeNaoTemFlag`, `SeTemItem`, `SeTemPokemonNaEquipe`, `SeInsigniasPeloMenos`, `SeDificuldade`, `SeTodasDestas`, `SeQualquerUmaDestas`. Puras (só leem `DadosJogador`). **Campo de flag lida chama-se `flagLida`** |
| `MotorDeRoteiro` / `ExecutorDeRoteiro` | O coração, **sem `MonoBehaviour`**: roda um roteiro por vez e recusa um segundo enquanto o primeiro não termina. Uma pilha de `IEnumerator` faz o papel de `StartCoroutine`. Separado do componente para o teste de EditMode avançar quadro a quadro |
| `IMundoDoRoteiro` / `MundoDoRoteiroNaCena` | Tudo que um passo precisa **fazer** no mundo (falar, lutar, andar, trocar de cena). Os passos não conhecem `SistemaDialogo`, `SistemaBatalha` nem cena; no jogo quem responde repassa ao sistema dono; no teste é um falso. **A escolha e o nome ainda não têm tela: são desenhados em IMGUI pelo executor (provisório)** |
| `AtorDeRoteiro` | Dá um id de cena a um objeto para os passos o acharem (mover NPC, virar, mostrar/esconder, ponto de chegada). Busca na hora do passo, nunca por quadro |
| Disparadores | `NpcComRoteiro` (`roteiro`; `flagLida` + `roteiroSeFlag` para uma 2ª versão depois da flag), `PlacaComRoteiro`, `GatilhoDeArea` (trigger; `flagDeUmaVezEscrita`), `GatilhoAoEntrarNaCena`, `ScriptDeMapa` (os *map scripts* do FireRed: volume que roda roteiro de passos imediatos na hora, sem ocupar o executor) |
| `ValidadorDeRoteiro` (menu `Pokemon → Validar roteiros`) | Passo vazio, batalha sem treinador, escolha sem opção, **flag lida que ninguém escreve** |

O sistema de **interação por cards** (grafo) roda **dentro** do `ExecutorDeRoteiro` como conversa avulsa e convive com o `RoteiroData`: [`INTERACAO_POR_CARDS.md`](INTERACAO_POR_CARDS.md).
