# Pacotes de terceiros

| | |
|---|---|
| **O que este documento responde** | Que pacotes de arte de terceiros o projeto tem, de onde vieram, o que há em cada um e a licença |
| **Quando abrir** | Antes de usar uma peça de fora, ou de mexer em `Assets/ThirdParty/` |
| **Onde ficam** | `Assets/ThirdParty/<pacote>/`. **Fora do Git** (`.gitignore`), exceto o AurynSky |
| **Organizado em** | 13/10/2026, a pedido de Higor. Os 14 arquivos de `Downloads/alguns assets` foram extraídos aqui; os três pacotes que já estavam importados (Holotna, Idyllic Fantasy Nature, MaxiBrut) e o AurynSky foram movidos para cá |

## Por que fora do Git

Pesam centenas de MB (o repositório já tem 950 MB de arte do Omega em LFS), têm licença própria e podem ser baixados de novo na origem. Um deles, o **Mapleford** (vila europeia), **proíbe redistribuir os arquivos-fonte**: versionar num repositório compartilhado seria arriscado. Nada do jogo aponta para estas pastas, **exceto o AurynSky** (por isso ele é o único versionado).

**Para usar uma peça:** copie o prefab (ou o FBX) para `Assets/Prefabs/` ou para a pasta do mapa e ligue por lá. Uma cena que cita um pacote daqui abre com referências quebradas em outra máquina.

## Os pacotes

| Pacote | Veio de | O que tem | Licença | Tamanho | No Git |
|---|---|---|---|---|---|
| `AurynSky` | Asset Store (AurynSky) | Floresta: árvores, pontes, barris, tochas, fogo (Forest Pack, variações Red e Blue). | não conferida | 8 MB (102 arquivos) | sim (3 cenas e prefabs do projeto o usam) |
| `BrokenVector_LowPoly_Trees` | Models (1).zip e Demo.zip (+ .blend em ArteOriginal) | Low Poly Tree Pack e demo do Free Low Poly Pack, da Broken Vector (árvores, cerca, pedra, .dae). | Asset Store (Broken Vector): conferir os termos na origem | 3 MB (52 arquivos) | não |
| `FreeTree_LowPoly_Pack` | free-tree-3d-low-poly-pack.zip | Pacote de árvores low poly (FBX). | licença Craftpix (o License.txt só aponta para craftpix.net/file-licenses) | 1 MB (24 arquivos) | não |
| `Holotna` | importado antes no Unity | Montanha (Mountain): materiais, modelos, prefabs, cena de demonstração; versões BRP, URP e HDRP. | não conferida | 487 MB (269 arquivos) | não |
| `Idyllic Fantasy Nature` | importado antes no Unity | Natureza de fantasia: modelos, prefabs, terreno, shader, scripts e documentação em PDF. | não conferida (há PDF de documentação no pacote) | 269 MB (432 arquivos) | não |
| `KayKit_Furniture_Bits` | KayKit_Furniture_Bits_1.0_FREE.zip | Móveis low poly: camas, poltronas, livros, mesas (FBX, versão "unity" e outras). | CC0 (arquivo License.txt; crédito a Kay Lousberg opcional) | 6 MB (328 arquivos) | não |
| `LowPolyNaturePackLite` | LowPolyNaturePackLiteUnityBRPandURP.unitypackage | Natureza low poly "Lite": 31 prefabs e 31 FBX, para BRP e URP. | Asset Store: conferir os termos na origem | 3 MB (68 arquivos) | não |
| `LowPoly_Environment_Pack` | LowPoly Environment Pack.rar | Ambiente low poly: terreno, árvores, arbustos, capim, troncos (FBX). | não identificada | 1 MB (31 arquivos) | não |
| `LowPoly_European_Village` | Low_Poly_European_Village_v1.1.0.zip | Vila europeia low poly "Mapleford": GLB extraído; a versão Blender ficou em ArteOriginal/ThirdParty. | Mapleford Asset License 1.0: uso comercial livre, **proibido redistribuir os arquivos-fonte** (por isso não vai para o Git) | 143 MB (248 arquivos) | não |
| `LowPoly_Props_Paletas` | Models.zip e Palettes.zip | Props .dae (caixas, barris, cestas, sacos, rack) e as 5 paletas de cor (azul, verde, roxo, vermelho, amarelo). | não identificada | 1 MB (40 arquivos) | não |
| `MaxiBrut` | importado antes no Unity | Pontes (BridgesPack): prefabs, arte e documentação. | não conferida | 304 MB (75 arquivos) | não |
| `Modular_House_Collection` | modular_house_collection.zip | Casa modular: props e peças de interior (banheiro etc., OBJ e MTL). | não identificada (o pacote não traz licença) | 9 MB (240 arquivos) | não |
| `Modular_Terrain_Collection` | modular_terrain_collection.zip | Terreno modular: praia, píer, props (OBJ). | não identificada (o pacote não traz licença) | 5 MB (306 arquivos) | não |
| `StylizedTrees_URP` | StylizedTreesURP.unitypackage | Árvores estilizadas: 25 prefabs, 24 FBX, shader graph das folhas (URP). | Asset Store: conferir os termos na origem | 8 MB (76 arquivos) | não |
| `Stylized_Nature_MegaKit` | Stylized Nature MegaKit[Standard].zip | Natureza estilizada (Quaternius), versão grátis com 68 dos 116 modelos: FBX, OBJ, glTF, texturas. | CC0 (License_Standard.txt) | 116 MB (454 arquivos) | não |

