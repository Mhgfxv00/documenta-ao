# Escolha do pacote de rede — como decidir (R1 / P13)

| | |
|---|---|
| **O que este documento responde** | Como se escolhe o pacote de netcode e de transporte; o que pesa neste projeto; quais são os candidatos e o que cada um custa; como a escolha é feita sem apostar no escuro |
| **Quando abrir** | Antes de instalar qualquer pacote de rede. É o desdobramento de [`REDE_E_MULTIPLAYER.md`](REDE_E_MULTIPLAYER.md) R1 e da decisão P13 em [`../Projeto/DECISOES.md`](../Projeto/DECISOES.md) |
| **Estado** | **Nada escolhido.** Este documento é o método e a análise; a decisão é do grupo. Nenhum pacote de netcode está instalado |
| **Conferido em** | 09/10/2026. Os fatos sobre pacotes vêm de busca na web nesta data (fontes no fim); o que **não** foi conferido está marcado como *a conferir* |

---

## 1. Primeiro: o que o projeto exige, independente do pacote

A escolha de pacote só faz sentido depois destas respostas, porque elas eliminam candidatos.

| Exigência | De onde vem | Consequência na escolha |
|---|---|---|
| **A batalha é por turnos e já é "servida por árbitro"** (`IArbitroDeBatalha`, ação como dado, registro reproduzível) | [`REDE_E_MULTIPLAYER.md`](REDE_E_MULTIPLAYER.md) §1 | Quase nada de batalha depende do pacote: ele só precisa **entregar uma mensagem confiável** (a `AcaoDeRede`) e a resposta. Batalha **não** é o critério de desempate |
| **O mundo é o que decide**: avatares, 2 a 15–18 jogadores, várias cenas por área | T20 em [`../Projeto/DECISOES.md`](../Projeto/DECISOES.md) | Pesam: sincronização de movimento, **relevância / interest management** por cena e distância, e como o pacote lida com **cenas aditivas** |
| **Autoridade no servidor** para PvP e para troca/captura (anti-trapaça) | decisão 3 e 4 de §3 do plano de rede | Precisa de modelo servidor-autoritativo. Fora o P2P puro sem dono |
| **O motor de batalha é C# de `MonoBehaviour` + corrotina** (`RotinaBatalha`) | `SistemaBatalha*.cs` | Um **servidor dedicado** precisa rodar a batalha **sem tela** (headless). Isso é trabalho do projeto, não do pacote, mas decide se "servidor dedicado" é viável cedo (ver §5) |
| **Determinismo fica** (semente + ações aceitas = mesma batalha) | T5 | A batalha roda **em um lugar só** (o dono da partida) e os outros recebem o resultado. Não é preciso um netcode de "lockstep" nem de rollback |
| **Ids são `string`** | catálogo | Irrelevante por turno; caro em alta frequência. Vale para o **movimento**, não para a batalha: movimento deve usar posição, não ids |
| **Não é DOTS/ECS** | o projeto inteiro é `MonoBehaviour` | **Netcode for Entities está fora.** Pede reescrever o jogo em ECS |
| **Unity 6000.5.10f1** | `ProjectVersion.txt` | Só pacotes que suportem Unity 6. O Netcode for GameObjects **1.x deixou de ser suportado a partir do 6000.3**; só a **2.x** serve |

Perguntas que o **grupo** precisa responder e que mudam a escolha (não dá para inferir do código):

1. **Quem hospeda?** (a) um jogador (host) com Relay para atravessar NAT; (b) servidor dedicado próprio ou alugado; (c) nuvem do fornecedor (Photon). Muda custo, anti-trapaça e quem paga a conta.
2. **Quantos jogadores simultâneos esperam de verdade?** 2 a 4 amigos é muito diferente de uma sala de 18 ou de milhares.
3. **Plataformas**: só PC? Mobile? Console? Cada uma tem restrição de transporte (WebGL, por exemplo, pede WebSocket).
4. **Orçamento mensal** aceito para infraestrutura. Zero muda tudo.
5. **Lançamento comercial ou projeto de grupo?** Licença e custo por jogador ativo entram.

---

## 2. Os candidatos

