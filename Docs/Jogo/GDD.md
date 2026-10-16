# GDD — o jogo como ele é

| | |
|---|---|
| **O que este documento responde** | O que é o jogo, como ele funciona em linhas gerais e que regras de desenho o projeto não quebra |
| **Quando abrir** | Para entender o jogo antes de ler o código; ao propor uma mudança de desenho |
| **Natureza** | **Descritivo, não aspiracional**: tudo aqui existe no código e foi verificado. O que ainda não existe está em [`../Projeto/PENDENCIAS.md`](../Projeto/PENDENCIAS.md), nunca aqui. É o inverso de um GDD normal — diz o que foi construído, para alguém entender o jogo sem ler 190 arquivos |
| **Conferido em** | 09/10/2026 contra o código (estados, eventos, avaliadores de IA, formato do save). Origem: seções 1 a 12 do antigo README, revistas |
| **Onde continuar** | Referência arquivo por arquivo: [`REFERENCIA_DE_SISTEMAS.md`](REFERENCIA_DE_SISTEMAS.md) · um sistema por seção: [`MODULOS_DO_JOGO.md`](MODULOS_DO_JOGO.md) · o que foi decidido: [`../Projeto/DECISOES.md`](../Projeto/DECISOES.md) |

## 1. O que é

RPG de exploração e captura de criaturas, **3D em terceira pessoa**, com **batalha por
turnos**. Dois estados de gameplay que se alternam: exploração e combate.

**Alvo declarado (04/09):** mundo grande, dividido em **múltiplas áreas conectadas**, com
**2 a 15–18 jogadores** simultâneos em rede. Isso não está construído — mas é o alvo que
decide o que entra no backlog e em que ordem.

## 2. O loop

```
Explorar ──► Encontrar ──► Interagir ──► Batalhar ──► Capturar / derrotar / fugir
   ▲                                                              │
   └──────────────── Progredir ◄──── Gerenciar equipe ◄───────────┘
```

Três laços secundários, todos fechados hoje: capturar → montar equipe → testar; explorar →
achar item → equipar/usar; batalhar → XP → evoluir → enfrentar mais.

## 3. Estados

O jogo tem **cinco** estados (`EstadoJogo`), e quem manda no controle do jogador é o
`GerenciadorEstados` — nunca a UI, nunca o jogador:

| Estado | Controle do jogador | Quem entra nele |
|---|---|---|
| `Exploracao` | livre | padrão |
| `Batalha` | bloqueado | `SistemaBatalha` |
| `Menu` | bloqueado | `Menu` |
| `Dialogo` | bloqueado | `SistemaDialogo` |
| `Transicao` | bloqueado | `SistemaTransicao`, até o jogador ser posicionado na cena nova |

A batalha tem a **sua própria** máquina de estados (`EstadoBatalha`), independente:
`Inativa → Inicio → EscolhaAcao → DeterminarOrdem → ExecutarAcoes → VerificarEstado →
ProximoTurno → Fim`.

## 4. Criatura: definição × instância

O princípio mais repetido do projeto. `EspecieData` (ScriptableObject, **compartilhado**)
guarda o que é permanente da espécie; `Pokemon` (classe serializável) guarda o indivíduo —
nível, HP, XP, golpes com PP, habilidade escolhida, held item, status, IVs, EVs e natureza.
Duas criaturas da mesma espécie são indivíduos diferentes.

O mesmo par se repete em toda parte: `StatusData`/`StatusAtivo`,
`EfeitoDeCampoData`/`EfeitoDeCampoAtivo`, `ItemData`/quantidade no inventário.

## 5. Batalha

Quatro ações (`TipoAcaoBatalha`): **Golpe, Troca, Item, Fuga** — enum fechado, por decisão
registrada. Suporta 1v1 e **dupla**, selvagem e treinador.

A peça central é a **`MesaDeRegras`**: a porta única por onde toda regra passiva é
consultada. Habilidade, held item, golpe em uso e efeito de campo são quatro *fontes* de
regra; a mesa junta as quatro e responde. Quem calcula dano, precisão ou prioridade nunca
sabe que habilidade existe.

**Ordem de resolução, declarada e total** (três critérios): `prioridade` → `OrigemDaRegra`
(habilidade → held item → golpe → campo) → posição na lista da fonte. Duas regras nunca
ficam empatadas decidindo no acaso da coleta.

