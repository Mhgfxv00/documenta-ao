# Pendências — o que falta fazer

| | |
|---|---|
| **O que este documento responde** | O que está **aberto**, com ID, esforço e dono. É o backlog vivo do projeto inteiro, menos o que tem plano próprio (tabela abaixo) |
| **Quando abrir** | Ao escolher o que fazer a seguir; ao dar um item por resolvido |
| **O que ele NÃO é** | Situação do projeto ([`ESTADO_DO_PROJETO.md`](ESTADO_DO_PROJETO.md)), decisões ([`DECISOES.md`](DECISOES.md)) nem histórico (o que foi fechado mora em [`../Historico/PENDENCIAS_FECHADAS.md`](../Historico/PENDENCIAS_FECHADAS.md)) |
| **Reescrito em** | 09/10/2026. Antes, o arquivo misturava backlog, diário de levas e análises; o que estava fechado ou era análise saiu para o histórico **sem edição**. Os IDs foram mantidos (o código e os documentos os citam) |
| **Como manter** | Resolveu um item: **apague a linha daqui e acrescente a explicação em `PENDENCIAS_FECHADAS.md`, no mesmo commit** do código. Item novo: ID novo (próximo número livre da faixa). Número sem recorte (data, cena, máquina) não entra |

**Esforço:** P = um dia ou menos · M = alguns dias · G = uma semana ou mais.
**Dono:** *Higor* (batalha, IA, regras, save — [`COMO_TRABALHAR.md`](COMO_TRABALHAR.md) §1) · *Mapa* (klz e grupo) · *UI* (outro lado do grupo) · *Conteúdo* (quem editar dados) · *Dev* (qualquer pessoa que mexa em código) · *Grupo* (decisão coletiva).

## 0. Onde mora cada backlog

Para não ter duas listas que divergem, o que tem plano próprio **não** é repetido aqui:

| Assunto | Onde está o que falta |
|---|---|
| Batalha: golpes, habilidades, itens, mecânicas de formato, IA de batalha | [`../Jogo/BATALHA_ATE_O_OFICIAL.md`](../Jogo/BATALHA_ATE_O_OFICIAL.md) §4 |
| Conteúdo contra o oficial (espécies, itens, golpes, habilidades) | [`../Jogo/AUDITORIA_OFICIAL.md`](../Jogo/AUDITORIA_OFICIAL.md) §2 e §3 |
| Toda a UI (telas, ordem, decisão da pilha) | [`../Jogo/UI_DO_PROJETO.md`](../Jogo/UI_DO_PROJETO.md) §6 e §7 |
| Peças de mapa que faltam; desempenho dos mapas | [`../Mundo/MONTAGEM_DE_MAPAS.md`](../Mundo/MONTAGEM_DE_MAPAS.md) §7 · [`../Mundo/MAPAS_E_DESEMPENHO.md`](../Mundo/MAPAS_E_DESEMPENHO.md) §6 |
| Rede de batalha | [`../Planos/REDE_E_MULTIPLAYER.md`](../Planos/REDE_E_MULTIPLAYER.md) §2 (R1 a R11) |
| Co-presença no mundo (A10) | [`../Planos/CO_PRESENCA_NO_MUNDO.md`](../Planos/CO_PRESENCA_NO_MUNDO.md) |
| Interação por cards | [`../Jogo/INTERACAO_POR_CARDS.md`](../Jogo/INTERACAO_POR_CARDS.md) §6 |
| Escolhas que esperam resposta | [`DECISOES.md`](DECISOES.md) |

## Ordem de trabalho (10/10/2026)

O plano do projeto é **criar as peças primeiro e só depois montar** mapas e conteúdo. A ordem abaixo segue isso e as dependências entre os itens. Os IDs remetem às tabelas deste arquivo, ao [`DECISOES.md`](DECISOES.md) (P) e aos planos. Ao fechar um item, atualize esta lista no mesmo commit.

**Fase 0: destravar o repositório.** É curta, e esperar custa caro, porque cada commit das cenas grandes fica no histórico para sempre.

1. Decidir P2 e P26: adotar a multicena **e** migrar antes os NPCs da história, que só existem na `Kanto3D`. **A migração foi validada em 11/10 com uma fatia pequena:** o começo de Pallet (mãe, irmã do rival, Carvalho, rival, dois moradores, o ponto de chegada ao laboratório e o gatilho norte) está em `Kanto_Pallet.unity`, montada por `OmegaHistoriaMulticena` (menu `Omega > Kanto multicena - 5.`), com os mesmos roteiros e as mesmas posições da `Kanto3D`; testes `HistoriaMulticenaTests` e `HistoriaMulticenaPlayModeTests`. A `Kanto3D` segue com a cópia dela. Falta o resto da história (Viridian em diante) e então resolver C3: tirar as cenas grandes do git ou pô-las no LFS.
2. ~~Unificar o Unity e fazer o merge na `main`.~~ **Feito em 10/10** (`5c1e3aeb`; a `main` foi avançada até o merge). Atenção: as cenas grandes foram junto (item 1 continua aberto).
3. ~~P10: mover o conteúdo não oficial para `Assets/Data/Conteudo/NaoOficial/`.~~ **Feito em 10/10** (291 assets; o auditor de alcançabilidade trata a pasta como quarentena). **Esvaziada em 13/10:** ficaram só `AssaultArmor` e `Eventide`; o resto foi apagado a pedido do Higor.

**Fase 1: medir a fidelidade da batalha.** O Node.js 24 LTS foi instalado em 10/10.

4. ~~D13 camada A.~~ **Feito em 10/10: dano 100% igual à calculadora oficial** (era 55,5%).
5. ~~D14: batalha sem tela.~~ **Feito em 10/10** (`BatalhaSemTela`).
6. ~~D13 camada B (fluxo contra o simulador).~~ **Feito em 11/10:** 156 cenários, 100% iguais ao Showdown, agora com Dynamax (formato gen 8) e Z-move (gen 7) ([`../Planos/TESTE_DE_CONFORMIDADE.md`](../Planos/TESTE_DE_CONFORMIDADE.md) §7).
7. Corrigir o que o teste acusar. **Feito em 10/10 para a camada A** (fórmula de dano, Life Orb, queimadura, Thick Fat e afins) e D1. **A camada B também, em 11/10:** arredondamentos, ganchos do multiplicador, gemas, Poison Heal, a areia sem dano e mais uma dúzia ([`../Jogo/BATALHA_ATE_O_OFICIAL.md`](../Jogo/BATALHA_ATE_O_OFICIAL.md) §15).
8. ~~Levantar os callbacks do Showdown.~~ **Feito em 10/10:** [`../Jogo/CALLBACKS_DO_SHOWDOWN.md`](../Jogo/CALLBACKS_DO_SHOWDOWN.md) (159 tipos; o que falta e o que conferir na §7).

