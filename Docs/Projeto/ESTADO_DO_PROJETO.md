# Estado do projeto — onde estamos, em 14/10/2026

> ## ⚠️ URGENTE: resolver a UI
>
> **(07/10/2026, revisto em 14/10) A UI é hoje o que mais trava o projeto.** Motor de batalha, mapa e conteúdo avançam, mas nada disso chega ao jogador sem tela:
>
> - **A pilha não foi decidida** (uGUI × UI Toolkit, quatro telas em duplicidade): toda tela nova espera essa resposta.
> - **As telas que existem são provisórias** (uGUI marcadas com `[PlaceholderDeUI]`; foram religadas em 12/10, depois de cinco dias desligadas): título, salvar/carregar, HUD, cartão, configurações. A escolha em conversa ganhou caixa própria em 11/10 (`EscolhaDeRoteiroUI`).
> - **Não existem:** tela de evolução, de aprender golpe, de resultado da batalha; clima, terreno, efetividade e golpe de vários turnos na batalha.
>
> **Ordem:** decidir a pilha → telas da batalha (incluindo Z, Dynamax e Tera, P31). Tudo em [`../Jogo/UI_DO_PROJETO.md`](../Jogo/UI_DO_PROJETO.md). A pergunta que destrava é a **P1** de [`DECISOES.md`](DECISOES.md).

| | |
|---|---|
| **O que este documento responde** | Qual é a situação do projeto **agora**: o que funciona, o que não, o que está em andamento, o que trava |
| **Quando abrir** | Antes de qualquer outro documento (depois do [`../../README.md`](../../README.md)). Os demais explicam o porquê |
| **O que ele NÃO é** | Referência de sistema ([`../Jogo/`](../Jogo/)), backlog ([`PENDENCIAS.md`](PENDENCIAS.md)), regras de trabalho ([`COMO_TRABALHAR.md`](COMO_TRABALHAR.md)) nem histórico |
| **Como foi feito** | Cada número abaixo foi **contado no repositório em 14/10/2026** (comandos na §9). Onde não foi medido, está escrito "não medido" |
| **Envelhece rápido** | A §2 foi contada e o resto revisto em 14/10. Se a data do título tiver mais de duas semanas, refaça a §2 inteira (não uma linha) antes de confiar nela |

---

## 1. Em uma página

**O que é.** Vertical slice de um RPG 3D de exploração e captura, com batalha por turnos, em Unity 6 (`6000.5.10f1`, URP). Projeto em grupo. Alvo declarado: mundo grande em várias áreas conectadas e **2 a 18 jogadores em rede** (é alvo, não existe ainda).

**Onde estamos.** O **motor de batalha é o ponto mais forte**: regras como dado (`MesaDeRegras`), determinismo, dupla, IA com dificuldade, e conteúdo de golpes, habilidades e itens cruzado com o Pokémon Showdown em 28 levas (Megas oficiais, bagas, formas por gatilho, Z-moves, Dynamax e Tera já implementados, **desligados por padrão**). Desde 13/10 a luta **se fotografa e se repõe** (a retomada dá o mesmo resultado que a original) e **várias batalhas convivem no mesmo processo** (cada Pokémon carrega a sua). O **mundo de Kanto é jogável** (mapa, história do FireRed importada, 744 treinadores, 327 tabelas de encontro) e o jogo entra pela **multicena por área** (`KantoMulticena`, DECISOES T51), embora só o começo da história esteja migrado para ela. O que **falta é apresentação e rede**: a UI existe em duas pilhas paralelas e com lacunas grandes, não há áudio, animação só do jogador, e o transporte de rede não foi escolhido.

**O que funciona hoje (coberto por teste automatizado):** batalha 1×1 e dupla, captura, evolução, save em 3 slots (formato 7, escrita atômica), inventário, loja, tutores de golpe, treinadores com IA, clima/terreno/efeitos de campo, formas em batalha (Mega, Primal, Deoxys, formas por gatilho), bagas, itens segurados que podem ser derrubados ou arremessados, item que concede qualquer habilidade, e a carga do mapa por áreas.

**O que não funciona ou não existe:** áudio (nenhum arquivo importado), animação de Pokémon/NPC/batalha, UI de clima/terreno/campo na batalha, tela de evolução, ginásios como sistema, conteúdo de missões (o modelo existe, nenhuma foi criada), mapa do mundo, co-presença e batalha em rede, e **nenhuma medição de desempenho numa build** (só no Editor).