| Candidato | O que é | Licença / custo | Fortes | Fracos neste projeto |
|---|---|---|---|---|
| **Netcode for GameObjects (NGO) 2.x** + Unity Transport | Pacote oficial da Unity. Modelos: cliente-servidor **ou** *distributed authority* (a autoridade se distribui entre clientes) | Pacote gratuito. Relay/Multiplayer Services do Unity têm cota grátis e cobrança por uso (*a conferir o preço atual*) | Suporte oficial; versão acompanha o Editor; Relay e lobby integrados; `com.unity.multiplayer.center` já instalado; a Unity mantém exemplos (Boss Room, citado no plano) | Interest management mais básico (*a conferir*); menos maduro em predição do que o Fish-Net |
| **Mirror** | Pacote de comunidade, sucessor do UNet | MIT, gratuito | Maduro, muita documentação, muitos jogos; interest management embutido (*a conferir*) | Sem suporte oficial; a manutenção depende da comunidade |
| **Fish-Net** | Pacote de comunidade focado em desempenho e recursos | Núcleo gratuito; **Pro** (pago, em torno de US$ 50–60 pontual, *a conferir*) acrescenta, por exemplo, *lag compensation* | Predição de cliente forte; observers por cena/distância; bom para mundo com muitas cenas (*a conferir*) | Mantido por poucas pessoas; parte dos recursos atrás de pagamento; menos oficial |
| **Photon Fusion 2** | Netcode em nuvem do fornecedor, simulação por *tick* | Grátis até 100 CCU; planos pagos acima (500 CCU ≈ US$ 125/mês em fonte de terceiros, *a conferir*) | Sem operar servidor próprio; matchmaking e relay prontos | **Preso ao fornecedor e a custo recorrente**; o modelo por *tick* de simulação encaixa pior num jogo por turnos; ids `string` e estado do catálogo não são o formato nativo dele |
| **Netcode for Entities** | Netcode oficial para DOTS | — | Muito escalável | **Descartado:** exige ECS |
| **Transporte próprio** (só o Unity Transport + código nosso) | Pacote de transporte oficial, usado por NGO e Netcode for Entities, também usável sozinho | Gratuito | Controle total; o nosso `AcaoDeRede` já é "mensagem como dado" | Reescrevemos spawn, sincronização de objetos, relevância e reconexão. É o caminho mais caro; só se justifica se nenhum pacote servir |

Observação: a comparação de **desempenho puro** entre NGO, Mirror e Fish-Net importa pouco aqui. Um mundo de até 18 jogadores com batalha por turnos está bem abaixo do que qualquer um deles aguenta. O que decide é **recurso (relevância, cena aditiva, predição), custo e risco de manutenção**.

---

## 3. Como pontuar

Tabela a preencher no spike (§4). Nota de 0 a 3; peso de 1 a 3 definido pelo grupo.

| Critério | Peso sugerido | Como medir |
|---|---:|---|
| Cabe no modelo servidor-autoritativo com `IArbitroDeBatalha` | 3 | Escrever o `ArbitroRemoto` mínimo e ver se a espera "Pendente" encaixa sem mexer no `SistemaBatalha` |
| Relevância / interest management por cena e distância | 3 | Dois clientes em cenas diferentes não recebem os objetos um do outro; um terceiro perto recebe |
| Convive com **cenas aditivas** e o `SistemaTransicao` (hoje `LoadSceneMode.Single`) | 3 | Trocar de cena de um jogador sem derrubar o outro |
| Custo mensal no cenário esperado (§1, perguntas 1 e 2) | 3 | Conta com CCU esperado |
| Risco de manutenção (quem mantém, ritmo de versões, suporte ao Unity 6) | 2 | Último release, issues abertas, mantenedores |
| Reconexão e queda de cliente (R8) | 2 | Derrubar o cliente no meio de uma batalha e ver o que o pacote entrega |
| Curva de aprendizado do grupo | 1 | Tempo até o primeiro `NetworkBehaviour` funcionando |
| Predição / reconciliação do movimento | 1 | Só pesa se o movimento tiver de parecer imediato com latência alta |
| Depuração (ferramentas, simulação de latência) | 1 | `Multiplayer Tools` da Unity, simulador de rede do pacote |

---

## 4. O método: spike curto e idêntico nos dois finalistas

Escolher por tabela de recursos erra; escolher por **protótipo igual nos dois finalistas** acerta. Passos:

1. **Eliminar pelo §1.** Netcode for Entities cai. O resto fica.
2. **Escolher dois finalistas** pelas respostas do grupo (§6 dá uma inclinação).
3. **Fatia vertical idêntica**, 2 a 3 dias por finalista, no mesmo ramo descartável:
   - dois clientes no editor (Multiplayer Play Mode) numa cena da `KantoMulticena`;
   - cada um move o próprio avatar e vê o outro;
   - um terceiro cliente numa **outra** cena **não** vê os dois;
   - um dos dois inicia uma batalha 1v1 e o outro é o dono da partida: o `ArbitroRemoto` mínimo aceita uma `AcaoDeRede` pelo fio e responde `Pendente` até confirmar;
   - derrubar um cliente no meio da batalha.