**Fase 2: terminar as peças.** Depois desta fase, montar é só preencher.

9. **Feito em 10/10:** ficha completa do membro de time (IV e EV por atributo, natureza, habilidade, gênero, tipo Tera, apelido: `TreinadorData.AplicarFicha`) e perfil por dificuldade em todos os caminhos de batalha (`PerfisPorDificuldade`). O que Fácil e Difícil fazem além da IA continua sendo a P17.
10. **Feito em 10/10:** cards de armazenamento, evolução por evento, dar Pokémon com ficha, câmera e mover objeto por `AncoraDeCena` ([`../Jogo/INTERACAO_POR_CARDS.md`](../Jogo/INTERACAO_POR_CARDS.md) §8.1). **Animar, instanciar e remover objeto saíram em 11/10.**
11. **Feito em 10/10:** repelente e multiplicadores de encontro por habilidade e item do líder ([`../Jogo/MODULOS_DO_JOGO.md`](../Jogo/MODULOS_DO_JOGO.md), "Chance de encontro e repelente"). Encontro fixo, vara e surf ficam com os cards e os componentes que já existem. Intimidate/Keen Eye e Magnet Pull/Static saíram em 11/10 (BATALHA_ATE_O_OFICIAL §15.1). Faltam as flautas: pedem a decisão de qual geração seguir.
12. **Feito em 10/10:** evolução por lugar, por espécie e tipo na equipe, por troca com a espécie certa, por dano sofrido, recuo e passos (com os produtores), e o Shedinja (Leva 19). Ainda como evento de roteiro: Milcery, Inkay, Kubfu.
13. **Feito em 10/10:** missões (`MissaoData`, quatro estados e etapas) e estatísticas do jogador. Insígnias continuam o inteiro de sempre até a P3; derrota e revanche esperam a P4 e a P18.
14. **Feito em 10/10:** Gigantamax (34 formas, 33 golpes G-Max, fator salvo na v6), Ogerpon (formas, máscaras, Tera obrigatório, Embody Aspect) e Maushold de Três ([`../Jogo/BATALHA_ATE_O_OFICIAL.md`](../Jogo/BATALHA_ATE_O_OFICIAL.md) §14). Os efeitos dos 10 golpes G-Max que faltavam saíram na Leva 21. A troca de golpe do Zacian/Zamazenta saiu em 11/10 (§15.1). Faltam os nomes da auditoria.
15. **Feito em 11/10:** o importador único do Showdown, descrito em [`FERRAMENTAS_DO_EDITOR.md`](FERRAMENTAS_DO_EDITOR.md) §3.
    - Um exportador Node lê o Showdown no commit fixo e gera um JSON. O importador compara e corrige os dados, e roda as 21 levas na ordem como receitas de comportamento, numa passada idempotente (a segunda dá 0 mudanças).
    - Reaplicar tudo achou e corrigiu:
      - **dados errados:** Keldeo Resoluto com Ataque e Ataque Especial trocados; Oricorio Pa'u, Pom-Pom e Sensu com o tipo do Baile (Fogo); Cosmog e Cosmoem com 100 em tudo; Aegislash com os números da Gen 7; Cresselia (110/120 no Showdown); PP de Take Heart e de 23 golpes G-Max; Burning Bulwark e Dragon Cheer sem "nunca erra";
      - **o bônus das máscaras da Ogerpon e do Adamant Crystal / Lustrous Globe**, que só valia para a espécie base, embora o item force outra forma; na prática, nunca valia;
      - **formas que eram base de si mesmas** (Indeedee F, Basculegion F, Oinkologne F, Enamorus Therian, Toxtricity, Squawkabilly), com nome "Nova especie(F)(F)(F)" que crescia a cada passada.
    - O relatório também lista o "oficial sem asset". Hoje é tudo diferença de nome, não lacuna: o projeto divide os Z-moves em `…Physical`/`…Special` e os cristais Z em `…Held`/`…Bag`, e tem um Hidden Power só, que calcula o tipo. O importador não cria asset sozinho.

**Feito em 11/10 junto das fases 1 e 2** (pedido de Higor): P3, P4, P17 e P18 viraram decisões (T47 a T50) e código: dificuldade e IVs no save v7, derrota sem apagão, revanche com política e renovação, perfil de IA por progresso. `ehMarco` marcado em 12/10 nos 8 líderes de Kanto e nas 6 variantes do campeão. Falta conteúdo: escolher os treinadores renováveis e preencher os perfis.

**Fase 3: UI.** Depende da P1, a pilha de UI.

16. (P31, lado da batalha **pronto em 12/10**: `SistemaBatalha.MecanicasParaUI.cs` responde motivo, resumo e previsão do golpe Z/Max; falta só a tela.) A ordem de [`../Jogo/UI_DO_PROJETO.md`](../Jogo/UI_DO_PROJETO.md) §6: as telas da batalha, incluindo as mecânicas novas (§2.H, P31). **Já feitos:** a caixa de escolha do roteiro (`EscolhaDeRoteiroUI`, 11/10) e, em 12/10, a volta das telas uGUI provisórias (título, salvar/carregar, HUD, aviso de interação, cartão, configurações), que estavam desligadas desde 07/10.

**Fase 4: montar o mundo.** É trabalho do dono do projeto e do grupo de mapa.

17. Mapas e áreas da multicena; NPCs com cards; perfis de IA preenchidos; lojas e preços (P5).
18. Em paralelo: arte (A11), colisão e desempenho (A3, medir uma build), áudio.

**Fase 5: rede.**

