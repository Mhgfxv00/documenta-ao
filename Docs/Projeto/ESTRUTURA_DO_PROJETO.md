# Estrutura do projeto: onde mora cada coisa

| | |
|---|---|
| **O que este documento responde** | Que pasta existe, o que vai em cada uma, e onde pôr um arquivo novo |
| **Quando abrir** | Ao criar um asset, um prefab, uma cena ou um dado; ao procurar um arquivo |
| **Organizado em** | 13/10/2026, a pedido de Higor (cenas, `Data`, arquivos soltos, prefabs de mapa e pacotes de terceiros). Os arquivos foram **movidos com o `.meta`**, então nenhuma referência por GUID quebrou; os caminhos escritos em código, testes, cenas e docs foram reescritos |
| **O que ele NÃO é** | Referência de sistemas de código ([`../Jogo/REFERENCIA_DE_SISTEMAS.md`](../Jogo/REFERENCIA_DE_SISTEMAS.md)) |

## A raiz do repositório

| Pasta | O que é |
|---|---|
| `Assets/` | Tudo o que o Unity importa |
| `Docs/` | Toda a documentação (fora de `Assets/`, para não virar asset) |
| `tools/` | Scripts de apoio em Node: teste de conformidade com o Showdown, exportador, gerador do catálogo de prefabs |
| `ArteOriginal/` | Fontes de arte que o Unity **não** importa: `PersonagemOmega/` (zip e imagens do Mixamo, versionado) e `ThirdParty/` (`.blend`, fora do Git) |
| `Packages/`, `ProjectSettings/` | Do Unity |
| `Library/`, `Temp/`, `Logs/`, `UserSettings/`, `*.csproj` | Gerados; ignorados pelo Git |

## `Assets/`

| Pasta | O que vai aqui |
|---|---|
| `Scripts/` | O jogo: `Data`, `Systems`, `World`, `UI`, `Roteiro`, `Interacao`, mais `Editor/` (ferramentas e testes de EditMode) e `Tests/` (PlayMode) |
| `Omega/` | O outro lado do grupo: UI Toolkit, criador de personagem, visual, ferramentas de mapa. Tem caminhos fixos no código: não renomear sem checar |
| `Data/` | O conteúdo, em três famílias (abaixo) |
| `Scenes/` | As cenas, por papel (abaixo) |
| `Prefabs/` | Prefabs do jogo: `Bootstrap` (sistemas, jogador), `Campo` (obstáculos), `Personagens` (corpos de NPC), `UI`. `Characters`, `Creatures` e `Gameplay` são reservas (só têm `LEIAME.md`) |
| `Art/`, `Animations/` | Arte própria. `Art/Environment/` guarda os materiais básicos (`MateriaisBasicos/`) e os terrenos (`Terrenos/`) das cenas de desenvolvimento |
| `UI/` | Ícones e prefabs de UI (uGUI) |
| `Settings/` | URP e perfis de volume |
| `ThirdParty/` | **Pacotes de terceiros, fora do Git** (exceto o AurynSky): [`../Mundo/PACOTES_DE_TERCEIROS.md`](../Mundo/PACOTES_DE_TERCEIROS.md) |
| `TextMesh Pro/`, `UI Toolkit/` | Do Unity |

### `Assets/Data/`

| Família | Conteúdo |
|---|---|
| `Conteudo/` | As regras do jogo como dado: `Especies`, `Golpes`, `Habilidades`, `Itens`, `HeldItems`, `Status`, `Tipagem` (e `TabelaTipos`), `Climas`, `Terreno`, `EfeitosDeCampo`, `EfeitosHabilidade`, `Caracteristicas`, `NaoOficial` (a quarentena: só `AssaultArmor` e `Eventide`) |
| `Mundo/` | O que se monta no mapa: `Encontros`, `TabelasDeEncontro`, `Treinadores`, `Roteiros`, `Lojas` (e `LojaInicial`), `Interacoes`, `Personagens`, `Professores` |
| `Sistema/` | O que o jogo precisa para ligar: `CatalogoJogo` (o índice de ids), `CatalogoModelos3D`, `PerfisDeIA` |

### `Assets/Scenes/`

| Pasta | Cenas |
|---|---|
| `Jogo/` | `Bootstrap` (a cena inicial, dona do jogador, da câmera e dos sistemas), `OmegaMenu` |
| `Mundo/` | `KantoMulticena/` (`KantoBase` e as `Areas/`): o mapa do jogo |
| `Interiores/` | `Casa`, `Quarto` |
| `Desenvolvimento/` | `DevScene`, `OmegaDev`, `OmegaBlocos`, `OmegaMapa`, `ArenaExemplo` e a `GaleriaDePrefabs` |
| `Legado/` | `Kanto3D` e `KantoPecas` (as cenas de 89 e 74 MB: **não abrir nem commitar**) |

## Prefabs para montar mapa

Estão em seis lugares, e o índice de todos é o [`../Mundo/CATALOGO_DE_PREFABS.md`](../Mundo/CATALOGO_DE_PREFABS.md). Para **ver** todos de uma vez: cena `Assets/Scenes/Desenvolvimento/GaleriaDePrefabs.unity` (menu **Omega → Prefabs de mapa - criar cena galeria**).

As 199 peças do Kanto (`Assets/Omega/Arte/Mapas/KantoPecas/Prefabs/`) estão em subpastas por tipo: `Interiores_e_Ilhas`, `Predios_e_Casas`, `Ginasios_e_Liga`, `Cavernas_e_Subterraneo`, `Agua`, `Relevo_e_Rochas`, `Vegetacao`, `Auxiliares`. **Peça nova:** ponha numa dessas subpastas e rode `node tools/gerar-catalogo-de-prefabs.js` (um teste confere o índice).

## Onde pôr um arquivo novo

| Tenho... | Vai para |
|---|---|
| Uma espécie, golpe, habilidade, item, status | `Assets/Data/Conteudo/<tipo>/` (e registre no `CatalogoJogo`) |
| Um treinador, loja, roteiro, tabela de encontro | `Assets/Data/Mundo/<tipo>/` |
| Uma cena de mapa do jogo | `Assets/Scenes/Mundo/` (e no Build Settings) |
| Uma cena de teste ou experimento | `Assets/Scenes/Desenvolvimento/` |
| Um prefab de peça de mapa | `Assets/Omega/Arte/Mapas/KantoPecas/Prefabs/<tipo>/` |
| Um NPC | `Assets/Prefabs/Personagens/` |
| Um pacote baixado | `Assets/ThirdParty/<Nome_Sem_Espacos>/` e uma linha em `PACOTES_DE_TERCEIROS.md` |
| Um `.blend`, `.psd`, `.zip` de fonte | `ArteOriginal/` (nunca em `Assets/`: o Unity tenta importar) |
| Um documento | `Docs/` (nunca em `Assets/`) |
