# Ferramentas do Editor — os menus `Pokemon/…` e `Omega/…`

| | |
|---|---|
| **O que este documento responde** | Que menu faz o quê, qual deles é seguro rodar, e qual está **congelado** |
| **Quando abrir** | Antes de rodar um menu que você não conhece; ao procurar "existe uma ferramenta para isso?" |
| **Estado** | 139 itens de menu (39 `Pokemon/…`, 100 `Omega/…`), contados em 09/10/2026 com `grep -rhoE 'MenuItem\("[^"]+"' Assets --include=*.cs`. As descrições vêm do nome do menu e do comentário da classe; **as ferramentas não foram executadas para este documento** |
| **Regra de ouro** | Rodou uma ferramenta que grava asset? **Leia o `git diff` antes de commitar**: a Unity também regrava material e cena ao abrir. E só confie em quem confere o arquivo em disco: logo depois de compilar, o CLI dá timeout mas o comando roda ([`COMO_TRABALHAR.md`](COMO_TRABALHAR.md) §4) |

**Legenda.** ❄️ = **congelado** (recusa rodar; ver abaixo) · ✅ = só lê/confere, seguro · ✍️ = grava asset ou cena · ♻️ = idempotente (rodar de novo não duplica) · 🕰️ = histórico: fez seu trabalho uma vez, rodar de novo é raro

## Os geradores congelados ❄️

Os geradores de dados criaram espécies, golpes, habilidades, itens e a história a partir do APK, da PokeAPI e do FireRed. Depois o conteúdo foi corrigido à mão (226 espécies, 255 itens, 19 habilidades e 3 golpes de fã ou lixo **removidos**; ~100 itens e vários golpes e habilidades ganharam comportamento). **Rodar um gerador de novo pode desfazer isso.** Por isso todos começam com `GeradoresCongelados.Permitir(...)` e recusam.

- Para rodar mesmo assim: menu `Pokemon → Geradores → Descongelar nesta sessao` (vale até fechar a Unity) ou, no batch, a variável de ambiente `NELORD_DESCONGELAR_GERADORES=1`. Depois, confira o `git diff` asset por asset.
- O conteúdo removido está em `Assets/Omega/Dados/conteudo_removido.txt`. O teste `GeradoresEFerramentasTests` falha se qualquer um voltar a existir em `Assets/Data` e se aparecer menu gerador sem a trava.
- Conteúdo **novo** entra por `ConteudoDoShowdown` (preenche comportamento, nunca cria espécie) ou por asset feito à mão.
- Os 23 menus congelados (em 16 arquivos): `Omega/Dados/` → *Colocar trocador e ponto de evolucao*, *Corretor de assets*, *Completar evolucoes e golpes por nivel*, *Normalizar slots de habilidade*, *Completar golpes*, *Completar habilidades*, *Importar dados FireRed*, *Lote 6, 7, 8, 9, 11 e 12* (passadas 1 e 2), *Ligar MTs e MOs*, *Professores de golpe (golpesPorTutor + prefabs)*; e `Omega → Historia Pallet`, `Omega → Importar historia FireRed`.

---

## 1. Conferir (rode antes de commitar conteúdo)

| Menu | O que faz | |
|---|---|---|
| `Pokemon → Validar assets de dados` | Confere ids, referências nulas, catálogo, linhas evolutivas (habilidades alinhadas por slot), contadores nomeados (todo `contadorNecessario` precisa de produtor). Roda também como teste | ✅ |
| `Pokemon → Validar roteiros` | Acusa passo vazio, batalha sem treinador, escolha sem opção e **flag lida que ninguém escreve** (erro de digitação que faz o "Se" cair no "Senão" para sempre, sem log) | ✅ |
| `Pokemon → Auditar alcancabilidade do conteudo` | Acusa asset que **ninguém aponta** (loja sem NPC, golpe que ninguém ensina). Prova que há um caminho declarado, **não** que o jogador consegue percorrê-lo. Exceções têm motivo escrito no código | ✅ |
| `Pokemon → Itens → Auditar habilidades vazias em especies oficiais` | Lista habilidades sem regra nem gatilho em espécie com número oficial de Pokédex (relatório em `Temp/habilidades_vazias.txt`) | ✅ |
| `Pokemon → Itens → Auditar todo o conteudo sem comportamento` | Idem para golpes, habilidades e itens (relatório em `Logs/conteudo_sem_comportamento.txt`) | ✅ |
| `Omega → Diagnostico - scripts ausentes` | Procura *missing script* e assets cujo tipo não carrega (útil depois de mexer em assembly) | ✅ |
| `Omega → Conferir merge completo`, `Conferir animacao`, `Conferir UI`, `Conferir colisao do chao e das arvores`, `Conferir especies importadas` | Conferências pontuais de integrações antigas | ✅ 🕰️ |
| `Pokemon → Depurar IA de Batalha` | Janela que mostra, em Play Mode, as candidatas da IA com nota e motivo (golpe/dano/efetividade/KO). Só custa quando está aberta | ✅ |

