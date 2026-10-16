# Rede e multiplayer — o que existe, o que falta, o que já foi decidido

| | |
|---|---|
| **O que este documento responde** | Até onde o projeto está preparado para jogar em rede, o que falta para a primeira partida real, e as decisões de arquitetura que valem antes de escolher um pacote |
| **Quando abrir** | Antes de escolher o pacote de rede, de escrever o primeiro `NetworkBehaviour`, ou de mexer em `SistemaBatalha.Rede.cs` / `BatalhaEmRede.cs` |
| **Estado** | **Não existe rede.** Nenhum pacote de netcode no `Packages/manifest.json` (o `com.unity.multiplayer.center` é só a janela de boas-vindas da Unity). O que existe são as **costuras** na batalha, todas com teste |
| **Alvo declarado (04/09)** | Mundo grande, várias áreas conectadas, **2 a 15–18 jogadores**. Com 18, *relevancy / interest management* deixa de ser opcional |
| **Conferido em** | 09/10/2026 no código (nomes de tipo e arquivo). As análises de 03 a 06/09 que sustentam as seções 3 e 4 estão em [`../Historico/AUDITORIA_ORGANIZACAO.md`](../Historico/AUDITORIA_ORGANIZACAO.md) §2 e §6 e **não foram refeitas** |

São duas frentes independentes: **rede de batalha** (turnos, autoridade, replay) e **co-presença no mundo** (avatares, estado por jogador, [`CO_PRESENCA_NO_MUNDO.md`](CO_PRESENCA_NO_MUNDO.md)).

---

## 1. O que já está pronto (rede de batalha)

| Peça | Onde | O que garante |
|---|---|---|
| **Ação como dado** (`AcaoDeRede`, `PostoDeBatalha`) | `Systems/BatalhaEmRede.cs` | O cliente manda a **escolha** ("golpe 2 no posto 3"), nunca o resultado. Todo caminho de escolha passa por `SistemaBatalha.Registrar`, que traduz para cá: **ação que não vira dado não acontece**, e um verbo novo de turno que esquecer de ser representável quebra na hora, não no dia da rede |
| **Autoridade** (`IArbitroDeBatalha`, `VeredictoDeAcao`: `Recusada` / `Aceita` / `Pendente`) | idem | Um ponto único de recusa. `Submeter(acao)` dá o veredicto de agora; `Consultar()` colhe o que faltava, uma vez por quadro. É por consulta e não por callback para a resposta da rede não mexer no estado no meio de um turno. `Pendente` devolve `true` à UI (a escolha foi feita; o menu fecha e aguarda); recusa tardia reemite `OnPedirAcao` |
| **Ritmo separado da regra** (`IRitmoDeBatalha`) | `Systems/RitmoDeBatalha.cs` | A regra não espera tempo; quem assiste escolhe o ritmo |
| **Registro reproduzível** (`RegistroDaBatalha`, `ReprodutorDeBatalha`) | `Systems/RegistroDaBatalha.cs` | Uma partida é estado inicial + semente + lista de ações aceitas. O registro leva a **narração** (falas, início, fim, troca, pedido de ação) |
| **Foto do estado** (`EstadoDeBatalhaSerializado`, `SistemaBatalha.Fotografar()`) | `Systems/SistemaBatalha.Rede.cs` | Reaproveita `SavePokemon` (um formato só) e, desde 13/10, leva também a luta de cada Pokémon (`SaveDeLuta`, `TradutorDeLuta`) e o estado da batalha; `SistemaBatalha.Restaurar` reconstrói uma batalha dela (R3/R4). Formato 2 |
| **Sorteio semeado por batalha** (`sorteioDaBatalha`, `SemearProximaBatalha`) | `Systems/Sorteio.cs`, `SistemaBatalha` | Mesma semente + mesmas ações = mesma batalha. Regra e lista de combinados: [`../Jogo/REFERENCIA_DE_SISTEMAS.md`](../Jogo/REFERENCIA_DE_SISTEMAS.md) "O acaso" |
| **Prazo para a UI local** | `SistemaBatalha.PosBatalha.cs` (`QuadrosDeEsperaDaEscolha` = 3600) | Quem aprende golpe e não responde (painel não montado, botão sem listener) não trava a batalha: estourado o prazo, não aprende e segue, com `LogError` |
| **Dono único do input** (`EntradaJogador`, `EscutaDeInput`) | `World/` | Cada jogador terá o próprio mapa de input; ninguém cria `new Inputs()` |

