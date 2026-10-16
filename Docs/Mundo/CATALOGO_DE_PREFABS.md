# Catálogo de prefabs para montar mapa

| | |
|---|---|
| **O que este documento responde** | Que prefabs existem para montar um mapa, onde cada um mora e para que serve |
| **Quando abrir** | Antes de procurar um prefab na janela Project; ao montar uma área nova |
| **Gerado por** | `node tools/gerar-catalogo-de-prefabs.js` (não edite à mão: um teste confere se ele está em dia) |
| **Ver tudo de uma vez** | Cena `Assets/Scenes/Desenvolvimento/GaleriaDePrefabs.unity` (menu **Omega → Prefabs de mapa - criar cena galeria**): todos os prefabs lado a lado, cada um com o nome no chão |
| **Os componentes** (porta, mato, NPC, loja...) | Não são prefabs: estão em [`MONTAGEM_DE_MAPAS.md`](MONTAGEM_DE_MAPAS.md) |
| **Pacotes de terceiros** (árvores, casas, móveis baixados) | [`PACOTES_DE_TERCEIROS.md`](PACOTES_DE_TERCEIROS.md) |

## Como usar

1. Abra a galeria (acima) e ache o que quer pelo nome, ou procure na tabela da seção "Procurando por...".
2. Na janela Project, vá até a pasta indicada e **arraste o prefab para a cena**. Prefab da pasta `Omega/Arte/Mapas/KantoPecas` já vem com o colisor certo (regra única `ColisaoDoMapa`).
3. Interior, ilha ou lugar inteiro (`I17...`, `I18...`) é um prefab **grande**: use no lugar de montar o mesmo à mão.
4. Não edite o prefab original para um uso só: arraste, e use **Prefab → Unpack** ou uma variante.

## Procurando por...

| Quero | Pasta |
|---|---|
| Uma casa, prédio, museu, cassino | `KantoPecas/Prefabs/Predios_e_Casas` |
| Um ginásio ou parte da Liga | `KantoPecas/Prefabs/Ginasios_e_Liga` (ou o lugar inteiro em `Interiores_e_Ilhas`) |
| Árvore, arbusto, flor, capim | `KantoPecas/Prefabs/Vegetacao`, `Omega/Prefabs/Blocos` ou o AurynSky |
| Lago, mar, píer | `KantoPecas/Prefabs/Agua` |
| Penhasco, pedra, montanha | `KantoPecas/Prefabs/Relevo_e_Rochas` |
| Caverna ou passagem subterrânea | `KantoPecas/Prefabs/Cavernas_e_Subterraneo` |
| Chão e paredes simples, bloco a bloco | `Omega/Prefabs/Blocos` |
| Um NPC | `Prefabs/Personagens` |
| Cortar, empurrar ou quebrar algo | `Prefabs/Campo` |
| Um lugar inteiro (Safari, Navel Rock, SS Anne, laboratório) | `KantoPecas/Prefabs/Interiores_e_Ilhas` |

## Peças do Kanto (199 prefabs)

Pasta: `Assets/Omega/Arte/Mapas/KantoPecas/Prefabs`, em subpastas por tipo. Vêm da `Kanto3D`, recortadas em peças (menu **Omega → Kanto - extrair pecas e montar cena nova**).

### Interiores e Ilhas (64)

Lugares inteiros e prontos: `I17...` são interiores (ginásios, Liga, laboratório, SS Anne) e `I18...` as ilhas e áreas especiais (Safari, Navel Rock, Spa). O sufixo `_Raiz` é a raiz do lugar inteiro, não uma peça.

`KantoPecas/Prefabs/Interiores_e_Ilhas`

