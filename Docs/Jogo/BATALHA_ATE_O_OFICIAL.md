# Batalha até o oficial — o que falta e como implementar

| | |
|---|---|
| **O que este documento responde** | O que separa a batalha do projeto da dos jogos oficiais, em que ordem fechar, e como cada peça entra no motor |
| **Quando abrir** | Antes de escolher a próxima "leva" de batalha, ou quando alguém perguntar "isto já funciona como no oficial?" |
| **O que ele NÃO é** | O backlog por ID ([`../Projeto/PENDENCIAS.md`](../Projeto/PENDENCIAS.md)) nem a referência do motor ([`GDD.md`](GDD.md) §5, [`REFERENCIA_DE_SISTEMAS.md`](REFERENCIA_DE_SISTEMAS.md)) |
| **Conferido em** | Escrito em 07/10/2026 e **ampliado até 12/10** (levas 12 a 28, §8 a §15; a tabela da §4 foi completada em 14/10). Os números marcados "(docs 30/09)" não foram recontados; os de 09/10 estão em [`../Projeto/ESTADO_DO_PROJETO.md`](../Projeto/ESTADO_DO_PROJETO.md) §2. **Ordem de leitura:** §4 (onde estamos e o que falta) primeiro; as seções 3 e 8 a 13 são o diário de cada leva |
| **Como ler "como implementar"** | É **proposta de desenho**, escrita a partir dos pontos de entrada que existem no código. Nenhuma dessas peças foi prototipada. Tamanhos (P/M/G) são estimativa de quem escreveu o plano |

---

## 1. Primeiro: "oficial" precisa de uma definição

"Igual ao oficial" não é um alvo único. Os jogos mudam de regra por geração e por formato. Sem fixar isto, cada leva discute o mesmo ponto.

| Decisão | Opções | Recomendação | Por quê |
|---|---|---|---|
| **D1. Geração de referência** | Última (Gen 9) · a do FireRed (Gen 3) · "a do Showdown" | **A do Pokémon Showdown, no modo da geração mais nova**, que é o que as levas 1 a 5 já seguem | O projeto já comparou golpe a golpe com o `moves.json` do Showdown; trocar de referência joga fora esse trabalho |
| **D2. Mecânicas de formato** (Tera, Dynamax, Z, Primal) | Todas · nenhuma · só uma | **Nenhuma agora** | Hoje há **zero** código de Terastal, Dynamax e Z-move (busquei nos fontes). Cada uma é do tamanho de um sistema |
| **D3. Formatos de batalha** | 1×1 e dupla (hoje) · + tripla/rotação · + 3×3 | **Ficar em 1×1 e dupla** | Tripla e rotação não aparecem em nenhum documento; fora da faixa do slice |
| **D4. Conteúdo fangame** (o lote `h1000+`, habilidades de Conquest) | Fazer · deixar sem regra | **Deixar** | Não tem equivalente "oficial" para seguir; sem fonte, vira invenção |

Tudo abaixo assume D1 = Showdown (Gen 9), D2 = nenhuma, D3 = 1×1 e dupla, D4 = deixar. Se alguma dessas mudar, as seções E e F mudam.

**Adotadas em 07/10/2026** como padrão de trabalho, quando a implementação começou (a pedido do dono do projeto). Mudar qualquer uma continua possível; só altera o que entra nas levas seguintes.

---

## 2. O que já existe (a base sobre a qual se constrói)

Resumo do que o motor sabe fazer, para dimensionar o resto. Detalhe em [`GDD.md`](GDD.md) §5 e no diário das levas 1 a 5 ([`../Historico/CONTEUDO_DO_SHOWDOWN_LEVAS.md`](../Historico/CONTEUDO_DO_SHOWDOWN_LEVAS.md)).

- **Núcleo:** dano, precisão, prioridade, status, clima, terreno, dupla, IA com dificuldade, determinismo por semente. 27 tipos de evento de batalha (contados em 09/10) (`EventoBatalha`, `Assets/Scripts/Data/Batalha/EventosDeBatalha.cs`).
- **Regras como dado:** `MesaDeRegras` (1.594 linhas) consulta peças tipadas (`RegraDeBatalha`, 41 classes só nas "famílias", `EfeitoDeBatalha`); o asset só combina peças.
- **Showdown, levas 1 a 5 (28 e 29/09):** 240 golpes e 82 habilidades ligados pelo `ConteudoDoShowdown`; as famílias de bloqueio, prisão, reflexo, dupla, consulta ao oponente, ordem de turno, fração do dano, redirecionamento, Instruct, Dancer, Telekinesis e outros estão fechadas (lista no diário das levas 1 a 5).
- **Formas em batalha:** Mega e Deoxys, por `Pokemon.TrocarForma` e `formasAlternaveis`.
- **Item segurado que se perde:** `Pokemon.DesativarHeldItem`, `HeldItemEmUso`, evento `ConsumiuHeldItem`.
- **Rede (costuras):** ação como dado (`AcaoDeRede`), árbitro (`IArbitroDeBatalha`), foto do estado, registro reproduzível. Falta transporte.
- **Testes:** 597 de EditMode passando (docs 30/09); PlayMode não lido; o `SistemaBatalha` só se prova em PlayMode.

---

## 3. As lacunas, por camada

Legenda de **tamanho**: P = um dia ou menos · M = alguns dias · G = uma semana ou mais. **Chão do trabalho:** o que o jogador percebe de diferente.

### A. Golpes sem peça própria (11 golpes) — tamanho M no conjunto

Os assets existem (gerados), mas sem comportamento. **Conferi hoje o Pursuit** (`Assets/Data/Conteudo/Golpes/Gerado/g17004_Pursuit.asset`): poder 40, `efeitos: []`, `regras: []`. Ou seja, hoje ele é um golpe comum de 40 de poder.

| Golpe | O que o oficial faz | Proposta de desenho | Ponto de entrada | Tam. |
|---|---|---|---|:-:|
| **Pursuit** | Se o alvo está **saindo** do campo, bate antes com poder dobrado | Em `TrocarPokemonSlot` e `TrocarOponenteSlot` (`SistemaBatalha.Acoes.cs`), antes de tirar quem sai: procurar na fila uma ação Pursuit contra ele, executá-la com poder ×2 e removê-la da fila. Precisa de um campo de contexto "alvo está trocando" | `SistemaBatalha.Acoes.cs`, `Golpes.cs` (fila) | M |
| **Snatch** | Rouba o efeito de um golpe de status "pegável" usado por outro nesse turno | Status de um turno, no molde do Magic Coat (`RegraDeRefletirGolpeDeStatus`). Pede uma etiqueta nova em `CaracteristicaGolpe` ("pegável"), como foi feito com `Danca` | `RegrasDeBatalhaFamilias.cs`, `ConteudoDoShowdown` | M |
| **Bide** | Guarda 2 turnos de dano e devolve o dobro | Status com contador e dano acumulado; ao fim, efeito de dano fixo (peça `DanosFixos` já existe). O oficial faz o usuário passar o turno inteiro sem agir | `MemoriaDeBatalha`, `DanosFixos.cs` | M |
| **Me First** | Usa o golpe escolhido pelo alvo, com poder ×1,5, antes dele | Lê a fila (como `ReposicionarAcaoDe` faz) e chama o golpe como "golpe chamado" (como Instruct). Falha se o alvo já agiu ou se o golpe é de status | `SistemaBatalha.Golpes.cs` | M |
| **Beat Up** | Um acerto por Pokémon são da equipe, com poder derivado do Ataque base de cada um | Fórmula nova em `FormulasDePoder.cs` + número de acertos vindo da equipe | `FormulasDePoder.cs` | P |
| **Speed Swap**, **Guard Split**, **Power Split** | Troca ou faz a média de Velocidade, Defesas ou Ataques entre os dois | Efeito que grava um "valor substituído" por Pokémon, lido no cálculo de atributo. É a mesma peça dos três | `EfeitosDeBatalhaFamilias.cs`, `Pokemon` | M |
| **Conversion 2** | Muda o tipo do usuário para um que resista ao último golpe **usado pelo alvo** (geração 5+) | Lê `Memoria.ultimoGolpe` **do alvo** (já existe; não precisa de campo novo) e sorteia entre os tipos que resistem | `EfeitosDeBatalhaFamilias.cs` | P |
| **Powder** | Se o alvo usar Fogo neste turno, o golpe falha e ele leva dano | Status de um turno que veta tipo e causa dano (a família "bloquear golpe do oponente" já tem a base: `PermiteGolpeRecebido`/`PermiteGolpeEscolhido`) | `RegrasDeBatalhaFamilias.cs` | P |
| **Ally Switch** | Em dupla, troca de posição com o aliado | **Precisa de operação nova de posição** no `SistemaBatalha` (hoje não há troca de posto sem sair de campo) | `SistemaBatalha.cs`, `Campo.cs` | M |
| **Teatime** | Todos comem a baga que seguram | Só faz sentido depois do lote de bagas (seção C) | — | P, depois de C |
| **Victory Dance, Shelter, Power Shift** | **São oficiais, do Pokémon Legends: Arceus** (2022), que tem outro sistema de batalha: sem estágios de atributo, com estilos Ágil/Forte e ordem por ação. As descrições do projeto são as desse jogo | Adaptar como o Showdown faz para jogos sem esse sistema (bônus em estágios comuns), deixar sem efeito, ou reproduzir o sistema do Legends (fora do escopo) | — | P, depois da decisão |

**Leva 7 feita em 07/10/2026** (`ConteudoDoShowdown.Leva7.cs`, menus `Pokemon/Showdown/Leva 7 - 1` e `- 2`): **Beat Up** (`RegraDeAcertosPelaEquipe` + `PoderDoBeatUp`), **Conversion 2** (`EfeitoConversao2`, lê o `ultimoGolpe` do alvo), **Powder** (status `Polvora` com `RegraDeFalharPorFiltro` e gatilho `GolpeFalhou`), **Speed Swap**, **Guard Split** e **Power Split** (`EfeitoDividirAtributos` + `MemoriaDeBatalha.brutoSubstituido`, lido em `Pokemon.AtributoEmBatalha`). Testes: `Leva7DoShowdownPlayModeTests` (5). Também em 07/10, golpes de Gen 8/9 que já cabiam nas peças: Power Trip, Scale Shot, Dragon Energy, Aqua Cutter, Esper Wing, Flower Trick, Infernal Parade. **Restam da tabela acima:** Pursuit, Snatch, Bide, Me First (Leva 8), Ally Switch (Leva 11), Teatime (depois das bagas).

**Leva 8 feita em 08/10/2026** (`ConteudoDoShowdown.Leva8.cs`, menus `Pokemon/Showdown/Leva 8 - 1` e `- 2`): **Pursuit** (`PoderDoPursuit`; antes de uma troca voluntaria, `SistemaBatalha.PerseguirQuemSai` tira da fila o Pursuit do outro lado que mira quem sai e o executa ali, com poder x2 — `PokemonSaindo` diz a formula quem esta saindo; nao pega quem sai por U-turn/Volt Switch), **Me First** (`EfeitoChamarGolpe` com a origem nova `GolpeEscolhidoPeloAlvo` e `multiplicadorDePoder` 1,5, que viaja no `ContextoGolpe.MultiplicadorDePoder` do golpe chamado; falha se o alvo ja agiu ou escolheu golpe de status; nao copia os 12 da lista "noMeFirst"), **Snatch** (status `Roubando` de um turno com `RegraDeRoubarGolpe` + a etiqueta nova `Pegavel` em 71 golpes; o roubo gasta o status) e **Bide** (`EfeitoIniciarCarga.turnosDePreparo` = 2 e o dano fixo `DanoAcumuladoNaCarga`, 2x o dano levado desde o comeco da carga; o asset gerado tinha poder 60, corrigido). A lista de golpes "pegaveis" e a do "noMeFirst" foram escritas de memoria do Showdown (o arquivo de dados dele nao esta nesta maquina): conferir. Testes: `Leva8DoShowdownPlayModeTests` (5), bateria 6/6. **Restam:** Ally Switch (Leva 11) e Teatime (depois das bagas).