19. Responder P29 e P30 (plataforma, quem hospeda, orçamento). Depois, o spike da P13 ([`../Planos/ESCOLHA_DO_PACOTE_DE_REDE.md`](../Planos/ESCOLHA_DO_PACOTE_DE_REDE.md)).
20. ~~Foto completa da batalha (R3, R4, R6)~~ **feita em 13/10**. Resta o árbitro remoto e o prazo (R2, R8), depois co-presença (A10).

**P9, P31, P29, P30 e P13 são do grupo de klz** (registrado em 12/10; ficam em [`DECISOES.md`](DECISOES.md) com esse dono). A explicação de cada uma, para o grupo, está na página compartilhável "Nel-Ord: decisões do grupo de klz" (12/10). **Decisões que destravam as fases, na ordem em que travam:** P2/P26 e a permissão da P10 (fase 0); P1 (fase 3); P9 e P31 (Z, Dynamax e Tera); P5 (preços); P29, P30 e P13 (fase 5). P3, P4, P17 e P18 já viraram as decisões T47 a T50.

**Entrada do jogo (12/10, decisão de Higor):** Bootstrap → título uGUI (`TelaDeTitulo`) → "Novo jogo" abre a `OmegaMenu` direto no criador de personagem → "Começar" leva à `KantoBase` (multicena), nascendo em Pallet. Antes ia para a `DevScene`, e Kanto não era alcançável jogando.

---

## 1. Crítico

| # | O quê | Como | Esf. | Dono |
|---|---|---|---|---|
| C3 | **Duas cenas gigantes já estão no `origin` sem LFS**: `Kanto3D.unity` (89 MB) e `KantoPecas.unity` (74 MB, medido em 09/10). O GitHub recusa arquivo acima de 100 MB e cada commit delas soma ~80 MB ao histórico para sempre. Também abertos: versão do Unity (`main` ainda em 9f1; esta branch em 10f1) e o pacote `draco` | Parar de commitar as duas. Adotar a `KantoMulticena` (PENDENCIAS O1, DECISOES) e tirá-las do git e do Build Settings. LFS é para binário (`.glb`, `.png`…); cena é texto e perde diff e merge no LFS | P/M | Grupo (Mapa + Higor) |
| C4 | Arquivos do Higor editados por outros; **perguntas sem resposta** nos commits `524e6c8`, `88d247a`, `acd9aed` e sobre determinismo | Um patch por assunto (save, evolução/gatilhos, validador/auditor, input). *Conferir se as perguntas ainda procedem antes de agir* | M | Dev + Higor |

## 2. Alto

| # | O quê | Como | Esf. | Dono |
|---|---|---|---|---|
| A2 | Rivais ainda são 3 `TreinadorData` por inicial, escolhidos pelo roteiro | Migrar para `TreinadorData.OpcaoDeEquipe` (mesma `CondicaoDeRoteiro`, nunca sorteio) é **opcional**: o mecanismo existe e tem teste | M | Conteúdo |
| A3 | `Kanto3D`: 53.978 `MeshCollider`. Na `KantoPecas` e na multicena a regra é única (`ColisaoDoMapa`) e a carga é ~2× mais rápida; **falta juntar as peças de cada casa em poucas caixas** (piloto no `PredioCeladon`: 318 → 94 caixas; estimativa de todas as estruturas: 26.782 → ~6.640) | Menu `Omega → Kanto - simplificar colisores dos predios` por prédio; ver [`../Mundo/KANTO_REVISAO_E_MULTICENA.md`](../Mundo/KANTO_REVISAO_E_MULTICENA.md) | M | Mapa |
| A4 | **Duas UIs (uGUI × UI Toolkit)** com quatro telas em duplicidade | Decidir: o Toolkit substitui? Sim → migrar tela a tela; não → congelar a nossa. Bloqueia toda tela nova ([`../Jogo/UI_DO_PROJETO.md`](../Jogo/UI_DO_PROJETO.md) §7) | M | Grupo (UI) |
| A5 | Aceite de `CombinadosPorPasta` no `AuditorDeAlcancabilidade` | Justificativa + meta de saída por área; o auditor imprime quantos assets cada par engole | P | Grupo |
| A6 | Bot de varredura chega ao Omega por reflexão | A assembly de testes de PlayMode já referencia `NelOrd.Omega` (07/10); falta tirar a reflexão do bot, usar `InputSystem.QueueStateEvent` e cobrir treinador, trocador e evolução com trava | M | Dev |
| A7 | Cenas sem dono e órfãs (`DevScene` com conteúdo nosso) | Conteúdo nosso vira prefab; remover `OmegaBlocos`/`OmegaMapa` se substituídas | P | Grupo |
| A8 | E-mail dos commits inconsistente | `git config user.email` local por máquina; **não** reescrever histórico | P | Grupo |
| A10 | Base local para co-presença no mundo | 5 etapas independentes (D → C1 → C2 → A → B): [`../Planos/CO_PRESENCA_NO_MUNDO.md`](../Planos/CO_PRESENCA_NO_MUNDO.md). Nada começado | M | Qualquer pessoa do grupo |
| A11 | **Arte faltando** (conferido 07/10) | `CatalogoModelos3D` liga **11 de 968** modelos 3D (rodar `Omega/Modelos - montar catalogo 3D` e conferir); **2.106 de 2.434 itens sem ícone**; formas novas sem arte própria (Castform ×3, Cramorant ×2, Mimikyu Descoberto, as 29 Megas criadas em 09/10) usam a folha da base — aceitável como provisório, nunca como final. Fluxo: [`COMO_TRABALHAR.md`](COMO_TRABALHAR.md) §4.2 | M | Arte |
| A12 | **Times de treinador: o que falta** (conferido 08/10). *Decidido (dono do projeto):* simples ou dupla é **escolha do jogador** em todo NPC treinador; `TreinadorData.dupla` do FireRed é só informativo; a batalha de roteiro é simples. | (1) **o que Fácil e Difícil fazem está decidido** (DECISOES T47: a IA dos treinadores, e os IVs deles numa opção separada, no save do mundo); **falta o conteúdo**: os 173 NPCs têm os três perfis por dificuldade vazios (hoje caem no perfil de reserva). (2) derrota: **feito**, igual para NPC e roteiro (T48). (3) revanche: **feito** (T49). (4) ficha completa: **feito** em 10/10 (`TreinadorData.AplicarFicha`: IV e EV por atributo, natureza, habilidade, gênero, tipo Tera, apelido); os 744 importados ainda usam a ficha rasa do FireRed, que é conteúdo. *Sem decisão:* `inicialDoJogador` sem uso em jogo. **Dificuldade selecionável de novo desde 12/10:** o dropdown dela vive na `Bootstrap` (override do prefab), mas o `ConfiguracoesMenu` ficou desligado de 07 a 12/10; o de IVs nunca tinha sido autorado e agora o painel o cria, copiando a linha do FPS, abaixo da última linha | M | Higor + Grupo |