**Os riscos que mais importam agora:**
1. **Duas UIs paralelas** (uGUI × UI Toolkit) com quatro telas duplicadas, sem decisão de qual fica.
2. **Duas cenas de 89 MB e 74 MB já estão no `origin`** sem LFS (`Kanto3D`, `KantoPecas`); o GitHub recusa arquivo acima de 100 MB. Desde 12/10 o jogo entra pela multicena (DECISOES T51), mas a história só foi migrada até o começo de Pallet: o resto ainda vive na `Kanto3D` e não é alcançável jogando.
3. **Os itens vieram do APK do Omega** com outra escala: preço (corrigido em 12/10 nos 50 itens de loja) e efeito (a Potion cura 400 HP). PENDENCIAS S15 e S20.

---

## 2. Números (contados em 14/10/2026)

### 2.1 Repositório

| | |
|---|---|
| Branch de trabalho | `mapa/blocos` (último commit antes da reorganização de 13/10: `456f9505`) |
| Commits | 362 no total; **236 à frente** da `main`, **0 atrás** |
| Unity | `6000.5.10f1` (a `main` ainda está em `9f1`) |
| Formato do save | **7** (`SistemaSave.VersaoDoFormato`); lê os anteriores migrando na memória |
| Remoto | `https://github.com/Mhgfxv00/Nel-Ord` |

### 2.2 Código (somente runtime: sem `Editor/` e sem testes)

| | Arquivos | Linhas |
|---|---:|---:|
| `Assets/Scripts/Data` (assets de dados, `Pokemon`, regras, efeitos) | 58 | 14.737 |
| `Assets/Scripts/Systems` (batalha, IA, save, inventário, mundo…) | 48 | 14.359 |
| `Assets/Scripts/World` (jogador, NPCs, interagíveis, obstáculos) | 33 | 3.918 |
| `Assets/Scripts/UI` (uGUI) | 19 | 3.800 |
| `Assets/Scripts/Roteiro` (roteiros de história) | 14 | 2.334 |
| `Assets/Scripts/Interacao` (protótipo de cards) | 7 | 1.631 |
| `Assets/Omega/Game/Runtime` (o outro lado do grupo: UI Toolkit, jogador Omega, dados Omega) | 33 | 5.065 |
| **Total de runtime** | **212** | **≈ 45.800** |

Fora do runtime: **52** ferramentas em `Assets/Scripts/Editor` e **67** em `Assets/Omega` (`Editor` e `Game/Editor`); **157** itens de menu (102 `Omega/…`, 55 `Pokemon/…`), catalogados em [`FERRAMENTAS_DO_EDITOR.md`](FERRAMENTAS_DO_EDITOR.md). Seis assembly definitions (§ "Mapa do código" em [`../Jogo/REFERENCIA_DE_SISTEMAS.md`](../Jogo/REFERENCIA_DE_SISTEMAS.md)).

### 2.3 Testes

| Suíte | Arquivos | Casos (`[Test]`/`[UnityTest]`/`[TestCase]` no código) | Última rodada completa registrada |
|---|---:|---:|---|
| **EditMode** | 112 | 898 | **898/898** em 15/10 |
| **PlayMode** | 40 | 261 | **260 passaram, 0 falharam, 1 ignorado** em 15/10 (o de teclado, que pede a Game View em foco) |

**Medido em 15/10/2026** (linha de comando, Editor fechado). O PlayMode é o que exercita o `SistemaBatalha` de verdade (é corrotina); mexer nele exige rodar as duas suítes. As conformidades com o Showdown (dano e fluxo) rodam dentro do EditMode e estão a 100%.

### 2.4 Conteúdo (assets em `Assets/Data`: `Conteudo/`, `Mundo/`, `Sistema/`)

| | Quantidade | Observação |
|---|---:|---|
| Espécies | 1.454 | inclui formas; `formaDe` agrupa as formas sob a base na Pokédex |
| Golpes | 955 | boa parte ligada às peças de regra pelo `ConteudoDoShowdown` (levas 1 a 28) |
| Habilidades | 324 | **13 sem regra nem gatilho**. Em 13/10 saiu todo o conteúdo não oficial, menos `Eventide` (quarentena) |
| Itens | 2.264 | todos em `Itens/` (`HeldItems/` ficou vazia). **1.824 têm preço 0**: nenhuma loja os vende. `AssaultArmor` fica na quarentena |
| Status / efeitos de campo | 56 / 31 | |
| Terrenos / climas | 4 / 4 | climas com comportamento: Sol, Neve, Areia e Ventos Fortes (Delta Stream); chuva vive na tabela do `SistemaMundo` |
| Perfis de IA | 6 | 4 sem uso (esperam adversários de fim de jogo) |
| Treinadores / encontros / roteiros | 744 / 327 / 2.026 | história do FireRed importada; **295 treinadores e 237 tabelas sem caminho** até o jogador em 30/09 (mapas sem sala 3D, revanches) — *não recontado* |
| Lojas / interações (cards) | 24 / 2 | + `LojaDeTeste_TodosOsItens` |
| Corpos de NPC (prefabs) / professores de golpe | 44 / 14 | |
| Eventos de batalha (`EventoBatalha`) | 28 | |
| Critérios de IA (avaliadores) | 11 | |