4. **Medir:** linhas escritas, o que quebrou no código existente, tamanho das mensagens, latência com o simulador de rede ligado, custo no cenário esperado.
5. **Preencher a tabela do §3 e registrar o resultado** em [`../Projeto/DECISOES.md`](../Projeto/DECISOES.md) (mover P13 para tomadas) e **apagar o ramo do spike**. O código descartável não volta para o ramo principal.

Critério de parada: se **nenhum** dos dois passar o item de cena aditiva e relevância (peso 3) sem remendo grande, reabrir a opção "transporte próprio".

---

## 5. O que fazer antes, para o spike não esbarrar no projeto

Estes itens independem do pacote e destravam qualquer escolha. Ordem sugerida:

1. **Foto completa da batalha** (R3/R4/R6 do plano de rede; **feita em 13/10**, ver `O_QUE_SINCRONIZA.md`), incluindo o estado das mecânicas de formato: Dynamax ativo e turnos restantes, tipo Tera, se o lado já usou Mega/Z/Dynamax/Tera. Sem isso a retomada devolve uma batalha parecida, não a mesma.
2. **Modo de batalha sem tela** rodando em teste de EditMode, com semente fixa. É o que um servidor dedicado precisaria e o que dá o teste de conformidade com o Showdown (ver [`../Historico/COMPARATIVOS_EXTERNOS.md`](../Historico/COMPARATIVOS_EXTERNOS.md), seção de 09/10).
3. **Escrever "o que sincroniza"** (R10) antes do primeiro `NetworkBehaviour`.
4. **Revisar o `float` do dano** se o PvP for entre plataformas diferentes (nota de determinismo do plano de rede).
5. **Medir uma build** (R11/O11): sem linha de base não há meta de rede.

Os itens 1 e 2 não escolhem pacote; deixam a decisão barata de reverter.

---

## 6. Inclinação (não é decisão)

Se as respostas do grupo forem as mais prováveis (poucos jogadores, PC, sem orçamento de servidor, host por jogador):

- **Netcode for GameObjects 2.x + Unity Transport + Relay** como primeiro finalista: oficial, integra com o Editor 6 e com o que já está instalado, e o risco de abandono é o menor.
- **Fish-Net** como segundo finalista, **se** a relevância por cena e a predição se mostrarem mais fortes no spike.
- **Mirror** só se o grupo já tiver experiência com ele.
- **Photon Fusion** só se o grupo **não** quiser operar nenhum servidor **e** aceitar custo recorrente e dependência do fornecedor.

A inclinação muda se a resposta à pergunta 2 for "sala de 18 ou mais" (aí relevância pesa mais e o Fish-Net sobe) ou à pergunta 4 for "zero" (aí o host por jogador com Relay grátis, ou servidor próprio, sobe e a nuvem paga cai).

---

## Fontes e limites desta análise

- [Documentação do Netcode for GameObjects no Unity 6.3 LTS](https://docs.unity3d.com/Manual/com.unity.netcode.gameobjects.html): 2.x é a versão para o 6000.3; distributed authority faz parte da v2.
- [Aviso de fim do suporte à 1.x no 6000.3](https://discussions.unity.com/t/netcode-for-gameobjects-1-x-deprecation-in-6000-3-editor/1674997).
- [Comparação NGO × Mirror × Fish-Net (fórum Unity)](https://discussions.unity.com/t/so-what-is-the-difference-between-netcode-for-gameobjects-mirror-and-fishnet-which-one-is-better-for-fast-co-op/1678432): opinião de comunidade; consenso de que a qualidade da implementação pesa mais que a biblioteca.
- [Relay, preço](https://support.unity.com/hc/en-us/articles/4410136449812-How-is-the-Relay-Service-Priced): a cota grátis citada é de 2023; **o preço atual não foi conferido**.
- [Photon, 100 CCU grátis](https://blog.photonengine.com/?p=5466) e [guia de preços de terceiros](https://crux.supercraft.host/blog/photon-fusion-pricing-2026/): valores pagos **vêm de terceiros**.
- [Fish-Net Pro](https://fish-networking.gitbook.io/docs/master/pro-learning-resources-and-support): preço e recursos do Pro de snapshots de 2025.
- [Unity Transport](https://docs.unity3d.com/Packages/com.unity.transport@2.5/manual/install.html): base de NGO e Netcode for Entities.

**Não foi conferido nesta data:** o preço atual do Relay e do Multiplayer Services; os preços vigentes do Photon e do Fish-Net Pro; as afirmações sobre relevância/interest management e cenas aditivas de cada pacote (vêm de conhecimento geral, daí o *a conferir*). O spike do §4 existe justamente para tirar essas dúvidas com medição.