Regras **consultam** (`RegraDeBatalha`), efeitos **aplicam** (`EfeitoDeBatalha`), e
gatilhos (`GatilhoDeEfeito`) ligam os dois a **27 eventos** de batalha. Habilidade nova
costuma ser dado novo, não código novo.

**Mecânicas de formato.** Além dos quatro verbos, um golpe pode ser escolhido junto de um *modificador*: **Mega Evolução**, **Z-move**, **Dynamax** e **Terastalização**. Cada lado usa cada uma **uma vez por batalha** e um Pokémon usa no máximo uma. Todas existem e passam pela rede como dado (`AcaoBatalha.UsarZ/Dinamaxizar/Terastalizar`), mas **só a Mega está ligada**: as outras três estão desligadas em `SistemaBatalha.MecanicasPermitidas` e o jogo ainda não decidiu quais usar ([`../Projeto/DECISOES.md`](../Projeto/DECISOES.md) P9; [`BATALHA_ATE_O_OFICIAL.md`](BATALHA_ATE_O_OFICIAL.md) §12 e §13). Cada treinador NPC declara as que usa (`NpcTreinador.mecanicasQueUsa`).

## 6. IA de combate

Existe **um** cérebro (`CerebroDeIA`), estático e puro: recebe estado, devolve ação. Nunca
executa a batalha. A dificuldade é **parâmetro**, não classe — três eixos separados:

- **O que considera**: uma caixa por critério no `PerfilDeIA`, decidindo quais dos 11
  avaliadores chegam a contar. Todas nascem ligadas;
- **Estilo** (`EstiloDeIA`): como prefere lutar, entre ações igualmente viáveis;
- **Imperfeição**: `ruidoDeAvaliacao` (erra a estimativa) e `chanceDeErro` (percebe a melhor
  e escolhe outra) — separados de propósito, porque são coisas diferentes.

Um perfil pode ainda ser **reativo** (`decideDepoisDoJogador`): decide depois de saber a
ação do jogador. `CapacidadeDeIA` sobrevive só como **preset de autoria** — marca as caixas
de uma vez e nada em runtime a relê.

O jogador escolhe a **dificuldade** no menu, e ela seleciona qual dos perfis do
`NpcTreinador` entra na luta. Dificuldade nunca é número maior.

## 7. Progressão e mundo

XP e EV ao derrotar; nível sobe; evolução é **sistema de condições** (nível, item, amizade,
clima, hora), avaliada no fim da batalha para não trocar atributos no meio do turno; golpe
novo com ficha cheia pergunta o que esquecer.

O mundo tem **clima e período do dia** (`SistemaMundo`) que avançam com o tempo, e a tabela
de encontro muda com eles (`ContextoDeEncontro`). A batalha copia o clima do mundo ao
começar e passa a ter o **seu**: uma Dança da Chuva não muda o tempo lá fora. O HUD de
exploração (`AmbienteUI`) mostra período, clima e quanto falta para o período virar.

## 8. Persistência

Três slots, JSON em `Application.persistentDataPath`. O save grava **ids de texto**, e o `CatalogoJogo` reencontra os assets ao carregar — por isso id vazio ou asset fora do catálogo é falha grave, e há um validador que roda como teste. `SaveJogo.versao` recusa save de build mais nova em vez de lê-lo pela metade. O formato atual é o **5** (`SistemaSave.VersaoDoFormato`): lê do 0 ao 4 migrando na memória (o arquivo só vira v5 no próximo Salvar), escreve de forma atômica (arquivo temporário + reserva `.anterior`; se o principal estiver ilegível, a leitura cai para a reserva e avisa) e grava, entre outras coisas, o item segurado com o estado de "desativado", a habilidade concedida por item e o tipo Tera do indivíduo. Cada degrau de versão está comentado em `SistemaSave.cs`. Regra ao mudar o formato: [`../Projeto/COMO_TRABALHAR.md`](../Projeto/COMO_TRABALHAR.md) §5.

## 9. Como o jogo se monta

`Bootstrap.unity` é a cena inicial e a única dona de jogador, câmera, UI, sistemas e
EventSystem. As cenas de jogo contêm só o conteúdo delas. Dar Play direto em qualquer cena
funciona: `Inicializacao.GarantirSistemas` traz os sistemas por cima.

