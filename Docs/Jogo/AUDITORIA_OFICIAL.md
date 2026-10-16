# Auditoria do conteúdo contra o oficial (09/10/2026)

O projeto tem tudo o que o oficial tem? E o que ele tem que **não** é oficial?

**Fontes de comparação**
- **PokeAPI** (`data/v2/csv`, branch master): lista de itens, espécies, golpes e habilidades dos jogos.
- **Pokémon Showdown** (commit `3065d24d`, `data/pokedex.ts`, `items.ts`, `moves.ts`, `abilities.ts`, `formats-data.ts`, `learnsets.ts`): formas, Megas, golpes e itens de batalha. Vai até Legends Z-A e a DLC Mega Dimension; o que o Showdown chama de "Past" (Mega Stones, Z-moves, Gen 2) conta como oficial.
- Casamento por **nome normalizado** (sem acento, sem pontuação) e, nas espécies, por **número da Pokédex**. As listas completas estão em [`auditoria_oficial/`](auditoria_oficial/) (TSV, para abrir em planilha).
- **Limite:** o que saiu depois do Showdown/PokeAPI (itens-chave e itens de história do Z-A, por exemplo) não tem como ser conferido por aqui.

## Resumo

| | Oficial | Projeto | Falta | Não é oficial |
|---|---:|---:|---|---|
| **Espécies** (arquivos) | 1.025 espécies + 354 formas = 1.379 | 1.432 | só formas: Gigantamax (34), Ogerpon (7) e Maushold de 3 | 1 forma de fã, 23 cópias do Omega |
| **Golpes** | 950 | 943 | 33 G-Max moves, Nihil Light | 21 de teste em português |
| **Habilidades** | 318 | 496 | 4 Embody Aspect (Ogerpon) | 165 de fã, 11 de teste |
| **Itens** | 2.223 (PokeAPI) + Gen 2 | 2.367 | Adventure Guide (conferir), Mail, Pink Bow | 95 de fã, 12 prováveis, 24 de teste |

## 1. O que faltava e foi feito hoje

- **Formas regionais (40):** Galar 20, Hisui 16, Paldea 4, com tipos, atributos, habilidades, tamanho e aprendizado do Showdown (nível, TM e tutor do Gen 9). Evoluções regional→regional ligadas (Hisui: Growlithe→Arcanine com Fire Stone, Voltorb→Electrode com Leaf Stone, Zorua→Zoroark nível 30, Sliggoo→Goodra nível 50 na chuva; Galar: Ponyta→Rapidash 40, Zigzagoon→Linoone 20, Darumaka→Darmanitan com Ice Stone, Slowpoke→Slowbro com Galarica Cuff e →Slowking com Galarica Wreath). As evoluções que só existem a partir de uma forma regional saíram da forma comum e foram para a regional: Perrserker, Runerigus, Clodsire, Sneasler, Overqwil, Cursola, Sirfetch'd, Obstagoon e Mr. Rime.
- **Formas que seguem o item (21):** Silvally ×17 (Memory), Dialga e Palkia Origin (Adamant Crystal, Lustrous Globe), Zacian e Zamazenta Crowned (Rusted Sword, Rusted Shield). O Giratina Origin passou para o mesmo desenho (Griseous Core). O item não sai da espécie que ele força nem vai para ela.
- **Outras formas:** Ursaluna Bloodmoon, Terapagos Terastal e Stellar (ligadas à Tera Shift e ao Tera), Darmanitan Galar Zen (ligada ao Zen Mode), 12 Totems, Indeedee, Basculegion e Oinkologne fêmeas, Enamorus Therian, Squawkabilly Blue, Yellow e White, Tatsugiri Droopy e Stretchy, Dudunsparce de 3 segmentos, Magearna Original, Basculin White-Striped, Rockruff Dusk, Gimmighoul Roaming, Poltchageist Artisan, Sinistcha Masterpiece, Sinistea e Polteageist Antique, Pichu Spiky-eared, Greninja Bond, Eternatus Eternamax.
- **Megas:** 97 formas e 92 pedras, completas (inclusive Magearna Original e Tatsugiri Droopy/Stretchy).
- **Leppa Berry** (restaura 10 PP) e as **10 bagas da Gen 2** (Berry, Gold, Bitter, Burnt, Ice, Mint, Miracle, Mystery, PRZ Cure, PSN Cure), com Natural Gift e Fling.
- A arte das formas novas é a da espécie base, até chegar a própria (`PENDENCIAS` A11).