## 2. Cena, catálogo e itens

| Menu | O que faz | |
|---|---|---|
| `Pokemon → Popular catalogo selecionado com todos os assets` | Com o `CatalogoJogo` selecionado, reconstrói as listas com tudo que existe no projeto. **Asset fora do catálogo grava no save e some ao carregar** | ✍️ ♻️ |
| `Omega → Itens - registrar no catalogo` | Enche o catálogo com todos os `ItemData` | ✍️ ♻️ |
| `Omega → Pokedex - popular catalogo e ligar tecla E` | Enche o catálogo com as espécies e instala o atalho da Pokédex | ✍️ 🕰️ |
| `Pokemon → Criar objeto de Sistemas` | Cria o GameObject `Sistemas` (estados, inventário, batalha, encontros, diálogo, save). Só para reconstruir do zero: o Bootstrap já é dono disso | ✍️ 🕰️ |
| `Pokemon → Criar tipos e caracteristicas basicos` | Cria os tipos e as características iniciais e a `TabelaTipos` clássica | ✍️ ♻️ 🕰️ |
| `Pokemon → Criar perfis de IA basicos` | Cria os 6 `PerfilDeIA` em `Assets/Data/Sistema/PerfisDeIA` a partir dos presets | ✍️ ♻️ 🕰️ |
| `Pokemon → Soltar sprites de Resources do catalogo` | Converte as referências diretas de sprite no catálogo em caminho de `Resources` (mantém o bootstrap leve) | ✍️ ♻️ |
| `Omega → Montar catalogo do avatar`, `Merge - fazer tudo` | Primeira ligação dos assets extraídos do APK (peças do avatar) | ✍️ 🕰️ |

## 3. Conteúdo do Showdown (`Pokemon → Showdown …` e `Pokemon → Itens …`)

Preenchem **comportamento** em assets que já existem (nunca criam espécie), a partir de peças tipadas, cruzando com o Pokémon Showdown. São **idempotentes** e, em geral, em **duas passadas** (`1 - criar status` e `2 - preencher`): rode na ordem. O plano e o que cada leva fez estão em [`../Jogo/BATALHA_ATE_O_OFICIAL.md`](../Jogo/BATALHA_ATE_O_OFICIAL.md). Depois de cada uma, rode as duas suítes.

**O caminho normal desde 11/10/2026 é o importador único**, não as levas uma a uma:

1. `node tools/showdown/exportar.mjs` baixa os dados do Showdown no commit fixo (`3065d24d`) e grava `tools/showdown/dados_showdown.json`, uma entrada por linha. Só precisa rodar de novo quando o commit fixo mudar, e o diff do JSON mostra o que o Showdown mudou.
2. `Pokemon → Showdown → Importador - 1 comparar (so le)` grava `Logs/importador_showdown.txt` com:
   - as divergências de dado: tipo, poder, precisão, PP, prioridade e categoria dos golpes; tipos e atributos base das espécies;
   - o que é oficial e não tem asset;
   - o que o projeto tem sem par no Showdown.
3. `Pokemon → Showdown → Importador - 2 aplicar tudo (dados e receitas)` roda todas as receitas abaixo na ordem certa, numa passada, e depois corrige os dados pelo Showdown, que tem a palavra final sobre número. Em seguida repovoa o catálogo e a loja de teste. Em lote: `-executeMethod ConteudoDoShowdown.ImportadorAplicarEmLote`.

O "aplicar tudo" é idempotente: rodar duas vezes dá **0 assets mudados** na segunda. Reaplicar recria as listas `[SerializeReference]` com `rid` novos, então o importador devolve ao disco, byte a byte, todo asset cujo texto só mudou em `rid`, fim de linha ou campo novo gravado com o valor padrão. O que sobra no git é mudança real, listada no relatório.