### 2.5 Cenas

| Cena | Tamanho | Papel |
|---|---|---|
| `Jogo/Bootstrap`, `Jogo/OmegaMenu` | pequenas | Cena inicial: dona de jogador, câmera, UI, sistemas; menu |
| `Interiores/Casa`, `Interiores/Quarto` | pequenas | Interiores |
| `Desenvolvimento/` (`DevScene`, `OmegaDev`, `ArenaExemplo`, `OmegaBlocos`, `OmegaMapa`, `GaleriaDePrefabs`) | pequenas | Desenvolvimento e testes |
| **`Legado/Kanto3D`** | **89 MB** | Mapa original em um arquivo só. **No `origin` sem LFS** |
| **`Legado/KantoPecas`** | **74 MB** | Mapa remontado em prefabs. **No `origin` sem LFS** |
| **`Mundo/KantoMulticena`** | pasta de 75 MB: `KantoBase` + 14 áreas | O mapa dividido, carregado por proximidade. **É por onde o jogo entra** (T51) |

27 arquivos `.unity` em `Assets/Scenes` (por papel: `Jogo`, `Mundo`, `Interiores`, `Desenvolvimento`, `Legado`: ver [`ESTRUTURA_DO_PROJETO.md`](ESTRUTURA_DO_PROJETO.md)); 23 entradas no Build Settings.

---

## 3. O que está pronto, por área

| Área | Estado | Onde ler |
|---|---|---|
| **Batalha** (regras, dano, precisão, prioridade, status, clima, terreno, dupla, formas, chamarizes) | ✅ sólida; 28 eventos; determinismo por semente; foto da luta restaurável (R3/R4); várias batalhas no mesmo processo (D3) | [`../Jogo/GDD.md`](../Jogo/GDD.md) §5; [`../Jogo/BATALHA_ATE_O_OFICIAL.md`](../Jogo/BATALHA_ATE_O_OFICIAL.md) |
| **Mecânicas de formato** (Mega, Z, Dynamax, Tera) | ✅ implementadas; **só a Mega está ligada** (decisão P9) | BATALHA_ATE_O_OFICIAL §12 e §13 |
| **IA de combate** | ✅ um cérebro, dificuldade por parâmetro; 3 níveis escolhíveis pelo jogador. Desde 14/10 joga o campo (armadilhas, telas, Protect, pivô…), conta prioridade e Trick Room, presume a troca do jogador (Pursuit) e escolhe a forma do Deoxys. Falta Illusion | [`../Jogo/MODULOS_DO_JOGO.md`](../Jogo/MODULOS_DO_JOGO.md) "IA de combate" |
| **Conteúdo do Showdown** | 🟡 levas 1 a 28 feitas (28/09 a 12/10). O que falta de verdade está em BATALHA_ATE_O_OFICIAL §4 (Gigantamax e as formas da Ogerpon foram descartados com o conteúdo não oficial) | BATALHA_ATE_O_OFICIAL §4; [`../Jogo/AUDITORIA_OFICIAL.md`](../Jogo/AUDITORIA_OFICIAL.md) |
| **Save** | ✅ formato 7, três slots, escrita atômica com reserva, migração em degraus | [`COMO_TRABALHAR.md`](COMO_TRABALHAR.md) §5 |
| **Inventário, loja, tutores, item de habilidade** | ✅ na lógica; a UI é o que falta | [`../Jogo/UI_DO_PROJETO.md`](../Jogo/UI_DO_PROJETO.md) §3 |
| **Roteiro (história)** | ✅ 22 tipos de passo; 2.026 roteiros importados; escolha pela `EscolhaDeRoteiroUI` (provisória) | [`../Jogo/REFERENCIA_DE_SISTEMAS.md`](../Jogo/REFERENCIA_DE_SISTEMAS.md) "Roteiro"; [`../Mundo/MONTAGEM_DE_MAPAS.md`](../Mundo/MONTAGEM_DE_MAPAS.md) R8 |
| **Mundo (Kanto)** | ✅ jogável pela multicena; 🟡 história migrada só até o começo de Pallet; desempenho não medido numa build | [`../Mundo/`](../Mundo/) |
| **Encontros por clima e período** | ✅ | MODULOS_DO_JOGO "Encontros por contexto" |
| **UI** | 🟡 duas pilhas, lacunas grandes | UI_DO_PROJETO |
| **Rede** | 🟡 as costuras existem (ação como dado, árbitro, foto do estado e da luta, registro); **transporte e árbitro remoto não** | [`../Planos/REDE_E_MULTIPLAYER.md`](../Planos/REDE_E_MULTIPLAYER.md) |
| **Áudio** | ❌ o sistema existe, **nenhum arquivo** | PENDENCIAS S1 |
| **Animação** | 🟡 só a locomoção do jogador (`AnimadorDoJogador`); **zero** para Pokémon, NPC e batalha | [`../Jogo/ANIMACAO.md`](../Jogo/ANIMACAO.md) |
| **Ginásios, insígnias, missões, estatísticas** | 🟡 estatísticas e missões existem (10/10) e aparecem no cartão de treinador (12/10); **nenhuma missão criada**; `badges` só é incrementado pelo passo `PassoDarInsignia`; líderes e campeão marcados como marco de revanche | PENDENCIAS S4, S6 |