`I17BikeShop_Raiz` · `I17CasaWarden_Raiz` · `I17Creche_Raiz` · `I17Dojo_Raiz` · `I17EscritorioSafari_Raiz` · `I17FanClub_Raiz` · `I17GinasioCerulean_Raiz` · `I17GinasioFuchsia_Raiz` · `I17GinasioPewter_Raiz` · `I17GinasioViridian_Raiz` · `I17Laboratorio_Raiz` · `I17LigaAgatha_Raiz` · `I17LigaBruno_Raiz` · `I17LigaCampeao_Raiz` · `I17LigaHallDaFama_Raiz` · `I17LigaLance_Raiz` · `I17LigaLorelei_Raiz` · `I17SSAnne_Raiz` · `I18BerryForest_Raiz` · `I18BirthIsland_Raiz` · `I18DescansoSafari1_Raiz` · `I18DescansoSafari2_Raiz` · `I18DescansoSafari3_Raiz` · `I18DescansoSafari4_Raiz` · `I18DescansoSafari5_Raiz` · `I18IlhaBirthIsland_Raiz` · `I18IlhaBosqueMarcado_Raiz` · `I18IlhaFlorestaFrutas_Raiz` · `I18IlhaSafari_Raiz` · `I18IlhaSpaBrasa_Raiz` · `I18IlhasEspumaB1F_Raiz` · `I18IlhasEspumaB2F_Raiz` · `I18IlhasEspumaB3F_Raiz` · `I18IlhasEspumaB4F_Raiz` · `I18IlhasEspumaB5F_Raiz` · `I18MartTres` · `I18NavelRock00_Raiz` · `I18NavelRock36_Raiz` · `I18NavelRock37_Raiz` · `I18NavelRock38_Raiz` · `I18NavelRock39_Raiz` · `I18NavelRock40_Raiz` · `I18NavelRock41_Raiz` · `I18NavelRock42_Raiz` · `I18NavelRock43_Raiz` · `I18NavelRock44_Raiz` · `I18NavelRock45_Raiz` · `I18NavelRock46_Raiz` · `I18NavelRock47_Raiz` · `I18NavelRock48_Raiz` · `I18NavelRock49_Raiz` · `I18NavelRock50_Raiz` · `I18NavelRock51_Raiz` · `I18NavelRock52_Raiz` · `I18NavelRock53_Raiz` · `I18NavelRock54_Raiz` · `I18NavelRock55_Raiz` · `I18PassagemIlhaTres_Raiz` · `I18PatternBush_Raiz` · `I18SafariCentro_Raiz` · `I18SafariLeste_Raiz` · `I18SafariNorte_Raiz` · `I18SafariOeste_Raiz` · `I18SpaBrasa_Raiz`

### Predios e Casas (40)

Fachadas e partes de prédio: casas (`Casa...`), cassino, centros, marts, museu, ruínas e torres. `Telhado`, `Teto`, `Forro` e `Piso` são partes que se escondem ou trocam ao entrar.

`KantoPecas/Prefabs/Predios_e_Casas`

`ArcoPilar` · `CaixaCorreio` · `CasaBatente` · `CasaBillForro` · `CasaBillJanela` · `CasaBillPiso` · `CasaBillTelhado` · `CasaCapeBrinkTelhado` · `CasaCerulean` · `CasaGuardiaoTelhado` · `CasaJogador` · `CasaLavender` · `CasaMrFuji` · `CasaPescadorTelhado` · `CasaPewter` · `CasaResortGorgeousTelhado` · `CasaRota8Telhado` · `CasaSaffron` · `CasaSeteJanela` · `CasaVermilion` · `CasaViridian` · `CassinoRocketForro` · `CassinoRocketTeto` · `CentroCincoIsland` · `CentroDoisIsland` · `LabCinnabarBancada` · `LabCinnabarMaquinaFossil` · `LaboratorioTeto` · `MartQuatroIsland` · `MuseuComputador` · `MuseuEscadaria` · `MuseuEstante` · `MuseuGuarda` · `PredioCeladon` · `RuinaTanobyBatente` · `RuinaTanobyBatente_2` · `RuinaTanobyJanela` · `Toldo` · `TorrePokemonCupula` · `TrainerTowerTerracoGrade`

### Ginasios e Liga (13)

Peças dos ginásios, da Liga Pokémon e do dojo: pisos, tetos, estátuas, divisórias.

`KantoPecas/Prefabs/Ginasios_e_Liga`

`DojoLutaTeto` · `GinasioCeladonCaminho` · `GinasioCeladonEstatua` · `GinasioCeladonPiso` · `GinasioCinnabarDivisoria` · `GinasioCinnabarLider` · `GinasioCinnabarPortaQuiz` · `GinasioCinnabarTeto` · `GinasioSaffronSubTeleporte` · `LigaAgathaPiso` · `LigaAgathaTeto` · `LigaBrunoPiso` · `LigaPokemonTeto`

