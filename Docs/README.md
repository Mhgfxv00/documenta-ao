# Documentação do Nel Ord — índice

| | |
|---|---|
| **O que este documento responde** | Qual documento responde a minha pergunta, e como a documentação é organizada e mantida |
| **Quando abrir** | Sempre que não souber onde algo está escrito, ou antes de criar um documento novo |
| **Conferido em** | 09/10/2026 (reorganização completa: o antigo README de 3.223 linhas foi dividido; documentos dispersos entre a raiz, `Assets/` e `Assets/Scripts/` foram reunidos aqui) |

## Como a documentação se organiza

Cada documento tem **um papel**. Quando dois discordam, vale o da coluna "fonte de verdade".

| Pasta | Papel | Verbo |
|---|---|---|
| [`Projeto/`](Projeto/) | A situação, o que falta, o que foi decidido e como trabalhar | **decidir e fazer** |
| [`Jogo/`](Jogo/) | O jogo e o código: o que é, onde mora, como funciona | **entender** |
| [`Mundo/`](Mundo/) | Mapas, cenas, montagem de conteúdo no mundo, desempenho | **montar** |
| [`Planos/`](Planos/) | Trabalhos grandes **ainda não feitos**, com desenho | **planejar** |
| [`Historico/`](Historico/) | O que foi feito e por quê. Arquivo: **não é fonte de verdade** | **lembrar** |

## Qual documento responde…

### Projeto — [`Projeto/`](Projeto/)

| Documento | Responde | Fonte de verdade para |
|---|---|---|
| [`ESTADO_DO_PROJETO.md`](Projeto/ESTADO_DO_PROJETO.md) | Onde estamos agora? O que funciona, o que trava, quais os números | a situação **de hoje** (números contados em 09/10/2026) |
| [`PENDENCIAS.md`](Projeto/PENDENCIAS.md) | O que falta, com ID, esforço e dono | **o que está aberto** |
| [`DECISOES.md`](Projeto/DECISOES.md) | O que já foi decidido? O que espera resposta e de quem? | as decisões (T = tomadas, P = pendentes) |
| [`COMO_TRABALHAR.md`](Projeto/COMO_TRABALHAR.md) | Como abrir, rodar, testar; regras de git, conteúdo, save, desempenho; plano de merge | as regras de trabalho |
| [`FERRAMENTAS_DO_EDITOR.md`](Projeto/FERRAMENTAS_DO_EDITOR.md) | O que cada menu `Pokemon/…` e `Omega/…` faz; quais estão congelados | o catálogo de ferramentas |

### Jogo — [`Jogo/`](Jogo/)

| Documento | Responde | Fonte de verdade para |
|---|---|---|
| [`GDD.md`](Jogo/GDD.md) | O que é o jogo? Que regras de desenho valem sempre? | **como o jogo é** (descritivo) |
| [`REFERENCIA_DE_SISTEMAS.md`](Jogo/REFERENCIA_DE_SISTEMAS.md) | Onde mora cada arquivo? Como crio uma regra, um efeito, um evento? Como funciona o roteiro? | o mapa do código |
| [`MODULOS_DO_JOGO.md`](Jogo/MODULOS_DO_JOGO.md) | Como cada sistema funciona **e por quê**? (clima, encontros, status, itens, formas, IA, evolução, save…) | o desenho de cada sistema |
| [`BATALHA_ATE_O_OFICIAL.md`](Jogo/BATALHA_ATE_O_OFICIAL.md) | O que separa a batalha da dos jogos oficiais? Em que ordem fechar? O que cada leva fez? | o **plano da batalha** |
| [`AUDITORIA_OFICIAL.md`](Jogo/AUDITORIA_OFICIAL.md) (+ [`auditoria_oficial/`](Jogo/auditoria_oficial/)) | O projeto tem tudo que o oficial tem? E o que ele tem que não é oficial? | a comparação de conteúdo com o oficial |
| [`CALLBACKS_DO_SHOWDOWN.md`](Jogo/CALLBACKS_DO_SHOWDOWN.md) | Cada gancho do Showdown tem peça equivalente aqui? Quais faltam? | o cruzamento callback × peça |
| [`UI_DO_PROJETO.md`](Jogo/UI_DO_PROJETO.md) | Que telas o jogo precisa? O que existe? Em que ordem? | **toda a UI** |
| [`MENUS_UGUI.md`](Jogo/MENUS_UGUI.md) | Como montar os painéis uGUI na cena? | montagem no Editor (uGUI) |
| [`INTERACAO_POR_CARDS.md`](Jogo/INTERACAO_POR_CARDS.md) | O protótipo de interação por grafo de cards | o protótipo |
| [`ANIMACAO.md`](Jogo/ANIMACAO.md) | Onde a animação vai morar? | regras de animação |