**Fora da conta de propósito:** Splash, Celebrate e Hold Hands (sem efeito legítimo no jogo). Happy Hour saiu na Leva 12 (dobra o dinheiro da vitória). Os golpes Shadow do Pokémon XD foram **apagados em 08/10** (o jogo não terá XD). Mountain Gale e Triple Arrows seguem a versão Scarlet/Violet e Showdown (decisão de 08/10); Upper Hand saiu na Leva 12.

### B. Habilidades — o bloco que mais pesa

**Situação (docs 30/09):** 207 das 513 habilidades sem regra nem gatilho; **47 estão em espécies jogáveis** (o resto é fangame e Conquest). Essas 47 são o alvo real, não as 207.

**B1. Formas por gatilho — um mecanismo para oito habilidades — tamanho G.**

| Habilidade | Gatilho | Forma |
|---|---|---|
| Zen Mode (Darmanitan) | HP ≤ 50% no fim do turno | Zen ↔ Padrão |
| Shields Down (Minior) | HP ≤ 50% | Meteoro ↔ Núcleo |
| Forecast (Castform) | Clima | Sol / Chuva / Neve / Normal |
| Disguise (Mimikyu) | Primeiro golpe recebido | Disfarçado → Descoberto; **absorve o golpe** (e dano de 1/8 do HP) |
| Ice Face (Eiscue) | Golpe físico recebido; volta com neve | Gelo ↔ Sem Gelo |
| Gulp Missile (Cramorant) | Usar Surfar/Mergulho; ao ser acertado, devolve | Normal → Com Presa / Com Raio |
| Hunger Switch (Morpeko) | Fim de cada turno | Satisfeito ↔ Faminto |
| Illusion (Zoroark) | Entrar em campo | Aparência de outro Pokémon da equipe |

**Proposta:** uma peça única **"forma por gatilho"**: *quando* (evento) + *se* (condição, via `CondicaoDeCampo`, que já lê HP, clima, status e gênero) + *qual forma* (`formasAlternaveis`) + *como volta* (ao sair de campo, ao fim da luta, por clima). A base já existe: `Pokemon.TrocarForma` (permanente) e o mecanismo da Mega (temporária, com `formaOriginal`). O que falta é o **gatilho** e a **volta automática**.

- **Disguise** e **Ice Face** exigem um segundo passo: absorver o dano. Isso cai no evento `AntesDoDano`, que já existe.
- **Illusion** é a exceção: é **apresentação** (o oponente vê outro nome e sprite) e **IA** (não deve saber). Deixe por último; o motor não precisa dela para o jogo funcionar.
- **Dependência de dados — conferida em 07/10/2026 nos assets (`Assets/Data/Conteudo/Especies/Gerado`):** as espécies de forma **já existem** para Darmanitan (Zen), Eiscue (Ice/Noice), Morpeko (FullBelly/Hangry), Minior (base + 7 núcleos `_2239` a `_2251`), Wishiwashi (School), Aegislash (Blade), Meloetta (Pirouette), Greninja (Ash) e Palafin (Hero, já feito). **Criadas em 07/10** (cópia da forma base, só tipo, nome, id e `formaDe` mudam): Castform Sol/Chuva/Neve (Fogo/Água/Gelo), Cramorant Com Presa/Com Raio e **Mimikyu Descoberto** — o `p0778_Mimikyu_10185` que parecia ser ele é uma "Mega Mimikyu". As formas novas **não têm imagem própria** (usam a da base) nem modelo 3D. A mesma peça cobre também **Stance Change** (Aegislash), **Schooling** (Wishiwashi), **Battle Bond** (Greninja) e **Relic Song** (Meloetta), que não estavam na lista acima.
- **Megas de fã — removidas em 07/10/2026 (aprovado):** 154 espécies que o gerador inventou como "Mega" (lendários e míticos, as 17 formas de Arceus por tipo, Rotom, Genesect, Tapus, iniciais e evoluções finais etc.), com as 51 evoluções-fantasma que as ligavam às bases e as 5 habilidades de fã que sobraram. Ficam as Megas oficiais da Gen 6/7, as Primais de Kyogre e Groudon e as 19 que parecem ser do Legends Z-A (Dragonite, Meganium, Feraligatr, Skarmory, Froslass, Heatran, Darkrai, Emboar, Excadrill, Chandelure, Chesnaught, Delphox, Greninja, Hawlucha, Golisopod, Zeraora e três de Zygarde; a conferência delas é do grupo). `p2016_Muka` está com nome "Mega Muk(Alola)" mas é um Muk de Alola mal nomeado. Ferramenta: `ConteudoDoShowdown.Limpeza.cs` (lê a lista de uma variável de ambiente, porque o Unity apaga a pasta `Temp` ao abrir). **Aviso:** os arquivos de origem do gerador (`Assets/Omega/Dados/especies_progressao.json`, `Assets/Omega/Editor/sprites_pokemon.json`) ainda citam essas espécies; rodar o gerador de novo pode recriá-las. **Congelados em 07/10** (`COMO_TRABALHAR.md` §4.1): os menus geradores recusam rodar, e um teste falha se algum conteúdo removido voltar.