### Cavernas e Subterraneo (14)

Cavernas, trilhas e passagens subterrâneas (Diglett, Cerulean, Altering, Icefall, Rota 6).

`KantoPecas/Prefabs/Cavernas_e_Subterraneo`

`AlteringCaveDegrau` · `AlteringCaveGuarda` · `AlteringCaveLaje` · `AlteringCaveRocha` · `CeruleanCaveLaje` · `DiglettCaveTrilha` · `DottedHoleLaje` · `EntradaCaverna` · `EntradaSubterraneoRota6` · `IcefallCaveLaje` · `UndergroundPath56Guarda` · `UndergroundPath56Guarda_001` · `UndergroundPath56Guarda_002` · `UndergroundPath56Guarda_003`

### Agua (19)

Água, lagoas (`Lagoa_2` a `Lagoa_12` são formatos diferentes), borda, píer, fundo do mar, vitórias-régias.

`KantoPecas/Prefabs/Agua`

`Agua` · `BordaLago` · `EmendaPier` · `FundoMar` · `Lagoa` · `Lagoa_10` · `Lagoa_11` · `Lagoa_12` · `Lagoa_2` · `Lagoa_3` · `Lagoa_4` · `Lagoa_5` · `Lagoa_6` · `Lagoa_7` · `Lagoa_8` · `Lagoa_9` · `Lily_large` · `Lily_small` · `MarBase`

### Relevo e Rochas (27)

Terreno, penhascos (`Cliff_...`), rochas e pedras, muros e a montanha de fundo.

`KantoPecas/Prefabs/Relevo_e_Rochas`

`Cliff_corner_rock` · `Cliff_half_rock` · `Cliff_large_rock` · `Cliff_rock` · `FundoMontanha` · `Muro` · `MuroBase` · `Ore_gigante` · `Pedra` · `Rock_largea` · `Rock_largeb` · `Rock_largec` · `Rock_largee` · `Rock_smalla` · `Rock_smallb` · `Rock_smallflata` · `Rock_talla` · `Rock_tallb` · `Rock_tallc` · `Rock_tallf` · `Stone_largeb` · `Stone_larged` · `Stone_smalla` · `Stone_smallc` · `Stone_smalle` · `TerrenoKanto` · `rock_smallFlatB`

### Vegetacao (20)

Árvores, arbustos, flores, capim, cogumelos, troncos e o mato.

`KantoPecas/Prefabs/Vegetacao`

`Arbusto_003` · `Arvore` · `Arvore_2` · `Arvore_3` · `Arvore_4` · `Arvore_5` · `Flor` · `Grass_large` · `Log` · `Log_large` · `Mato` · `Mushroom_redgroup` · `Mushroom_tangroup` · `Plant_flattall` · `Sebe` · `Stump_oldtall` · `Tronco` · `plant_bushSmall` · `tree_palmBend` · `tree_palmTall`

### Auxiliares (2)

Referências para montar (câmera aérea, boneco de 1,75 m para conferir escala). Não vão para o jogo.

`KantoPecas/Prefabs/Auxiliares`

`CameraAerea_Alvo` · `ReferenciaJogador_1m75`

## Blocos genéricos (10)

Blocos de 1 unidade para montar terreno à mão: grama, água, areia, terra, caminho, mato alto, parede, pedra, árvore, degrau. Os mais simples e os mais fáceis de combinar.

`Assets/Omega/Prefabs/Blocos`

`BlocoAgua` · `BlocoAreia` · `BlocoArvore` · `BlocoCaminho` · `BlocoDegrau` · `BlocoGrama` · `BlocoMatoAlto` · `BlocoParede` · `BlocoPedra` · `BlocoTerra`

## Obstáculos de campo (3)

Árvore que se corta, pedra de Força, rocha que se quebra. Já trazem o colisor sólido e o gatilho `Interagivel`; só falta a MO, a insígnia e o golpe (ver `MONTAGEM_DE_MAPAS.md` R9).