## 3. Médio

| # | O quê | Como | Dono |
|---|---|---|---|
| M1 | Plano de batalha: **etapa 8** (barramento único; `MesaDeRegras` vira fachada). O PRNG no contexto e as batalhas simultâneas **já estão feitos** (13/10, D3). Falta o barramento em si, que também resolve a ordem dos resíduos (D14). Risco alto. As etapas 4 e 11 foram resolvidas ou dispensadas | [`../Planos/BATALHA_BARRAMENTO_E_DETERMINISMO.md`](../Planos/BATALHA_BARRAMENTO_E_DETERMINISMO.md). Um PR por etapa; `OrdemDeResiduosTests` | Higor |
| M5 | Categorias de inventário | `CategoriaDeItem` em `ItemData`; abas na UI (Captura / Cura / Batalha / Chaves); depende de A4 | Higor + UI |
| M6 | Ícones e sprites: 2.106 itens sem ícone e 23 formas regionais | Apelidos `a` → `-alola`; ícone por categoria; ver A11 | Arte |
| M8 | Interiores e cavernas | Decidir cena aditiva antes de modelar | Grupo |
| M9 | Shaders e URP assets | Documentar passes; fixar URP 17.5 | Dev + Grupo |
| M12 | Atalho de Pokébola em batalha selvagem | Botão/tecla que abre direto no item mais recente de captura | Dev |
| M13 | Opção de nenhum ataque ser letal | Config. de dificuldade/acessibilidade: dano que zeraria o HP para em 1 | Grupo |
| M19 | Sprites de batalha: o que sobrou | (2) `0003` e `0026` são o sprite de andar no mundo, não de batalha — falta a tira certa (a origem `YAPU_Demo` não está nesta máquina); (3) **mobile no futuro**: tiras acima de 4096 px (a maior tem 13.625) precisam virar grade. Visto: adversário alto encosta no painel de HP do jogador (posição da caixa no prefab) | Dev + Arte |

## 4. Baixo

| # | O quê | Dono |
|---|---|---|
| B1 | Habilidades de fã e de Conquest e golpes sem `meta` na PokeAPI: fora do jogável. *Recontar:* 07/10 removeu 18 habilidades de fã presas às formas apagadas | Conteúdo |
| B2 | **Bugs conhecidos:** rematch paga infinito (o dinheiro agora só muda por `GanharDinheiro`/`GastarDinheiro`/`DefinirDinheiro`, teto 999.999; a revanche ganhou política e renovação em 11/10: DECISOES T49; a revanche paga igual, por decisão); caixa cheia gasta a bola; `AmbienteUI.cs:77`; 2 `Debug.Log` | Higor |
| B3 | Rede/árbitro remoto (a foto da batalha já se repõe, 13/10) | Higor — ver [`../Planos/REDE_E_MULTIPLAYER.md`](../Planos/REDE_E_MULTIPLAYER.md) |
| B4 | **Conteúdo da troca de habilidade.** O card **Trocar habilidade** está feito (12/10; regra decidida: o NPC oferece as habilidades **dele** a qualquer espécie, pela caixa de escolha, e a troca fica no indivíduo). Falta o conteúdo: montar os NPCs e distribuir as habilidades entre eles, e decidir **se a troca custa algo** (dinheiro, item, flag). O **Ability Capsule e o Ability Patch funcionam desde 12/10** (`ItemTrocaDeHabilidade`, Leva 26), mas preço 0 e nenhuma loja, NPC ou item de chão os entrega. O molde `CapsulaDeHabilidade` (qualquer habilidade por id) também segue sem quem o entregue | Conteúdo |
| B5 | Formas `10xxx` sem ocupar slot de Pokédex: **não é do save** (os ids não mudam); a Pokédex só precisa agrupar as formas sob a base (`formaDe`). Decidir junto da UI da Pokédex | Grupo |
| B7 | README "como regenerar tudo" em `tools/` (o extrator do APK, `tools/omega-extract`, não está nesta máquina; ver [`COMO_TRABALHAR.md`](COMO_TRABALHAR.md) §4.1) | Dev |

---

## 5. Dívidas de código e de teste

Nada disto quebra o jogo hoje; cada item é um defeito, uma inconsistência ou uma pergunta de desenho conhecida.

