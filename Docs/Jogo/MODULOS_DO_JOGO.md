# Módulos do jogo — um sistema por seção

| | |
|---|---|
| **O que este documento responde** | Como cada sistema funciona **e por quê** foi desenhado assim: clima e período, encontros, status, buffs, itens segurados, golpes de vários turnos, formas, IA, atributos, evolução, armazenamento, batalha de treinador, input, save |
| **Quando abrir** | Antes de mexer num desses sistemas; quando o "porquê" de uma regra importar |
| **O que ele NÃO é** | A lista de arquivos ([`REFERENCIA_DE_SISTEMAS.md`](REFERENCIA_DE_SISTEMAS.md)) nem o plano da batalha ([`BATALHA_ATE_O_OFICIAL.md`](BATALHA_ATE_O_OFICIAL.md)) |
| **Origem** | Seção "Módulos" do antigo README, movida em 09/10/2026. Os cabeçalhos com data (`(05/09)`) marcam o dia em que o desenho foi fechado; o texto descreve o que **é**. Itens corrigidos em 09/10 estão dito no próprio parágrafo |
| **Sobre os "(mapa N)"** | Números de um mapa de tarefas antigo (setembro). Não procure por eles |

Cada bloco abaixo é independente: dá para mexer num sem tocar nos outros.

## Mundo: clima e período (mapa 15)

`SistemaMundo` é o único que sabe que horas são e se está chovendo — os outros
**perguntam** e **escutam** (`OnAmbienteMudou`), em vez de descobrir sozinhos.
O ciclo anda sozinho (`duracaoDoPeriodo`, 0 = parado) e o clima é sorteado na
virada de período entre os `climasPossiveis`. Alimenta hoje: encontros, dano em
batalha (`modificadores`: chuva reforça Água e enfraquece Fogo, etc.) e evolução.

## Encontros por contexto (mapa 3)

`AreaDeEncontro` não tem "uma" tabela: tem uma lista de `ContextoDeEncontro`
(tabela + clima + período). Regra: **todas** as condições preenchidas precisam
valer; "Qualquer" não é exigência. Para *chuva OU noite*, use dois contextos.
Entre vários válidos vence o mais específico ("chuva + noite" ganha de "chuva");
**empatados em especificidade, as tabelas SOMAM** — numa noite chuvosa aparece o
conteúdo dos dois contextos, com a maior das duas chances.
A tabela em uso só é recalculada quando o `SistemaMundo` avisa (e uma vez no
início) — nunca a cada frame. Montagem antiga (uma tabela só) migra sozinha para
um contexto sem condições.

### Chance de encontro e repelente (10/10/2026)

A chance de um encontro é composta em duas etapas, porque os efeitos oficiais são de dois tipos diferentes:

1. **Multiplicadores da chance:** a chance da **área** (`TabelaEncontros.Chance`) vezes o que o **líder da equipe** traz, pela `RegraDeTaxaDeEncontro` na habilidade dele **e** no item que ele segura. Illuminate, Arena Trap e No Guard ×2; Stench, White Smoke, Quick Feet e Infiltrator ×0,5; Cleanse Tag e Pure Incense ×1/3 (Leva 18). Um efeito novo desse tipo é só essa regra num asset.
2. **Filtros depois do sorteio:** o **repelente** (`ItemRepelente`: Repel 100, Super 200, Max 250 passos) não diminui a chance. Ele **barra** o selvagem de nível menor que o líder, como no oficial. Cada verificação de encontro gasta um passo; quando acaba, o jogo avisa; com um valendo, outro não é usado. Os passos restantes são a variável "repelente" do personagem (o save não muda).

Por que não "um número no jogador": o repelente oficial é um filtro de nível, e a habilidade do líder é um multiplicador. Somar os dois num número só daria um repelente que deixa passar Pokémon fraco de vez em quando, o que o oficial nunca faz. Ainda faltam, no mesmo molde: Intimidate e Keen Eye (50% de barrar o mais fraco), Magnet Pull e Static (puxam o tipo), as flautas.

## Status (mapa 5)

Um status é um **asset** (`StatusData`), não um `if` espalhado: dano por turno,
chance de perder a vez, dano ao perder a vez, multiplicadores de atributo,
duração, tipos imunes e bônus de captura. A ficha (`Pokemon`) guarda os
`StatusAtivo` e responde as perguntas; a batalha só narra e aplica:
- antes do golpe → `SortearPerdaDeTurno`
- fim de turno → `ResolverStatusDeFimDeTurno` (dano + contagem + cura)
- ao sair de campo/fim de batalha → voláteis somem, não-voláteis ficam.

Só cabe **um** status não-volátil por vez; voláteis (confusão) somam.
Status novo = asset novo + `EfeitoAplicarStatus` num golpe. Nada de código.

## Buffs/debuffs (mapa 4)

Estágios -6..+6 por atributo em `Pokemon` (`AlterarEstagio`), multiplicador
clássico, zerados ao sair de campo. `EfeitoAlterarAtributo` é o efeito de golpe
que mexe neles. Precisão e evasão entram no cálculo de acerto.

## Held items (módulo Habilidades/Efeitos/Golpes/Held Items, seções 24-26)

`HeldItemData` **herda de `ItemData`** de propósito: assim já mora na mochila, aparece na
loja, é resolvido por id pelo `CatalogoJogo` no save, e usa o fluxo de "usar item" que já
existia — usar um held item na tela da equipe é *equipar* ele naquele Pokémon. `Pokemon`
tem `heldItem` (serializado e salvo), `EquiparHeldItem`, `RemoverHeldItem` e
`ConsumirHeldItem`.

Os três comportamentos do módulo saem das mesmas duas listas, sem código:
**passivo** (`regras`), **reativo/condicional** (`gatilhos` + `CondicaoDeCampo`) e
**consumível** (gatilho com "consome o held item" ligado). O limite
`usosPorEntradaEmCampo` é contado por indivíduo e zera junto com os estágios ao sair de
campo (`Pokemon.LimparEstadoDeGolpe`).

`PartyMenu` mostra o held item equipado (ícone + nome) no painel de detalhe e tem um
botão "Remover" que chama `Pokemon.RemoverHeldItem()` e devolve o item para a mochila — de
lá, usar o item em outro Pokémon da equipe já é o fluxo de equipar de sempre, fechando o
"trocar". Equipar em cima de um item já segurado continua **recusado** em vez de trocar
sozinho, de propósito: o antigo só volta pela mochila, nunca é descartado em silêncio.