`Assets/Prefabs/Campo`

`ArvoreCortavel` · `PedraDeForca` · `RochaQuebravel`

## Corpos de NPC (44)

O corpo de cada personagem (líderes, Liga, rival, comuns). O componente de fala vai num filho `Interacao` com a tag `Interagivel` (R4).

`Assets/Prefabs/Personagens`

`Agatha` · `Blaine` · `Brock` · `Bruno` · `CacadorDeInsetos` · `Campeao` · `Carvalho` · `Cavalheiro` · `Cientista` · `EnfermeiraJoy` · `Erika` · `FaixaPreta` · `Garota` · `Garoto` · `Giovanni` · `Idoso` · `Koga` · `Lance` · `Lorelei` · `LtSurge` · `Mae` · `Marinheiro` · `Medium` · `MeninoPequeno` · `Misty` · `MocaDaPraia` · `Montanhista` · `Morador1` · `Morador2` · `Morador3` · `Morador4` · `Motoqueiro` · `Nadador` · `Nadadora` · `Passaro` · `Pescador` · `Rival` · `RocketRecruta` · `RocketRecrutaF` · `Sabrina` · `Supernerd` · `TreinadorAs` · `TreinadoraAs` · `Vendedor`

## Professores (tutores de golpe) (14)

Os 14 professores prontos, já com o golpe que ensinam.

`Assets/Data/Mundo/Professores`

`Professor_Ancestral` · `Professor_Chamas` · `Professor_Dojo` · `Professor_Espiritos` · `Professor_ForcaBruta` · `Professor_Fundamentos` · `Professor_Graca` · `Professor_Imitacao` · `Professor_Iniciais` · `Professor_Instintos` · `Professor_Mente` · `Professor_Punhos` · `Professor_Sono` · `Professor_Vendaval`

## Cachoeira (1)

Efeito visual de cachoeira.

`Assets/Omega/Visual/Cachoeira`

`RespingoCachoeira`

## Vegetação do AurynSky (Forest Pack) (45)

Pacote de terceiros de floresta (único versionado em `ThirdParty`). Árvores, pontes, barris, tochas, fogo. Os prefabs `Red` e `Blue` são variações de cor.

`Assets/ThirdParty/AurynSky/Forest Pack/Prefabs`

`Fire` · `ForestBarrel` · `ForestBlock01` · `ForestBlock02` · `ForestBridgeRegular` · `ForestBridgeWide` · `ForestCastleWindow` · `ForestCrate` · `ForestGrass01` · `ForestGrass02` · `ForestGround01` · `ForestGround02` · `ForestGroundDirt` · `ForestLadder` · `ForestTreeAppleShort` · `ForestTreeAppleTall` · `ForestTreeDShort` · `ForestTreeDTall` · `ForestTreePineShort` · `ForestTreePineShort2` · `ForestTreePineTall` · `ForestTreePineTall2` · `ForestBanner_Blue` · `ForestBrazzierBlue` · `ForestCastleDoorL_Blue` · `ForestCastleDoorR_Blue` · `ForestCastle_Blue` · `ForestCatapult_Blue` · `ForestFlag_Blue` · `ForestTower_Blue` · `ForestBanner_Red` · `ForestBrazzierRed` · `ForestCastleDoorL_Red` · `ForestCastleDoorR_Red` · `ForestCastle_Red` · `ForestCatapult_Red` · `ForestFlag_Red` · `ForestTower_Red` · `RiverQuadCombined` · `RiverQuadSolo` · `Smoke` · `WaterFall` · `WaterPixiesSolo` · `WaterQuadPixies` · `WaterQuadSolo`

## Nomes: como ler

- `I17...` e `I18...`: lugares inteiros (interiores e ilhas). `..._Raiz` é a raiz do lugar.
- `Telhado`, `Teto`, `Forro`: parte de cima de um prédio (some ao entrar). `Piso`: chão do interior.
- `Lagoa_2`, `Arvore_3`: formatos diferentes do mesmo tipo, numerados.
- Nomes em inglês (`Cliff_rock`, `Grass_large`, `Rock_smalla`) vêm de pacotes de terceiros usados na `Kanto3D`.