### Mundo — [`Mundo/`](Mundo/)

| Documento | Responde | Fonte de verdade para |
|---|---|---|
| [`CATALOGO_DE_PREFABS.md`](Mundo/CATALOGO_DE_PREFABS.md) | Que prefabs existem para montar mapa e onde cada um mora? | **índice de prefabs** (gerado) |
| [`PACOTES_DE_TERCEIROS.md`](Mundo/PACOTES_DE_TERCEIROS.md) | Que pacotes de arte de fora o projeto tem, de onde vieram e com que licença? | **pacotes de terceiros** |
| [`ESTRUTURA_DO_PROJETO.md`](Projeto/ESTRUTURA_DO_PROJETO.md) | Que pasta existe, o que vai em cada uma e onde pôr um arquivo novo? | **mapa de pastas** |
| [`MONTAGEM_DE_MAPAS.md`](Mundo/MONTAGEM_DE_MAPAS.md) | Como monto um mapa, uma porta, um NPC, uma loja, um roteiro? O que ainda não dá? | **receitas de montagem** |
| [`COMO_MONTAR_A_CENA.md`](Mundo/COMO_MONTAR_A_CENA.md) | Como se liga a cena do zero (sistemas, assets, jogador, grama, batalha)? | a montagem manual de uma cena |
| [`MAPAS_E_DESEMPENHO.md`](Mundo/MAPAS_E_DESEMPENHO.md) | Como se mede? O que foi medido? Diretrizes para mapas leves | números de desempenho dos mapas |
| [`KANTO_REVISAO_E_MULTICENA.md`](Mundo/KANTO_REVISAO_E_MULTICENA.md) | Por que o mapa é assim? Como e por que foi dividido em cenas por área? | a razão da estrutura do mapa |

### Planos — [`Planos/`](Planos/)

| Documento | Responde |
|---|---|
| [`REDE_E_MULTIPLAYER.md`](Planos/REDE_E_MULTIPLAYER.md) | O que existe de rede, o que falta para a primeira partida, o que já foi decidido |
| [`ESCOLHA_DO_PACOTE_DE_REDE.md`](Planos/ESCOLHA_DO_PACOTE_DE_REDE.md) | Como se escolhe o pacote de netcode? Candidatos, critérios, custo e o spike que decide |
| [`TESTE_DE_CONFORMIDADE.md`](Planos/TESTE_DE_CONFORMIDADE.md) | Como medir em número o quanto a batalha bate com o Showdown (D13) e o que a batalha sem tela (D14) precisa |
| [`CO_PRESENCA_NO_MUNDO.md`](Planos/CO_PRESENCA_NO_MUNDO.md) | Como preparar o mundo para vários jogadores sem escolher pacote de rede |
| [`BATALHA_BARRAMENTO_E_DETERMINISMO.md`](Planos/BATALHA_BARRAMENTO_E_DETERMINISMO.md) | O desenho proposto para o motor (barramento único) e o que ficou decidido sobre determinismo |

### Histórico — [`Historico/`](Historico/)

Arquivo. Cada documento tem no topo o que ele foi e onde mora hoje o que ainda vale. **Não atualize**: acrescente ao arquivo só se for registrar uma lição.