Ficam **fora** da corrente as migrações de uma vez só, que apagam ou renomeiam: `Megas - 1` e `- 3`, remover espécies de fã e a limpeza. Uma leva nova entra como mais uma linha em `ConteudoDoShowdown.Receitas()` (`ConteudoDoShowdown.Importador.cs`). Nomes que o Showdown escreve diferente (o Minior Meteoro, as formas de Alola `p20NN_<Nome>a`) ficam em `Apelidos`. A quarentena (`Assets/Data/Conteudo/NaoOficial`) não entra.

Os menus de cada leva continuam valendo para rodar uma só:

| Menu | Leva |
|---|---|
| `1 - Criar status e efeitos de campo que faltam`, `2 - Preencher golpes, habilidades, status e campo` | Levas 1 a 5 (28 e 29/09) |
| `Leva 7 - 1 criar o status do Powder`, `- 2 preencher golpes` | Beat Up, Conversion 2, Powder, Speed Swap, Guard/Power Split |
| `Leva 8 - 1 criar status e etiqueta`, `- 2 preencher golpes` | Pursuit, Snatch, Me First, Bide |
| `Leva 10 - formas por gatilho` | Zen Mode, Shields Down, Forecast, Disguise, Ice Face… |
| `Leva 11 - aliado em dupla e itens` | Symbiosis, Receiver, Ally Switch; Quick Claw, Air Balloon… |
| `Leva 12 - 1 criar status`, `- 2 preencher` | O que a auditoria completa achou (40 golpes, 5 habilidades, 20 itens) |
| `Leva 13 - bagas, Mega e itens de especie` | Bagas (`Baga`), pedras Mega, placas, itens de origem |
| `Leva 14 - Multitype, Judgment e restricoes de habilidade` | Arceus e as 32 restrições de habilidade |
| `Leva 15 - Z, Dynamax e Tera` | Cristais Z, Max moves, Tera |
| `Leva 16 - Leppa e bagas da Gen 2` | |
| `Leva 17 - Queimadura no dano (Guts e Facade)` | O que o teste de conformidade de dano achou nos dados: queimadura corta o dano físico (não o Ataque), Guts e Facade ignoram, Thick Fat/Water Bubble/Purifying Salt/Heatproof cortam o ataque de quem bate |
| `Leva 18 - Repelente e taxa de encontro` | Repel, Super Repel e Max Repel como `ItemRepelente`; a regra de taxa de encontro em Stench, White Smoke, Quick Feet, Infiltrator, Arena Trap, No Guard, Cleanse Tag e Pure Incense |
| `Leva 19 - Evolucoes por equipe, troca, dano, passos e Shedinja` | As evoluções oficiais que dependiam das peças novas (Mantyke, Pancham, Karrablast, Shelmet, Yamask de Galar, Basculin, Pawmo, Bramblin, Rellor, Nincada) |
| `Leva 20 - 1 habilidades Embody Aspect e golpes G-Max`, `- 2 ligar Ogerpon, Gigantamax e Maushold` | Parte 1 antes, parte 2 depois do menu `Formas` (que cria as formas da Ogerpon e as Gigantamax a partir de `dados_showdown_formas.tsv`) |
| `Leva 23 - Struggle` | Depois da Leva 22. O Struggle sem tipo, com o recuo oficial (1/4 arredondado, direto), e o `SistemaBatalha` do prefab de sistemas apontando para ele |
| `Leva 24 - Anuncio de habilidade ao entrar` | Depois da Leva 23. Preenche `HabilidadeData.AnuncioAoEntrar` nas 13 habilidades que o Showdown anuncia na entrada (Pressure, Mold Breaker, as auras, Unnerve…) |
| `Leva 25 - Dupla (Intimidate em todos os adversarios)` | Depois da Leva 24. Liga `emTodosOsAdversarios` no Intimidate. O alvo "todos os adjacentes" dos 20 golpes de área não é leva: vem do importador, que compara o alvo com o Showdown |
| `Leva 26 - Ability Capsule e Ability Patch` | Depois da Leva 25. Troca a classe dos dois assets de `ItemAtributo` para `ItemTrocaDeHabilidade` (modo 0 = Capsule, 1 = Patch) editando o texto do asset, para manter o arquivo e o guid |
| `Leva 27 - Golpes da varredura por golpe` | Depois da Leva 26. O que a varredura por golpe achou contra o Showdown (12/10): 6 golpes com o efeito em si mesmo no alvo, Tera Blast, Electro Shot e Meteor Beam (+1 no preparo), as habilidades ignoradas, Victory Dance e Shelter, os recuos de Steel Beam, Mind Blown e Chloroblast, Fillet Away e Belly Drum, Flail e Reversal pela tabela, Heat Crash e Heavy Slam, Syrup Bomb, as falhas por condição (Dream Eater, Snore, Dark Void, Aura Wheel, Hyperspace Fury, Spit Up, No Retreat, Shed Tail, Aurora Veil), as salas que alternam, Gravity, Burning Jealousy, Alluring Voice, Collision Course, Shell Side Arm, Aromatic Mist, Coaching e Magnetic Flux. Usa as peças novas `RegraDeFalhaAoUsar` (motivos `CondicaoValendo`, `CondicaoNaoValendo`, `SemParaQuemTrocar`), `RegraDeGolpesProibidos`, `RegraDoMelhorDano`, `PoderPorHpEmFaixas` e `EfeitoSeAlvoSubiuEstagio` |
| `Leva 28 - Efeitos dos itens` | Depois da Leva 27. Cura, EV, Exp. Candy, Tera Shard e quatro itens segurados contra o texto da geração mais nova (T53). **Troca a classe do asset no próprio texto** (mesmo arquivo e guid): as 18 Tera Shards viram `ItemTipoTera`, as 5 Exp. Candies `ItemExperiencia`, e Lucky Egg, Amulet Coin, Luck Incense e Everstone `HeldItemData`. Duas passadas internas; idempotente |
| `Leva 22 - Achados da camada B de conformidade` | Depois da Leva 21. Troca o gancho do multiplicador (poder base e atributo de ataque em vez de dano final: 69 + 10 habilidades e itens), Gemas em ×1,3, Poison Heal, Espinhos Tóxicos de duas camadas, Intimidate barrada por Inner Focus e afins, Rest, a areia (no `Sistemas.prefab`), Zacian/Zamazenta Coroados e o encontro do líder (Intimidate, Keen Eye, Static, Magnet Pull) |
| `Leva 21 - Efeitos dos 10 G-Max que faltavam` | Depois da Leva 20. Cria os campos `GMaxWildfire`/`Cannonade`/`VineLash`/`Volcalith` e `AcoAfiado`, o status `ChiStrike`, e liga os efeitos |
| `Pokemon/Manutencao/Repovoar catalogo e loja de teste` | Depois de mexer em conteúdo em lote: repopula o catálogo (tira duplicatas) e refaz a loja de teste com todo item que tem preço |
| `Formas - regionais, Silvally, Origin e Crowned` | Formas que a auditoria contra o oficial achou |
| `Megas - 1 renomear, criar e apagar`, `- 2 preencher pelo Showdown`, `- 3 apagar formas base de fa` | As 94 Megas oficiais |
| `Golpe no aliado` | Golpe de dano mirando o aliado em dupla; Pollen Puff |
| `Pokemon → Itens → Preencher itens segurados (so os vazios / refazer todos com receita)` | `HeldItemData` com as peças existentes |
| `Pokemon → Itens → Preencher golpes de geracao nova que ja cabem nas pecas` | Golpes de Gen 8/9 |
| `Pokemon → Itens → Remover especies de fa listadas em Temp` | ⚠️ Apaga espécies listadas num arquivo temporário (a Unity apaga `Temp` ao abrir: a lista vem de variável de ambiente). Registre o que removeu em `conteudo_removido.txt` |