Todos os itens segurados são os oficiais, em `Assets/Data/Conteudo/Itens/Gerado/`, preenchidos por `ConteudoDoShowdown`. Os nove feitos à mão em `Assets/Data/Conteudo/HeldItems/` (Cinta Muscular, Espinhos de Ferro, Baga Mirtilo...) eram conteúdo de teste e foram **apagados em 10/10/2026**, com as referências trocadas pelos oficiais (Muscle Band, Rocky Helmet, Sitrus Berry...); um exemplo de cada categoria hoje: `ChoiceBand` (passivo), `RockyHelmet` (reativo não consumível), `SitrusBerry` (consumível). Pedras Mega, placas, cristais Z, bagas e os itens de origem de espécie existem como `HeldItemData` (ver [`BATALHA_ATE_O_OFICIAL.md`](BATALHA_ATE_O_OFICIAL.md) §9 a §12).

### Bagas, formas presas ao item e restrições de habilidade

- **Bagas**: `Baga` é `HeldItemData` e também item de mochila (`ICuraDeMochila`, cura compartilhada em `CuraDeItem`). Quem decide se e quando comer é a `MesaDeRegras` (`ComerBaga`, `LimiarDeBaga` para Gluttony, `DobraBagas` para Ripen, `BagasImpedidas` para Unnerve); os efeitos que mexem em baga alheia (Bug Bite, Pluck, Teatime, Stuff Cheeks, Cud Chew) ficam em `EfeitosDeBatalhaFamilias.cs`. `Leppa` e `Mystery Berry` usam a condição `algumGolpeSemPp`.
- **Forma presa ao item**: `EspecieData.formaSegueOItem` + `HeldItemData.formaForcada` (placas do Arceus, Memory do Silvally, Adamant/Lustrous/Griseous, Rusted Sword/Shield). `Pokemon.SincronizarFormaDoItem` roda ao equipar e remover. `HeldItemData.PodeMudarDeMao` impede que o item saia da espécie que ele força, ou vá para outra. `tipoNaoMuda` trava o tipo (Arceus, Silvally); `Judgment` e `Multi-Attack` usam o tipo do item.
- **Restrições de habilidade**: `HabilidadeData.restricoes` (`RestricaoDeHabilidade`: não rastreável, imitável, receptável, trocável, ensinável, suprimível) segue as flags do Showdown. Os efeitos de copiar, trocar e suprimir habilidade consultam a flag certa (`EfeitoCopiarHabilidade.barradoPor`).
- **Golpe em aliado**: `GolpeData.PodeMirarAliado`/`naoMiraAliado`; Pollen Puff cura o aliado em vez de causar dano (`EfeitoCurarAliadoEmVezDeDano`).

### Z-move, Dynamax e Tera

Regras e motivos completos em [`BATALHA_ATE_O_OFICIAL.md`](BATALHA_ATE_O_OFICIAL.md) §10 a §13; o resumo do desenho:

- Cada lado usa **uma** mecânica de formato por batalha, e um Pokémon só uma. Pedra Mega ou cristal Z na mão bloqueia Dynamax e Tera.
- **Quem pode** é por lado, em `SistemaBatalha.Permite(mecanica, pokemon)`: o jogador pelas `mecanicasPermitidas` (padrão: só Mega) mais as listas de item-chave `itensQueHabilitam*` na mochila; o treinador por `NpcTreinador.mecanicasQueUsa` (não precisa de item-chave, mas a pedra/cristal tem que estar no Pokémon); selvagem só Mega.
- **Z e Max** vêm de `GolpeData.Potencializar`, furam Protect com 1/4 do dano (Max Guard bloqueia). **Dynamax** dura 3 turnos com HP ×2 (`HpSemDinamax` guarda o original). **Tera** dura até o fim da batalha: STAB ×2 se o tipo Tera já era original, piso de poder 60, Stellar suportado.
- Tudo está no arquivo `SistemaBatalha.Mecanicas.cs`. Falta a tela para o jogador escolher o tipo Tera.

## Golpes de vários turnos: trava genérica (`TravaDeAcao`)

Golpe de carga (Solar Beam: um turno de preparo, ataca de verdade no seguinte) e de
repetição forçada (Thrash: continua sozinho por N turnos, confuso ao parar) não são um
campo fixo em `GolpeData` — são efeitos comuns (`EfeitoIniciarCarga`,
`EfeitoIniciarRepeticaoForcada`) na mesma lista `efeitos` de qualquer golpe, e escrevem
numa trava genérica em `Pokemon` (`TravaDeAcao`: o golpe travado + `ModoDeTrava` +
turnos restantes + status ao expirar). `SistemaBatalha` só lê essa trava
(`GolpeData.Efeito<T>()` acha a marca sem precisar de um campo fixo por marca) — um golpe
normal não carrega nenhum campo a mais no Inspector por causa disso. A trava se solta
sozinha ao sair de campo, junto de estágios e status voláteis.

## Forma em batalha (Mega Evolução / forma de clima)