**Leva 10 feita em 08/10/2026** (`ConteudoDoShowdown.Leva10.cs`, menu `Pokemon/Showdown/Leva 10 - formas por gatilho`). Uma peca, `EfeitoAlternarForma` (assume a forma, troca de uma forma temporaria para outra ou volta a base; `alternar` para ir e voltar; `soDeEspecies` para golpe), pendurada nos gatilhos de cada habilidade com a condicao de especie, HP e clima: **Zen Mode** (fim do turno, HP <= 1/2), **Shields Down** (entrada e fim do turno; o nucleo e o vermelho, `_2239`), **Forecast** (entrada e o evento novo `ClimaMudou`, que a batalha dispara depois de qualquer acao que mudou o clima), **Disguise** e **Ice Face** (a regra nova `RegraDeAbsorverGolpe` zera o golpe de dano — Ice Face so fisico — e o evento novo `GolpeAbsorvido` troca a forma; Disguise perde 1/8; o gelo volta quando comeca a nevar ou ao entrar na neve; disfarce caido e face quebrada ficam ate o fim da luta), **Gulp Missile** (Surf e Dive enchem a boca conforme o HP; quem o acertar leva 1/4 do proprio HP e perde Defesa ou e paralisado), **Hunger Switch** (alterna no fim do turno), **Stance Change** (antes do golpe: dano vira Lamina, King's Shield volta a Escudo), **Schooling** (nivel 20+ e HP > 1/4) e o golpe **Relic Song** (alterna Aria e Pirouette, so no Meloetta). Pecas novas de apoio: `CondicaoDeCampo.nivelMinimo` e `semNenhumDestesClimas`, `CondicaoGolpe.golpes` (filtro por golpe, comparado pelo id). **Battle Bond** pela regra da Gen 9: ao derrubar um oponente, +1 Ataque, Ataque Especial e Velocidade, uma vez por batalha (contador por batalha + a condicao nova `contadorAbaixoDe`); sem forma Ash. **Fora:** Illusion (apresentacao e IA), Power Construct, a imunidade a status do Minior Meteoro. Testes: `Leva10FormasPlayModeTests` (10).

**B2. Aliado em dupla — tamanho M.** Telepathy, Symbiosis, Commander, Receiver, Power of Alchemy. Já há `RegraDeBatalha.alcance` + `EfeitoEmAliado`. Falta, por habilidade, o gancho: Symbiosis passa o item ao aliado quando ele o consome (usa `ConsumiuHeldItem`); Receiver e Power of Alchemy copiam a habilidade do aliado que desmaiou (padrão do Trace/Doodle); Commander é a mais estranha (Tatsugiri entra na boca de Dondozo) e pede operação de posição, como o Ally Switch.

**Leva 11 feita em 08/10/2026** (`ConteudoDoShowdown.Leva11.cs`, menu `Pokemon/Showdown/Leva 11 - aliado em dupla e itens`). **Aliado em dupla:** o gatilho ganhou `escutaOAliado` (dispara com o evento de um aliado em campo; a `MesaDeRegras.Disparar` roda, depois dos gatilhos do dono, os de quem escuta) — **Symbiosis** (o aliado gastou o item: passa o seu), **Receiver** e **Power of Alchemy** (o aliado caiu: copia a habilidade dele, sem a lista de excecoes do oficial); **Telepathy** (`RegraDeIgnorarGolpeDoAliado`; aviso: nenhum golpe de area do motor atinge o proprio lado, entao hoje quase nao age); **Ally Switch** (`EfeitoTrocarDePosicao` + `SistemaBatalha.TrocarDePosicaoComAliado`: troca os postos e quem ja mirava um passa a mirar o outro). **Itens:** Quick Claw (`RegraDeAgirPrimeiroNaFaixa`, 20%, sorteado so para quem tem), Air Balloon (bloqueia Terra; estoura ao levar golpe), Power Herb (`RegraDeDispensarCarga`; nao apressa o Bide), Eject Button (`RecebeuDano` + troca forcada), Eject Pack (evento novo `AtributoCaiu`, qualquer queda, inclusive Overheat), Damp/Heat/Icy/Smooth Rock e Light Clay (`RegraDeDuracaoProlongada`: 8 turnos). Todos com preco provisorio e na loja de teste. **Fora:** Commander, Big Root, Binding Band, Grip Claw, Zoom Lens, Metronome (item); o Eject Button gasta o item mesmo sem reserva para entrar. Testes: `Leva11PlayModeTests` (10), bateria 6/6.

**B3. Pequenas — tamanho P cada.** Aura Break (inverte as Auras: `RegraDe…` consultada pelas Auras que já existem), Opportunist (copia a subida de atributo do oponente: evento `AtributoMudou`), Ball Fetch (fora de batalha).

**B4. Bagas — adiadas de propósito.** Cud Chew, Gluttony, Unnerve, Ripen não têm sobre o que agir: o projeto tem **18 itens segurados e 2 bagas**. Dependem da seção C.

**B5. A lista real — auditada em 07/10/2026** (menu `Pokemon/Itens/Auditar habilidades vazias em especies oficiais`, relatório em `Temp/habilidades_vazias.txt`). **41 habilidades** sem regra nem gatilho estão em espécie com número de Pokédex oficial (1 a 1025). Elas se dividem assim:

| Grupo | Habilidades | Onde entra |
|---|---|---|
| **De fã, em formas "Special" e Megas inventadas** | Armadura Móvel, Aura da Vida, Aura do Abismo, Aura do Domínio Secreto, Aura do Inverno, Domínio do Espaço/do Tempo, Electric/Fire Mirage, Espaço paralelo, Glória Sombria, Guarda do Deus do Mar, Invocar Tempestade, Luz Eterna, Território Congelado/das Chamas/do Trovão, Trovão e Relâmpago (18) | **Todas removidas em 07/10**, junto com as formas e as Megas de fã que as usavam |
| Forma por gatilho | Disguise, Ice Face, Hunger Switch, Shields Down, Forecast, Gulp Missile, Illusion | Leva 10 |
| Aliado em dupla | Commander, Receiver, Symbiosis, Telepathy (46 espécies) | Leva 11 |
| Bagas | Cud Chew, Gluttony (28 espécies), Ripen, Unnerve (30 espécies) | Depois de haver bagas de pinça (Sitrus, Lum, as de 25%): hoje só existem as de resistência e 2 feitas à mão |
| Só sabor (a forma já fixa o tipo) | Multitype (36), RKS System | **Ficam vazias de propósito**, como as Memórias |
| Pequenas | Aura Break, Opportunist, Poison Puppeteer, Piercing Drill | Leva 12. Piercing Drill (Legends Z-A) atravessa proteção com dano reduzido: falta o valor exato antes de ligar |
| Fora do escopo | Tera Shift (D2), Ball Fetch (fora de batalha) | — |

### C. Itens segurados e bagas — a maior distância, tamanho G

**Correção de 07/10/2026.** A primeira versão desta seção dizia que só 9 itens tinham comportamento e que Choice Band, Leftovers etc. eram `...Scroll` inertes. Isso estava errado: os itens reais existem como `HeldItemData` em `Assets/Data/Conteudo/Itens/Gerado/` (347 assets), e os `...Scroll` são outra coisa (`ItemGenerico`). **344 dos 347 estavam vazios** (sem regra e sem gatilho); só `ClearAmulet`, `CovertCloak` e `LoadedDice` tinham algo, além dos 9 feitos à mão em `Assets/Data/Conteudo/HeldItems`.

**Feito em 07/10/2026** (`ConteudoDoShowdown.Itens.cs`, menu `Pokemon/Itens/Preencher itens segurados`): **248 itens preenchidos** com peças que já existiam (depois, 157 deles foram removidos como itens de fã, ver abaixo; ficam **91**): 23 reforços de tipo oficiais (Charcoal, Black Belt, Mystic Water, incensos…), 17 gemas, 18 bagas de resistência, Leftovers, Flame/Toxic Orb, Life Orb, Muscle Band, Wise Glasses, Expert Belt, Scope Lens, Razor Claw, Wide Lens, Bright Powder, Lax Incense, Lagging Tail, Full Incense, Float Stone, Protective Pads, Assault Vest, Focus Sash, Rocky Helmet, Shell Bell, King's Rock, Razor Fang, Weakness Policy, Absorb Bulb, Luminous Moss, Cell Battery, Snowball, Red Card, Mental Herb e as 4 sementes de terreno. Valores de Fling e de Natural Gift onde o oficial é conhecido. Para isso houve duas peças de runtime: o filtro de efetividade passou a valer também em gatilhos (`MesaDeRegras.Disparar` recebe a efetividade) e o efeito `EfeitoAnunciar`, que deixa um gatilho "só consome" (gema, baga) acontecer. Testes: `ItensSeguradosPlayModeTests` (8).

**Fechados depois, ainda em 07/10 (resto da Leva 9):** Choice Band/Specs/Scarf (status `PresoAoGolpe`: o Encore sem prazo; limite: a trava não some se o item for derrubado), Black Sludge e Eviolite (condições novas `semNenhumDestesTipos` e `aindaPodeEvoluir` na `CondicaoDeCampo`), White Herb (`EfeitoDesfazerQuedas` + condição `comAlgumEstagioNegativo`) e Focus Band (`chance` na `RegraDeSobrevivenciaGarantida`, sorteada só quando é menor que 100, para não mexer na sequência do PRNG do conteúdo antigo). Testes: Choice Band, Black Sludge e White Herb em `ItensSeguradosPlayModeTests` (11 no total).

**Continuam sem comportamento**, por falta de peça ou por decisão: Quick Claw (falta "agir primeiro na faixa" com chance), Air Balloon, Power Herb, Eject Button/Pack, rochas de clima, Light Clay, Big Root e os demais listados no relatório. **Itens de fã removidos (07/10):** toda a família de reforço com nível (Tiny/Normal/Rare/Advanced/Epic/Legend, "Collect 3 to compose"), que não existe no oficial e não tem no jogo a mecânica de juntar 3: **168 itens** (157 preenchidos + 11 contraditórios). Os 11 que tinham nome contradizendo a descrição (EpicBlackBelt, LegendDragonFang, NeverMeltIce_100411, NormalBlackBelt e NormalBlackBelt_115028, NormalNeverMeltIce, NormalPinkLetter, NormalPolkadotBow, TinyPolkadotBow, NormalSeaIncense, NormalSilkScarf) eram da família Tiny/Normal/Rare/Advanced/Epic/Legend ("Collect 3 to compose"), que não existe no oficial. Foram apagados do projeto, do `CatalogoJogo` e da loja de teste (nada mais os referenciava). As Memórias do Silvally ficam vazias de propósito (a forma já fixa o tipo e dar tipo à memória mudaria o Techno Blast).

**Alcance e preço:** os 159 itens preenchidos que estavam com preço 0 receberam **preço provisório de 2000** e entraram em `LojaDeTeste_TodosOsItens`. O preço definitivo continua sendo decisão de conteúdo ([`../Projeto/DECISOES.md`](../Projeto/DECISOES.md) P5).

**Itens de lendário corrigidos em 07/10/2026.** Os itens oficiais exclusivos de espécie existiam com o **tipo errado** (a Light Ball era `ItemPokebola` e podia ser usada como Poké Bola em batalha; Lustrous Orb, Thick Club, Stick, Lucky Punch e Soul Dew eram `ItemEvolucao`). Foram convertidos no lugar para `HeldItemData` (o id e o save não mudam) e ganharam o efeito oficial, só para a espécie certa, pela nova condição `comAlgumaDestasEspecies`: **Adamant Orb** (Dialga: golpes de Dragão e Aço ×1,2), **Lustrous Orb** (Palkia: Dragão e Água ×1,2), **Griseous Orb** (Giratina: Dragão e Fantasma ×1,2), **Soul Dew** (Latias e Latios: Dragão e Psíquico ×1,2, como no Showdown Gen 9; a descrição do projeto ainda diz a regra antiga), **Light Ball** (Pikachu: Ataque e Ataque Esp. ×2), **Thick Club** (Cubone e Marowak: Ataque ×2), **Stick** (Farfetch'd e Sirfetch'd: crítico +2) e **Lucky Punch** (Chansey: crítico +2). Seus seis `...Scroll` de fã foram apagados. Testes: Light Ball e Thick Club em `ItensSeguradosPlayModeTests` (13 no total). **Fora de escopo:** Metal Powder e Quick Powder (precisam saber se o Ditto se transformou), as Rusted Sword/Shield (forma Coroada), Berserk Gene e os itens-chave (Jade Orb, Wings, Stones). As Orbs de Groudon e Kyogre foram feitas depois (ver abaixo).

**Os `...Scroll` de fã foram removidos em 07/10/2026**, por decisão do dono do projeto: ficam só `PulseCannonScroll` (golpes Pulse +20%) e `PowerRingScroll` (Defesa +10%), que agora têm esse efeito. Os outros 81 saíram do projeto, do catálogo e da loja de teste.
**Reversão Primal feita em 07/10/2026.** `RedOrb` e `BlueOrb` eram itens de fã (ItemGenerico: "Groudon (Omega)" e "convoca o Kyogre para a Embedded Tower"). Foram convertidas para item segurado oficial: ao **entrar em campo**, só o Groudon (Red Orb) ou só o Kyogre (Blue Orb) vira a forma Primal; o item não pode ser derrubado. Não é a Mega: não é escolha do jogador nem uma por batalha. Peças: gatilho `EntrouEmCampo` + `EfeitoAssumirForma` com a nova opção `redispararEntrada` (depois da troca dispara a entrada de novo, para a Desolate Land / Primordial Sea da forma agirem; a Mega não usa) + a condição de espécie `=p0383_Groudon` (só a base, nunca a Primal). As formas Primais já tinham as habilidades certas; faltava o `formaDe` apontando para a base, agora ligado. **Regra de clima, conferida em batalha:** clima comum (Rain Dance) não tira o clima forte; clima forte tira o outro forte (a Primordial Sea tira a Desolate Land); cada um volta quando o dono sai de campo. O motor já distinguia fraco de forte pelo dono do clima (`MudarClima` com `donoForte`), com a mesma regra de "nível" que o grupo descreveu (comum < forte; forte contra forte troca), então **não foi preciso refatorar** para números. Testes: `ReversaoPrimalPlayModeTests` (5).

**Pulse Cannon e Power Ring (os dois `...Scroll` que ficaram)** viraram item segurado com o efeito da própria descrição: golpes de pulso ×1,2 e Defesa ×1,1.

**Proposta, no mesmo molde do `ConteudoDoShowdown`:**
1. Escolher o **núcleo competitivo** primeiro: Leftovers, Choice Band/Specs/Scarf, Life Orb, Focus Sash, Assault Vest, Rocky Helmet, Eviolite, Expert Belt, as caixas de reforço de tipo, Black Sludge, Shell Bell, Sitrus e as bagas de status.
2. Cada item = `HeldItemData` com as regras e os gatilhos que a descrição pede (as peças de Showdown da seção anterior cobrem a maior parte: reforço de atributo, cura no fim do turno, prender a um golpe, sobreviver a um golpe).
3. **Poder de Fling e de Natural Gift** (hoje 0, então os golpes falham) e `efeitosAoArremessar` são campos já prontos; é preencher.
4. Ligar ao catálogo e a **uma loja** (senão o auditor acusa "ninguém dá").
5. **Preço:** 2.202 itens têm preço 0 e nenhuma loja os vende (docs 30/09). Isso é decisão de conteúdo ([`../Projeto/DECISOES.md`](../Projeto/DECISOES.md) P5).

**Decisão D5:** o `ItemGenerico` gerado não é o item oficial; o catálogo pode ficar com os dois nomes (`Choice Band` e `Choice Band Scroll`)? Conferir antes de gerar, para não dobrar a Pokédex de itens.

### D. Campo — quase fechado, tamanho P

Terrenos: `CampoDeGrama`, `CampoEletrico`, `CampoNebuloso`, `CampoPsiquico` (conferi hoje: 4 assets) — o conjunto oficial. Climas com `ClimaData`: Sol, Neve, Areia e Ventos Fortes (4 assets). **Atenção:** não achei `ClimaData` de **Chuva**; o `COMO_TRABALHAR` §4 diz que o bônus de Água na chuva já vive na tabela do `SistemaMundo`. Confirme que a chuva **em batalha** (Thunder sem erro, Swift Swim, Rain Dance) está coberta por esse caminho antes de dar por fechado.

Aberto no backlog: Grama × Earthquake (golpe fraco sobre o Campo de Grama).

### E. Mecânicas de formato (decisão D2) — tamanho G cada

| Mecânica | O que exige | Quando faz sentido |
|---|---|---|
| **Terastal** | Tipo extra por Pokémon (no asset e no save), botão por turno, regra de STAB com o novo tipo, o evento de mudança de tipo | Se o alvo for Gen 9 competitivo |
| **Dynamax** | Golpes Max (18 tipos), HP dobrado por 3 turnos, imunidades, a UI | Se o alvo for Gen 8 |
| **Z-moves** | Item Z, golpe Z por tipo, uma vez por luta | Se o alvo for Gen 7 |
| **Primal** (Groudon/Kyogre) | Já parcialmente coberta pelos "climas fortes com dono" | Pequeno |

**Atualização 09/10:** Z, Dynamax e Tera foram implementados (§12), desligados por padrão; quais o jogo usa continua sendo a decisão D2.

### F. Apresentação do que já se calcula — tamanho M, **maior ganho percebido**

O motor já calcula tudo isto; o jogador não vê (`UI_DO_PROJETO.md` §3.5):

1. **Clima, terreno e efeitos de campo por lado** (Espinhos, telas, Vento a Favor, Trick Room…) — hoje nada na tela.
2. **Efetividade do golpe** contra o alvo ("é super efetivo" antes de escolher).
3. **Golpe de vários turnos explicado** (carregando, recarregando) — hoje lê-se como bug.
4. **Estágios, forma atual, substituto e marca de proteção** (estágios e status já aparecem).
5. **Resultado da batalha** (XP, nível, golpe novo, dinheiro) e a **tela de aprender golpe**.
6. **Revival Blessing sem escolha** (hoje volta o de maior nível; o oficial pergunta).

Depende da decisão da pilha de UI (uGUI × Toolkit).

### G. IA — tamanho M

A IA **não troca de forma** (`PENDENCIAS` M14). Ao entrar a seção B1, ela precisa de regra para "vale a pena mudar?". Também não conhece Pursuit, Snatch, Me First ou Ally Switch. Cada peça nova de A e B1 precisa de uma linha na `CerebroDeIA` ou, no mínimo, de um teste de que a IA não escolhe o golpe de forma burra (o `AnaliseDeIAWindow` mostra o que ela pensa).

### H. Rede — tamanho G, frente separada

Faltam: reconstruir a batalha a partir da foto (hoje só tira), transporte, árbitro remoto, e **prazo com padrão para o cliente remoto que não responde**. (A espera da UI local de aprender golpe já tem prazo: `QuadrosDeEsperaDaEscolha` = 3600 quadros, depois não aprende e segue. Falta o equivalente para uma escolha remota.) Lista completa em [`../Planos/REDE_E_MULTIPLAYER.md`](../Planos/REDE_E_MULTIPLAYER.md). Os requisitos de determinismo estão na memória do projeto: semente só reproduz as rolagens; o estado inicial (IV, natureza, quem entrou) precisa viajar. Não é "igualar o oficial" em regra, mas é parte de ter batalha como no oficial.

---

## 4. Ordem recomendada

Do que dá mais resultado por esforço para o que dá menos. Cada leva fecha com bateria de 6+ rodadas de PlayMode.

| Leva | Conteúdo | Situação |
|---|---|---|
| **6** | **Apresentação (F), itens 1 a 3**: clima, terreno e campo na tela; efetividade; golpe de vários turnos | **Não começada.** Depende da decisão da pilha de UI ([`../Projeto/DECISOES.md`](../Projeto/DECISOES.md) P1); a `BatalhaUI` é uGUI e é a única tela de batalha |
| **7** | Golpes pequenos (A): Beat Up, Conversion 2, Powder, Speed Swap, Guard/Power Split (+7 golpes de Gen 8/9) | ✅ 07/10 |
| **8** | Golpes que mexem na fila (A): Pursuit, Snatch, Me First, Bide | ✅ 08/10 |
| **9** | Itens segurados (C) | ✅ 07/10 no que cabia (98 itens); o resto saiu nas levas 11 e 13 |
| **10** | Formas por gatilho (B1), sem Illusion: 9 habilidades e o Relic Song; Battle Bond pela regra da Gen 9 | ✅ 08/10 |
| **11** | Aliado em dupla (B2) e Ally Switch; os 11 itens que faltavam da leva 9 | ✅ 08/10 |
| **12** | O que a auditoria completa achou (§8): 40 golpes, 5 habilidades, 20 itens | ✅ 08/10 |
| **13** | Bagas, Mega pela espécie certa, placas, itens de origem (§9); golpe de dano mirando o aliado e as 94 Megas oficiais (§10). IA: `AvaliadorDeGolpeQueVaiFalhar` | ✅ 08 a 09/10 |
| **14** | Multitype, Judgment, restrições de habilidade; formas base de fã apagadas (§11) | ✅ 09/10 |
| **15** | Z-moves, Dynamax e Tera, e quem usa cada mecânica (§12, §13) | ✅ 09/10, **desligados por padrão** |
| **16** | Leppa Berry e as bagas da Gen 2 | ✅ 09/10 |
| **17** | Queimadura corta o dano físico; Guts e Facade ignoram | ✅ 10/10 |
| **18** | Repelentes e taxa de encontro por habilidade e item | ✅ 10/10 |
| **19** | Evoluções por equipe, troca, dano, passos e Shedinja | ✅ 10/10 |
| **20 e 21** | Ogerpon e Gigantamax (formas, 33 golpes G-Max) e os 10 efeitos que pediam peça nova (§14) | ✅ 10/10 (conteúdo de fã removido depois, em 13/10) |
| **22** | O que a conformidade de fluxo achou: Intimidate barrada, Poison Heal, espinhos tóxicos, Flash Fire, areia (§15) | ✅ 11/10 |
| **23** | Struggle | ✅ 11/10 |
| **24** | Anúncio de habilidade ao entrar em campo (13 habilidades) | ✅ 11/10 |
| **25** | Dupla: Intimidate baixa todos os adversários; alvo "todos os adjacentes" vem do importador | ✅ 12/10 |
| **26** | Ability Capsule e Ability Patch | ✅ 12/10 |
| **27** | Varredura por golpe contra o simulador do Showdown (efeitos em si mesmo, Tera Blast, preparo, ignorar habilidade) | ✅ 12/10 |
| **28** | Efeitos dos itens contra o texto de Scarlet/Violet (Potion 20, EV, revives) | ✅ 12/10 |
| **Em paralelo** | Rede (H) | Frente separada: [`../Planos/REDE_E_MULTIPLAYER.md`](../Planos/REDE_E_MULTIPLAYER.md) |

### O que falta de verdade (consolidado em 09/10, atualizado em 14/10)

Tirado das seções acima; as levas 17 a 28 estão na tabela acima e o que elas fecharam está riscado abaixo. Os itens 2, 3, 5, 6, 7 e 9 **não foram reconferidos em 14/10**. **Esta lista é a fonte de verdade do que falta na batalha.**

1. **Apresentação (leva 6)** — clima, terreno, efeitos de campo por lado, efetividade, golpe de vários turnos, estágios/forma/substituto, resultado da batalha, tela de aprender golpe. Destrava o que o jogador *vê* do motor. Bloqueada pela decisão da pilha de UI.
2. **Illusion** (apresentação + IA: o oponente vê outro nome e sprite; a IA não pode saber do disfarce; cai no primeiro dano de golpe) e **Commander + Order Up** (pede um estado novo de posto "ocupado, mas fora do alcance e sem agir" que a fila, os alvos, a troca e o fim da batalha respeitem; faltam as formas Droopy e Stretchy do Tatsugiri).
3. **Victory Dance, Shelter e Power Shift** (oficiais do Legends: Arceus): decisão ([`../Projeto/DECISOES.md`](../Projeto/DECISOES.md) P11).
4. ~~**Formas que o projeto não tem:** Gigantamax, Ogerpon, formas Tera, evolução por lugar~~ **Feito nas levas 19 a 21** (10/10); o conteúdo de fã foi removido em 13/10 — [`AUDITORIA_OFICIAL.md`](AUDITORIA_OFICIAL.md).
5. **Peças soltas:** Micle Berry (precisão ×1,2 no próximo golpe), Booster Energy (Protosynthesis e Quark Drive sem clima nem terreno), Piercing Drill (falta o valor exato), Metal Powder e Quick Powder (precisam saber se o Ditto se transformou; hoje `ItemEvolucao`), Rusted Sword/Shield (forma Coroada), Berserk Gene, Destiny Bond e Pain Split com o HP sem Dynamax, Revival Blessing com tela de escolha, Grama × Earthquake (golpe fraco sobre o Campo de Grama), Telepathy (nenhum golpe de área do motor atinge o próprio lado), Punching Glove "sem contato" só para socos (a regra de contato não tem esse filtro), o Eject Button gasta o item mesmo sem reserva, Choice Band: a trava não some se o item for derrubado.
6. **IA:** feito em 15/10: quem entra no lugar de quem cai e depois de pivô (o melhor contra o jogador), a opção de abrir com o melhor do time e a troca por escolha que desconta a pancada na entrada. Feito em 14/10: joga o campo (critério `Tatica`), conta prioridade e Trick Room, presume a troca do jogador (Pursuit) e escolhe a forma do Deoxys. **Falta:** Illusion, e a estimativa de dano não conta o poder Z/Max (aproximação declarada).
7. **Dados:** a descrição do Soul Dew ainda diz a regra antiga; os nomes das formas do Arceus ainda são os de fã ("Arceus(Flame)"); Blank Plate e Legend Plate seguem como estão; o Gastro Acid ainda "gruda" na Multitype (o status entra, mas não faz nada); `Mega Magearna Original` e `Mega Tatsugiri Droopy/Stretchy` ficam de fora (sem forma base).
8. **Decisões:** quais mecânicas de formato o jogo liga (P9). ~~Apagar o que não é oficial (P10)~~ **respondida em 13/10**: removido, ficam `AssaultArmor` e `Eventide` na quarentena.
9. **Erro de regra conhecido:** dano e cura em fração do HP arredondam; o oficial trunca (PENDENCIAS D1).
10. **Rede (H).**

---

## 5. Como se prova "igual ao oficial"

1. **Cada peça nova sai com teste de EditMode** (regra pura) **e de PlayMode** (com a batalha de verdade, em dupla quando couber), no padrão `Leva4DoShowdownTests` / `Leva4DoShowdownPlayModeTests`.
2. **Criar o Pokémon de teste só por `ArnesDoJogador.PokemonDeTeste`** (habilidade fixa): o que é sorteado na criação vira ruído e causou um flake de 1 em 6.
3. **Bateria de 6+ rodadas** antes de dizer "estável" (duas rodadas não pegam um flake de 5%).
4. **Rodar as duas suítes inteiras**: o EditMode prova que compila e que o resto não regrediu; só o PlayMode prova que a batalha se comporta.
5. **Casos de referência contra o Showdown (proposta):** montar uma tabela pequena de cenários (Pokémon, golpe, clima → dano esperado ou efeito) tirada do Showdown **fora do projeto**, e rodá-la como teste. O `COMPARATIVOS_EXTERNOS.md` já fez isso por leitura de fonte; falta transformar em tabela executável. (Não verifiquei se há Node instalado nesta máquina para gerar a tabela do Showdown.)
6. **Auditor de alcançabilidade** (`Pokemon → Auditar alcancabilidade do conteudo`) depois de cada leva de conteúdo: peça pronta que ninguém alcança não conta.

---

## 6. Riscos

| Risco | Mitigação |
|---|---|
| Mexer em `SistemaBatalha` quebra batalha sem teste automatizado de EditMode | PlayMode inteiro depois de cada mudança nele; revisão a cada 2 pares, como combinado |
| Formas por gatilho dependem de espécies que não estão no projeto (Castform, Minior) | Decidir os dados antes de escrever a regra |
| Itens gerados (`...Scroll`) e itens oficiais duplicados no catálogo | Decisão D5 antes de gerar |
| Escopo cresce com "igual ao oficial" | Fixar D1 a D4 e tratar tudo fora deles como novo item de backlog |
| Rede e batalha local divergem | Cada peça nova passa por `AcaoDeRede`/`ParaRede`: ação que não vira dado não acontece |

---

## 7. Decisões desta frente

Revisto em 07/10/2026 e 09/10/2026. D1 a D4 foram adotadas. **As decisões em aberto de toda a documentação, com dono e recomendação, estão em [`../Projeto/DECISOES.md`](../Projeto/DECISOES.md) §2**; as desta frente são P9, P10 e P11. O que o texto original deste plano deixou aberto:

1. ~~Os `...Scroll`~~ **Feito em 07/10:** 81 removidos; ficam `PulseCannonScroll` e `PowerRingScroll`.
2. ~~Megas e formas de fã~~ **Feito em 07/10:** 68 formas "Special" + 154 Megas de fã removidas, com as habilidades de fã presas a elas.
3. **Victory Dance, Shelter e Power Shift** (oficiais do Legends: Arceus, ver §3.A): adaptar para estágios comuns, como o Showdown, ou deixar sem efeito?
4. ~~Castform e Cramorant~~ **Feito em 07/10:** Castform Sol/Chuva/Neve, Cramorant Com Presa/Com Raio e Mimikyu Descoberto criados, sem imagem própria nem modelo 3D.
5. **Bagas de pinça** (Sitrus, Lum, Liechi e afins): elas **existem**, mas como `ItemCura` (uso da mochila: Sitrus, Lum, Oran, Cheri, Chesto…) ou `ItemGenerico` inerte (Liechi, Salac…), não como item segurado. Para o Pokémon comer sozinho em batalha precisam virar `HeldItemData` (troca o tipo do asset; o id e o save continuam) ou ganhar um par segurado. Destravam Cud Chew, Gluttony, Unnerve, Ripen e Teatime (60+ espécies). Qual dos dois caminhos?
6. **A pilha de UI**, para a Leva 6.

---

## 8. Leva 12 (08/10/2026): o que a auditoria completa achou

Menu `Pokemon/Itens/Auditar todo o conteudo sem comportamento` (relatório em `Logs/conteudo_sem_comportamento.txt`). Ferramenta: `ConteudoDoShowdown.Leva12.cs` (menus `Pokemon/Showdown/Leva 12 - 1` e `- 2`). Testes: `Leva12PlayModeTests` (11).

**Só dados (as peças já existiam):** Sacred Sword e Darkest Lariat (ignoram os estágios do alvo), Throat Chop (status novo `GargantaFerida`: 2 turnos sem golpe de som), Glitzy Glow e Baddy Bad (telas), Freezy Frost (Haze), Sparkly Swirl (cura a equipe), Pika Papow e Veevee Volley (poder pela amizade, nunca erram), Eerie Spell (−3 PP), Stone Axe e Ceaseless Edge (armadilha no campo), Chloroblast (perde metade do HP), Psyblade (×1,5 no Campo Elétrico), Collision Course e Electro Drift (×1,33 super efetivo), Hyper Drill e Mighty Cleave (atravessam proteção), Magical Leaf (nunca erra), Grassy Glide (+1 de prioridade no Campo de Grama), Core Enforcer (suprime a habilidade de quem já agiu), Lunar Blessing (regra da Gen 9: cura 1/4 e status de quem usa e do aliado), Bitter Malice (Gen 9: −1 Ataque), Glaive Rush (status novo `Exposto`: golpes contra quem usou nunca erram e causam o dobro até o fim do próximo turno), Fickle Beam (30% de poder dobrado). Itens: Fairy Feather, Punching Glove (só o ×1,1; o "sem contato" so para socos pede filtro que a regra de contato nao tem), Throat Spray, Blunder Policy (dispara em qualquer falha, não só erro de precisão), Zoom Lens, Safety Goggles (só os golpes de pó), Shed Shell, Adrenaline Orb (aproximação: qualquer queda imposta pelo oponente, não só Intimidate), Sticky Barb, Ring Target, Mirror Herb.

**Peças novas pequenas:** `RegraDeFalhaAoUsar` com cinco motivos novos (Poltergeist sem item no alvo, Upper Hand contra golpe sem prioridade, Burn Up/Double Shock sem o tipo, Synchronoise sem tipo em comum, Steel Roller sem terreno); `EfeitoPerderTipo` (Burn Up, Double Shock; com o único tipo, fica sem tipo); `EfeitoRoubarEstagiosPositivos` (Spectral Thief: rouba depois de acertar e antes do dano, como no oficial — `EfeitoDeBatalha.AntesDoDano` e o dano recalculado); `EfeitoRemoverTerreno` (Steel Roller, Ice Spinner); `RegraDoMaiorAtaque` (Photon Geyser); `RegraDeMudarTipo` por terreno + `PoderNoTerreno` (Terrain Pulse); `CondicaoDoTurno.MeuAtributoCaiuNesteTurno` (Lash Out); `RegraDeClimaQueReforca` (Hydro Steam); `EfeitoDobrarRecompensa` (Happy Hour); `EfeitoAplicarUmDosStatus` (Dire Claw e, de quebra, Tri Attack); `EfeitoCopiarSubida` + `GatilhoDeEfeito.escutaOOponente` (Opportunist, Mirror Herb); o evento `AplicouStatusNoOutro` (Poison Puppeteer); `EfeitoDanoDeHabilidade.aura` + `RegraDeQuebrarAura` (Aura Break inverte Dark/Fairy Aura); `EfeitoDeBatalha.PodeAcontecer` (efeito sem o que fazer nem sorteia). Itens: Big Root (`RegraDeCuraPorDreno`), Binding Band e Grip Claw (`RegraDeStatusImposto`: quem impõe o status muda o dano e a duração), Metronome (`RegraDeReforcoPorRepeticao`), Ability Shield (`RegraDeProtegerHabilidade`), Heavy-Duty Boots (`RegraDeImunidadeAArmadilhas`), Utility Umbrella (`RegraDeIgnorarSolEChuva`, lida direto no clima), Room Service (evento `CampoInstalado`), Terrain Extender (duração de terreno).

**Continua de fora:** Pollen Puff curando aliado (o motor não deixa um golpe de dano mirar o aliado; ver §9), Illusion, Commander/Order Up, Booster Energy, Piercing Drill (falta o valor), Z-moves, Max moves e Tera (decisão de formato). Teatime, as bagas, as habilidades de baga e as máscaras da Ogerpon saíram na Leva 13 (§9).

## 9. Leva 13 (09/10/2026): o que faltava comparando com o Showdown

Ferramenta: `ConteudoDoShowdown.Leva13.cs` (menu `Pokemon/Showdown/Leva 13 - bagas, Mega e itens de especie`, uma passada, idempotente). Testes: `Leva13PlayModeTests` (10).

**Itens com a classe errada.** As bagas, as pedras Mega, as placas e os itens de origem existiam como `ItemGenerico` ou `ItemEvolucao`, que a batalha não lê. Foram convertidos no lugar (troca do `m_Script`; o guid, o id e o save não mudam): 69 bagas viraram `Baga` (classe nova, filha de `HeldItemData`, que também é usada da mochila pela mesma cura do `ItemCura`, agora em `CuraDeItem`), e as pedras Mega (as 45 oficiais e as duplicadas), 17 placas, Adamant Crystal, Lustrous Globe e Griseous Core viraram `HeldItemData`. As seis bagas de EV (Pomeg, Kelpsy, Qualot, Hondew, Grepa, Tamato) ficam `ItemAtributo`: são de mochila.

**Bagas (runtime).** `MesaDeRegras.ComerBaga` come uma baga fora do gatilho dela (roda os efeitos do primeiro gatilho que a gasta, sem as condições de HP; o que ela faz contra quem bateu e a fala de resistência não acontecem, como no Showdown). Em cima disso:
- Bug Bite e Pluck: `EfeitoComerBagaDoAlvo`, no lugar do "remover item".
- Stuff Cheeks: `EfeitoComerBagaPropria` e o motivo de falha `SemBaga`.
- Teatime: `EfeitoTodosComemBaga`, que ignora o Unnerve, como no Showdown.
- Cud Chew: `EfeitoRuminar`, que come de novo no fim do turno seguinte, uma vez.
- Gluttony: `RegraDeGula` e a condição `limiarDeBaga`, que leva o aperto de 1/4 para 1/2.
- Ripen: `RegraDeAmadurecer`. Os efeitos da baga rodam duas vezes.
- Unnerve e As One: `RegraDeNervosismo`. As bagas do outro lado não valem, nem como regra nem como gatilho.
- Custap: `RegraDeAgirPrimeiroNaFaixa.consomeOItem`. A ordem do turno gasta a baga.
- Belch passou a exigir **baga comida**, e não qualquer item gasto.

**Bagas (dados).**
- Oran (10 HP) e Sitrus (1/4) na metade do HP.
- Figy, Wiki, Mago, Aguav e Iapapa (1/3) com 1/4 do HP. A confusão por natureza não entra: o projeto não tem natureza.
- Enigma: golpe super efetivo.
- Cheri, Chesto, Pecha, Rawst, Aspear, Persim e Lum: na hora em que o status gruda.
- Liechi, Ganlon, Salac, Petaya, Apicot, Lansat e Starf, mais Custap, Kee, Maranga, Jaboca e Rowap.
- Toda baga de batalha tem Fling 10 e o tipo e o poder do Natural Gift da Gen 6+.
- A cura da mochila foi corrigida: a Sitrus curava 99999 e a Cheri curava qualquer status. A `Baga` ganhou `fracaoDoHpMaximo` para isso.

**Mega Evolução.**
- As 45 pedras oficiais cujas formas Mega existem no projeto ganharam `formaVoluntaria` e não podem ser derrubadas.
- Cada forma Mega passou a ser `formaDe` da base e dura até o fim da batalha.
- **Dois erros de regra corrigidos.** Antes, qualquer Pokémon que segurasse uma pedra virava aquela Mega (um Charmander com Venusaurite virava Mega Venusaur). Agora a forma precisa ser da espécie de quem segura (`SistemaBatalha.FormaDaEspecie`). E a Mega passa a ser **uma por lado por batalha**, como no Showdown.

**Outros itens.**
- Placas: ×1,2 no tipo, Fling 90.
- Adamant Crystal, Lustrous Globe e Griseous Core: ×1,2 nos dois tipos, só na espécie certa. O Core leva o Giratina à forma Origin ao entrar.
- Máscaras da Ogerpon: ×1,2, só na Ogerpon.

**Continua de fora, e por quê:**
- ~~Pedras sem forma no projeto e pedras duplicadas~~: resolvido na §10 (todas as Megas oficiais, as de fã apagadas).
- **Pedem peça nova:** Micle (precisão ×1,2 no próximo golpe), Leppa (PP) e Booster Energy (Protosynthesis e Quark Drive sem clima nem terreno).
- ~~Judgment e Multitype~~: feitos na §11. **Faltam** as formas da Ogerpon (para quando as mecanicas de formato entrarem).
- **Ditto:** Metal Powder e Quick Powder continuam `ItemEvolucao`.
- **Illusion:** a UI precisa mostrar outro Pokémon.
- **Commander:** dupla, Tatsugiri dentro do Dondozo.

### O que pede um golpe de dano mirando o aliado, Z-moves, Max moves e Tera

| Mecânica | O que muda no projeto | Tamanho |
|---|---|---|
| ~~**Dano no aliado**~~ | **Feito em 09/10** (§10): golpe de um alvo mira o aliado em dupla, Pollen Puff cura o aliado, UI e IA. | — |
| **Z-moves** | Os cristais Z como itens segurados (tipo + golpe exigido); a tabela de poder Z por poder base; os ~35 golpes Z (18 por tipo + os exclusivos); os efeitos Z dos golpes de status; o botão na UI; a trava "um por lado por batalha" (a da Mega serve). | G |
| **Max moves (Dynamax)** | Estado de 3 turnos com HP multiplicado; os golpes viram Max (18 + Max Guard) com efeitos de campo (clima, terreno, atributo do lado); imunidades (flinch, golpes de peso, Encore…); some na troca; formas Gigantamax; UI. | G |
| **Tera** | Tipo Tera por Pokémon (no asset e no save); a troca de tipo defensivo; as regras de STAB (os tipos originais mantêm STAB, Tera igual a um original dá ×2, piso de 60 de poder); Tera Blast; o tipo Stellar; a trava uma por batalha; a UI. | G |

As três últimas são a decisão D2 (§7): o Showdown só as usa em formatos específicos (Z e Dynamax ficaram na National Dex). Nenhuma tem código hoje.

## 10. Megas como no oficial e golpe no aliado (09/10/2026)

### Megas: só as oficiais, com os dados do Showdown

Fonte: `data/pokedex.ts` e `data/items.ts` do Showdown (commit `3065d24d`), com as Megas de X/Y, ORAS, Legends Z-A e da DLC Mega Dimension. Ferramenta: `ConteudoDoShowdown.Megas.cs`, em duas passadas (`Pokemon/Showdown/Megas - 1 renomear, criar e apagar` e `- 2 preencher pelo Showdown`). As duas são idempotentes.

- **94 formas Mega** no projeto, todas com tipos, atributos, habilidade, altura e peso do Showdown. Os dados que vieram do gerador estavam errados nas Megas de Z-A (status de uma previsão anterior ao lançamento), na Mega Beedrill e na Mega Zygarde. Cada forma é `formaDe` da base exata e dura até o fim da batalha. A habilidade espelha os slots da base, e a oculta vale onde existe (Mega Heatran, Mega Tatsugiri).
- **19 formas renomeadas** do nome numérico para o padrão (`p0154_Meganium_10198` → `p0154_MeganiumMega`). O guid e o id não mudam.
- **29 formas criadas:** Raichu X/Y, Clefable, Victreebel, Starmie, Chimecho, Absol Z, Staraptor, Garchomp Z, Lucario Z, Scolipede, Scrafty, Eelektross, Golurk, Pyroar, Floette, Meowstic (macho e fêmea), Malamar, Barbaracle, Dragalge, Crabominable, Drampa, Magearna, Falinks, Scovillain, Glimmora, Tatsugiri e Baxcalibur. O sprite é o da base até a arte chegar (`PENDENCIAS` A11). O id é o do Showdown (`raichumegax`).
- **92 pedras oficiais**, todas item segurado que não pode ser derrubado. 47 delas eram `ItemEvolucao` e foram convertidas no lugar; nada além do catálogo as referenciava.
- **Apagadas, por serem de fã:**
  - 9 formas: Mewtwo Sombrio X/Y, Mewtwo Steel e Steel Y, Zacian Mega, Zamazenta Mega, Mega Muk de Alola, Mega Zygarde 10% e Mega Zygarde 50%;
  - 39 pedras duplicadas `Mega…ite`;
  - a "evolução" do Zygarde para a Mega Zygarde 50%.

  Todas entraram em `conteudo_removido.txt`.
- **Habilidade nova:** Sharpness (Mega Absol Z): golpes cortantes ×1,5.
- **Peças de runtime:**
  - A Mega exige a **base exata**: `EspecieData.FormaDe` da forma tem de ser a espécie de quem segura. A Zygardite só vale na Zygarde Complete, e a Floettite só na Floette de Flor Eterna.
  - `HeldItemData.outrasFormasVoluntarias` serve às pedras de várias formas: a Meowsticite vale para macho e fêmea, e a Tatsugirinite para cada forma do Tatsugiri.
  - `EspecieData.formaPorGolpe` e `golpeDaForma` dão a Mega Rayquaza por Dragon Ascent, sem pedra.
  - `SistemaBatalha.FormaVoluntariaDe` responde tudo isso.
- **Ficam de fora**, porque a forma base não existe no projeto: Mega Magearna Original e Mega Tatsugiri Droopy e Stretchy.
- **Itens que parecem pedra mas não são:** a BlackAugurite (evolução do Kleavor, do Legends Arceus) e a Meteorite (formas do Deoxys).

### Golpe de dano mirando o aliado

No oficial, em dupla, todo golpe de um alvo ("normal" no Showdown) pode mirar o aliado. Agora o projeto também:

- **Batalha:** `ResolverAlvosDoGolpe` aceita o aliado como alvo de um golpe `UmInimigo` quando quem escolheu o pediu. Chamariz (Follow Me) não desvia esse golpe. Se o aliado caiu antes, o golpe volta para o inimigo.
- **Golpes que nunca miram o aliado:** `GolpeData.naoMiraAliado`. No Showdown são os de alvo "só inimigo" (Doodle, Me First), os de alvo aleatório (Outrage, Petal Dance, Thrash, Raging Fury, Uproar, Struggle) e os de alvo decidido pelo próprio golpe (Counter, Mirror Coat, Metal Burst, Comeuppance).
- **Pollen Puff:** `EfeitoCurarAliadoEmVezDeDano`. No aliado, cura metade do HP em vez de causar dano e falha com Heal Block; no inimigo, é dano comum.
- **UI:** em dupla, o seletor de alvo de um golpe assim lista também o aliado, marcado "(aliado)".
- **IA:** usa o Pollen Puff no parceiro ferido (`AvaliadorDeApoioAoAliado`). Nunca bate no aliado de propósito.
- **Rede:** o alvo já viajava por posto; nada mudou.
- Ferramenta: `ConteudoDoShowdown.Aliado.cs` (menu `Pokemon/Showdown/Golpe no aliado`).

Testes: `Leva13PlayModeTests` (Pollen Puff no aliado e no inimigo, golpe de dano no aliado, golpe que não mira o aliado, Mega pela base exata, pedra de várias formas e Rayquaza).

### Judgment e Multitype, Illusion, Commander: onde estão

| | Como é no oficial | Como está no projeto | O que falta |
|---|---|---|---|
| **Multitype + Judgment** (**feito em 09/10, §11; a coluna "Como está" abaixo descreve o estado de antes**) | O Arceus segurando uma placa vira a forma daquele tipo; o Judgment tem o tipo da placa; com Multitype a placa não sai e a habilidade não é trocada. | As 18 formas do Arceus **existem**, com os tipos certos, mas sem `formaDe` (não estão ligadas ao Arceus). Multitype está vazia. O Judgment usa o tipo de quem o usa: se o Arceus estivesse na forma certa, já sairia com o tipo certo. | É pequeno, com as peças que já existem: ligar as formas ao Arceus; cada placa leva o Arceus à forma dela ao entrar em campo (o mesmo desenho da Red Orb); Multitype protege a placa e a habilidade. Pendente: se a forma vale também fora de batalha (no oficial, vale). |
| **Illusion** (Zoroark) | Entra disfarçado do último Pokémon vivo da equipe: o nome e o sprite são do outro. Cai ao levar dano de golpe. | Habilidade vazia. Zorua e Zoroark existem. | Médio, e quase tudo de **apresentação**: a batalha guarda "como este Pokémon aparece" separado de quem ele é. A UI, a narração e a IA do outro lado passam a ler a aparência (a IA não pode saber do disfarce), e o disfarce cai no primeiro dano de golpe. Depende de a UI de batalha estar resolvida (`PENDENCIAS`, UI urgente). |
| **Commander + Order Up** (Tatsugiri + Dondozo) | Em dupla, o Tatsugiri entra na boca do Dondozo aliado: sai de campo sem sair (não age, não pode ser atingido), e o Dondozo ganha +2 em todos os atributos. O Order Up do Dondozo sobe um atributo conforme a forma do Tatsugiri. | Habilidade vazia; Order Up vazio. Só existe a forma Curly do Tatsugiri. | Grande. Precisa de um estado novo de posto ("ocupado, mas fora do alcance e sem agir") que a fila, os alvos, a troca e o fim da batalha respeitem. Depois, o +2 do Dondozo (peça existente) e o Order Up lendo a forma do Tatsugiri que está dentro dele. Faltam também as formas Droopy e Stretchy. |

## 11. Judgment, Multitype, formas de fã e como o Showdown faz as outras mecânicas (09/10/2026)

### Formas base de fã apagadas

Mewtwo Sombrio (`p2501_MewtwoBlack`), Mewtwo Steel (`p0150_MewtwoSteel`), Magearna 10143 (a "Magearna" de status que não existem no oficial), Beedrill e Pidgeot "Kanto". Junto saiu a evolução da Magearna para a 10143. Todas estão em `conteudo_removido.txt`, e as Megas deles já tinham saído na §10. Ferramenta: `Pokemon/Showdown/Megas - 3 apagar formas base de fa`.

### Multitype e Judgment (feitos)

Como o Showdown faz (`abilities.ts`, `items.ts`, `moves.ts`, `pokemon.ts`):
- O Arceus com uma placa é a **forma** dela (`forcedForme` da placa). A placa **não sai do Arceus nem vai para ele** (`onTakeItem` das placas olha a espécie, não a habilidade): Knock Off, Thief, Trick, Switcheroo e Fling falham.
- O **Judgment** muda para o tipo da placa (`onModifyType`); sem placa é Normal. O tipo de quem usa não entra na conta.
- A **Multitype** só tem flags: `failroleplay`, `noreceiver`, `noentrain`, `notrace`, `failskillswap`, `cantsuppress`. Ninguém a copia, troca, impõe nem cala.
- O primeiro tipo do Arceus (e do Silvally) **não muda** (`setType` recusa): Soak, Conversion, Conversion 2, Camouflage, Reflect Type, Protean, Burn Up…

No projeto:
- **Forma que segue o item:** `EspecieData.formaSegueOItem` no Arceus base. As 17 formas passaram a ter `Forma de` = Arceus (a Pokédex mostra uma entrada só). `Pokemon.EquiparHeldItem` e `RemoverHeldItem` sincronizam a espécie, como uma evolução permanente que o save grava. Sem placa, volta ao Normal. Isso vale também fora de batalha, como nos jogos.
- **Placas:** `HeldItemData.tipoDePlaca` e `formaForcada`, preenchidas nas 17 placas. `HeldItemData.PodeMudarDeMao(dono, outro)` trava a placa em Knock Off, Trick e afins e no Fling.
- **Judgment:** a regra nova `RegraDeTipoDaPlaca` substituiu `RegraDeTipoDoUsuario`.
- **Tipo travado:** `EspecieData.tipoNaoMuda` no Arceus e no Silvally. `Pokemon.AssumirTipo` e `FicarSemTipo` recusam, e os seis efeitos de mudar tipo respondem "Mas falhou!". Trick-or-Treat e Forest's Curse, que **somam** um tipo, continuam funcionando.
- **Restrições de habilidade:** `HabilidadeData.restricoes` (`RestricaoDeHabilidade`). As flags de **32 habilidades** foram copiadas do Showdown (Multitype, Disguise, Schooling, Zen Mode, Stance Change, Battle Bond, Commander, As One, Trace, Illusion…). Trace, Role Play, Receiver/Power of Alchemy, Doodle, Skill Swap/Wandering Spirit, Entrainment, Mummy, Worry Seed, Simple Beam e Gastro Acid/Neutralizing Gas passaram a respeitá-las. Transform copia a Multitype mesmo assim. Embody Aspect (Ogerpon) fica para quando as outras mecânicas entrarem.
- Ferramenta: `ConteudoDoShowdown.Leva14.cs`. Testes: `Leva14PlayModeTests` (12).
- **Limites:** o Gastro Acid ainda "gruda" na Multitype (o status entra), mas não faz nada. A Blank Plate e a Legend Plate seguem como estão: o Showdown não as trata como placa. Os nomes das formas do Arceus ainda são os de fã ("Arceus(Flame)", "Arceus(Splash)").

### Como o Showdown faz as outras mecânicas (`sim/battle-actions.ts`, `sim/pokemon.ts`, `data/conditions.ts`)

| | Como funciona no Showdown | O que o projeto teria de ter |
|---|---|---|
| **Mega** (já feita) | `canMegaEvo`: a pedra mapeia a espécie **exata** para a Mega; a Rayquaza vai por golpe. `runMegaEvo` troca a forma e zera `canMegaEvo` em toda a equipe (uma por lado). Mega, Z e Tera se excluem entre si. | Feito na §10. |
| **Z-moves** | O cristal (`item.zMove`) vale para um tipo; o lado só usa **um** Z por batalha (`side.zMoveUsed`), o golpe-base precisa ter PP. O golpe de dano vira o Z do tipo, com o poder da tabela do golpe-base (`zMove.basePower`) e a categoria dele; o de status mantém o nome e dá um bônus (`zMove.boost`: +1 em algo, ou `effect`: curar tudo, curar o próximo que entrar, limpar quedas, chamariz, crítico). Cristais exclusivos (`zMoveFrom` + `itemUser`) trocam um golpe específico por um Z próprio (Pikanium Z: Catastropika). Atravessa Protect com 25% do dano. | Itens Z (tipo ou golpe+espécie), a tabela de poder Z por golpe (um campo por golpe, ~900), os 18 Z de tipo e os ~17 exclusivos, os bônus dos golpes de status, o botão na tela e a trava de um por lado (a da Mega serve). **Não pede estado em campo**: o Z é só uma escolha de golpe naquele turno. |
| **Dynamax / Max** | Estado `dynamax` de 3 turnos (`onResidual` conta). Ao entrar: HP máximo e atual × (1,5 + 0,05 × nível Dynamax; 2,0 no nível 10); tira Substitute, Minimize e Torment. Imune a hesitar; **não é arrastado** (Roar, Whirlwind, Dragon Tail); Behemoth Bash/Blade e Dynamax Cannon causam o dobro. Ao acabar, o HP volta proporcional. Os golpes viram **Max moves** (`MAX_MOVES` por tipo; status vira Max Guard), com poder da tabela do golpe-base (`maxMove.basePower`) e efeitos de campo/atributo; G-Max por espécie (`canGigantamax`). | O estado de 3 turnos com HP multiplicado (peça nova), as imunidades (peças existentes: bloqueio de status e de troca forçada), os 18 Max moves + Max Guard com seus efeitos (clima, terreno, atributos do lado: peças existentes), a tabela de poder, os G-Max (opcional) e a tela. Depende do projeto ter golpes que mudam de nome/efeito por estado (o Z precisa do mesmo). |
| **Tera** | `teraType` por Pokémon (no set). `terastallize`: **um por lado**, sem tipo base novo — `getTypes()` devolve só o tipo Tera (o Stellar mantém os tipos originais). **STAB:** 2× quando o golpe é do tipo Tera **e** o Pokémon já tinha esse tipo; senão 1,5× se bate com o Tera ou com um tipo original. **Piso de poder 60** para golpes do tipo Tera com poder menor (menos prioridade, multi-hit e poder variável). Terastalizado não tem o tipo base trocado nem somado. Stellar: bônus único por tipo (1,2× sem STAB, 2× com). Ogerpon e Terapagos trocam de forma ao terastalizar. Tera Blast vira Físico se Atq > Atq. Esp. | Tipo Tera por Pokémon (asset de espécie? indivíduo? save), o estado "terastalizado" **que não some ao trocar** (o `tipoAssumido` atual some ao sair de campo, então não serve), a regra de STAB e o piso de 60 em `CalculoDano`, a trava de um por lado, Tera Blast, Stellar (opcional) e a tela. O mais barato dos três: quase tudo é uma camada no tipo e no cálculo. |

**Recomendação de ordem:** Tera → Z → Dynamax. Mas é a decisão D2 (§7): o jogo escolhe **uma** mecânica de formato; o Showdown só as junta em formatos (Z e Dynamax só na National Dex). Se o jogo escolher uma só, as outras duas não precisam existir.

## 12. Z-moves, Dynamax e Tera (09/10/2026)

Implementados como o Showdown os faz (`sim/battle-actions.ts`, `data/conditions.ts`, `sim/dex-moves.ts`). Os dados vêm de `data/moves.ts` e `data/items.ts`.

**Desligados por padrão.** Cada batalha liga o que quiser em `SistemaBatalha.MecanicasPermitidas`; o padrão é `MecanicasPadrao` = só a Mega, como antes. Quais mecânicas o jogo usa é a decisão D2 (§7), e fica com o dono do projeto. Com elas ligadas, a IA de treinador passaria a usá-las, o que muda as lutas atuais.

**Regras comuns:**
- Cada mecânica vale **uma vez por lado por batalha**.
- Um Pokémon usa no máximo uma. Quem segura pedra Mega ou cristal Z não Dinamaxiza nem Terastaliza, como no Showdown.
- Pokémon **selvagem** não usa Z, Dynamax nem Tera.
- Na tela de batalha, o botão de forma percorre nenhuma → Mega → Z → Dynamax → Tera → formas alternáveis, mostrando só as que valem. A ação (`AcaoBatalha.UsarZ/Dinamaxizar/Terastalizar`) também viaja pela rede.
- **IA:** usa na primeira oportunidade, como já fazia com a Mega. A ordem é Mega, senão Z no golpe que tem Z, senão Tera, senão Dynamax. A estimativa de dano da IA não conta o poder Z/Max (aproximação declarada).

### Z-moves
- **Cristal Z** (`HeldItemData`): os 35 "...ZHeld" viraram item segurado. Os 18 de tipo transformam os golpes daquele tipo; os 17 exclusivos transformam um golpe base, e só nas espécies certas (Pikanium Z: Volt Tackle → Catastropika, só no Pikachu). Os "...ZBag" ficam como item de mochila.
- **Golpe de dano** vira o Z do tipo, com o poder da tabela do golpe base (`GolpeData.poderZ`: o do `moves.ts`, senão a fórmula do `dex-moves.ts`) e a categoria dele. O Z exclusivo tem poder próprio. Golpe Z nunca erra.
- **Golpe de status** sai ele mesmo, com o poder Z antes: os estágios (`bonusZ`, por exemplo Z-Splash +3 Ataque) ou o efeito (`efeitoZ`): curar tudo, curar quem entrar (Z-Memento, Z-Parting Shot), limpar quedas, chamariz ou crítico (por status), e o Z-Curse.
- O golpe base paga o PP e é o que fica na memória (Encore e Disable miram ele).
- Z e Max **atravessam Protect com 1/4 do dano**.
- Efeitos dos Z exclusivos: 10.000.000 Volt Thunderbolt (crítico +2), Stoked Sparksurfer (paralisa), Extreme Evoboost (+2 em tudo), Clangorous Soulblaze (+1 em tudo, área), Genesis Supernova (Campo Psíquico), Splintered Stormshards (tira o terreno), Guardian of Alola (3/4 do HP), Searing Sunraze Smash / Menacing Moonraze Maelstrom / Light That Burns the Sky (ignoram habilidade; o último usa o maior ataque).

### Dynamax
- **3 turnos**, contando o turno em que começa. HP máximo e atual dobram (nível Dynamax 10 do Showdown). Ao acabar, o HP volta na proporção, arredondando para cima.
- Acaba ao sair de campo e no fim da batalha.
- **Golpes:** todo golpe vira o Max do tipo, com o poder da tabela do golpe base (`poderMax`), e status vira Max Guard.
- **Efeitos dos Max:** clima (Max Flare, Geyser, Hailstorm, Rockfall), terreno (Lightning, Overgrowth, Mindstorm, Starfall), sobe o lado de quem usa (Knuckle, Steelspike, Ooze, Quake, Airstream) ou baixa o lado do alvo (Strike, Wyrmwind, Phantasm, Flutterby, Darkness), com a peça nova `EfeitoAlterarAtributoDoLado`. O Max Guard segura até Z e Max.
- **Dinamaxizado:**
  - não hesita e não é arrastado (Roar, Whirlwind, Dragon Tail, Red Card);
  - Choice Band, Specs e Scarf não valem;
  - Low Kick, Grass Knot, Heat Crash, Heavy Slam, Encore, Torment, Instruct e Entrainment falham (motivo novo `AlvoDinamaxizado`);
  - Behemoth Blade, Behemoth Bash e Dynamax Cannon dobram contra ele;
  - Super Fang, Nature's Madness, Ruination, Endeavor e Guardian of Alola usam o HP sem Dynamax.
- Zacian, Zamazenta e Eternatus não Dinamaxizam.
- **Falta:** Gigantamax (o projeto não tem as formas nem os G-Max moves), Destiny Bond e Pain Split com HP sem Dynamax.

### Tera
- **Tipo Tera** por indivíduo (`Pokemon.tipoTera`, padrão: o primeiro tipo da espécie base; `teraEstelar` para Stellar). Vai no **save formato 5**, e o catálogo passou a indexar os tipos.
- **Terastalizado:** vale até o **fim da batalha**, mesmo trocando. O Pokémon tem só o tipo Tera (Stellar mantém os tipos), e não ganha nem troca tipo (Soak, Trick-or-Treat, Protean…).
- **STAB:**
  - 2× quando o tipo Tera já era um tipo original;
  - 1,5× para o tipo Tera e para os tipos originais;
  - piso de **poder 60** para golpe do tipo Tera, menos prioridade, multi-hit e poder variável.
- **Stellar:** reforço único por tipo de golpe (2× com STAB, 1,2× sem).
- **Tera Blast:** sai do tipo Tera e vira Físico quando o Ataque é maior. Stellar: poder 100, 2× em quem está Terastalizado, e −1 Ataque e Ataque Especial.
- **Falta:** as formas Tera de Terapagos e Ogerpon (Tera Starstorm), que ficam para quando a Ogerpon entrar, e a escolha do tipo Tera em jogo (por enquanto só por código: `Pokemon.DefinirTipoTera`).

**Peças novas:**
- **Dados:** `MecanicasDeFormato.cs` (`EfeitoZ`, `BonusDeAtributo`, `MecanicaDeFormato`); campos Z/Max em `GolpeData` e `GolpeData.Potencializar` (cópia de batalha com o poder e a categoria do golpe base); campos de cristal Z em `HeldItemData`; `EspecieData.naoDinamaxiza`.
- **Pokémon:** `Pokemon.TipoTera`, `Terastalizar`, `ComecarDinamax`, `EncerrarDinamax` e `HpSemDinamax`.
- **Batalha:** `SistemaBatalha.Mecanicas.cs`; `EfeitoProteger.seguraZEMax`.
- **Condições:** `semDinamax`, `terastalizado`, `oponenteDinamaxizado`, `oponenteTerastalizado`.
- **Tera Blast:** `RegraDeTeraBlast` e `EfeitoTeraBlastEstelar`.
- **Ferramenta:** `ConteudoDoShowdown.Leva15.cs`.
- **Testes:** `Leva15PlayModeTests` (10).

## 13. Quem usa cada mecânica de formato (09/10/2026)

"Desligadas por padrão" não quer dizer que ninguém possa usar: é só a **lista da cena** (`SistemaBatalha.MecanicasPermitidas`), que vale para o **jogador**. Cada treinador NPC tem a **lista dele**.

- **Jogador:** `SistemaBatalha.MecanicasPermitidas` (padrão: só a Mega). Cada mecânica pode exigir um **item-chave na mochila**, configurado em quatro listas (`itensQueHabilitamMega`, `itensQueHabilitamZ`, `itensQueHabilitamDinamax`, `itensQueHabilitamTera`). Lista vazia = não exige item. Preenchida = o jogador precisa ter algum dos itens: Mega Ring, Mega Bracelet ou Key Stone; Z-Ring ou Z Power Ring; Dynamax Band; Tera Orb (os seis já existem como item).
- **Treinador NPC:** `NpcTreinador.mecanicasQueUsa`, uma lista de marcar (Mega, Z, Dynamax, Tera). Padrão: só a Mega, como sempre foi. Marque só o que ele deve usar.
- **O NPC não precisa de inventário nem de item-chave.** O que cada mecânica pede dele, por Pokémon, é o que o oficial pede:
  - **Mega:** a pedra no Pokémon dele (sem pedra, não evolui).
  - **Z:** o cristal Z no Pokémon dele.
  - **Dynamax e Tera:** nada. Não exigem item no Showdown nem no código; a Dynamax Band e a Tera Orb são item-chave **do jogador**.
- **Pokémon selvagem:** só a Mega, como antes.
- **A IA** usa a mecânica na primeira oportunidade, se o NPC a tiver na lista. Uma por vez: Mega, senão Z, senão Tera, senão Dynamax.
- Cada lado usa cada mecânica **uma vez por batalha**, e um Pokémon usa no máximo uma.

Testes: `Leva15PlayModeTests` (NPC que só usa o que a config dele permite; item-chave só vale para o jogador).

**Falta ligar na cena:** as quatro listas de itens-chave estão vazias (nenhuma mecânica exige item hoje) e o `mecanicasPermitidas` da cena continua só com a Mega. Isso é decisão de jogo (D2).

## 14. Gigantamax e Ogerpon (10/10/2026)

**Gigantamax.**
- Peças: `Pokemon.FatorGigantamax` (salvo; save v6), `EspecieData.formaGigantamax` e `golpeGMax`.
- Ao dinamaxizar com o fator, a espécie assume a forma G-Max (`MudarFormaTemporaria`), e ela volta quando o Dynamax acaba (`EncerrarDinamax`).
- Na forma G-Max, golpe de dano do **mesmo tipo** do golpe G-Max vira o G-Max (`SistemaBatalha.GolpeGMaxDe`). O poder é o da tabela Max do golpe base, ou o próprio do G-Max quando ele tem (Drum Solo, Fireball e Hydrosnipe: 160).
- Dados: 34 formas pela ferramenta de formas; 33 golpes G-Max pela Leva 20.
- Efeitos dos G-Max: copiados do golpe oficial de mesmo efeito (Gravitas do Gravity, Resonance do Aurora Veil, Stonesurge do Stealth Rock, Wind Rage do Defog, Terror do Mean Look, Meltdown do Torment, Snooze do Yawn, Sweetness do Aromatherapy, Depletion do Spite, Centiferno do Fire Spin, Sandblast do Sand Tomb, Cuddle do Attract), ou montados com as peças (Volt Crash paralisa, Malodor envenena, Smite e Gold Rush confundem, Foam Burst −2 Velocidade, Tartness −1 evasão, Drum Solo/Fireball/Hydrosnipe ignoram habilidade, One Blow/Rapid Flow ignoram proteção).
- **Os 10 que pediam peça nova (Leva 21):**
  - Wildfire, Cannonade, Vine Lash e Volcalith: campo no lado inimigo por 4 turnos; no fim de cada turno, 1/6 do HP de quem não é do tipo.
  - Steelsurge: armadilha de entrada de Aço (`AcoAfiado`, a Stealth Rock com Aço).
  - Befuddle e Stun Shock: `EfeitoStatusAleatorio`.
  - Chi Strike: status volátil `ChiStrike`, +1 de crítico por uso, até +3, em cada um do lado.
  - Replenish: 50% de cada um do lado recuperar a baga consumida (`EfeitoRecuperarItemConsumido.soBaga`).
  - Finale: 1/6 do HP de cada um do lado.
  - Peças novas: `EfeitoEmTodoOLado` e `EfeitoStatusAleatorio`.
- **Bug antigo corrigido no caminho (10/10):** a leitura do campo (`SistemaBatalha.Juntar`) tratava um efeito de "lado inimigo" como se estivesse no lado de quem o instalou. Como a instalação já o guarda no lado da vítima, Espinhos, Pedras Furtivas, Rede Pegajosa e afins pegavam **quem os instalou**. Agora a lista de um lado vale para quem está nele, e a do outro lado só para o que é do campo inteiro.
- Também em 10/10: instalar algo que já está no campo só diz "Mas não funcionou!" quando o golpe é de status. Num golpe de dano (G-Max, Stone Axe), a instalação é secundária e o oficial fica calado.
- **Máscaras e cristais (corrigido em 11/10 pelo importador único):** o bônus de dano das máscaras da Ogerpon e do Adamant Crystal / Lustrous Globe exigia a espécie **base**. Como o item força a forma, o bônus nunca valia. Agora a lista inclui as formas, e `ImportadorDoShowdownTests` impede a volta do bug.
- Testes:
  - `OgerponEGigantamaxTests`: o Charizard gigantamaxiza, usa G-Max Wildfire e volta.
  - `GMaxEfeitosTests`: as Chamas ferem o Snorlax no fim do turno numa batalha de verdade; o alcance de lado inimigo; Chi Strike até +3; Finale; Befuddle; Replenish só com baga; Steelsurge.

**Ogerpon.**
- As máscaras forçam a forma, como as placas do Arceus (`formaForcada`, `formaSegueOItem`), e não podem ser derrubadas.
- Cada forma tem o **tipo Tera obrigatório** (`EspecieData.tipoTeraFixo`, que o `Pokemon.TipoTera` respeita) e a forma Tera (`formaAoTerastalizar`).
- Wellspring, Hearthflame e Cornerstone dão ×4915/4096 no poder base. O Ivy Cudgel fica do tipo da máscara (`RegraDeTipoDaPlaca`, como o Judgment).
- As 4 **Embody Aspect** dão +1 no atributo da máscara pelo **evento novo `Terastalizou`** e ao entrar em campo já terastalizada.

**Maushold.** A Família de Três entrou como forma da de Quatro. O id da de Quatro já ia nos saves, por isso ela segue como a entrada da Pokédex. O Tandemaus evolui com peso 99 para a de Três e 1 para a de Quatro (o 1% do oficial).

## 15. O que a camada B de conformidade achou (11/10/2026)

A camada B ([`../Planos/TESTE_DE_CONFORMIDADE.md`](../Planos/TESTE_DE_CONFORMIDADE.md) §7) compara o fluxo de batalhas inteiras com o simulador do Showdown, turno a turno. Ela começou em 73,5% e foi a **100%** em 139 cenários. Do que errava, a maior parte era do próprio teste; o resto são estes achados, já corrigidos:

**Arredondamento** (a diferença é de 1 a 2 de HP, mas aparece em quase todo golpe):
- **Toxic:** o oficial trunca 1/16 e multiplica pelo degrau (235 HP: 14, 28, 42). O projeto multiplicava a fração e truncava depois (14, 29, 44).
- **Recuo, dreno e cura de golpe:** o oficial usa `Math.round` (2,5 vira 3). O `Mathf.RoundToInt` da Unity arredonda a metade para o par (2,5 vira 2). Agora `CalculoDano.ArredondarMetadeParaCima`.
- **Poder com fração** (Knock Off ×1,5): o oficial arredonda a metade para **baixo** (65 → 97). Era 98.
- **Aqua Ring e Ingrain:** curam truncando (14), não arredondando como Recover (15). `EfeitoCuraPropria.truncar`.

**Onde o multiplicador entra** (o Showdown tem três ganchos diferentes, e o projeto usava um só, o dano final):
- **Poder base** (`onBasePower`), em 69 habilidades e itens: Muscle Band, Wise Glasses, Iron Fist, Tough Claws, Strong Jaw, Sheer Force, Sand Force, as placas, os incensos, as máscaras, e assim por diante. Passaram para `RegraDePoderBase`.
- **Atributo de ataque** (`onModifyAtk/SpA`), em 10 habilidades: Blaze, Torrent, Overgrow, Swarm, Flash Fire, Steelworker, Transistor, Dragon's Maw, Stakeout e Rocky Payload. Usam `noAtaqueDeQuemBate` com o lado "quando eu uso" (novo `ModificarAtaqueProprio`).
- **Gemas:** eram ×1,5 no dano. Do Gen 6 em diante o oficial dá **×1,3 no poder base**, e é esse o valor agora. *Mudança visível para o jogador.*

**Fórmulas:**
- **Gyro Ball:** `floor(25 × velocidade do alvo / a sua) + 1`, com teto de 150.
- **Electro Ball:** faixas da razão inteira entre as velocidades (40, 60, 80, 120, 150). Usava a mesma fórmula do Gyro Ball ao contrário.

**Regras e dados:**
- **Rocky Helmet, Rough Skin e Iron Barbs** agora também devolvem o troco quando o golpe derruba o dono. Só o que é de contato; bagas e Weakness Policy continuam sem reagir a quem caiu.
- **Inner Focus, Oblivious, Own Tempo e Scrappy** barram a queda de Ataque da Intimidate (e só dela). `EfeitoAlterarAtributo.daIntimidacao`.
- **Poison Heal:** o veneno cura 1/8 **em vez de** ferir. Antes ferindo e curando, com saldo errado.
- **Espinhos Tóxicos** com duas camadas aplicam o veneno grave. Antes sempre o comum.
- **Venoshock e Barb Barrage** dobram contra veneno grave também.
- **Rest** dorme 2 turnos inteiros (a contagem era 2 e o Pokémon acordava depois de um).
- **Tempestade de areia:** o dano por turno **não estava configurado em lugar nenhum**. Nenhum Pokémon se machucava na areia. Agora 1/16 do HP, menos Pedra, Terra e Metal (no `Sistemas.prefab`).

Tudo isso está na **Leva 22** (`ConteudoDoShowdown.Leva22.cs`), idempotente.

**Fica registrado, sem correção:**
- **Struggle:** achado aqui e **feito em 11/10** (Leva 23; ver `PENDENCIAS_FECHADAS.md`, D15). Cobre também o Assault Vest em quem só tem golpes de status, que antes pulava a ação.
- **Habilidades que também trocam o gancho** e ainda não têm cenário: Hustle (atributo de ataque), Solar Power e Guts passaram, mas a lista de ganchos de todas as habilidades merece uma varredura, como a do poder base.
- **Sono:** o projeto cura no fim do turno e o Showdown no início do seguinte. O Pokémon age no mesmo turno nos dois; o teste aceita essa diferença só de momento.

### 15.1 Sobras da fase 2 que saíram junto

- **Zacian e Zamazenta:** com a espada ou o escudo, o Iron Head vira Behemoth Blade ou Behemoth Bash no mesmo slot, e volta sem o item (`EspecieData.golpeQueSaiNaForma` e `golpeQueEntraNaForma`). Sem Iron Head, ganha a forma e não o golpe.
- **Encontro, pelo líder da equipe:**
  - Intimidate e Keen Eye: um selvagem 5 níveis ou mais abaixo do líder tem 50% de chance de nem lutar (`RegraDeEvitarEncontroFraco`, depois do sorteio, como o repelente).
  - Static e Magnet Pull: 50% de o sorteio sair só entre Elétricos e Metálicos, se o lugar tiver (`RegraDeAtracaoDeEncontro`).
- **Flautas** (Black e White Flute): *não feitas.* O efeito muda de geração para geração; precisa de decisão do dono do projeto (qual geração seguir).