### 3.1 Ferramentas de linha de comando (Node, fora do Editor)

Precisam do Node.js 24 LTS. Nenhuma entra no build.

| Comando | O que faz |
|---|---|
| `node tools/showdown/exportar.mjs` | Baixa os dados do Showdown no commit fixo `3065d24d` e grava `tools/showdown/dados_showdown.json`. O Editor lê esse arquivo no importador único |
| `cd tools/conformidade && npm install && npm run dano` | Gera `esperado/dano.json` com a calculadora oficial (camada A do teste de conformidade) |
| `cd tools/conformidade && npm run simulador` | Baixa e compila o simulador do Showdown em `.showdown/` (fora do git; uma vez) |
| `cd tools/conformidade && npm run golpes` | Um cenário por golpe do Showdown, em duas variantes (neutra e com o alvo queimado, de HP baixo e com Ataque +6): grava `esperado/golpes.json` e `esperado/golpes-estado.json`. Os testes `Golpes_*` do `ConformidadeDeFluxoTests` os leem; relatórios em `Logs/conformidade_golpes*.txt`. Cada golpe que fica fora tem o motivo no `FORA` do gerador |
| `cd tools/conformidade && npm run fluxo` | Roda os cenários de `cenarios-fluxo*.js` no simulador e grava `esperado/fluxo.json` (camada B). O teste do Unity lê o arquivo e não precisa de Node |