> **Quarentena esvaziada (13/10/2026):** a pedido do dono do projeto, **tudo o que estava em `Assets/Data/Conteudo/NaoOficial/` foi apagado, menos o item `AssaultArmor` e a habilidade `Eventide`** (289 assets: as 21 espécies, 164 habilidades e 104 itens). As referências foram limpas: saíram do `CatalogoJogo` e da loja de teste, e 17 espécies oficiais perderam uma entrada de `evolucoes` que apontava para a duplicata do Omega (Azumarill, Tranquill, Cosmog, Cosmoem, os Ultraentes, Urshifu e Zarude Aba). A descrição abaixo é o histórico de 10/10.

> **Quarentena (10/10/2026):** a pedido do dono do projeto, os assets de fã, prováveis de fã e as duplicatas do Omega foram **movidos** (com os `.meta`, então nenhuma referência quebrou) para `Assets/Data/Conteudo/NaoOficial/{Especies,Habilidades,Itens}/{DeFa,ProvavelDeFa,DuplicataDoOmega}`: 1 espécie de fã, 20 duplicatas, 165 habilidades, 93 itens de fã e 12 prováveis. **Ficaram no lugar:** os de classe TESTE (depois, ainda em 10/10, a pedido do dono do projeto, **apagados**: os 56 de teste e as 9 espécies feitas à mão, com todas as referências trocadas pelo equivalente oficial — Poção por Potion, Encomenda por Oak's Parcel, os `g1*` pelas espécies oficiais — e os testes reescritos com conteúdo oficial; as 7 Pokébolas da escada de captura e as duas pedras de forma de teste, sem equivalente oficial, saíram das listas), os 2 de SISTEMA, os 5 a REVISAR, e `PowerRingScroll`/`PulseCannonScroll` (um teste PlayMode os carrega pelo caminho). As Megas de Magearna Original e de Tatsugiri Droopy/Stretchy eram classificadas como duplicata por engano e foram corrigidas para OFICIAL na lista.

**Desde 11/10/2026 esta conferência é automática:** `Pokemon → Showdown → Importador - 1 comparar` compara os números de todo golpe e espécie com o Showdown e lista o que falta ([`../Projeto/FERRAMENTAS_DO_EDITOR.md`](../Projeto/FERRAMENTAS_DO_EDITOR.md) §3). A primeira passada corrigiu 49 números ([`../Projeto/PENDENCIAS.md`](../Projeto/PENDENCIAS.md), item 15).

## 2. O que ainda falta, e o que é preciso

| Falta | O que é preciso |
|---|---|
| ~~Gigantamax~~ | **Feito em 10/10** ([`BATALHA_ATE_O_OFICIAL.md`](BATALHA_ATE_O_OFICIAL.md) §14). Os efeitos dos 10 golpes G-Max que pediam peça nova saíram na Leva 21 (10/10) |
| ~~Ogerpon~~ | **Feito em 10/10** (§14) |
| ~~Maushold de 3~~ | **Feito em 10/10**: forma da de Quatro (o id dela já ia nos saves), com o peso 99/1 na evolução |
| **Evolução por lugar** (Quilava→Typhlosion de Hisui, Dewott→Samurott de Hisui, Petilil→Lilligant de Hisui, Rufflet→Braviary de Hisui, Goomy→Sliggoo de Hisui, Dartrix→Decidueye de Hisui, Bergmite→Avalugg de Hisui, Koffing→Weezing de Galar, Mime Jr.→Mr. Mime de Galar) | **A peça existe desde 10/10** (`ExigeLugar` + `LugarDoMundo`); falta marcar os lugares no mapa e ligar as espécies. Antes: Uma condição de evolução de **região/lugar**: o projeto não tem. Enquanto isso, essas formas aparecem por roteiro, evento ou treinador. |
| ~~Troca de golpe da forma~~ (Zacian Crowned, Zamazenta Crowned) | **Feito**: `EspecieData.golpeDaForma`. |
| **Hidden Power por tipo** (15 entradas do Showdown) | Não faltam: no oficial é um golpe só; o Showdown separa por tipo. |
| **Nomes e numeração** | Eternatus está em `p0889` (o certo é #890); `ZarudeAba` é o Zarude Dada; as formas do Arceus têm nomes de placa ("Arceus(Flame)", "Arceus(Splash)"). Não afetam o jogo; ficam para uma revisão de nomes. |

## 3. O que o projeto tem que não é oficial

Não apaguei nada: a lista é para decisão. Cada linha do TSV tem a classe e o motivo.

### Espécies ([`especies_do_projeto_fora_da_lista_oficial.tsv`](auditoria_oficial/especies_do_projeto_fora_da_lista_oficial.tsv))
- **Fã (1):** `p2018_MarowakAlolan_2052` (Marowak de Johto).
- **Cópias do Omega (23):** a mesma espécie com o mesmo nome e id `10xxx` ou `2xxx`: Azumarill, Shaymin Sky, Cosmog, Cosmoem, Nihilego, Buzzwole, Pheromosa, Xurkitree, Guzzlord, Necrozma, Marshadow, Naganadel, Stakataka, Blacephalon, Urshifu ×2, Zarude ×2, Mothim ×2, Unfezant. O oficial tem uma só de cada. Antes de apagar, falta ver quais roteiros, encontros e treinadores apontam para a cópia e quais para o original.
- **Alola:** as 18 formas de Alola existem como `p2001`–`p2018`, fora da entrada da Pokédex da espécie base. Não faltam, mas estão numeradas à parte e sem "Forma de".

### Habilidades ([`habilidades_do_projeto_fora_da_lista_oficial.tsv`](auditoria_oficial/habilidades_do_projeto_fora_da_lista_oficial.tsv))
- **Fã (165, sendo 153 de fã mesmo e 12 com número oficial):** `h1020`–`h1126` (Divine Inferno, Dragon Roams the World, Shining Diamond…) e `h100002`–`h100060` (Wave Rider, Skater, Thrust…). Pertencem às Megas e formas de fã que já saíram, a Pokémon do Omega ou a habilidades de uma versão anterior do jogo. Falta conferir se alguma espécie ainda usa alguma.
- **Os 12 com número oficial, mas nome do Omega:** `h9`, `h16`, `h22`, `h35`, `h37`, `h46`, `h49`, `h50`, `h59`, `h60`, `h279`, `h280` (Allure, Feedback, Content, Eventide, Permafrost, Scavenger, Conditioning, Grass Aura, Ice Aura, Siphon, Paledysprosium, Reversal Field).
- **Teste (11):** os stubs em português (Copia, Flexibilidade, Graça Serena, Mutação, Olho Composto, Pressagio, Refrigerar, Robustez, Rompe Molde, Teimosia, Travessura).

### Golpes ([`golpes_do_projeto_fora_da_lista_oficial.tsv`](auditoria_oficial/golpes_do_projeto_fora_da_lista_oficial.tsv))
- **Teste (21):** `atkagua`, `atkfogo`, `hidrobomba`, `lancachamas`, `raiosolar`… em português, na pasta raiz de golpes.
- **Grafia ou pontuação diferente (3):** Vice Grip (o oficial é Vise Grip), Sword Dance (o oficial é Swords Dance) e "10 000 000 Volt Thunderbolt".

### Itens ([`itens_do_projeto_fora_da_lista_oficial.tsv`](auditoria_oficial/itens_do_projeto_fora_da_lista_oficial.tsv))
- **Fã (95):** flautas de natureza e de treino (`AdamantFlute`, `AdvancedAttackFlute`, `FlashFlute10`…), sistema de criação do Omega (`*CraftCrystal`, `*CraftSand`), 23 orbes de forma (`FormOrb*`), pergaminhos, `AdvancedLifeOrb`, `RareOrb`, `DayPenaltyOrb`, `DivineAngerOrb`, `MegaGlalie`, `RayquazaMegaStone`, `BondEvolution`, `ChangingShield`, `ChangingSword`, `AssaultArmor`.
- **Prováveis (12):** `AirStone`, `CoronetStone`, `RainbowStone`, `WhitishStone`, `SkyStone`, `HeartStone`, `RockStone`, `ResetStone`, `RefiningStone`, `MossRock`, `FlameFlower`, `CactusFlower`.
- **Teste (24) e sistema (2):** itens com nome em português (`Pocao`, `Antidoto`, `PedraFocal`, `Pokebola 1`–`8`…) e as duas cápsulas do projeto (`CapsulaDeHabilidade`, `SeloDeHabilidade`).
- **São oficiais, só com nome diferente do PokeAPI (451):** os 300 Dynamax Crystals (o PokeAPI chama de "★And458"), os 35 cristais Z em duas versões (mochila e segurado), as bolas do Legends Arceus (prefixo `La`), as Feathers (no projeto, `Wing`) e outros.

## 4. Como refazer

Os arquivos de `auditoria_oficial/` são gerados por um conjunto de scripts descartáveis. Para refazer a auditoria, baixe de novo os arquivos do Showdown e do PokeAPI e compare pelos nomes normalizados. As formas foram criadas por `ConteudoDoShowdown.Formas.cs`, que lê `Docs/Jogo/auditoria_oficial/dados_showdown_formas.tsv` e é idempotente (menu `Pokemon/Showdown/Formas - regionais, Silvally, Origin e Crowned`).