| # | O quê | Detalhe | Dono |
|---|---|---|---|
| D2 | `TipoDeAcontecimento.AtualizouStatus` é declarado e **nunca escrito** | O `RegistroDaBatalha` leva narração, não estado. Ver Rede R5 | Higor |
| D5 | **Vocabulário de reaplicar status** | Hoje cada chamador decide caso a caso se reaplicar empilha, estende ou recusa. Nomear as quatro intenções (padrão do Shattered Pixel Dungeon: *append* = sempre cria, *affect* = pega ou cria sem duplicar, *prolong* = estende, *detach* = remove todos da classe) em cima de `StatusAtivo` evitaria bugs do tipo "veneno reinicia o contador ao ser reaplicado" | Higor |
| D7 | Membros públicos sem consumidor | Recontado em 11/10: `TrocarPosicoes`, `EhForma`, `TotalDeSlots` e `PrimeiroGolpeUtilizavel` já têm uso (saíram da lista); `UltimoGolpeDe`, `PodeSerUsado` e `Proximos` foram removidos. Ficam `EquipeAdversaria` (a foto da batalha da rede vai precisar, R3) e `BotaoAvancar` (gancho de botão de UI que ninguém ligou ainda) | Dev |
| D8 | `ItemCura.curaTodosOsStatus` (bool) + `statusCurados` (lista) é um par ad hoc | Um enum de alcance (`Nenhum`/`Todos`/`Especificos`) seria mais direto. Qualidade de vida, não dívida | Dev |
$114/10) | **UI** (A4/P1). **Dados:** `Assets/Data` + `CatalogoJogo` × `Omega/.../GameDatabase.cs` (TSV do APK), que hoje só alimenta um log e o texto da pausa do `GameBootstrap`: candidato a sair. **Validador:** `ValidadorDeCatalogo` × `OmegaCorretorDeAssets` **não são um par**: o primeiro confere e relata, o segundo corrige o que tem resposta única (e está atrás de `GeradoresCongelados`). Ficam os dois. **NPC que fala:** três caminhos (`Npc` só nas cenas de dev, `NpcComRoteiro`, `NpcComInteracao`), ver P27. **Movimento:** não é par de verdade: o jogo usa `PlayerMovement`; o `OmegaPlayerController` só entra quando a `OmegaMenu` é aberta sozinha (modo protótipo). **Primeira cena:** valem `Inicializacao.primeiraCena` = `OmegaMenu` (na Bootstrap) e `GameBootstrap.cenaDoMundo` = `KantoBase` (na OmegaMenu). **Corrigido em 14/10:** `OmegaKanto3D` e `OmegaAcabamento` só preenchem o `cenaDoMundo` se estiver vazio, em vez de regravar `Kanto3D` e `OmegaDev` por cima | Decidir qual fica em cada par. **Antes de criar algo em Equipe/Mochila/Pokédex/movimento/dados, conferir se já existe do outro lado.** Não rodar `OmegaKanto3D`/`OmegaAcabamento` sem conferir o `cenaDoMundo` depois | Grupo |
| D10 | **Teste de teclado do Play Mode precisa da Game View em foco**; `InputTestFixture` não resolve | O teste `SaveViaJogadorTests.EscapeAbreOMenu_PeloInputDeVerdade` se declara `Ignore` sem foco. O ponto cego (desfazer o mapeamento do Escape) está fechado por `BindingsDeInputTests` (EditMode). Retrofit viável exigiria que os sistemas nasçam *depois* do `Setup()` do fixture. Ver [`COMO_TRABALHAR.md`](COMO_TRABALHAR.md) §6 | Dev |
| D11 | **Perfis de IA nunca tiveram os parâmetros escolhidos um a um** | Os 6 assets vieram do preset da ferramenta (tabela "Configuração esperada por faixa" em [`../Jogo/MODULOS_DO_JOGO.md`](../Jogo/MODULOS_DO_JOGO.md) é **proposta a validar**). 4 dos 6 não são usados por nenhum adversário (`IAVeterana`, `IAVeteranaDefensiva`, `IAEspecialista`, `IACampea`): esperam uma linha de adversários de fim de jogo. O `perfilSelvagem` não varia com a dificuldade | Higor + Conteúdo |
| D12 | **Habilidade atravessa a evolução por convenção** | `Pokemon.Evoluir` preserva o **slot**; a linha evolutiva inteira precisa listar as habilidades na mesma ordem e o `ValidadorDeCatalogo` cobra. A alternativa (mapa explícito na `Evolucao`) só se paga com a primeira evolução de habilidade exclusiva. Recomendação: manter até aparecer o caso | Higor |
| D13 | **Teste de conformidade com o Showdown** | **Camadas A (dano, 10/10) e B (fluxo, 11/10) feitas, as duas a 100%.** A B tem 169 cenários: simples, Dynamax, Z-move e **dupla** (12/10). Fica para crescer: troca forçada depois de nocaute em dupla (os cenários de dupla não derrubam ninguém), os terrenos e climas em dupla, e as habilidades de aliado (Plus/Minus, Friend Guard). Plano e resultado em [`../Planos/TESTE_DE_CONFORMIDADE.md`](../Planos/TESTE_DE_CONFORMIDADE.md). Origem: [`../Historico/COMPARATIVOS_EXTERNOS.md`](../Historico/COMPARATIVOS_EXTERNOS.md) §6.1 | Higor |

| D14 | **Ordem dos resíduos do fim do turno** (achado em 12/10 pela varredura com alvo preparado) | O Showdown cura com o Leftovers (ordem 5) **antes** da queimadura (10); aqui o status cobra antes e os gatilhos de item vêm depois (`SistemaBatalha.FimDeTurno.cs`). Com HP perto de uma queimadura decide quem sobrevive. Resolver é dar a cada gatilho e a cada status a ordem numérica do oficial (peso 1: clima; 4: Wish; 5: terreno e Leftovers; 8: Leech Seed; 9: veneno; 10: queimadura; 28: orbes, Sticky Barb), ou seja, o M1 etapa 8. A varredura usa Light Clay no alvo para não esbarrar nisto | Higor |

## 6. Desempenho e build

Regras: medir antes, A/B na mesma rodada, **número sem recorte (cena, máquina, data) não entra**, "não medi" quando não mediu. Máquina de referência: Ryzen 5 4600G, 16 GB, HD + SSD. Método e medições: [`../Mundo/MAPAS_E_DESEMPENHO.md`](../Mundo/MAPAS_E_DESEMPENHO.md).