Quando mexer num cenário ou subir o commit do Showdown: rode o comando, leia o `git diff` do JSON, e rode `ConformidadeDeFluxoTests`. Detalhes em [`../Planos/TESTE_DE_CONFORMIDADE.md`](../Planos/TESTE_DE_CONFORMIDADE.md) §7.

## 4. Arte

Fluxo e regras (nome do arquivo no padrão, rodar o menu de ligação, conferir no asset): [`COMO_TRABALHAR.md`](COMO_TRABALHAR.md) §4.2. Os menus de arte são **livres**: só ligam o que já existe.

| Menu | O que faz |
|---|---|
| `Omega → Sprites - 1. importar sprites de batalha`, `2. ligar nas especies`, `fazer tudo`, `refatiar pelas celulas` | Folhas de batalha por número de Pokédex (`0256`, `0256b`…) → `EspecieData.folhaEmResources`/`quadroDaFolha` |
| `Omega → Batalha 2D - 1. sprites para Resources`, `2. ligar na UI`, `corrigir import do fundo`, `fazer tudo`, `posicionar sprites`, `arrumar menus`, `diagnosticar layout/painel/paineis` | Apresentação 2D da batalha pendurada na `BatalhaUI` (sem reescrevê-la) |
| `Omega → Especies - importar e ligar sprites da PokeAPI` | Arte alternativa (retrato). O mapa `sprites_pokemon.json` ainda cita espécies removidas; é inofensivo |
| `Omega → Modelos - montar catalogo 3D` | `CatalogoModelos3D`: número da Pokédex → modelo `.glb`. **Hoje liga só 11 dos 968** (PENDENCIAS A11) |
| `Omega → Itens - fatiar e ligar icones` | Atlas de ícones → `ItemData.icone`. 2.106 itens ainda sem ícone |
| `Omega → UI - importar arte 9-slice` | Sprites de UI com borda |
| `Omega → Animacoes - importar Mixamo e montar Animator`, `Mixamo - exportar personagem` | Animação do jogador (Humanoid retargetado); exportação do avatar em OBJ para o Mixamo |
| `Omega → Personagens NPC → Gerar todos` (e `+ previa PNG`) | Corpos de NPC (`AparenciaDeNpc` + prefab) a partir das peças do criador. Resultado: `Assets/Prefabs/Personagens/` |
| `Omega → Pacotes - instalar Draco (modelos 3D)`, `reimportar modelos 3D` | Os `.glb` do Pokémon-3D-api vêm comprimidos em Draco |

## 5. Mapa de Kanto

Receitas e contexto: [`../Mundo/MONTAGEM_DE_MAPAS.md`](../Mundo/MONTAGEM_DE_MAPAS.md) e [`../Mundo/KANTO_REVISAO_E_MULTICENA.md`](../Mundo/KANTO_REVISAO_E_MULTICENA.md). **Atenção à memória:** as ferramentas que abrem a origem de 76 a 89 MB já derrubaram o Editor.