## Tipos de arquivo em cada pacote

- **AurynSky**: .prefab 45, .fbx 34, .mat 7, .cs 4, .controller 3, .png 3
- **BrokenVector_LowPoly_Trees**: .dae 42, .png 8, .txt 2
- **FreeTree_LowPoly_Pack**: .fbx 21, .txt 2, .png 1 · licença no pacote: License.txt
- **Holotna**: .png 146, .exr 60, .mat 15, .prefab 15, .asset 11, .fbx 10
- **Idyllic Fantasy Nature**: .prefab 157, .fbx 94, .png 83, .mat 65, .cs 8, .asset 7
- **KayKit_Furniture_Bits**: .fbx 106, .bin 53, .gltf 53, .mtl 53, .obj 53, .png 7 · licença no pacote: License.txt
- **LowPolyNaturePackLite**: .fbx 31, .prefab 31, .mat 2, .unity 2, .unitypackage 1, .png 1
- **LowPoly_Environment_Pack**: .fbx 31
- **LowPoly_European_Village**: .png 119, .glb 111, .json 7, .md 5, .txt 4, .blend 2 · licença no pacote: LICENSE.txt
- **LowPoly_Props_Paletas**: .dae 35, .png 5
- **MaxiBrut**: .png 28, .prefab 13, .mat 12, .fbx 11, .shadersubgraph 4, .shadergraph 3
- **Modular_House_Collection**: .mtl 120, .obj 120
- **Modular_Terrain_Collection**: .obj 305, .mtl 1
- **StylizedTrees_URP**: .prefab 25, .fbx 24, .png 14, .mat 10, .unity 1, .asset 1
- **Stylized_Nature_MegaKit**: .fbx 136, .bin 68, .gltf 68, .mtl 68, .obj 68, .png 40 · licença no pacote: License_Standard.txt

## Fontes que o Unity não importa

Ficam em `ArteOriginal/ThirdParty/` (fora de `Assets`, fora do Git): `BrokenVector_LowPolyTreePack.blend` e `LowPoly_European_Village_mapleford-low-poly-village-blender-v1.1.0.zip` (a versão Blender da vila). O Unity só importa `.blend` se o Blender estiver instalado.

## Como acrescentar um pacote novo

1. Extraia para `Assets/ThirdParty/<Nome_Do_Pacote>/` (sem espaços; mantenha o arquivo de licença dentro).
2. Acrescente a linha na tabela acima (origem, o que tem, **licença**) em `Docs/Mundo/PACOTES_DE_TERCEIROS.md`.
3. Não ligue cena nem prefab do jogo ao pacote; copie a peça (ver acima).