---

## 4. Em andamento

| Frente | Situação | Próximo passo |
|---|---|---|
| **Conteúdo contra o oficial** | Levas 1 a 28. **P10 respondida em 13/10:** o conteúdo não oficial foi removido (ficam `AssaultArmor` e `Eventide`) | Sobras a decidir: Power Ring/Pulse Cannon (pergaminhos), Flash Flute (%), moldes de sistema. Depois, as peças em BATALHA_ATE_O_OFICIAL §4 |
| **KantoMulticena** | Base + 14 áreas funcionando (2 testes de PlayMode); o jogo entra por ela desde 12/10 | Assar occlusion por cena; regenerar as cenas com as estruturas altas no horizonte (código pronto, **não rodado**); medir numa build; migrar o resto da história (P26) e tirar `Kanto3D`/`KantoPecas` do git (P2) |
| **UI** | Caixa de escolha (11/10) e placeholders religados (12/10); o resto da §6 de UI_DO_PROJETO segue | Decisão da pilha (P1); depois as telas da batalha |
| **Mapas** | Spawn, tutores, vendedor e treinador de teste em Pallet; zona de mato na Rota 1. Em 13/10 prefabs, cenas e `Data` foram reorganizados, com índice e galeria de prefabs ([`../Mundo/CATALOGO_DE_PREFABS.md`](../Mundo/CATALOGO_DE_PREFABS.md)) | [`../Mundo/MONTAGEM_DE_MAPAS.md`](../Mundo/MONTAGEM_DE_MAPAS.md) §7: faltam barranco, música/clima por zona, surf de verdade |
| **Desempenho** | Sombra do URP reduzida (60 m, 2 cascatas, mapa 2048): −3 a −4 ms/quadro **no Editor** | Uma build de desenvolvimento na máquina mínima (O11); nenhum número de build existe |
| **Merge com a `main`** | **Feito em 10/10** (`5c1e3aeb`); desde então a `main` acompanha a `mapa/blocos`. As cenas grandes foram junto (C3 segue aberto) | — |
| **Barramento único de eventos (M1)** | Parado de propósito por decisão de Higor (13/10): fica o estado atual | Retomar só se a rede pedir |

---

## 5. Decisões que esperam resposta (as que mais travam)

A lista completa, com opções, recomendação e dono, está em [`DECISOES.md`](DECISOES.md) §2 (P1 a P31). As que travam trabalho hoje:

| Decisão | Por que trava | Dono |
|---|---|---|
| **P1. A pilha de UI** — o UI Toolkit substitui a uGUI? | Quatro telas em duplicidade; toda tela nova pergunta isso | UI (outro lado do grupo) |
| **P2. Adotar a multicena e tirar as duas cenas gigantes do git** | Limite de 100 MB do GitHub; o merge levaria as duas | Mapa + Higor |
| **P5. Preço dos 1.824 itens de preço 0** | A loja de teste vende só os que têm preço | Conteúdo |
| **P9. Quais mecânicas de formato o jogo liga** | Z, Dynamax e Tera existem e estão desligadas | Grupo de klz |
| **P7. Diálogo ramificado** | Adiado de propósito, espera material externo | Higor |
| **P8. Cor/textura do mapa** | Decisão do grupo do mapa, sem prazo | Mapa |