## 10. As três regras que o projeto não quebra

Não são aspiração — foram **medidas** por varredura no código inteiro (ver
[AUDITORIA_ORGANIZACAO.md](../Historico/AUDITORIA_ORGANIZACAO.md) §0):

1. **UI pede, sistema decide.** Nenhum botão escreve em `DadosJogador`/`Pokemon`.
2. **`MesaDeRegras` é a porta única das regras.** Zero consultas fora dela.
3. **`EntradaJogador` é o dono único do mapa de input.** Zero `new Inputs()` fora dele.

## 11. Estado para rede

Pronto: ação como **dado** (`AcaoDeRede`, sem referência de objeto), **autoridade** com
ponto único de recusa (`IArbitroDeBatalha`), **ritmo** separado da regra
(`IRitmoDeBatalha`), **registro** de acontecimentos tipados e reproduzíveis
(`RegistroDaBatalha` + `ReprodutorDeBatalha`), **foto** serializável do estado, e sorteio
semeado por batalha.

Falta: transporte, árbitro remoto, e reconstruir a batalha a partir da foto — lista completa em [`../Planos/REDE_E_MULTIPLAYER.md`](../Planos/REDE_E_MULTIPLAYER.md).

## 12. Volume de conteúdo

Os números atuais (espécies, golpes, habilidades, itens, treinadores, encontros…) moram em [`../Projeto/ESTADO_DO_PROJETO.md`](../Projeto/ESTADO_DO_PROJETO.md) §2.4, porque envelhecem. Boa parte veio de importação (PokeAPI, Showdown, FireRed) e foi depois corrigida à mão; o conteúdo de fã foi removido em 07/10 e os geradores de dados, congelados ([`../Projeto/COMO_TRABALHAR.md`](../Projeto/COMO_TRABALHAR.md) §4.1).

O critério é literal: **todo mecanismo do projeto tem pelo menos um asset que o usa em partida**, e o `AuditorDeAlcancabilidade` roda como teste para impedir que isso volte a não ser verdade. "Implementado e testado" e "alcançável jogando" são perguntas diferentes: a suíte monta o próprio conteúdo e só responde a primeira. Exemplos do que a varredura achou e corrigiu: `Mutacao` (Protean, via `GolpeEscolhido`), `Teimosia` (via `AtributoMudou`), `Pressagio` (via `AntesDoDano`, alimentando o contador que a evolução do Charmeleon exige), o `Colete de Assalto` (proibição de golpe) e o `Amuleto de Alívio` (via `StatusRemovido`), os dois à venda na loja. Característica de golpe sem nenhum golpe que a carregue é filtro morto.

## 13. Critérios de sucesso do vertical slice

Os quatro fluxos que o GDD original definiu como critério, completos de ponta a ponta e sem travar. Avaliados em 02/09 contra o código (a avaliação linha a linha está em [`../Historico/NOTAS_DA_CENA_DE_TESTE_2026-09.md`](../Historico/NOTAS_DA_CENA_DE_TESTE_2026-09.md)); **os testes de PlayMode que dirigem o `SistemaBatalha` cobrem os fluxos 1 a 4**, mas a suíte inteira não foi relida em 09/10:

1. Explorar → grama → encontro → batalha → golpe → derrota do selvagem → XP → volta ao mapa.
2. Encontro → Pokébola → captura → o Pokémon entra na equipe (ou na caixa, se cheia). *Mecanicamente completo; a fórmula é a oficial da Gen III+ ([`../Historico/CAPTURA_VALORES_DE_TESTE.md`](../Historico/CAPTURA_VALORES_DE_TESTE.md)).*
3. NPC → diálogo → fecha → continua explorando.
4. NPC treinador → diálogo de desafio → fecha → batalha (equipe inteira, sem captura) → vitória → recompensa em dinheiro → flag registrada (não desafia de novo, a menos que `permiteRevanche`).

Além da linha de base já entraram: efeitos de status, batalha dupla, evolução, loja, itens segurados, IA com capacidade/estilo/imperfeição, IA usando item e apoiando o parceiro, formas de batalha reversíveis, roteiros de história e as mecânicas de formato.