Reaproveita a mesma ideia de `EspecieData.formaDe` (que já modelava "isto é uma forma de
X" para a Pokédex) com uma troca **reversível** de espécie — diferente de `Evoluir`
(permanente, nunca desfeita). `Pokemon.AssumirForma`/`ReverterForma` trocam a espécie
ativa preservando o dano absoluto já sofrido (mesma técnica de `Evoluir`, mesmo com HP
máximo diferente entre forma e base); `EfeitoAssumirForma` é o efeito que aplica isso,
normalmente pendurado num held item reagindo a `EventoBatalha.EntrouEmCampo` (com
`usosPorEntradaEmCampo` 1). Uma forma nova é só uma `EspecieData` com atributos/habilidade
diferentes e `formaDe` apontando pra base.

**Quando a forma volta** é um campo da **forma**, não do item (`EspecieData.voltaDaForma`, enum `VoltaDaForma`) — quem sabe quanto a transformação dura é ela mesma, e a mesma forma alcançada por caminhos diferentes deve durar o mesmo: **`AoSairDeCampo`** (padrão: troca, desmaio ou fim; certo para forma de clima e Zen Mode) e **`NoFimDaBatalha`** (a Mega: quem vai para o banco continua transformado). `SairDeCampo` recebe `aBatalhaAcabou`, e só o encerramento o passa verdadeiro. Com a forma sobrevivendo à troca, o `TradutorDePokemon` grava a **`EspecieBase`** no save — senão a transformação viraria permanente no arquivo. `AssumirForma` recusa quando já há uma forma ativa: trocar de uma forma para outra passa por reverter primeiro (base ⇄ forma, não cadeia); Aegislash e Darmanitan são dois gatilhos (um assume, outro reverte).

Existe também a troca **permanente** (`Pokemon.TrocarForma`, `EspecieData.formasAlternaveis`: o Deoxys, o Arceus pela placa), a forma **por gatilho** (`EfeitoAlternarForma`) e a ativação **voluntária** junto com o golpe (Mega, Z, Dynamax, Tera). O conteúdo de forma existente e o que falta: [`BATALHA_ATE_O_OFICIAL.md`](BATALHA_ATE_O_OFICIAL.md) §3.B1, §10 a §12.

## Inventário de treinador e IA usando item

`NpcTreinador` tem dois recursos de item, com papéis diferentes:

- **`itensDeRecompensa`**: lista de `RecompensaDeItem` (item + quantidade +
  `apenasUmaVez`). Concedido em `AoTerminarBatalha`, junto do dinheiro, só quando
  `Resultado == Vitoria`. `apenasUmaVez` é **por item**, não pelo treinador inteiro —
  dá pra montar uma pedra rara só na primeira vitória e uma Poção toda vez que uma
  revanche acontecer, na mesma lista.
- **`inventarioDeBatalha`**: lista de `ItemData` que este treinador usa DE VERDADE na
  própria luta — só itens marcados `UsavelEmBatalha` contam (`OnValidate` avisa no
  Console se algo não servir; o getter público `InventarioDeBatalha` filtra de novo na
  leitura). `SistemaBatalha` copia essa lista para `inventarioAdversario` ao iniciar a
  luta (`IniciarBatalhaTreinador(Dupla)`) — uma cópia isolada por batalha, nunca a lista
  original do `NpcTreinador`; batalha selvagem sempre fica vazia.

`CerebroDeIA.GerarItens` (capacidade `Competente`+, igual troca) gera uma candidata por
`ItemCura` × alvo plausível: cura/status em quem está em campo (eu e o parceiro em
dupla), revive só em quem está desmaiado **no banco**. `AvaliadorDeItem` pontua pela
fração de HP que falta curar (até o teto do próprio item), reviver sempre alto, e cura de
status só quando o alvo tem mesmo a condição listada. A execução é
`SistemaBatalha.UsarItemDoOponente` — equivalente a `UsarItem` do lado do jogador, mas sem
passar pelo `SistemaInventario` (que é só a mochila do jogador): chama `ItemData.Usar`
direto e consome de `inventarioAdversario`. Pokébola e pedra de evolução nunca viram
candidata (não fazem sentido pro oponente usar em si mesmo em batalha).

## IA de combate (módulo de IA)

A IA **não executa regras**: produz uma decisão e entrega ao `SistemaBatalha`.
Existe **um cérebro só** — a dificuldade é parâmetro (`PerfilDeIA`), não uma classe
de IA por nível, senão cada dificuldade nova viraria um sistema paralelo de manter.

```
estado → ações legais (golpe, troca, item) → avaliação → estilo → imperfeição → escolha
```

**Análise × estilo são ortogonais.** O que a IA considera é uma coisa; como ela tende a
lutar (`Agressivo`/`Defensivo`/`Estrategista`/`Equilibrado`) é outra. Dois treinadores que
enxergam exatamente a mesma coisa podem jogar bem diferente, e nenhum dos dois ganha
atributo por isso — um treinador difícil **luta melhor, não tem números maiores**.

**Como isso se traduz em código:** a lista de avaliadores é **fixa** (os doze de
`CerebroDeIA.AvaliadoresPadrao()`), o cérebro percorre todos e pergunta ao perfil
`Considera(avaliador.Criterio)` antes de somar a nota. Não existe "IA básica" como classe —
é a mesma lista de critérios, filtrada pelo asset. Critério novo = uma classe nova em
`AvaliadoresPadrao()` mais uma caixa no perfil; nem o cérebro nem a batalha mudam.

Os cinco presets (`AplicarPreset`), que é o que cada nível **marca** — não o que ele é:

| Preset | O que passa a considerar |
|---|---|
| `Basica` | só dano imediato; nem gera trocas ou itens |
| `Competente` | vantagem de tipo, nocaute, trocas simples, uso de item |
| `Avancada` | risco, status, cura, apoio ao parceiro (dupla) — e só conhece os golpes **já revelados** |
| `Especialista` | planeja 2 lances, prepara setup (com bônus extra no primeiro turno), lê a ficha inteira do adversário |
| `Reativa` | decide **depois** de saber a escolha do jogador no turno |

### O perfil declara o que CONSIDERA (05/09)

Um asset feito **para ser** o especialista carregava um campo dizendo `capacidade =
Especialista` — um rótulo no lugar das características. O teste dessa crítica é direto:
para saber o que `IAEspecialista` fazia era preciso ler **onze arquivos** (o asset mais o
construtor de cada avaliador). O oposto de explícito.

Agora as características estão no asset:

| Campo | O que é |
|---|---|
| `consideraDano` … `consideraTatica` | uma caixa por critério (10) |
| `decideDepoisDoJogador` | a antiga capacidade `Reativa` |
| `podeUsarItemEmBatalha` | chega a gerar candidata de item |
| `leFichaCompletaDoAdversario` | lê a ficha inteira, em vez de só os golpes já vistos |

A pergunta no cérebro deixou de ser `estado.Sabe(avaliador.capacidadeMinima)` e virou
`perfil.Considera(avaliador.Criterio)`. Repare que **nenhuma informação foi duplicada**: o
avaliador continua declarando só *o que ele é* (`Criterio`, que ele já declarava), e o
perfil responde se aquilo conta. O que mudou foi quem responde.

**Todas as caixas nascem ligadas**, e essa é a propriedade que sustenta o desenho. Quando um
critério novo entra, a Unity aplica o inicializador aos assets já gravados que não têm o
campo — então ele passa a valer em todos. É o oposto exato da lista opt-in que existia
antes, em que `ApoioAoAliado` e `Item` ficaram de fora dos seis presets para sempre sem
ninguém notar. O `Considera` de um critério desconhecido também devolve `true`, pelo mesmo
motivo, e há teste para os dois casos.

**`CapacidadeDeIA` virou preset de autoria, não dado.** `AplicarPreset(nivel)` marca as
caixas como a escada marcava, e nada em runtime relê o nível — então ajustar uma caixa
depois é edição legítima, não inconsistência. É assim que "um especialista que nunca troca"
passa a ser configurável, o que a escada não permitia: descer de nível para não trocar tirava
junto status, cura e segurança.

Dois detalhes que só apareceram implementando:

- **`AvaliadorDeCura` e `AvaliadorDeSeguranca` compartilham o critério `Seguranca`**, então
  a caixa liga e desliga os dois. Hoje é o comportamento certo (os dois entravam no mesmo
  nível); se um dia precisarem ser separados, é um valor novo no enum.
- **Trocar e usar item eram o mesmo gate** (`Sabe(Competente)`) e agora são dois. A escada
  não tinha como separá-los; as caixas têm, e gerar candidata que o perfil ignora era
  trabalho jogado fora.

### O perfil NÃO carrega mais lista de avaliadores (05/09)

O campo `avaliadores` **saiu** do `PerfilDeIA`, e o motivo é concreto. Ele nunca foi um
complemento da capacidade — a relação era a inversa: cada avaliador declarava a própria
`capacidadeMinima` e o cérebro pulava quem estava acima do nível do perfil, então **a mesma
lista servia a todos os níveis**, e todos os seis presets carregavam o conjunto padrão. Era
duplicação pura.

**E duplicação que derivou.** Os presets foram gerados antes de `AvaliadorDeApoioAoAliado` e
`AvaliadorDeItem` existirem, e ficaram com **8 dos 10** para sempre. Na prática, *todo*
treinador do projeto era incapaz de apoiar o parceiro ou de usar item, em qualquer
capacidade — e nada acusava, porque uma lista preenchida parece configuração deliberada.

Agora o cérebro usa sempre `CerebroDeIA.AvaliadoresPadrao()`: critério novo passa a valer
para todos os adversários no dia em que a classe entra. O que se perdeu foi **ajuste de peso
por critério** — não em uso, e o peso por critério já existe pelo **estilo**, que é o dial
certo para isso. (Naquele dia "este treinador nunca troca" também foi listado como perda; a
caixa `consideraTroca` devolveu isso no mesmo dia, ver a seção acima.)

Os doze avaliadores, o critério que cada um declara, e o preset que marca a caixa:

| Avaliador | `Criterio` | Marcado a partir de |
|---|---|---|
| `AvaliadorDeDano` | `Dano` | `Basica` |
| `AvaliadorDeNocaute` | `Nocaute` | `Competente` |
| `AvaliadorDeVantagemDeTipo` | `Tipo` | `Competente` |
| `AvaliadorDeTroca` | `Troca` | `Competente` |
| `AvaliadorDeItem` | `Item` | `Competente` |
| `AvaliadorDeStatus` | `Status` | `Avancada` |
| `AvaliadorDeSeguranca` | `Seguranca` | `Avancada` |
| `AvaliadorDeCura` | `Seguranca` **(o mesmo)** | `Avancada` |
| `AvaliadorDeApoioAoAliado` | `Aliado` | `Avancada` |
| `AvaliadorDePreparacao` | `Preparacao` | `Especialista` |
| `AvaliadorDeGolpeQueVaiFalhar` | `Seguranca` **(o mesmo)** | `Avancada` — penaliza o golpe que certamente falha agora (Ally Switch fora de dupla, Me First sendo mais lenta, Snatch sem golpe "pegável", Bide sem ameaça); não avalia o que o golpe faz quando funciona |
| `AvaliadorDeTatica` | `Tatica` | `Avancada` — o que o golpe faz além de bater: armadilha de entrada (só com reserva do outro lado; rende menos a cada camada), tela e vento a favor (uma vez), clima e terreno (se já não valem), Trick Room (se sou o mais lento), Protect (quando a ameaça é real; cai a cada uso seguido), Substitute, golpe de pivô e Baton Pass, Roar/Dragon Tail contra quem acumulou subidas, Rest e Heal Bell. Em golpe de dano o brinde só soma, nunca castiga |

**Ordem de ataque (14/10).** `EstadoDeIA.AgiriaAntesDe` considera a prioridade do golpe e o Trick Room, não só a velocidade; é o que o Nocaute, a Segurança e o planejamento usam. Da prioridade do adversário a IA não presume nada.

**Quem entra (15/10).** Três momentos em que o adversário escolhe quem manda, todos por `LeituraDoJogador` e sem acaso (empate fica com o primeiro da fila):
- **Abertura** (opção `PerfilDeIA.escolheAAbertura`, **desligada por padrão**): abre com o melhor do time contra o time INTEIRO do jogador (quem está em campo conta em dobro); em dupla, os dois melhores.
- **Substituto de quem caiu** e **entrada depois de golpe de pivô** (U-turn, Volt Switch, Baton Pass): o melhor contra quem está em campo do lado do jogador. Vale só em batalha de treinador e só para o perfil que considera Troca (`CriterioDeIA.Troca`); o Novato segue a fila, como nos jogos.
- **A troca por escolha** (`AvaliadorDeTroca`) passou a descontar a pancada que o Pokémon leva na entrada (nunca entra numa que derruba) e a pesar os estágios: quem está com −3 de Ataque ganha em sair, quem acumulou subida perde.

**Ler o jogador (14/10), `LeituraDoJogador`.** Duas coisas, ambas tiradas do estado de agora (nunca de memória de turnos, porque a foto da batalha não guarda isso e a retomada tem que decidir igual):
- **Pursuit.** A Reativa sabe que o jogador troca e mira o Pursuit em quem *sai*, com o dobro. Quem planeja (profundidade ≥ 1) presume uma chance de troca pelo confronto (0,5 se o jogador vai cair, 0,35 se leva mais da metade, 0,25 se está com menos de 30% de HP, 0,1 no resto; metade disso se ele me bate mais do que eu nele; zero sem reserva ou preso) e pesa o dano do Pursuit por ela.
- **Forma por escolha (Deoxys).** A IA que considera Tática escolhe uma vez por decisão a forma alternável de melhor nota (ataque pelo meu lado físico/especial, defesa pelo dele, velocidade para passar na frente) e só troca com 12% de folga, porque a troca é permanente. A troca vai junto de qualquer golpe, de graça, como a Mega.

`Reativa` não acrescenta avaliador nenhum: ela marca `decideDepoisDoJogador`, que muda
*quando* a IA decide, não *o que* ela enxerga.

### Configuração esperada por faixa

Os seis assets em `Assets/Data/Sistema/PerfisDeIA` vieram do preset da ferramenta, não de decisão de
design. Esta é a leitura recomendada de cada faixa — **proposta a validar**, não medição. A
coluna "preset" diz de onde as caixas de critério vieram; ela **não** está guardada no asset,
e desmarcar uma caixa depois não desfaz nada:

| Perfil | Preset de origem | Profundidade | Ruído | Erro | Lê-se como |
|---|---|---|---|---|---|
| `IANovato` | `Basica` | 0 | 0.45 | 0.35 | vê só dano e erra bastante |
| `IACompetente` | `Competente` | 0 | 0.25 | 0.20 | tipo e nocaute; ainda escorrega |
| `IAVeterana` | `Avancada` | 1 | 0.15 | 0.12 | pesa risco e status; pensa um lance |
| `IAVeteranaDefensiva` | `Avancada` | 1 | 0.15 | 0.12 | igual, com estilo `Defensivo` |
| `IAEspecialista` | `Especialista` | 2 | 0.08 | 0.05 | planeja e prepara; estilo `Estrategista` |
| `IACampea` | `Reativa` | 2 | 0.05 | 0.02 | responde à sua escolha, quase não erra |

### Como configurar um perfil novo

`Create > Pokemon > Perfil de IA`, salvo em `Assets/Data/Sistema/PerfisDeIA`. O caminho rápido é
aplicar um preset (`AplicarPreset`, ou o menu `Pokemon > Criar perfis de IA basicos`) e
ajustar o que quiser a partir dali. Os campos:

| Campo | O que ele faz, exatamente |
|---|---|
| as 9 caixas `considera…` | se aquele critério entra na soma. Critério não considerado nem é consultado |
| `podeUsarItemEmBatalha` | se a IA chega a **gerar** a candidata de item (sem isso, considerar o critério Item não adianta) |
| `leFichaCompletaDoAdversario` | desligado = só conhece os golpes já usados na frente dela |
| `decideDepoisDoJogador` | decide sabendo a escolha do jogador no turno |
| `estilo` | multiplica o peso **por critério** (`PesoDoEstilo`). É o único dial de preferência |
| `profundidade` | 0 = só o resultado imediato · 1 = também como fica a batalha depois · 2 = também a melhor resposta provável do adversário |
| `ruidoDeAvaliacao` | ruído somado a cada nota antes de escolher — faz **não perceber** qual era a melhor ação |
| `chanceDeErro` | chance de, tendo percebido, escolher outra — erro de **execução** |

Depois é só apontar o asset num slot de dificuldade do `NpcTreinador`.

**Sim, só o estilo mexe em peso por critério** — e vale a nuance: cada avaliador tem um
`peso` próprio, mas como todos vêm do conjunto padrão, esse peso é uma constante igual para
todo mundo. Na prática, dois perfis que consideram os mesmos critérios só se distinguem por
estilo, profundidade e imperfeição.

Um perfil novo **não precisa** de classe nova nem de código: só faz sentido escrever C# aqui
para inventar um *critério* (uma `AvaliadorDeIA` nova, que passa a valer para todos os
perfis já existentes) ou um preset novo.

### Dificuldade escolhida pelo jogador (05/09)

`Dificuldade` (`Facil`/`Normal`/`Dificil`) mora em `DificuldadeDoJogo`, uma estática apoiada
em `PlayerPrefs` — mesmo tratamento de volume e qualidade gráfica, e pelo mesmo motivo: quem
pergunta são objetos espalhados por cenas, e nenhum deveria carregar referência para
descobrir isso. O dropdown fica no `ConfiguracoesMenu`.

`NpcTreinador` tem **um campo de perfil por dificuldade**, mais um `perfilDeReserva` usado
quando o campo daquela dificuldade está vazio. A escolha é lida **no momento de começar a
batalha**, não guardada: trocar a dificuldade no menu passa a valer na luta seguinte sem
ninguém avisar o NPC.

Dificuldade aqui **não é número maior**: ela só troca qual `PerfilDeIA` o adversário usa, e
o perfil só muda o quanto a IA analisa e o quanto ela erra. Nível novo = um valor no enum,
um campo no `NpcTreinador` e uma opção no dropdown; o cérebro não muda.

*Ainda não ligado:* o `perfilSelvagem` do `SistemaBatalha` não varia com a dificuldade —
encontro selvagem usa sempre o mesmo. Se for para variar, o lugar é o mesmo padrão de
campos por dificuldade.

**A regra de coerência**, que é o que realmente importa ao ajustar: *ruído e erro caem
enquanto a capacidade sobe*. São coisas diferentes — **ruído** é não perceber qual era a
melhor ação (erro de análise), **erro** é perceber e escolher outra (erro de execução) —, e
inverter a curva de qualquer um deles produz um adversário incoerente: enxerga tudo e joga
mal, ou enxerga pouco e nunca erra. **Profundidade** só passa de 0 a partir de `Avancada`;
abaixo disso ela custa processamento para uma IA que nem enxerga o que planejar.

**Informação é limitada de propósito** (`EstadoDeIA` + `HistoricoDeBatalha`): com
`leFichaCompletaDoAdversario` desligado, perguntar pelos golpes do adversário devolve só os
já usados; ligado, devolve a ficha inteira. Nenhum avaliador precisa saber disso — ele só
pergunta.
`CerebroDeIATests.InformacaoLimitada_...` prova isso com números: a mesma ameaça, vista
com só o golpe fraco revelado, mede ~19 pra `Avancada` e ~61 pra `Especialista` (que lê a
ficha e reconhece o golpe forte nunca usado).

**A reativa não precisou mudar o fluxo do turno.** A batalha já perguntava à IA *depois*
que o jogador escolhia; o que faltava era permissão para olhar. Quando o jogador troca,
quem recebe o golpe é quem **entra** (troca tem prioridade), então avaliar contra ele não
é trapaça — é a leitura correta do turno que vai acontecer.

**Imperfeição são dois mecanismos separados**, porque são falhas diferentes:
`ruidoDeAvaliacao` erra a *análise* (pode nem perceber a melhor ação) e `chanceDeErro`
erra a *execução* (percebeu e escolheu outra). Nos dois casos a escolha continua saindo
das ações analisadas — nunca vira sorteio. `ImperfeicaoDeIATests.cs` prova isso com uma
seed fixa: sem ruído, 100% de acerto; com ruído crescente, o acerto degrada
progressivamente sem nunca cair abaixo do que um chute aleatório entre as ações já daria.

**Simular sem clonar a batalha** só é possível porque regra é *consultada*, não aplicada:
`CalculoDano.DanoEstimado` pergunta a dano/precisão às habilidades e held items dos dois
lados sem que nada aconteça.

**Personalidade só se prova quando a escolha em si diverge entre estilos, não só a
nota** — a lição de calibrar os cenários de teste: vantagem/desvantagem esmagadora ou
risco letal fazem **todo** estilo concordar (só existe uma resposta sensata), então não
provam que o peso do estilo (`PerfilDeIA.PesoDoEstilo`) está fazendo diferença nenhuma.
Separação real pede uma situação **ambígua** do ponto de vista neutro (`Equilibrado`),
onde o multiplicador do estilo é o que decide para qual lado a balança pende. Os quatro
cenários calibrados de
[EstiloDeIATests.cs](../../Assets/Scripts/Editor/Tests/EstiloDeIATests.cs) reproduzem os
exemplos literais do módulo (seções 6 e 8) e isolam todos os pares de estilo, inclusive
`Defensivo`×`Estrategista` (o par que os outros cenários não separam: nos dois, os dois
concordam). `PerfilDeIA.PesoDoEstilo` tem hoje uma entrada explícita pra todo `CriterioDeIA`
em todo estilo nomeado — nada cai no `default` por esquecimento; onde o valor é neutro
(1x) por decisão deliberada, o comentário no código diz por quê.

A única coisa do módulo que **ainda não existe** é a "previsão" da seção 11 (estimar a
jogada do adversário *antes* de saber) — escopo deliberado: a seção 20 do módulo já lista
previsão como expansão posterior, separada da "reação" (decidir *depois* de saber), que
foi a priorizada.

Menu **`Pokemon > Criar perfis de IA basicos`** cria seis presets em
`Assets/Data/Sistema/PerfisDeIA`. Atribua-os nos campos `Perfil Facil`/`Perfil Normal`/`Perfil
Dificil` do `NpcTreinador`; o `Perfil De Reserva` cobre a dificuldade que ficar vazia, e
com os quatro vazios cai no perfil dos selvagens, configurado no `SistemaBatalha`.

**Troca do adversário:** `TrocarOponente`/`TrocarOponenteDupla` disparam
`SaiuDeCampo`/`EntrouEmCampo` como a troca do jogador — antes só existia a substituição
automática de quem desmaiava.

## Atributos individuais: IV e EV (mapa 7)

`EspecieData` só guarda **base**. O valor final sai de base + nível + IV + EV
(fórmula clássica), em `Pokemon.Atributo`. IVs são sorteados uma vez por
indivíduo (por isso dois Charmander do mesmo nível não são iguais); EVs vêm de
derrotar (cada espécie concede `evConcedido`/`quantidadeDeEv`) e de vitaminas,
com teto de 252 por atributo e 510 no total.

## Evolução (mapa 7/8)

`EspecieData.evolucoes` é uma lista, e cada linha é um destino + uma lista de **condições
como objetos** (`[SerializeReference]`, com o mesmo botão "Tipo ▾" das regras e efeitos).
**Todas** precisam valer; lista vazia = evolui só por chegar ali.

> *Correção de 05/09:* quando esta seção foi escrita, o botão "Tipo ▾" **não** cobria
> condição de evolução — o drawer estava registrado só para `EfeitoDeBatalha`,
> `RegraDeBatalha` e `AvaliadorDeIA`. Na prática, o modelo de dados melhorou e montar uma
> evolução à mão piorou: caía no menu "Add Managed Reference" nativo, que é justamente o
> que não funciona direito nesta versão e o motivo de o drawer existir. Registrado o
> `CondicaoDeEvolucaoDrawer`; agora a frase acima é verdade.

| Condição | O que exige |
|---|---|
| `ExigeNivel` | nível ≥ X |
| `ExigeAmbiente` | clima e/ou período agora ("Qualquer" não é exigência) |
| `ExigeItemUsado` | este item foi **usado** nele |
| `ExigeHeldItem` | está **segurando** este held item |
| `ExigeGolpeConhecido` | conhece este golpe |
| `ExigeAmizade` | amizade ≥ X |
| `ExigeCriticos` | críticos acertados (vitalício) ≥ X |
| `ExigeUsoDeGolpe` | usou aquele golpe N vezes (vitalício) |
| `ExigeContador` | um contador de **nome livre** ≥ N |
| `QualquerUmaDestas` | basta **uma** das internas valer — o **OU** |

Eram dez campos fixos até 05/09, e cada condição nova custava um campo na classe, uma linha
no `Vale` e uma no `Especificidade`. Pior: todas eram obrigatórias juntas — a própria doc
mandava criar **duas evoluções idênticas** para dizer "chuva ou noite", o que dobrava o peso
dela no sorteio de desempate. Agora condição nova é uma classe, e o OU é uma condição.

As três **naturezas** continuam distintas de propósito, porque não são intercambiáveis:
**estado** (chovendo, segurando, conhece — pergunta-se agora), **acumulação** (contadores) e
**evento** (o item foi usado). `ExigeItemUsado` e `ExigeHeldItem` são o exemplo: um é item
gasto no Pokémon, o outro é equipamento que ele carrega.

Uma regra **não** cabe em condição nenhuma, porque é sobre a ausência delas: *evolução que
não menciona item usado nunca é disparada por um item* — senão toda poção evoluiria alguém.
Ela mora no `Evolucao.Vale`, e pergunta recursivamente (um `ExigeItemUsado` dentro de um
`QualquerUmaDestas` conta).


### Condições e produtores acrescentados em 10/10/2026

- **Condições:** `ExigeLugar` (o lugar atual: a cena, ou a área de um `LugarDoMundo`; ex. "campo_magnetico"), `ExigeEspecieNaEquipe` (Mantyke com Remoraid), `ExigeTipoNaEquipe` (Pancham com um Sombrio), `ExigeTrocaPor` (Karrablast trocado por Shelmet). O `ContextoDeEvolucao` passou a levar `Lugar`, `Equipe` e `EspecieDaTroca`; o `SistemaEvolucao.Jogador` é o único ponto que procura o jogador.
- **Produtores de contador feitos pelo código** (o `ValidadorDeCatalogo` os conhece): `dano_sofrido` (em `Pokemon.ReceberDano`; Yamask de Galar), `dano_de_recuo` (no recuo; Basculin listrado de branco), os dois **zeram ao desmaiar**; `passos` (o `ContadorDePassos`, que o `DadosJogador` acrescenta sozinho ao jogador, credita no líder da equipe; Pawmo, Bramblin, Rellor).
- **O que sobra ao evoluir:** `EspecieData.Evolucao.especieQueSobra` (Shedinja: com vaga na equipe e uma Poké Ball na mochila).
- **Dados:** Leva 19 (`ConteudoDoShowdown.Leva19`) liga tudo isso nas espécies oficiais.
- **Ainda como evento de roteiro** (gatilho Evento): girar o personagem (Milcery), virar o aparelho (Inkay), as torres do Kubfu, o arco do Yamask de Galar.
### Contadores: quem escreve, quem lê, e por quanto tempo vivem

**Dois produtores**, e a diferença entre eles é onde o fato acontece:

- `EfeitoContar`, num `GatilhoDeEfeito` — conta o que acontece **na batalha**.
- `ContadorDeAmbiente` (no prefab de Sistemas) — conta o que acontece **no mundo**: escuta
  `SistemaMundo.OnAmbienteMudou` e soma, **por indivíduo da equipe**, a cada virada que
  case com o clima/período declarado. É o que faz "evolui depois de passar por 3 chuvas"
  existir. Por indivíduo, e não um contador só no jogador, porque a resposta é diferente
  em cada Pokémon — o capturado ontem atravessou menos chuvas que o do começo do jogo.

**Dois consumidores**: `Evolucao.contadorNecessario` e `CondicaoDeCampo.contadorNecessario`
— este último é o que permite regra, gatilho e efeito de campo lerem contagem **durante** a
luta ("já apanhei 3 vezes, mudo de forma").

**Dois escopos**, escolhidos por quem cria o contador:

| Escopo | Vive até | Vai para o save? | Para quê |
|---|---|---|---|
| vitalício (padrão) | sempre | sim | memória do indivíduo — é o que a evolução lê |
| `porBatalha` | fim da batalha | **não** | estado da luta em curso |

O escopo é definido na **primeira** contagem e não muda depois: deixar um segundo produtor
convertê-lo em silêncio faria a evolução parar de progredir sem nada dizer. Não confundir
com `usosPorEntradaEmCampo`, do gatilho, que é um terceiro escopo (mais curto) e já existia.

**O nome é uma string escrita em dois lugares**, e errar a grafia daria uma condição que
nunca fecha, para sempre, sem log. Por isso o `ValidadorDeCatalogo` **cruza os nomes**: todo
`contadorNecessario` precisa ter algum produtor com o mesmo nome, ou o teste reprova.

**Quando é avaliado** — e são só dois momentos: fim de batalha, para quem subiu de nível
naquela luta (nunca no meio, senão os atributos mudariam com o turno em andamento); e uso
de item pelo `SistemaInventario`, que tenta primeiro a evolução do item e depois a de
nível/ambiente (um doce raro pode ter acabado de cruzar o nível exigido). Consequência
prática de configuração: **uma evolução só por held item, sem `nivelMinimo`, nunca
dispara sozinha** — equipar não é um gatilho.

**Empate**: entre evoluções válidas ao mesmo tempo ganha a de maior **especificidade**
(mais condições preenchidas); empatou nisso também, é sorteio por `peso`.

**Habilidade na evolução**: `Evoluir` preserva o **slot**, não a habilidade. Por isso a
linha evolutiva inteira precisa listar as habilidades **na mesma ordem** — o
`ValidadorDeCatalogo` reprova desalinhamento (contagem de slots e a marca `oculta` por
posição).

### Habilidades da espécie: peso e oculta

`EspecieData.habilidades` é uma lista de `habilidade + peso + oculta`. Cada indivíduo
sorteia **uma** ao nascer, pelo peso — a mesma ideia de peso que a `TabelaEncontros` usa
para escolher espécie, e um número relativo, não porcentagem.

- **peso 1** = comum; **0.1** = rara; **0** = nunca sai sozinha, só por concessão.
- **`oculta`** não afeta o sorteio: marca qual delas `DarHabilidadeOculta()` concede e qual
  a ficha mostra como "(oculta)".
- Espécie com **uma** habilidade só não consome sorteio nenhum — acrescentar a segunda a
  qualquer espécie não pode reordenar a sequência aleatória de todo mundo depois dela.

## Armazenamento (mapa 10)

`CaixaMenu` mostra equipe e caixa lado a lado, move de um lado para o outro e
libera (com confirmação em dois cliques). `PontoDeArmazenamento` é o objeto do
mundo (o "PC") que abre essa tela direto da exploração — é ele que fecha o buraco
de capturar com a equipe cheia. Regras ficam em `DadosJogador`
(`MoverParaEquipe` / `MoverParaCaixa` / `Liberar`), que nunca deixa o jogador sem
ninguém em pé na equipe.

## World / UI

`Interagivel` (base MonoBehaviour) implementa a interface `IInteractable`. Herdam dela:
`AreaDeEncontro` (sorteia encontro ao interagir **e** ao andar dentro), `ItemColetavel` (vai para a
mochila e marca flag para não reaparecer), `Npc` (abre diálogo), `NpcTreinador` (desafia com
equipe própria — ver seção abaixo).
`PontoTransicao` faz a saída da área (outro ponto da cena ou outra cena) — a rota de volta
de `Casa.unity` pra área externa aponta pra `DevScene` por nome de cena
(`SceneManager.LoadScene`), então trocar o nome de arquivo de uma cena que serve de
destino exige atualizar o `cenaDestino` de quem leva até ela.

Detecção de interação: o Player carrega só o colisor físico + Rigidbody, numa layer física
própria (`"Jogador"`, excluída da máscara de obstáculo da câmera); o volume de **trigger**
é de cada interagível, e o raio dele é o alcance de interação — eventos de trigger chegam
tanto ao GameObject do collider quanto ao do Rigidbody dele, então o corpo físico do
jogador entrando no trigger de um interagível já dispara `OnTriggerEnter` nele.

### Batalha de treinador (GDD 23) e recompensa (GDD 24)

`SistemaBatalha` tem duas portas de entrada: `IniciarBatalha(dados, selvagem)` (um só
Pokémon, pode capturar, sem dinheiro) e `IniciarBatalhaTreinador(dados, nome, equipe,
recompensa, ..., inventarioDeBatalha)` (equipe inteira — `Oponente` troca sozinho para o
próximo vivo quando um desmaia — sem captura, dinheiro só quando `Resultado ==
Vitoria`, e agora pode usar item durante a luta se o treinador carregar algum). `EhTreinador`
e `NomeTreinador` ficam expostos para a UI/narração diferenciar as duas.

`NpcTreinador` é a ponte pronta: falas de desafio → fecha o diálogo → chama
`IniciarBatalhaTreinador` com a lista de `Pokemon` montada no Inspector (a mesma ficha
usada em `DadosJogador.equipe`) → registra uma flag de progresso só se `Resultado ==
Vitoria` (perder permite tentar de novo; vencer trava o desafio e mostra `falasVencido`
depois — a menos que a **política de revanche** deixe: *renovável* volta a lutar quando se vence um líder de ginásio ou o campeão
(`TreinadorData.ehMarco`) ou ao entrar na manhã (`SistemaMundo.periodoQueViraODia`, um para o mundo todo); *sem limite* sempre.
A revanche paga igual e usa `falasRevanche` no lugar de `falasDesafio` se preenchida; o time muda só pelas `opcoes` do `TreinadorData`,
que também trocam o perfil de IA por dificuldade. A flag de vitória fica; a espera é `<flag>#espera` e vai no save). Precisa
de `SistemaDialogo` e `SistemaBatalha` na cena (acha sozinho, como o `Npc` de diálogo).

Na UI, `BatalhaUI` e `DialogoUI` só assinam eventos dos sistemas. Todos os campos são
opcionais: o que ficar vazio não é desenhado, então dá para testar a batalha inteira
lendo o Console antes de montar a interface.

### Aprender golpe novo com a ficha cheia

Com menos de 4 golpes, `Pokemon.GanharXp` aprende direto. Com 4, o golpe novo **não**
substitui ninguém sozinho: entra numa fila (`TemGolpePendente` / `ProximoGolpePendente`)
até alguém chamar `ConfirmarAprendizado(indice)` (ou `-1` para recusar). `SistemaBatalha`
drena essa fila logo depois do XP (`OnPedirEscolhaDeGolpe`), narrando e esperando a UI —
`BatalhaUI` abre `painelAprenderGolpe` com os 4 golpes atuais + "Não aprender" e chama
`SistemaBatalha.EscolherGolpeParaEsquecer(indice)` no clique. Como o XP só é dado ao
aliado ativo, só ele pode ter pendência por vez; a fila existe pra não perder um segundo
golpe pendente se o mesmo salto de XP cruzar dois níveis com golpe novo cada um. Sem
ninguém ouvindo `OnPedirEscolhaDeGolpe` (painel não montado, ou testando só pelo Console),
esquece o golpe mais antigo sozinho em vez de travar a batalha esperando uma escolha.

### Um único mapa de Input por jogador (`EntradaJogador`)

`PlayerMovement`, `CameraRotate` e `PlayerInteracao` não criam o próprio `new Inputs()`:
todos pegam o mesmo via `GetComponentInParent<EntradaJogador>().Mapa`, um componente com
`[DefaultExecutionOrder(-100)]` no objeto do jogador. Isso resolve um bug real (duas
instâncias de `Inputs` ouvindo o mesmo botão em ordem que o código não controla — apertar
"Interagir" na última fala de um diálogo podia fechar e reabrir a conversa no mesmo
aperto, porque `PlayerInteracao` via `podeInteragir` liberado a tempo de interagir de novo
com o mesmo alvo) e é o ponto certo pra crescer pra multiplayer: cada jogador só vai
precisar do seu próprio `EntradaJogador` (futuramente com um `PlayerInput` e o device
certo), sem mexer nos outros scripts. `SistemaDialogo` também usa esse mesmo mapa (acha o
`EntradaJogador` da cena) em vez do seu próprio — hoje só existe um jogador, então
funciona; virar multiplayer de verdade ainda vai exigir que diálogo/batalha deixem de ser
singletons de cena e passem a saber de qual jogador vieram (fora do escopo deste slice).

Mesmo bug de fundo tinha outra ponta: `SistemaDialogo.Encerrar()` chamava
`VoltarParaExploracao()` sem checar se alguém (o `NpcTreinador`, ao encadear pra uma
batalha assim que a fala de desafio fecha) já tinha trocado de estado durante
`OnDialogoTerminado` — o que fazia a HUD de interação (e os controles de exploração)
voltarem por engano durante a luta. Agora só volta pra exploração se o estado ainda for
`Dialogo`.

### ID de mundo não depende de `name`

`ItemColetavel.idNoMundo` e `NpcTreinador.flagDeProgresso` têm `OnValidate()` gerando um
GUID sozinho na primeira vez que o Editor toca no objeto (adicionar o componente,
duplicar, ou depois de um recompile) — nunca dependem do nome do GameObject na cena (dois
objetos com o mesmo nome, fácil de acontecer copiando e colando, dividiriam a mesma flag
de progresso senão). Ainda vale saber: isso só roda no Editor — um objeto instanciado em
runtime a partir de um prefab (loot procedural, por exemplo) herda o GUID já gravado no
prefab, então várias cópias do mesmo prefab colidiriam entre si; se um dia existir spawn
procedural desses objetos, o id tem que ser sorteado na hora do spawn, não herdado do
prefab.

### Save

`SistemaSave` grava em JSON (`Application.persistentDataPath`), ids resolvidos pelo
`CatalogoJogo`. Também grava a rotação da câmera (horizontal/vertical, restaurada com
`CameraRotate.DefinirRotacao`) e o nome da cena atual; `Carregar()` leva à cena gravada pelo
`SistemaTransicao` quando ela não é a atual. Formato atual: **7** (`SistemaSave.VersaoDoFormato`). A v5 acrescentou `tipoTera` e `teraEstelar` em `SavePokemon`, a v6 (10/10/2026) `fatorGigantamax`, e a v7 (11/10/2026) `dificuldade` e `ivsDosTreinadores` em `SaveJogo`; as migrações v4→v5 e v5→v6 têm teste (`ItemDeHabilidadeTests`) e a v6→v7 também (`RevanchesDificuldadeEDerrotaTests`). Variáveis do personagem (missões, estatísticas, repelente) vão como flag com prefixo e não mudam o formato.

Ainda fora do escopo deste slice: estado de objetos persistentes que
não passam por uma flag genérica (ex: um `Rigidbody` solto no mundo que o jogador empurrou).