---

## 6. Desempenho

- **Máquina mínima de referência:** Ryzen 5 4600G (gráfico integrado), 16 GB, HD + SSD. Toda meta é contra ela.
- **Medido, no Editor** ([`../Mundo/MAPAS_E_DESEMPENHO.md`](../Mundo/MAPAS_E_DESEMPENHO.md)): a KantoMulticena roda a ~16 a 22 ms/quadro depois de reduzir as sombras; 51.584 renderers, todos com sombra e sem LOD; a sombra é a maior alavanca.
- **Medido numa build:** o bootstrap deixou de carregar as imagens de todas as espécies (985 → 77 MB de textura); **é o único número de build que existe**.
- **Não medido:** memória e quadro numa build; a carga das áreas da multicena no HD; o custo do occlusion culling assado por cena.
- **Regra:** número sem recorte (cena, máquina, data) não entra; "não medi" quando não mediu.

---

## 7. Riscos

| Risco | Gravidade | Mitigação |
|---|---|---|
| Cenas de 74 e 89 MB no `origin` sem LFS; cada commit delas soma ~80 MB ao histórico | alta | Não commitar as duas; adotar a multicena |
| UI em duas pilhas diverge a cada mudança | alta | Decidir a pilha (P1) |
| História só na `Kanto3D`, e o jogo entra pela multicena | alta | Migrar o resto da história (P26), como a fatia de Pallet |
| Editor cai por falta de memória (16 GB) depois de horas de suíte, ou ao abrir a cena de 76 MB | média | Reiniciar o Editor antes de suíte grande; não abrir a origem da multicena sem necessidade |
| A Unity cai sozinha em batch de vez em quando (08/10) | baixa | Repetir a rodada; só vale o que está no XML ([`COMO_TRABALHAR.md`](COMO_TRABALHAR.md) §6) |
| Batalha só tem teste real em Play Mode (`SistemaBatalha` é corrotina) | média | Sempre rodar o PlayMode inteiro depois de mexer nele |
| Nenhuma medição de build | média | O11 |
| Teste de teclado do Play Mode falha por foco da Game View quando a suíte é cheia | baixa | Passa isolado; PENDENCIAS D10 |

---

## 8. Onde está cada documento

O mapa completo, "qual pergunta cada documento responde", mora em [`../README.md`](../README.md). Regra de manutenção: ao resolver algo do backlog, atualize [`PENDENCIAS.md`](PENDENCIAS.md) **no mesmo commit**; ao mudar um número daqui, refaça a §2 inteira; documento novo entra no índice.

---

## 9. Como reproduzir os números

No Git Bash, na raiz do projeto:

```bash
git branch -vv ; git rev-list --count main..HEAD ; git rev-list --count HEAD..main ; git rev-list --count HEAD   # §2.1
grep -n "VersaoDoFormato *=" Assets/Scripts/Systems/SistemaSave.cs                                              # formato do save
for d in Data Systems World UI Roteiro Interacao; do find Assets/Scripts/$d -name '*.cs' -not -path '*/Editor/*' -not -path '*/Tests/*' | xargs cat | wc -l; done   # §2.2
find Assets/Data/Conteudo/Especies -name '*.asset' | wc -l                                                               # §2.4 (repita por pasta de Data)
grep -rhoE 'MenuItem\("[^"]+"' Assets --include=*.cs | wc -l                                                    # menus
grep -rE '^\s*\[(Test|TestCase)' Assets/Scripts/Editor/Tests | wc -l                                            # casos EditMode
grep -rE '^\s*\[(UnityTest|Test|TestCase)' Assets/Scripts/Tests/PlayMode | wc -l                                # casos PlayMode
find Assets/Scenes -name '*.unity' ; grep -c 'path:' ProjectSettings/EditorBuildSettings.asset                  # §2.5
```

Rodar os testes de verdade: **Window → General → Test Runner**, ou [`COMO_TRABALHAR.md`](COMO_TRABALHAR.md) §6 (linha de comando).

Habilidades vazias: contar os `.asset` de `Assets/Data/Conteudo/Habilidades` que têm `regras: []` **e** `gatilhos: []`. Itens com preço 0: contar os `.asset` de `Assets/Data/Conteudo/Itens` e `HeldItems` com `preco: 0` (use `find -print0`: há nomes com espaço).