| Documento | O que é |
|---|---|
| [`HISTORICO_DE_SESSOES.md`](Historico/HISTORICO_DE_SESSOES.md) | Log de marcos e lições, do mais recente ao mais antigo |
| [`PENDENCIAS_FECHADAS.md`](Historico/PENDENCIAS_FECHADAS.md) | Como cada item do backlog foi fechado |
| [`CONTEUDO_DO_SHOWDOWN_LEVAS.md`](Historico/CONTEUDO_DO_SHOWDOWN_LEVAS.md) | Qual peça implementa cada família de habilidade/golpe (levas 1 a 5) |
| [`BACKLOG_DO_README_ATE_2026-09.md`](Historico/BACKLOG_DO_README_ATE_2026-09.md) | O backlog antigo do README (setembro), com a análise de cada achado |
| [`CAPTURA_VALORES_DE_TESTE.md`](Historico/CAPTURA_VALORES_DE_TESTE.md) | Fórmula de captura e as chances com o conteúdo de teste |
| [`NOTAS_DA_CENA_DE_TESTE_2026-09.md`](Historico/NOTAS_DA_CENA_DE_TESTE_2026-09.md) | Conferência da `DevScene` de 02/09 e a avaliação do vertical slice |
| [`COMPARATIVOS_EXTERNOS.md`](Historico/COMPARATIVOS_EXTERNOS.md) | Comparação com Showdown, IA dos jogos, YAPU, Essentials, R.O.W.E. e outros ("vale copiar isso?") |
| [`AUDITORIA_ORGANIZACAO.md`](Historico/AUDITORIA_ORGANIZACAO.md) | A medição das invariantes do projeto, auditoria de áudio e rede (setembro) |
| [`GUIA_DE_IMPLEMENTACAO_2026-09.md`](Historico/GUIA_DE_IMPLEMENTACAO_2026-09.md) | O antigo `Assets/message.md`: planejamento técnico de setembro (o vivo foi para `Projeto/` e `Planos/`) |
| [`GUIA_DE_IMPLEMENTACAO_main_2026-09.md`](Historico/GUIA_DE_IMPLEMENTACAO_main_2026-09.md) | A versão do mesmo guia que estava na `main` (`Assets/message.md`), trazida no merge de 10/10 |
| [`CATALOGO_DE_FEATURES_2026-09.md`](Historico/CATALOGO_DE_FEATURES_2026-09.md) | Retrato das features do código em 16/09, feito na `main` e trazido no merge de 10/10 |
| [`plano_apresentado_e_nao_feito.txt`](Historico/plano_apresentado_e_nao_feito.txt) | Um relatório antigo de sistemas ausentes |
| [`sessoes/`](Historico/sessoes/) | Logs brutos das sessões de 30/08 a 05/09 |

---

## Como manter a documentação

1. **Um fato mora em um lugar.** Os outros documentos **apontam** para ele; não o copiam. (Foi a cópia — do aviso de UI em seis arquivos, das tabelas de pendência em três — que fez a documentação anterior divergir do código.)
2. **Resolveu algo do backlog?** Apague a linha de [`PENDENCIAS.md`](Projeto/PENDENCIAS.md) e explique em [`PENDENCIAS_FECHADAS.md`](Historico/PENDENCIAS_FECHADAS.md), **no mesmo commit** do código.
3. **Decidiu algo?** Mova a linha em [`DECISOES.md`](Projeto/DECISOES.md) de "esperam resposta" para "tomadas".
4. **Mudou como um sistema funciona?** Atualize a [`REFERENCIA_DE_SISTEMAS.md`](Jogo/REFERENCIA_DE_SISTEMAS.md) ou o [`MODULOS_DO_JOGO.md`](Jogo/MODULOS_DO_JOGO.md). O texto descreve o que **é**, nunca o que se pretende.
5. **Número sempre com recorte** (cena, máquina, data). "Não medi" quando não mediu. Os números do projeto estão em [`ESTADO_DO_PROJETO.md`](Projeto/ESTADO_DO_PROJETO.md) §2, com os comandos para recontar (§9): refaça a seção inteira, não uma linha.
6. **Documento novo** entra neste índice, com a pergunta que ele responde.
7. **Todo documento começa com a tabela** "O que responde / Quando abrir / O que NÃO é / Conferido em".
8. **Nada de documento dentro de `Assets/`**: a Unity cria um `.meta` e importa como asset. Nomes de arquivo citados em comentários de código (`COMO_TRABALHAR.md §6`, `BATALHA_ATE_O_OFICIAL.md §4`, `UI_DO_PROJETO.md §2.1`, `PENDENCIAS`) valem por nome: **mantenha esses nomes e numerações de seção**.

## Avisos para quem chega

- Os textos moram em português, sem acento nos nomes de arquivo e identificadores do código; nos documentos há acento.
- "Higor" é o autor e dono do motor de batalha, IA, regras e save; "klz" e o "grupo do mapa" cuidam do mapa e da arte; o "outro lado do grupo" cuida da UI. Quem é dono de quê: [`COMO_TRABALHAR.md`](Projeto/COMO_TRABALHAR.md) §1.
- Os documentos do [`Historico/`](Historico/) falam, em vários pontos, na primeira pessoa de uma sessão de trabalho que já acabou ("eu havia afirmado…", "esta máquina"). Leia como registro, não como instrução.