Testes: `BatalhaEmRedeTests`, `ReplayDeBatalhaTests`, `AcasoDaBatalhaTests`, `AcasoGlobalTests` (este último trava o uso do sorteio global).

---

## 2. O que falta para a primeira partida em rede

| # | O quê | Por quê | Tamanho |
|---|---|---|---|
| R1 | **Escolher o pacote de transporte** (Netcode for GameObjects, Mirror, Fish-Net, Photon…) | Nada começa sem isto. A escolha é do grupo ([`../Projeto/DECISOES.md`](../Projeto/DECISOES.md) P13). O método, os candidatos e o spike estão em [`ESCOLHA_DO_PACOTE_DE_REDE.md`](ESCOLHA_DO_PACOTE_DE_REDE.md) | decisão |
| R2 | **`ArbitroRemoto`**: um `IArbitroDeBatalha` que manda a ação ao dono da partida e responde `Pendente` até ele confirmar | É o outro lado do fio; a interface já aceita a espera | M |
| R3 | **FEITO em 13/10** (e, no mesmo dia, o D3: várias batalhas simultâneas no mesmo processo, ver DECISOES T57). ~~Repor a foto~~: `SistemaBatalha.Restaurar(foto, dados, catalogo, perfilDeIa)` reconstrói a batalha e segue **no início da escolha do turno** em que a foto foi tirada (`NoInicioDaEscolha`). Provado por `Retomada_*` (`ConformidadeDeFluxoTests.Retomada.cs`): a batalha restaurada joga o resto igual à original, falas e foto a cada turno, em 338 + 1.294 + 1.294 cenários | G |
| R4 | **FEITO em 13/10.** ~~A foto precisa guardar o que só existe dentro da luta~~: `SaveDeLuta` por Pokémon (estágios, trava, forma e tipo assumidos, habilidade copiada, item desativado, substituto, proteções, Dynamax, Tera, usos de gatilho, status com golpe vinculado, contadores de batalha, a memória inteira) e, no nível da batalha, fuga, recompensa, inventário do adversário, mecânicas já usadas por lado, terreno e clima forte, golpes revelados à IA, ataques atrasados e o acaso (semente + consumos). Limites em [`O_QUE_SINCRONIZA.md`](O_QUE_SINCRONIZA.md) §5 | M |
| R5 | **O registro precisa carregar estado, não só narração** | `TipoDeAcontecimento.AtualizouStatus` está **declarado e nunca escrito** (conferido 09/10: uma ocorrência, na declaração). Um cliente remoto não consegue desenhar uma barra de HP a partir do registro | M |
| R6 | **FEITO em 13/10 (a metade que faltava era usar a foto).** ~~O estado inicial viaja junto~~: A semente reproduz as **rolagens**, não quem entrou em campo nem com quais IV e natureza (`Pokemon.Criar` sorteia do `Sorteio` global, de propósito: é criação, não batalha). A foto já carrega isso; falta o outro lado usar | M |
| R7 | **Delta em vez de foto inteira** | A foto cabe para abrir/retomar; para sincronizar turno a turno é desperdício. O candidato natural é o `RegistroDaBatalha` enriquecido (R5). Ids são `string`: irrelevante por turno, caro em alta frequência | M |
| R8 | **Prazo com padrão para o cliente remoto que não responde** | O prazo existente cobre só a UI local de aprender golpe. Um cliente remoto que cai no meio de qualquer escolha precisa de regra (ação padrão? derrota?) | M |
| R9 | **Metade de apresentação nos efeitos** | `EfeitoDeBatalha` só publica evento. No Boss Room (sample oficial da Unity) cada `Action` tem `OnStart` (servidor) e `OnStartClient` (cliente). Um gancho **nomeado** de apresentação fecha a porta para regra vazar para o lado do desenho e resolve "a quem a animação se liga" | M |
| R10 | **FEITO em 12/10: [`O_QUE_SINCRONIZA.md`](O_QUE_SINCRONIZA.md).** ~~Escrever "o que sincroniza" antes do primeiro `NetworkBehaviour`~~ | Sincroniza: posição/rotação, estado de criatura, HP, ação escolhida. **Reconstrói-se localmente:** partícula, efeito de câmera, animação secundária. Sem isto escrito o impulso natural é sincronizar o `transform` de tudo | P (documento) |
| R11 | **Relevancy / interest management** | Com 18 jogadores, cada cliente receberia 17 outros mais o mundo inteiro. As áreas já são cenas separadas (multicena) e o `SistemaTransicao` é o dono único da troca: é o começo natural | G |