| # | O quê | Por que agora | Como |
|---|---|---|---|
| O1 | **Adotar a `KantoMulticena` e tirar as cenas gigantes** | Memória do Editor (caiu em 26/09), limite de 100 MB do GitHub, merge. Falta: occlusion culling por cena, medir carga/memória/quadro numa build, decidir | Ver C3, [`DECISOES.md`](DECISOES.md) e o "Falta" em [`../Mundo/KANTO_REVISAO_E_MULTICENA.md`](../Mundo/KANTO_REVISAO_E_MULTICENA.md) |
| O2 | **Sprites de batalha sem compressão** (845 folhas, ~759 MB descomprimidas, em `Resources`) | A máquina mínima tem ~0,5 GB de VRAM dedicada. O cache é liberado no fim da luta, mas cada folha entra inteira | Compressão de qualidade alta (Point continua) ou grade (que o mobile já exige, M19); depois `Resources` → Addressables (DECISOES) |
| O4 | `GramaPontos.bytes` (27 MB) e o pacote visual (`Assets/Omega/Visual`) **nunca aplicados à cena versionada** | 27 MB no repositório que nenhuma cena usa | Aplicar ou apagar — decisão do grupo do mapa (DECISOES) |
| O5 | 18.615 `MeshCollider` na `KantoPecas` | Carga e memória de física | Caixas por casa (A3) |
| O7 | +2,3 KB/quadro quando o culling por área esconde áreas | Vem do motor/URP (conferido), não do componente | Profiler numa build; só agir se aparecer como pico de GC |
| O8 | Carga da `Kanto3D`: 62 s em HD mecânico (26/09) | Tempo de iteração de todo mundo | O1 resolve; o projeto num SSD ajuda |
| O10 | Pares de sistemas duplicados | Ver D9 | — |
| O11 | **Nenhuma build medida, nenhuma meta numérica** | Todo número de desempenho do projeto é do Editor (a exceção: o bootstrap, 985 → 77 MB de textura, medido numa build de desenvolvimento) | Uma build de desenvolvimento na máquina mínima + metas por cena (ms/quadro, carga, memória). Método em MAPAS_E_DESEMPENHO §1 |
| O12 | **~38 Megas com as habilidades da forma base + "Pickup" como oculta** (achado 26/09) | Preenchimento padrão do gerador. As Megas foram refeitas pelo Showdown em 09/10 (94 formas, habilidade espelhando os slots da base — `BATALHA_ATE_O_OFICIAL` §10), *então conferir se o O12 ainda procede*; Excadrill → Piercing Drill segue sem regra | Conferir; fechar ou reescrever |
| O13 | **LOD e impostors** | Zero `LODGroup` no projeto (o mecanismo é trivial, mas exige malhas de menor detalhe: arte). Octahedral impostors (objeto distante vira um quad com atlas pré-renderizado) só se o *instancing* da vegetação não bastar a distância grande — **medido, não suposto** | Depois do O11. Pooling de objetos temporários também só depois de medir: os 12 `Instantiate` do runtime são botões de lista de UI, nunca por quadro |

## 7. Conteúdo, design e sistemas que ainda não existem