| Menu | O que faz | |
|---|---|---|
| `Omega → Kanto multicena - 1. relatorio das areas`, `2. separar em cenas por area`, `3. relatorio de pecas altas` | Divide um mapa grande já pronto em uma base + 14 áreas (`Assets/Scenes/Mundo/KantoMulticena`) | ✍️ ♻️ |
| `Omega → Kanto multicena - 4. tutores, vendedor, treinador, spawn e mato em Pallet` | **Exemplo de montagem por script**: coloca NPCs, spawn e mato nas cenas de área | ✍️ ♻️ |
| `Omega → Kanto - extrair pecas e montar cena nova`, `reaplicar regras nos prefabs`, `simplificar colisores dos predios` | Mapa em prefabs (`KantoPecas`) e a regra única de colisor | ✍️ |
| `Omega → Kanto - salvar na cena as correcoes de runtime` | Pedras da costa e paisagem distante assadas na cena | ✍️ 🕰️ |
| `Omega → Montar Kanto 3D` | Monta a `Kanto3D` original a partir do `kanto.glb` | ✍️ 🕰️ |
| `Omega → Colocar professores de golpe na Kanto3D` | Posiciona os 14 tutores | ✍️ |
| `Omega → Historia Pallet (roteiros + NPCs)`, `Importar historia FireRed (NPCs, treinadores, placas, encontros)` | História importada | ❄️ |
| `Omega → Visual → …` (Montar visual da Kanto 3D, Água estilizada, Cachoeiras vivas, Grama instanciada, Toon, Vegetação com vento) | Pacote visual do klz. **Nunca foi aplicado à cena versionada** (PENDENCIAS K2) | ✍️ |
| `Omega → Prefabs de mapa - criar cena galeria` | Cria `Assets/Scenes/Desenvolvimento/GaleriaDePrefabs.unity` com todos os prefabs de mapa em grade, para ver e escolher. O índice escrito é [`../Mundo/CATALOGO_DE_PREFABS.md`](../Mundo/CATALOGO_DE_PREFABS.md), gerado por `node tools/gerar-catalogo-de-prefabs.js` (um teste confere) | ✍️ |
| `Omega → Blocos`, `Chao`, `Campo`, `Grama - campo com batalha` | Cena de teste feita de blocos-prefab, do começo do Omega | ✍️ 🕰️ |

## 6. UI (do outro lado do grupo)

| Menu | O que faz |
|---|---|
| `Omega → UI - instalar telas em UI Toolkit`, `instalar menu principal em UI Toolkit`, `instalar PC Pokemon` | Penduram as telas Toolkit no `UI_Manager` e apontam o `Menu` para elas; a tela uGUI antiga fica no prefab, sem referência |
| `Omega → UI - consertar PanelSettings` | Corrige as duas causas de "abre e fica vazio": PanelSettings sem tema e `sortingOrder` 0 |
| `Omega → UI - reempilhar botoes do menu`, `diagnosticar tela de equipe` | Layout do menu principal |

## 7. Jogador, ambiente e utilitários

| Menu | O que faz |
|---|---|
| `Omega → Jogador - instalar avatar, animador e Pokedex no prefab`, `diagnosticar geometria`, `normalizar escala e colisor` | Prefab do jogador (`Assets/Prefabs/Bootstrap/Player.prefab`) |
| `Omega → Dev - instalar modo desenvolvedor` | Põe o `ModoDesenvolvedor` junto do `DadosJogador` (mochila cheia e Pokédex liberada) |
| `Omega → Ligar criador ao mundo` | Cenas no Build Settings e `AvatarDoJogador` no jogador |
| `Omega → Acabamento - colisao, animacao e menu`, `Diagnostico do campo` | Ajustes e diagnóstico do terreno de teste |
| `Omega → Cena - deixar so um AudioListener` | Remove ouvinte duplicado |
| `Omega → Fotografar o mundo` | Captura a cena a partir do jogador (batch sem `-nographics`) |
| `Omega → Ponte - estado` | Ponte de comandos que roda métodos do Editor aberto lendo `omega-comando.txt` e escrevendo `omega-resposta.txt` na raiz (substituída, na prática, pelo CLI oficial `unity`; os dois arquivos estão no `.gitignore`) |
| `Pokemon → Interacao (prototipo) → Editor de interacao`, `Montar demo na KantoMulticena` | Editor de grafo e demo do sistema de cards ([`../Jogo/INTERACAO_POR_CARDS.md`](../Jogo/INTERACAO_POR_CARDS.md)) |
| `Pokemon → Geradores → Descongelar nesta sessao`, `Congelar de novo` | Liga e desliga a trava dos geradores |

## 8. Como a lista foi montada

```bash
grep -rE 'MenuItem\("' Assets --include=*.cs | sed -E 's|^([^:]+):[0-9]+:.*MenuItem\("([^"]+)".*|\2\t\1|' | sort
grep -rl "GeradoresCongelados.Permitir" Assets --include=*.cs     # quais são geradores
```

Menu novo: **acrescente a linha aqui**. Se ele grava dado em massa, comece por `GeradoresCongelados.Permitir(...)` (o teste cobra).