**Teste de determinismo entre máquinas:** a batalha usa ponto fixo 4096 para encadear multiplicadores e sorteio semeado, e, desde 10/10/2026, o dano é calculado em inteiros na ordem do cartucho; o que ainda usa `float` é a estimativa da IA e partes de efeitos (ver [`BATALHA_BARRAMENTO_E_DETERMINISMO.md`](BATALHA_BARRAMENTO_E_DETERMINISMO.md)). PvP entre plataformas diferentes só é garantido se isso for revisto.

---

## 3. Decisões de arquitetura já tomadas

1. **Determinismo fica** (04 a 06/09). Serve a teste (flake de 5% vira reproduzível), a bug (o anexo vira teste) e a rede (sem reenviar resultado). Não é mecânica de jogador.
2. **Autoridade por consulta, não por callback** (06/09; `IArbitroDeBatalha.Consultar`). A resposta da rede mexe no estado em um ponto conhecido do quadro.
3. **`TipoAcaoBatalha` é enum fechado**: Golpe, Troca, Item, Fuga. Mega, Z, Dynamax e Tera chegaram como **modificador da ação de golpe**, não como verbo novo ([`../Projeto/DECISOES.md`](../Projeto/DECISOES.md)).
4. **O cliente manda escolha, nunca resultado.**
5. **Predição do jogador mora na ação**, não num sistema à parte (padrão do Boss Room, `AnticipateActionClient`). Vale para movimento em rede, não para batalha por turnos (o turno inteiro é uma resposta só).
6. **Pooling só depois de medir** (os 12 `Instantiate` do runtime são botão de lista de UI, não por quadro).

---

## 4. Para a co-presença no mundo

Ver [`CO_PRESENCA_NO_MUNDO.md`](CO_PRESENCA_NO_MUNDO.md). Resumo do que impede mais de um jogador hoje: 15 `FindAnyObjectByType<DadosJogador>()` no código de jogo, `GerenciadorEstados` como singleton de cena (decide o controle de **todos**), `ObjetoPersistente` com uma instância por chave, e `SistemaTransicao` com `LoadSceneMode.Single` em torno de um jogador cacheado.

## 5. Linha de base de desempenho

Não há **nenhuma medição de build** nem meta numérica de rede (largura de banda, pacotes por segundo, tick). Medir cliente (CPU/GPU/memória/GC), servidor (tick) e rede separadamente é o item 0 antes de escolher técnica. Ver [`../Mundo/MAPAS_E_DESEMPENHO.md`](../Mundo/MAPAS_E_DESEMPENHO.md) e PENDENCIAS O11.