| # | O quê | Situação |
|---|---|---|
| S1 | **Áudio** | `SistemaAudio` pronto desde 05/09 (música em laço com troca suave, efeitos, 3 volumes); **nenhum arquivo de som importado** (música de exploração, batalha, vitória; efeitos de menu, dano, desmaio, captura, item). Sem clipe, tudo é no-op silencioso |
| S2 | **Tela de título** definitiva | A provisória (`TelaDeTitulo`, uGUI) voltou a ser a entrada em 12/10: Continuar (slot mais recente, `SistemaSave.SlotMaisRecente`), Novo jogo (abre a `OmegaMenu` direto no criador de personagem, por `Inicializacao.NovoJogoJaEscolhido`) e Sair. O título em UI Toolkit da `OmegaMenu` continua existindo e aparece no "Voltar" do criador. A da branch de assets tem preferência no merge; o que vale preservar é o fluxo |
| S3 | **Diálogo ramificado** | `SistemaDialogo` faz sequência linear; a **escolha** vive no roteiro e nos cards, pela caixa `EscolhaDeRoteiroUI` (uGUI, 11/10; substituiu o IMGUI). Adiado de propósito até revisar material externo (canal Renzk), que mexe também em `Interagivel`, encontros e batalha — revisar os três juntos |
| S4 | **Insígnias (`badges`)** | Campo morto na leitura: só o passo `PassoDarInsignia` do roteiro o incrementa e as MOs o consultam; não há ginásio como sistema. Opções: virar sistema de ginásios, virar eixo de escala de nível (padrão do R.O.W.E.: uma tabela que ajusta nível de selvagem/treinador/chefe, tamanho do time, dinheiro perdido e teto de poder — **o desenho é ponto de partida, os valores precisam de validação jogando**), ou sair |
| S6 | **Missões: conteúdo e diário** | O modelo existe desde 10/10 (`MissaoData`, quatro estados, etapas, `PassoMissao`, `SeMissao`) e o cartão de treinador mostra as em aberto desde 12/10. Falta: **nenhuma `MissaoData` foi criada** (0 assets; o `CatalogoJogo.missoes` está vazio), recompensa como dado e o diário de verdade (UI_DO_PROJETO P2-2c) |
| S8 | **Capacidade da caixa** | Ilimitada; a equipe é 6 |
| S9 | **Rematch de treinador** | **Feito em 11/10** (DECISOES T49). `ehMarco` marcado em 12/10 nos 8 líderes de Kanto e nas 6 variantes do campeão. Falta escolher quem é renovável (conteúdo) |
| S11 | **Habilidade — pontas que os jogos do gênero têm** | (a) **feito em 11/10** (anúncio ao entrar); (b) **feito em 12/10**: a Pokédex lista as habilidades possíveis da espécie capturada, com a oculta marcada; (c) a oculta não tem fonte especial (é peso baixo em qualquer encontro; nos jogos vem de encontro marcado), e isso pede decisão; (d) não há herança (não existe reprodução) |
| S12 | **Captura: revista com o conteúdo de hoje (14 e 15/10)** | **Feito.** (1) **A fórmula é a da Gen VIII/IX, conferida em 15/10** (antes era a Gen III/IV): shake `Y = ⌊65536/(255/X)^(3/16)⌋`, sono e congelamento ×2,5, **bônus de nível baixo da Gen IX** (até o nível 13, `(36 − 2×nível)/10`), **captura crítica** (pelas espécies capturadas, uma verificação só) e Ultra Ente ×410/4096 contra bola comum. A conta é contínua, em `double`, sem piso: as 47 espécies do APK com taxa 1 ou 2 davam `a` = 0 na fórmula antiga (captura impossível), e agora têm uma chance pequena de verdade. A Timer Ball conta os turnos **passados** (×1 no primeiro). (2) **Todas as bolas que dependem da situação carregam a regra no asset** (`BonusDeBola`, Leva 29): Quick (×5 no 1º turno), Timer (+0,3 por turno até ×4), Net (×3,5 em Água e Inseto), Repeat (×3,5 em espécie já capturada), Level, Nest, Fast (velocidade-base ≥ 100, ×4), Love (×8), Dream (×4 dormindo), **Dusk (×3 de noite ou em caverna), Dive (×3,5 na água e pescando), Lure (×4 pescando), Moon (×4 contra quem evolui com a Moon Stone), Heavy (soma na taxa pelo peso: −20, 0, +20, +30) e Beast (×5 em Ultra Ente, ×0,1 nos demais; as 11 espécies estão marcadas)**. A origem do encontro (`OrigemDoEncontro`: mato, água, pesca, caverna, evento) vem de `AreaDeEncontro.origem` (padrão Mato), da pesca e do roteiro. **Ficou de fora:** Safari, Sport e Park Ball (só valem em modos que o jogo não tem), o modificador de dificuldade D da Gen VIII (a Gen IX o trocou por efeitos de fora da batalha) e o Charme de Captura (o gancho `AmuletoDeCaptura` existe, falta o item). **Para quem monta mapa:** a zona de água precisa de `origem` = Água (e a de caverna = Caverna) no Inspector; as já montadas ficam em Mato até alguém mudar. Sono e congelamento passaram de ×2 para ×2,5 (Gen V+); paralisia, veneno e queimadura já eram ×1,5. O histórico de 02/09 continua em [`../Historico/CAPTURA_VALORES_DE_TESTE.md`](../Historico/CAPTURA_VALORES_DE_TESTE.md) |
| S13 | **Save/load sem confirmação ao jogador** | "Jogo salvo no slot N em <caminho>" é log técnico. Decidir se o jogador vê uma confirmação |
| S14 | **Animação** | Só a locomoção do jogador (`AnimadorDoJogador`); zero para Pokémon, NPC e batalha ([`../Jogo/ANIMACAO.md`](../Jogo/ANIMACAO.md)) |
| S15 | **Preço dos itens** | Os itens vieram do APK do Omega, com a economia dele (Potion 3, Ultra Ball 20, Poké Ball 200). **Em 12/10 os 50 itens que alguma loja vende** (fora a `LojaDeTeste_TodosOsItens`) receberam o preço oficial de compra da PokeAPI (`items.csv`, geração mais nova, T21): Potion 200, Repel 400, Ultra Ball 800… Ficaram de fora os TMs (o número de cada TM muda por geração, então o preço da PokeAPI não é do mesmo golpe), `RetroMail` e `SeloDeHabilidade` (sem preço oficial). O resto do catálogo segue com o preço do Omega ou 0: a PokeAPI mistura moedas (Mints, Ability Patch e Candies aparecem com o preço em Battle Points), então não dá para aplicar em massa. Preço definitivo é decisão de conteúdo (P5) |
| S17 | **Conteúdo importado sem caminho até o jogador** | 295 treinadores e 237 tabelas de encontro importados do FireRed **não têm caminho até o jogador** (mapas sem sala 3D, revanches) — o auditor os lista com motivo escrito |
| S18 | **PP como recurso** | **Ether, Max Ether, Elixir e Max Elixir feitos em 14/10** (`ItemRestauraPp`, Leva 30): os quatro vinham do APK como `ItemGenerico`, que não fazia nada. O Ether cura o golpe que a tela escolher (`ContextoUsoItem.IndiceDoGolpe`) ou, sem escolha, o mais vazio; a **tela de escolher o golpe é da UI e não existe**. **Falta:** PP Up e PP Max (aumentam o PP máximo do golpe, que o save precisa guardar; hoje são `ItemAtributo` com o padrão de EV, ou seja, errados), Pressure e a IA usando Ether |
| S19 | **Golpes ligados que nenhuma espécie aprende** | Barrage, Scale Shot, Triple Axel e Hard Press estão ligados como multi-hit, mas sem caminho até o jogador (o auditor os lista) |
| S20 | **Efeito dos itens e dos golpes contra o oficial** (revisão de 12/10) | **Feito em 12/10.** *Golpes:* varredura por golpe contra o simulador do Showdown (`tools/conformidade/gerar-golpes.js`, [`../Planos/TESTE_DE_CONFORMIDADE.md`](../Planos/TESTE_DE_CONFORMIDADE.md) §8): de 91,6% para 99,8% (neutra) e 99,5% (alvo preparado); ~40 golpes corrigidos pela Leva 27. *Itens:* a Leva 28 conferiu cura, EV, Exp. Candy e itens segurados contra o texto da geração mais nova (a Potion curava 400 HP; o Antidote curava qualquer status); `ItemTipoTera` (as 18 Tera Shards), `ItemExperiencia` (5 Exp. Candies) e quatro itens segurados com efeito (Lucky Egg, Amulet Coin, Luck Incense, Everstone). **Falta, em golpes:** Psychic Noise (Heal Block), Fusion Bolt e Fusion Flare (dobram depois do outro), os três Pledge, Flying Press (tipo duplo), Beak Blast, Uproar, Supercell Slam (dano de queda), Steel Beam ao errar, Tera Starstorm, Order Up e Commander; Floral Healing com Grassy Terrain (2/3); Magnetic Flux só vale com Plus e Minus em dupla; **Instruct só mira o aliado** (o enum de alvo não tem "qualquer adjacente"). Golpes que pedem preparo próprio (Counter, Fling, Belch, Sucker Punch…) e os de sorteio ficam fora da varredura, com o motivo no gerador. **Falta, em itens** (todos `ItemGenerico`, ~1.590, sem efeito): X items e Dire Hit/Guard Spec (9 + 2), PP (Ether, Elixir e os Max, PP Up, PP Max: ver S18), Escape Rope, Poké Doll, iscas e flautas, Mints (natureza), fósseis, doces de espécie, mulches, apricorns, Sacred Ash (revive a equipe inteira), Exp. Share, Soothe Bell, Fresh-Start Mochi, Dynamax Candy, e as **Poké Balls especiais**: só Great, Ultra e Master têm multiplicador, as outras 40 ficam em ×1 sem as condições (Net, Dive, Nest, Timer, Quick, Dusk, Level, Lure, Moon, Love, Fast, Heavy, Repeat, Beast, Heal, Luxury…). A quantidade de EV dos Candies e Mochi (10) **não tem fonte** e os Candies L e XL ainda não exigem nível 30 e 60. Itens de fã sem par oficial (Magic Candy, Rainbow Candy, Health Feather…) continuam no jogo: ver P10 |
| S21 | **TMs e TRs de depois da Gen 4 não ensinam nada** (auditoria de 15/10) | 280 itens (`Tm51` em diante e `Tr00` a `Tr99`) são `ItemGenerico`: usar não faz nada. Só `Tm01` a `Tm50` e as 8 MOs são `ItemEnsinaGolpe` (58 no total). Falta o mapa máquina → golpe (a PokeAPI tem `machines.csv`) e a compatibilidade das espécies: `golpesPorMaquina` vem só da ROM do FireRed. Sem isso, a maioria dos golpes de TM da Gen 5 a 9 só se aprende por nível |
| S22 | **~1.600 itens sem efeito (`ItemGenerico`)** (15/10) | Nem todos precisam de efeito: 1.183 são materiais, enfeites e itens de chave. Os que pedem peça nova: **Mints** (21, trocam a natureza), **Bottle Caps** (treino hiper), **fósseis** (14, pedem um laboratório para reviver), **bagas de plantar**, **Apricorns** (Poké Balls do Kurt) e as 31 **cartas**. Falta a triagem do que entra no jogo (decisão de conteúdo) |
| S23 | **Cromáticos (shiny)** | Zero código e zero dado: sem chance (1/4096), sem marca no `Pokemon`, sem visual alternativo (arte), sem save. É decisão de design antes de ser código |
| S24 | **Criação: ovos, creche, golpes de ovo e grupos de ovo** | Nada existe (nem o dado de grupo de ovo na espécie). Pede NPC da creche, ovo como Pokémon que choca por passos, herança de golpes e natureza, save |
| S25 | **Relembrar e apagar golpes** | Não há NPC nem tela. "Esquecer" existe só quando o Pokémon aprende um golpe novo com a ficha cheia |
| S26 | **Controles e conforto** | O gamepad já tem binding (stick, botão sul, gatilho). Faltam a **tela de remapear as teclas** (`PerformInteractiveRebinding` não é usado), o **salvamento automático** (hoje só manual, em slot) e **renomear o Pokémon** (o campo `apelido` existe; não há tela nem NPC) |
| S27 | **Mundo vivo** | Nenhum NPC anda ou patrulha, nem muda por horário; não há Pokémon seguidor nem Pokémon visível no mapa; não há bicicleta, plantação de bagas, Itemfinder, minimapa, mapa do mundo ou Voar |
| S28 | **Formatos de batalha** | Simples e dupla (inclusive treinador em dupla) existem. Faltam a **dupla com parceiro NPC** (um aliado controlado pela IA ao lado do jogador), a tripla e a de rotação |
| S29 | **Evolução por lugar sem dados** | A peça (`ExigeLugar` + `LugarDoMundo`) existe, mas **nenhuma espécie a usa e nenhum mapa tem um `LugarDoMundo`**: as evoluções de Hisui e Galar (Quilava, Dewott, Petilil, Rufflet, Goomy, Dartrix, Bergmite, Koffing, Mime Jr.) são impossíveis jogando. Falta marcar os lugares e ligar as espécies |

## 8. Mapa de Kanto: o que está aberto da revisão de 26/09

Detalhe e evidência em [`../Mundo/KANTO_REVISAO_E_MULTICENA.md`](../Mundo/KANTO_REVISAO_E_MULTICENA.md). Resolvidos: colisor com regra única (A), correções de runtime na fonte (D), resgate de queda (E). **Abertos:**

| # | O quê | Dono |
|---|---|---|
| K1 | **Cor e textura (item C): dois diagnósticos que se contradizem.** O klz documentou em `OmegaVisualToon.cs` que a cor vinha de nós de shader que o glTF não exporta e que o atlas pela UV dá "listras" (por isso o toon usa a paleta `kanto_paleta.json`); a correção M17 religou o `kanto_atlas` em 306 clones. Nenhum dos dois está aplicado de forma coerente. **O grupo decide antes de qualquer mudança visual.** Não mexer até lá | Grupo (Mapa) |
| K2 | **Pacote visual nunca aplicado (item B)**: toon (63 materiais), água, céu, vegetação, cachoeira e grama instanciada têm zero referências nas cenas versionadas; só rodam `PaisagemDistante` e `MarSemLimbo`. Aplicar ou descartar de vez (e apagar os 28 MB de grama se descartar) | Grupo (Mapa) |
| K3 | **Peças repetidas viram prefab** (as 2.933 malhas compartilhadas): colisor, LOD e material definidos uma vez. A `KantoPecas` fez 199 prefabs; falta LOD (zero `LODGroup` no projeto: exige malhas de menor detalhe) | Mapa |
| K4 | **Occlusion culling por cena** na multicena (o dado herdado da origem foi removido da base em 30/09 porque fazia sumir o chão perto da Torre Pokémon — *a confirmar em jogo*) | Mapa |
| K5 | **3 tutores de golpe (Espíritos, Fundamentos, Sono) ficaram no `y` da `Kanto3D`** na multicena, porque o raio não achou o chão: podem flutuar ou afundar. Conferir em jogo, e que o `Menu`/`SistemaDialogo` que os NPCs procuram existam onde a multicena roda | Mapa |
| K6 | Regenerar as áreas com as estruturas altas no horizonte (código pronto, **não rodado**: o Editor caiu ao abrir a origem de 76 MB) | Mapa |

## 9. Expansões futuras (sem prazo)

- **Ginásios e insígnias** (S4): a infraestrutura (perfil de IA, flags, diálogo) já existe quase inteira; falta a regra "este treinador concede insígnia" e o que a insígnia libera.
- **Localização**: decisão é PT-BR único ([`DECISOES.md`](DECISOES.md)).
- **Missões e conquistas** (S5, S6).
- **Mais formas em batalha**: o mecanismo está pronto; falta arte e conteúdo.
- **Mais áreas conectadas**: suportado (`SistemaTransicao` + `PontoTransicao` + `CarregadorDeAreas`); falta conteúdo.
- **Dificuldade de jogo configurável**: decidido em 11/10 (DECISOES T47): só a IA dos treinadores, mais uma opção separada de IVs; no save do mundo. Falta preencher os perfis dos NPCs e ligar o dropdown dos IVs na cena do menu (`ConfiguracoesMenu.dropdownIvsDosTreinadores`; é opcional).
- **Substituir `RegraDeBatalha`/`EfeitoDeBatalha` por handlers nomeados por convenção, ao estilo Showdown**: refinamento sem prazo e **sem decisão** ([`../Planos/BATALHA_BARRAMENTO_E_DETERMINISMO.md`](../Planos/BATALHA_BARRAMENTO_E_DETERMINISMO.md)).
