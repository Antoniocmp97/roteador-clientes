# Roteador de Clientes — Hagamorfis

> Documento de contexto do projeto. Mantido atualizado a cada passo para permitir
> migração do chat para o Claude Code sem perda de contexto.
>
> **Última atualização:** 19/09/2026 — v7.8.0 (botões da parada: navegação em contorno âmbar)

---

## 1. Objetivo

Aplicativo web que lê pontos geográficos de clientes/filiais (arquivo GeoJSON,
originalmente criado no uMap) e traça rotas de visita **dentro do próprio app**,
sem depender de serviços externos de navegação.

Contexto de uso: empresa em Criciúma, Santa Catarina (Brasil). Interface em
português do Brasil.

---

## 2. Estado atual

**Arquivo principal:** `index.html` — HTML único, sem build,
sem dependências instaladas. Abre direto no navegador.

**Status:** funcional e testado pelo usuário.

### Funcionalidades implementadas

- Upload do **backup completo do uMap (`.umap`)** por clique ou arrastar-e-soltar.
  **Módulo 1 enxuto** (v7.4.0, escolha do usuário entre 5 propostas montadas numa
  página de comparação): o **título faz o trabalho do módulo**. Com base
  carregada ele é uma linha só — `1 · Clientes (6) ✓`, com o ✓ teal —, e as ações
  ficam em três **ícones** à direita, no padrão dos ícones do módulo 3:
  **corrente** (retomar), **seta para cima** (trocar base) e **✕ em círculo**
  (esquecer a base guardada, só quando existe uma). Contagem de clientes, nome do
  arquivo e data ficam na **dica do título**. Sem base, a caixa de upload é **uma
  linha de 38px** com ícone; soltar arquivo em qualquer ponto do módulo carrega, e
  com base carregada é o **módulo inteiro** que se acende ao receber o arquivo.
  A linha de avisos só aparece quando tem o que dizer (erro de arquivo, confirmação
  de "esquecer"). Medido no painel de 340px: **151px → 61px** sem base e
  **87px → 18px** com base.
  ⚠️ O que isso substituiu (v6.2 a v6.2.3): uma linha "✓ Base carregada" de 14,5px,
  que era a maior fonte do painel, repetia o título e vinha com dois links iguais
  em linhas seguidas ("esquecer base" e "trocar base"). O usuário: "não achei que
  ficou bom visualmente"
- ⚠️ **Dois toques para esquecer a base, sem texto no botão** (v7.4.0): como o
  botão virou ícone, o "confirmar?" que ficava dentro dele passou para a linha de
  avisos ("Clique de novo no ✕...") e o ícone fica vermelho enquanto armado.
  Desarma sozinho em 4s, como antes
- Parse das camadas: cada camada é um **cliente**, os pontos dentro dela são as
  **filiais**. Camadas vazias são ignoradas
- Checklist com **cascata de três níveis** quando o uMap tem grupos:
  **Grupo → Camada → Unidades**. Camadas sem grupo continuam no primeiro nível,
  lado a lado com os grupos
- **Base ordenada alfabeticamente** ao carregar, nos três níveis, com
  `Intl.Collator('pt-BR')` — acentos e cedilha entram no lugar certo. A ordem da
  rota não é afetada: quem decide a sequência da viagem é a otimização
- **Painel de largura ajustável**: a divisória entre o mapa e o painel é
  arrastável e a largura escolhida fica guardada no navegador (duplo clique
  volta ao padrão). Piso de **320px** no painel e na guia (v5.6): abaixo disso o
  nome da parada ficava com ~40px e quebrava letra a letra. Existe porque o mesmo site é usado numa TV de 1366x768 e
  em monitores Full HD — nenhuma largura fixa serve para as duas
- **Nome do cliente nunca é cortado**: quebra em duas ou três linhas quando o
  painel está estreito, em vez de terminar em "…" (o final do nome é justamente
  o que distingue uma unidade da outra)
- **Busca no checklist** (Fase 3): filtra por nome do cliente ou da filial;
  grupos com resultado abrem sozinhos durante a busca. Ao lado do campo, dois
  ícones (v6.6.0, sugestão de layout 6): **engrenagem** = tipos de serviço,
  **caixa marcada** = marcar/desmarcar todos; acendem em âmbar quando ligados e
  aparecem só com base carregada. Saíram do cabeçalho porque no painel de 340px
  os três itens pediam ~430px numa linha de 307 e cada um quebrava em duas — com
  isso o título voltou a caber inteiro (cabeçalho de 28px para 14px)
- **Parada avulsa** (v7.0.0): ícone de alfinete na linha da busca abre um painel
  com nome + coordenada colada e o 🎯 para marcar no mapa. Aceita o formato do
  Google Maps, espaço ou ponto-e-vírgula, vírgula decimal e URL de mapa colada;
  recusa texto sem números e coordenada fora de faixa. A parada entra em
  `clientPoints` e num grupo próprio do checklist ("PARADAS AVULSAS"), então vale
  em tudo: seleção, ★, tipo, rota, link e tela do campo. ⚠️ **Removida da viagem,
  ela é apagada** (v7.2.0): sai do mapa, de `clientPoints` e do checklist, e o
  grupo vazio some. Diferente da parada da base, que continua no mapa quando
  desmarcada — a avulsa só existe por causa daquela viagem, e antes ficava uma
  bolinha órfã (relatado pelo usuário). O 🎯 da origem e o da
  avulsa dividem o mesmo mecanismo, agora com **modo** (`'origem' | 'avulsa'`) —
  um desarma o outro. Sem nome digitado, o rótulo vem do endereço de volta
  (Nominatim). ⚠️ Vive só na sessão: recarregar limpa, como a seleção do dia; para
  recuperar, retoma-se pelo link
- **A cascata fecha ao selecionar** (v5.0): marcar uma filial fecha a cascata do
  cliente, para a lista não ficar poluída. O contador no cabeçalho (ex.: "1/9")
  e a bolinha âmbar continuam mostrando que há seleção ali dentro. Desmarcar
  **não** fecha. Dentro de um grupo do uMap, só a camada fecha — o grupo
  continua aberto
- **Base guardada no navegador** (Fase 3): depois de carregar o `.umap` uma vez,
  a base volta sozinha na próxima abertura. Também é a rede de segurança se o
  uMap sair do ar. Link "esquecer base" apaga a base guardada (com confirmação)
- **Tipo de serviço por parada** (Fase 2): lista configurável (editável pelo
  próprio escritório, salva no navegador — ex.: "Entrega de toner",
  "Manutenção"). Cada parada selecionada ganha um seletor para escolher o
  tipo; aparece na lista de paradas do escritório e como etiqueta na tela
  do campo
- **Modo noturno** (v3.3): botão no cabeçalho das **duas telas**
  (escritório e campo) troca o app inteiro entre tema escuro e claro. A escolha
  fica guardada no navegador e é aplicada antes da primeira pintura, para a tela
  não piscar no tema errado ao abrir. Padrão: escuro.
  **Desde a v7.6.0 é um ícone de 32px** (ponto 7 da revisão de design) que mostra
  o **estado atual** — lua no escuro, sol no claro —, no lugar do interruptor de
  trilho com "MODO NOTURNO" escrito ao lado (~110px em caixa alta). Continua
  sendo `<button role="switch">` com `aria-checked`: o CSS escolhe o ícone a
  partir dele e o leitor de tela anuncia a partir dele, num lugar só. A dica do
  mouse diz para onde o clique leva
- **Ações do cabeçalho num bloco só** (v7.6.0): `[disquete] [lixeira] | [tema]`
  encostados à direita, com um divisor de 1px entre o arranjo e o tema — 121px no
  total. Antes o disquete e a lixeira ficavam colados na marca, como se fossem
  parte dela. Em tela estreita (<760px) o arranjo some, como já era, e o divisor
  some junto
- **O mapa acompanha o tema**: no escuro usa o Esri Dark Gray com um tratamento
  de cor que reproduz a referência do usuário (ver seção 5); no claro **troca de
  ladrilhos** para o Esri Light Gray, que é onde ruas e rótulos foram desenhados
  para luz do dia. Rota, marcadores e popups não são afetados em nenhum dos dois
- Plot dos pontos no mapa (Leaflet) com popup mostrando filial e cliente
- Definição de origem, por três caminhos: geolocalização do navegador, endereço
  digitado (Nominatim, com viés geográfico para o sul de SC) ou **clique direto no
  mapa** (🎯), que dá precisão exata. Quando a busca devolve mais de um endereço
  possível, o app lista e o usuário escolhe
- **O mapa vai até a origem** assim que ela é definida pela localização ou por
  endereço (v3.5). Antes o pino era colocado sem mover o mapa: se caísse fora da
  vista, parecia que o botão não tinha feito nada
- **Endereço de conferência na localização** (v3.5, mantido na v3.7): depois do
  📍, o endereço do ponto é buscado de volta e aparece no campo de origem, para
  dar para conferir onde o pino caiu
- **O botão 📍 aceita a posição que o navegador devolve** (v3.7, igual à v2.5):
  uma chamada, sem opções, sem julgar a margem. Entre a v3.5 e a v3.7 houve
  tentativas de filtrar pela margem — todas pioraram o botão (ver seção 7)
- **Origem padrão salva** no navegador: definida uma vez, volta pronta a cada
  abertura — a operação sai quase sempre do mesmo lugar
- Checklist de seleção de quais clientes visitar na viagem
- Lista de paradas reordenável manualmente: **arrastando pela alça** (⋮⋮), como
  nas listas do celular (v3.8), ou pelas setas ▲▼ para mover uma casa; e remoção.
  Soltar uma parada no bloco das prioritárias a torna ★; soltar uma ★ entre as
  demais tira a estrela. Na divisa, quem decide é o título "Demais": soltou
  acima dele, a parada entra no **fim das ★** (v3.9); abaixo, fica no começo das
  demais.
  Esc cancela o arraste. O arraste começa só pela alça, então no celular o resto
  da linha continua rolando a lista. A alça tem o mesmo destaque âmbar translúcido
  da alça dos módulos, ao passar o mouse e durante o arraste (v6.1.6).
  **Em coluna de até 400px a parada fica em uma linha só** (v6.2): as setas ▲▼ saem
  e ★ ✕ ficam na linha do nome; em coluna mais larga as setas voltam
- **Todos os módulos em até três colunas** (v4.4–v4.7; ampliado na v5.7): ordem
  na tela **mapa · coluna 3 · coluna 2 · painel**. Os sete módulos — Clientes,
  Origem, Selecionar paradas, Ordem da viagem, Rota (botões + status),
  Enviar para o campo e Roteiro — têm alça no título.
  ⚠️ **Os títulos deixaram de ser numerados na v7.5.1** (ponto 6 da revisão de
  design, escolha do usuário entre numerar todos ou tirar): a sequência na tela
  era "1, 2, 3, 4, Rota, 5, Roteiro" e, com os módulos móveis desde a v6.0, o
  número não indicava mais a ordem em que eles aparecem. **Os apelidos internos
  (`m1`..`m7`) e este documento continuam falando em "módulo 3", "módulo 4"** —
  é o vocabulário do código, não o que aparece na tela. A alça passou a ser a
  versão "sem número" (24px) em todos; a marcação continua saindo do texto do
  título, não fixa no código. Arrastar pela alça até uma
  coluna a contorna em âmbar; soltar sobre o mapa abre a próxima coluna; Esc
  cancela; duplo clique leva à coluna seguinte (painel → 2 → 3 → painel).
  **Posição livre** (v6.0): soltar entre dois módulos põe ali — vale no painel e
  nas colunas (ex.: o 4 no topo, acima do 1). Durante o arraste, além do contorno
  da coluna, o módulo sai da tela e um **vão de encaixe** tracejado do tamanho dele
  (v6.1, escolha do usuário: "abrir espaço real") vai para onde ele cairia — os
  módulos em volta se afastam de verdade. O contorno âmbar da coluna só aparece
  quando o módulo **muda de coluna** (ou abre uma nova); na mesma coluna, só o vão
  (v6.1.1). O contorno é **só a borda**, sem preenchimento (v6.1.2). A alça do
  módulo é um **retângulo compacto em volta dos pontinhos e do número** do título
  (v6.1.5, pelo print do usuário): 39×30px, 47×44 no toque; Rota e Roteiro, sem
  número, só nos pontinhos (classe `sem-numero`, marcada no JS). Fica acima do
  número (`z-index:1`), com destaque âmbar translúcido. ⚠️ Histórico para não
  repetir: 15×13 (só os pontinhos) era pequeno; a v6.1.4 fez 60×64 invisível,
  estendendo para cima do título, e o usuário achou **desproporcional** — a área
  deve corresponder ao que se vê. Números dos títulos alinhados no mesmo x
  (v6.1.4: o 3 tinha 5px a mais). Vão limitado a 60% da altura visível do
  destino; sem destino, volta à origem. Posição calculada com o vão aberto (é
  estável). Eventos do arraste escutados na **janela**, porque a alça some junto
  com o módulo. Perto do topo ou do fim de uma coluna que rola, ela rola sozinha. O arranjo guarda `local` (coluna) e
  `ordem` (lista única, sempre agrupada por coluna — `canonizarArranjo`, para dois
  arranjos iguais terem o mesmo JSON). Duplo clique só troca de coluna e põe o
  módulo na posição da numeração. Arranjo salvo antes da v6.0 (sem `ordem`) abre
  na ordem da numeração. A divisória 3↔4 do painel só aparece com o 4 **logo
  abaixo** do 3 (classe `sem-divisoria-34` no body); fora disso vale o puxador de
  cada um.
  Regras: a coluna 3 **só existe junto com a 2** — se a 2
  esvazia, o conteúdo da 3 passa para ela. Módulo escondido (4 sem paradas, 5 e
  Roteiro sem rota) aparece na coluna como título + "aparece depois…", com alça
  própria; no painel continua só escondido.
  Estrutura: cada módulo é `<section class="modulo-movel" data-modulo="mN">` com um
  único `.modulo-corpo`. No painel a section é `display:contents` (painel idêntico
  ao de antes — medido); nas colunas vira a caixa. Mostrar/esconder é detectado
  por `MutationObserver` no corpo. ⚠️ Módulo 4 se esconde por `hidden`, não por
  `style.display` — o inline venceria o flex das colunas
- **Salvar arranjo / voltar ao normal** (v5.7): mover módulos vale só enquanto a
  página está aberta (regra da v4.6). No cabeçalho, dois ícones em SVG (pedido do
  usuário: ícones, não texto nem emoji): o **disquete** salva o arranjo e faz o
  site abrir assim (`hg_arranjo_modulos`) — acende em âmbar com bolinha quando a
  tela difere do salvo e vira ✓ por 1,8 s ao salvar; a **lixeira** volta ao
  normal: tudo no painel e **apaga** o salvo (confirma se havia um). Escolha do
  usuário: botão, e não gravação automática
- **Altura dentro das colunas — e no painel** (v4.9, generalizado na v5.7 e, para
  o painel, na v6.4): os módulos com lista
  (3, 4, Roteiro) dividem a sobra da coluna na proporção de `--peso`; os outros
  ocupam o que precisam. Entre dois módulos com lista na mesma coluna o JS cria
  uma divisória. Os pesos viram as alturas em pixels ao arrastar (proporção se
  mantém se a janela muda), são zerados na coluna quando um módulo entra ou sai, e
  fazem parte do arranjo salvo. Duplo clique divide igual.
  **Puxador na borda de baixo** (v5.9) do último módulo com lista de cada coluna —
  sem ele, um módulo sozinho na coluna não tinha controle de altura. O primeiro
  arraste passa a coluna para **altura fixa** (`arranjo.fixas[col]`, classe
  `.alturas-fixas`): o peso vira a altura em pixels, sobra espaço embaixo ou a
  coluna rola. Duplo clique no puxador volta a preencher (mesma proporção); na
  altura fixa o duplo clique na divisória iguala pela média. O modo é salvo no
  arranjo e zerado quando a coluna muda de módulos. ⚠️ Ao entrar no modo, medir
  **todos** antes de aplicar — aplicar um redistribui a coluna e o próximo é
  medido encolhido. Módulos 1, 2, Rota e 5 não têm puxador (sem lista).
  **O painel usa exatamente este mecanismo desde a v6.4** (`LOCAIS = painel, g1,
  g2`): ele deixou de rolar por baixo das listas — eram até 4 barras empilhadas —
  e os módulos com lista dividem a altura da tela. Só rola quando a janela é baixa
  demais para os mínimos de 120px. Em tela estreita (layout empilhado) volta o
  comportamento antigo: painel rolando, com teto por lista (200/170/220px)
- **Colunas com largura ajustável** (v4.8): cada coluna auxiliar tem divisória
  própria. Mínimo de 320px, teto calculado para tudo caber (painel + outra coluna
  + mapa de 380px), duplo clique volta a 360px. A largura **é** guardada na hora
  (`hg_largura_guia`, `hg_largura_guia2`), fora do arranjo. A largura máxima do
  painel também desconta as colunas abertas
- **Controles de altura antigos do painel, aposentados na v6.4** — ficam
  registrados porque explicam decisões que voltam a aparecer: divisória 3↔4
  (v4.3, soma constante), puxador do módulo 3 (v4.5/v5.4), puxador do módulo 4
  (v4.6, crescia empurrando o resto; a v4.2 fazia isso e foi substituída), e a
  chave `hg_alturas_modulos` com as alturas em pixels. Tudo isso saiu quando o
  painel passou a repartir altura como as colunas: hoje o ajuste é a **divisória
  entre módulos com lista** e o **puxador do último**, iguais nos três lugares.
  ⚠️ Lição da v5.8 que continua valendo: altura de lista nunca sai de
  `max-height` nem de medida da tela — com `max-height` uma busca de poucos
  resultados encolhia a caixa e travava os controles
- **Área de pegada da alça de arrastar** (v5.2 a v5.5): os seis pontinhos que
  movem a parada têm área de 54×altura-da-linha **quando a coluna é larga** e
  27 quando ela aperta — a troca é automática, por `@container`, e vale tanto no
  painel quanto na guia, cada um com sua largura. Os pontinhos desenhados
  continuam do mesmo tamanho. Os botões ★ ▲ ▼ ✕ são `flex:none`: nunca se
  deformam; quem cede espaço é a alça e depois o nome, que quebra linha
  ⚠️ A v5.1 tinha aumentado os botões ★ ▲ ▼ ✕ por engano — o pedido era a alça;
  os botões voltaram ao tamanho original
- **Paradas prioritárias** (★): as marcadas ficam fixas no início, na ordem
  escolhida, e "Otimizar ordem" reordena só as demais — para quando é preciso
  passar num lugar antes do resto do roteiro
- **Traçar nesta ordem** — rota respeitando a ordem escolhida (OSRM Route API)
- **Otimizar ordem** — OSRM Trip API recalcula a melhor sequência e reordena a
  lista automaticamente (`source=first`, `roundtrip=false`)
- **Módulo Rota sempre visível** (v6.5.0, sugestão de layout 5): o módulo gruda na
  coluna em que estiver — `position:sticky` com **top:0 e bottom:0**, então fica
  preso na base enquanto o lugar natural dele está abaixo e no topo depois de
  rolar por ele. Vale no painel e nas colunas, com o módulo em qualquer posição.
  Enquanto grudado fica compacto (classe `grudado`, calculada por uma sentinela
  logo depois do corpo): título, botões e o resumo km/min; a opção de retorno e o
  status voltam no lugar natural. ⚠️ **Folga de 120px para soltar** (v7.1.0): sem
  ela isto oscilava — compacto encolhe ~80px, a sentinela reaparece, solta, cresce
  e gruda de novo, e o painel ficava piscando (relatado pelo usuário). O
  observador de mudanças olha só painel e colunas; em `main` os ladrilhos do mapa
  disparavam verificações à toa. ⚠️ O título **fica**: a alça mora nele, e sem ela
  não dá para mover o módulo grudado. ⚠️ Dois tropeços registrados: `display:contents`
  não muda a árvore do HTML (o seletor precisa descer pela `section`), e a marca
  não pode ser atualizada dentro de `requestAnimationFrame` — o navegador pausa o
  relógio quando a aba não está desenhando
- **Voltar para a origem no fim** (v6.3): interruptor no módulo Rota, guardado no
  navegador (`hg_voltar_origem`), **ligado por padrão**. A origem entra como último
  ponto do traçado, o otimizador fecha o círculo (`destination=last`, com a origem
  nas coordenadas e o waypoint dela descartado ao remontar as paradas), o Roteiro
  ganha "Retorno · <origem>" e o link leva o retorno (formato v7). Trocar o
  interruptor desfaz a rota traçada — km, tempo e ordem mudam. Medido na base de
  teste: 16,2 km sem volta, 27,1 km com volta na mesma ordem, e 24,1 km quando o
  otimizador trabalha sabendo que precisa voltar
- Marcadores numerados conforme a ordem final de visita
- **Clicar na parada leva o mapa até ela** (v6.9.0): clique no nome ou no número
  centraliza o mapa na parada com zoom 16, abre o balão e acende o marcador. O
  zoom **nunca afasta** (se já estiver mais perto, só centraliza). Alça e botões
  mantêm a função deles. ⚠️ Em aba de segundo plano o `setView` animado do Leaflet
  não chega a ser aplicado (relógio de quadros pausado) — é artefato de teste,
  não do app
- **Botão que bloqueia e libera esse zoom** (v7.3.0, pedido do usuário): lupa no
  **título do módulo 4**, à direita — no módulo em que o clique acontece, então
  ela acompanha o módulo quando ele muda de coluna. Liberado (padrão) é a lupa
  com "+", discreta; bloqueado é a lupa riscada, acesa em âmbar como os ícones do
  módulo 3. **Bloqueado, o clique ainda acende o marcador e abre o balão** — só o
  mapa fica parado, que era o ponto em aberto: conferindo a lista com o mapa
  enquadrado num bairro, cada clique jogava a vista para outro canto. A escolha é
  guardada no navegador (`hg_zoom_parada`). ⚠️ Exceção de propósito: a **parada
  avulsa recém-criada** leva o mapa até ela mesmo bloqueado
  (`verParadaNoMapa(i, true)`) — o ponto acabou de nascer e pode estar fora da
  vista
- **Realce mapa ↔ lista** (v6.7.0, sugestão de layout 7): o mouse numa parada da
  lista acende o marcador dela (1,6x, anel âmbar, à frente dos vizinhos); o mouse
  num marcador acende a linha e rola a lista até ela (só nesse sentido, senão a
  lista se mexeria sob o ponteiro). Vale com a bolinha teal do cliente antes da
  rota e com a numerada depois. ⚠️ O `transform` vai no ícone **de dentro**: o
  elemento de fora é do Leaflet e carrega a posição do marcador
- Distância total e tempo estimado **logo abaixo de Traçar/Otimizar**, no módulo
  Rota (v6.2; antes ficavam no topo do Roteiro, no fim do painel). Instruções passo
  a passo agrupadas por parada no módulo Roteiro
- **Modo campo (Fase 1):** depois de traçar a rota, um botão gera um link com o
  roteiro inteiro. Quem abre esse link (ex.: recebido pelo WhatsApp) cai numa
  tela separada, feita para celular: lista das paradas na ordem certa, botão
  de navegação para Waze ou Google Maps em cada uma, e um botão para marcar a
  parada como concluída. O progresso marcado fica salvo no aparelho de quem
  abriu (sobrevive a fechar o navegador) e nenhum dado de cliente passa pelo
  servidor — o roteiro trafega inteiro dentro do link, no trecho depois do
  `#` (ver ADR-01 no documento de arquitetura).
  Desde a v6.3 o **retorno à origem** vai no link e aparece como último cartão,
  com ↩ no lugar do número, etiqueta RETORNO, navegação e marcação próprias; o
  cabeçalho conta "N paradas · + retorno".
  **A barra de progresso do dia** (teal, no cabeçalho, com "2/4" ao lado) só
  passou a aparecer na **v7.5.0** — existia desde a Fase 1 com largura zero,
  ver seção 5.
  **A parada da vez é a única em cartão inteiro** (v7.7.0, ponto 2 da revisão de
  design): ela ganha a etiqueta **AGORA** em âmbar e a borda âmbar que já tinha;
  a **já concluída vira uma faixa de 44px** (✓ teal, "nome · cliente" riscado) e
  a **pendente que não é a da vez, uma linha de 55px** (número, nome, cliente,
  ★ se prioritária). As duas **abrem ao toque** — é de lá que se navega fora de
  ordem e que se reabre uma parada marcada por engano —, e o topo do cartão
  aberto fecha de volta. O cartão da parada da vez **não fecha**: é a âncora da
  tela. Resolver uma parada (concluir ou reabrir) fecha o cartão dela sozinho, e
  a que está pedindo confirmação fica aberta à força, porque o par
  Confirmar/Cancelar mora dentro do cartão. Medido num celular de 375×812 com 3
  paradas + retorno: a página caiu de **1077px para 812px** — cabe sem rolar,
  que era o problema (todos os cartões tinham 204px e o grande não significava
  nada). ⚠️ O que está aberto vive **só na tela** (um `Set` ao lado do
  `confirmando`): recarregar volta ao padrão. O progresso salvo continua sendo
  só o conjunto de concluídas, por coordenada.
  **Hierarquia dos botões do cartão** (v7.8.0, ponto 3 da revisão de design):
  Waze e Maps ficam **sem preenchimento, com texto e contorno em âmbar** (forma
  pedida pelo usuário: "sem cor, porém o contorno com a cor laranja usada no
  layout"), e **"Marcar como concluída" é o único cheio** (teal). Numa parada já
  concluída o botão vira "✓ Parada concluída" — que é REABRIR, ação rara — e fica
  em contorno; é o inverso de antes. ⚠️ Até a v7.7.0 os dois botões de navegação
  eram âmbar cheio: a cor mais forte do app, reservada à origem e à rota, estava
  nos dois botões que fazem a mesma coisa por caminhos diferentes, e o único que
  muda o estado do roteiro era o mais apagado do cartão.
  **Formato v8** (v6.8.0): 7º grupo = retorno ("1"/vazio), 8º = **id do roteiro**
  (rid), 9º = **origem** "lat*lng*rótulo". No v7 a coordenada do retorno vinha no
  7º grupo; a leitura entende os dois, e links v5/v6/v7 continuam abrindo.
- **Retomar um roteiro pelo link** (v6.8.0, pedido do usuário: "meia hora depois
  surge mais uma parada"): colar o link no **módulo 1** (v6.8.1, sempre à mão — o
  campo do módulo 5 só existe depois de uma rota traçada; desde a v7.4.0 fica num
  **painel dobrável**, atrás do ícone de corrente do título, que abre com o foco no
  campo e fecha sozinho quando a retomada dá certo), no módulo 5 ou
  clicar em **"editar no escritório"** no rodapé da tela do roteiro (escondido em
  tela estreita; guarda o link e recarrega sem o `#`, retomando depois da base
  carregada). Volta ordem, ★, tipo, origem e o interruptor de retorno; o **id do
  roteiro é mantido**, e a rota é desfeita porque a lista mudou. Parada que não
  está na base **entra como avulsa** (v7.0.0), com aviso de quantas e quais — é o
  que faz um roteiro com avulsas voltar inteiro. **Sem base carregada o link
  espera**: fica guardado e a retomada acontece sozinha quando a base entra (v6.8.1)
- **Progresso do campo por parada e por roteiro** (v6.8.0): a chave é o id do
  roteiro (`hg_prog_r_<rid>`) e as marcas são a **coordenada** da parada
  ("retorno" para a volta). Antes era o código do link com **índices**: link novo
  zerava tudo e inserir no meio deslocava as marcas. Índices de links antigos são
  convertidos na primeira abertura; a contagem só considera paradas do roteiro
  atual. Medido: com 2 de 5 marcadas, o escritório inseriu uma parada na 2ª
  posição e regerou o link — as duas seguiram marcadas (uma delas já na 3ª
  posição) e a contagem virou 2/6

---

## 3. Stack técnica

| Camada | Escolha | Observação |
|---|---|---|
| Mapa | Leaflet 1.9.4 (CDN cdnjs) | Mesma base que o uMap usa |
| Tiles | Esri Dark/Light Gray Canvas (base + rótulos) | Sem cadastro. Substituiu o CARTO em 29/08/2026. O conjunto muda com o tema (v3.3) |
| Rotas | OSRM — `router.project-osrm.org` | **Servidor público de demonstração** |
| Otimização | OSRM Trip API | Resolve TSP aproximado |
| Geocodificação | Nominatim (OpenStreetMap) | Viés por caixa geográfica da região sul de SC. Sujeito a política de uso justo |
| Fontes | Space Grotesk, JetBrains Mono, Inter | Google Fonts |

### Identidade visual

Dois temas, escolhidos no interruptor do cabeçalho. **Toda** cor é variável
CSS — inclusive fundos de hover, chips e botões armados, que antes estavam
fixos no meio do CSS e não teriam como acompanhar a troca. O tema escuro
continua sendo o padrão e a identidade do app.

Variáveis em `:root` (tema escuro):

```
--ink:#0C1418  --panel:#16232B  --panel-2:#1D2E37  --line:#2A3D47
--text:#E7EEF2 --muted:#7C93A0  --amber:#F2A93C   --amber-dim:#B87F26
--teal:#2FB6A6 --danger:#E2604F
--hover:#22343E       --teal-fraco:#0e2422   --teal-hover:#132229
--amber-fraco:#2a1f0e --amber-claro:#FFC163  --teal-escuro:#28a094
--map-bg:#0C1418      --sombra:rgba(0,0,0,.4)
```

Tema claro em `:root[data-tema="claro"]`. **Não é o escuro invertido**: o âmbar
e o teal precisaram escurecer (`#C9821A`, `#12796E`) para continuarem legíveis
sobre branco — os tons do tema escuro sobre fundo claro ficam lavados. Já os
textos que ficam *sobre* preenchimento âmbar/teal (`#241705`, `#062420`) valem
nos dois temas, porque o preenchimento continua sendo a cor forte.

Âmbar = origem, rota traçada e paradas numeradas. Teal = clientes não
selecionados e estados de sucesso.

O escritório **não tem rodapé** (removido em 10/09/2026, a pedido do usuário).
O crédito obrigatório ao OpenStreetMap e à Esri fica no canto do próprio mapa,
no controle de atribuição do Leaflet (opção attribution da camada base) — é
exigência das licenças dos mapas, não remover. A tela do campo mantém o rodapé
dela ("Roteiro recebido por link · nada é enviado para servidor").

---

## 4. Decisões tomadas

1. **uMap não faz roteamento interno.** Ele apenas abre um link externo para o
   OSRM ao pedir direções. Por isso construímos app próprio sobre Leaflet + OSRM,
   usando o uMap apenas como ferramenta de criação/exportação dos pontos.
2. **Dados carregados por upload**, não fixos no código nem buscados de URL.
   Usuário carrega o backup do uMap (`.umap`) quando quiser.
3. **Múltiplas paradas** em vez de destino único — o caso de uso é visitar
   vários clientes na mesma viagem.
4. **Filiais de exemplo removidas** (Rio Maina / Próspera do primeiro protótipo).
   Apenas os pontos carregados pelo usuário aparecem.
5. **Arquivo HTML único** por enquanto — sem build step, sem framework.
6. **O alvo é monitor Full HD** (decidido em 11/09/2026). A TV de 1366x768 deixou
   de guiar as decisões de layout: limites de altura amarrados a telas pequenas
   foram removidos. O layout empilhado para telas estreitas continua existindo,
   mas não é mais o caso a otimizar.

---

## 5. Problemas resolvidos

**"Failed to fetch" na pré-visualização do chat.** A pré-visualização embutida
do Claude.ai bloqueia `fetch()` para domínios externos, quebrando as chamadas ao
OSRM e Nominatim. O código estava correto — funciona ao abrir o arquivo baixado
diretamente no navegador.

Solução aplicada: função `safeFetchJSON()` que envolve todas as chamadas de rede
e retorna mensagem explicativa em português quando a conexão falha, orientando o
usuário a baixar o arquivo.

**Marca d'água "API KEY REQUIRED" sobre o mapa (29/08/2026).** A CARTO passou
a exigir chave de acesso para os tiles `dark_all`. O servidor não recusava o
pedido — devolvia a imagem carimbada, sujando o mapa inteiro, inclusive no site
já publicado.

Solução aplicada: troca para o **Esri Dark Gray Canvas**, que serve sem
cadastro. Vem em duas camadas empilhadas (ruas + rótulos) e a ordem dos números
no endereço é `{z}/{y}/{x}`, invertida em relação ao CARTO. O Esri só tem imagem
real até o zoom 16 nesta região, então usamos `maxNativeZoom: 16` com
`maxZoom: 19` — o Leaflet continua deixando aproximar, ampliando a última
imagem disponível, em vez de deixar o mapa em branco.

Custo da troca: o fundo ficou cinza médio, mais claro que o preto-azulado
anterior, portanto menos integrado ao painel escuro. Alternativa avaliada e
descartada: Stadia Maps combinaria melhor visualmente, mas responde 401 fora do
`localhost` — exigiria cadastro e uma chave exposta no repositório público.

**Mapa claro demais (09/09/2026).** O custo acima virou incômodo real: o usuário
mandou uma referência (modo escuro do Waze) pedindo um mapa mais escuro.

Solução aplicada na v3.2: **filtro de cor por CSS sobre os ladrilhos**, sem
trocar de provedor — nenhuma requisição a mais, nenhum cadastro, nenhuma chave.
As duas camadas recebem filtros opostos (`.camada-base` escurece,
`.camada-rotulos` clareia), senão os nomes de rua sumiriam junto com o fundo.
A classe vai no `className` da camada, e não no estilo de cada ladrilho — assim
vale também para os que só chegam depois, ao arrastar ou dar zoom.

Foram comparados três tratamentos no mapa real, numa página de comparação
lado a lado com a referência: azul-marinho tipo Waze, azul suave e cinza
escuro. O usuário escolheu o **cinza escuro** — escurecer sem introduzir cor.

⚠️ Limite do método: como a origem é um mapa cinza, o filtro pinta tudo no
mesmo tom. Não dá para tratar parque, água e via principal com cores
diferentes, como faz o Waze. Isso exigiria um provedor **vetorial**
(OpenFreeMap, Protomaps), que é troca de stack, não ajuste de CSS.

**Parque "borrado" no mapa escuro (09/09/2026).** O usuário reclamou duas
vezes do mapa escuro: primeiro que os parques ficavam "distorcidos" (v3.2) e
depois, com um print, que um parque ficava "borrado" (v3.3).

⚠️ **A primeira medição que fiz estava errada** e levou a v3.3 para o lado
errado. Eu marcava como "parque" todo pixel esverdeado do ladrilho, o que
incluía as ruas que cortam o parque, e concluí que o parque saltava do fundo.
Medindo o parque isolado — máscara feita no ladrilho **claro**, onde o parque
é verde de verdade, e lida no ladrilho **escuro** — o parque está a **3 tons**
do fundo já no mapa original. Nunca foi uma mancha clara.

O que existia de verdade era **perda de definição**: a distância entre rua e
fundo, que é o que dá desenho ao mapa. Sem rua desenhada em volta, a área do
parque vira uma mancha sem forma — o "borrado" do print.

Luminância medida (0–255) no ladrilho do Parque Municipal Morro do Céu:

| Tratamento | Fundo | Rua | Parque | Definição (rua−fundo) |
|---|---|---|---|---|
| Referência do usuário (Waze) | 37 | 82 | 45 | **45** |
| Esri Dark Gray sem tratamento | 71 | 102 | 75 | 31 |
| v3.2 (contraste no escuro) | 13 | 30 | 15 | 17 |
| v3.3 (véu) | 26 | 34 | 27 | **8** |
| v3.4 (atual) | 37 | 82 | 42 | **45** |

Solução na v3.4: **a ordem das operações**, não a força delas.

```
brightness(1.425)  sobe o mapa até o fundo chegar perto do meio da escala
contrast(2.035)    abre a distância entre rua e fundo
brightness(.5)     desce de volta para o tom escuro
```

O `contrast()` do CSS gira em torno do **meio** da escala. Aplicado direto num
mapa escuro — que é o que a v3.2 fazia — ele não abre nada: empurra tudo para o
preto. Clareando antes, o contraste passa a trabalhar na faixa em que ele de
fato separa os tons.

Os três números foram **resolvidos** para bater com a referência, não
tentados: dá um sistema de duas equações (fundo final = 37, definição = 45).
Nenhum pixel satura — o ladrilho mais claro da região tem 163, e o primeiro
brilho só saturaria acima de 179.

O véu e o painel exclusivo dos rótulos, criados na v3.3, foram removidos: com
o tratamento novo eles não faziam mais nada.

**Tela do campo aparecendo embaixo do escritório (10/09/2026).** Relatado pelo
usuário como "no rodapé criou uma espécie de outra página". Rolando até o fim
do escritório, aparecia o cabeçalho da tela do técnico com o interruptor de
modo noturno, e depois uma área vazia.

Causa: a tela do campo tem o atributo hidden no HTML, mas o CSS dela declara
display:flex — e **qualquer display do CSS passa por cima do hidden**, que no
navegador é a regra mais fraca que existe. Existia **desde a v1.0**
(conferido no commit 2c87046): a página sempre teve uma segunda tela inteira
embaixo. Ficou visível o bastante para ser notado quando a v3.3 pôs o
interruptor no cabeçalho dessa tela.

Medido numa tela de 768 px: a página tinha 1536 px — 768 px sobrando abaixo do
rodapé. Solução na v4.0: regra global [hidden]{display:none !important}, que
faz o atributo sempre valer. Com ela, a página tem exatamente 768 px, e o link
do técnico continua abrindo a tela do campo normalmente (o JS tira o hidden).

**Barra de progresso do campo com largura zero (17/09/2026, v7.5.0).**
Achado numa revisão de design do app inteiro, a pedido do usuário. A tela do
campo tem barra de progresso desde a Fase 1 e **nunca apareceu**: o cabeçalho
dela é um `<header class="campo-head">`, e a regra geral `header{align-items:
baseline}` chegava nele. Numa **coluna** flex, qualquer `align-items` que não
seja `stretch` faz os filhos encolherem até o conteúdo — `.campo-progresso`
ficava com 33px e a barra, que é `flex:1` dentro dele, com **0px**. Restava só
o "0/4" de 12,5px no canto.

Solução: `.campo-head{align-items:stretch}`. Medido depois: barra de 307px num
celular de 375px, preenchimento em 50% com duas de quatro paradas marcadas.
De brinde, o interruptor de tema foi para a direita da tela (a linha do topo
também vinha encolhida), como no cabeçalho do escritório.

⚠️ **A lição**: regra de elemento (`header{...}`) alcança qualquer bloco que use
aquela tag, inclusive um com classe própria e layout diferente. A mesma pegadinha
já tinha aparecido na v4.0, com `[hidden]` perdendo para o `display` do CSS — em
ambos os casos o culpado foi uma regra genérica vencendo em silêncio, sem erro.

**Revisão de código completa (10/09/2026, v4.1).** A pedido do usuário, o
index.html inteiro foi revisado atrás de bugs. Seis achados, todos corrigidos e
testados um por um antes de passar ao próximo:

1. **"Marcar todos" tirava as ★ da frente** — trocava a lista pela ordem da
   base (reproduzido: ★ na 11ª posição de 39). Agora as já escolhidas ficam
   como estavam, com as ★ na frente, e as que faltavam entram depois.
2. **Botão do balão do mapa não fazia nada com a busca ativa** — ele dependia
   do checkbox, que a busca não desenha. Agora troca direto e avisa.
3. **Link do roteiro saía desatualizado** — mudar as paradas depois de traçar
   não desfazia a rota: o técnico podia receber outra lista, fora da ordem
   traçada, com km/min da viagem antiga. Nova função invalidarRota(),
   chamada em todo ponto que muda a lista. Mudar só o tipo de serviço mantém a
   rota e tira apenas o link já gerado.
4. **Carregar outra base deixava os números da rota antiga no mapa** — mesma
   função, chamada em importClients (e limpa o aviso "Rota traçada…").
5. **Balões do mapa interpretavam símbolos como HTML** — "POSTO <CENTRO>"
   aparecia como "POSTO ". Agora passam por escaparHtml.
6. **Abrir outro roteiro na mesma aba não trocava a tela** — o modo era
   decidido só ao carregar, e trocar apenas o # não recarrega. Um listener de
   hashchange recarrega quando o # é (ou deixa de ser) um roteiro.

---

## 6. Formato de dados esperado

**Backup completo do uMap (`.umap`)** — é o único formato que preserva a
separação por camada. No uMap: painel **"Compartilhar e baixar"** → backup
completo.

```json
{
  "type": "umap",
  "layers": [
    {
      "type": "FeatureCollection",
      "_umap_options": { "name": "LABORATORIO BURIGO" },
      "features": [
        { "type": "Feature",
          "geometry": { "type": "Point", "coordinates": [-49.3697, -28.6775] },
          "properties": { "name": "CENTRAL" } }
      ]
    }
  ]
}
```

**Cada camada é um cliente; cada ponto dentro dela é uma filial.** O nome do
cliente muda de lugar conforme a instalação do uMap, e o app lê as três formas,
nesta ordem:

| Onde | Instalação |
|---|---|
| `properties.name` | `umap.hotosm.org` — **é o que a base real usa** |
| `_umap_options.name` | instalações mais recentes do uMap |
| `_storage.name` | instalações antigas |

O nome da filial vem sempre de `properties.name` da feature. Coordenadas em
ordem GeoJSON: `[longitude, latitude]`.

### Grupos de camadas

O uMap permite agrupar camadas. No arquivo, o grupo é **uma camada especial**:
tem `properties.group: true`, **nenhum ponto próprio**, e carrega as camadas de
verdade num array `layers` aninhado dentro dela.

```json
{ "type": "FeatureCollection", "features": [],
  "properties": { "name": "PMNV", "group": true },
  "layers": [ { "type": "FeatureCollection", "features": [...],
                "properties": { "name": "PREFEITURA MUNICIPAL NOVA VENEZA - EDUCAÇÃO" } } ] }
```

⚠️ Ler só o primeiro nível fazia essas camadas **e todos os pontos delas
sumirem em silêncio** — sem erro, sem aviso. Verificado em 08/09/2026 com um
arquivo real: 23 camadas / 39 pontos no arquivo, 20 / 36 lidos pelo app.

O app agora percorre as camadas aninhadas. A lista interna continua **plana**
(uma entrada por camada com pontos), com o nome do grupo como campo a mais —
a árvore de três níveis é montada só na hora de desenhar o checklist. Por isso
seleção, rota, link e tela do campo não precisaram mudar.

Na tela do campo **nada muda**: a parada continua mostrando a camada como
cliente e a unidade como nome. O grupo é organização do escritório.

⚠️ **O download simples em `.geojson` não serve.** Ele achata todas as camadas
numa lista única e descarta os nomes — a informação de cliente não chega ao app.
Verificado em 29/08/2026 com um export real: as 11 features vinham com
`properties` contendo apenas `name`. O app recusa esse arquivo com uma mensagem
que ensina o caminho certo, em vez de carregar os pontos sem agrupamento.

⚠️ **Exportação do uMap só inclui camadas visíveis.** Camada com o "olho"
desligado fica de fora do arquivo. Se faltar cliente no app, é o primeiro lugar
a conferir.

Arquivo de exemplo versionado: `Exemplos/exemplo.umap` — 4 clientes fictícios
com 6 filiais na região de Criciúma. Cobre de propósito as **três** variantes de
nome de camada (`properties`, `_umap_options`, `_storage`) e inclui uma camada
vazia, para servir de teste de todos os casos.

O arquivo com os pontos reais fica **fora do repositório**, porque o
repositório é público (ver `.gitignore`).

## 7. Limitações conhecidas

- **OSRM público** é servidor de demonstração, sem garantia de disponibilidade
  nem uso comercial. Para produção: instância própria de OSRM ou GraphHopper
  (auto-hospedadas, gratuitas) ou API paga (Mapbox Directions, Google Directions).
- **Nominatim** tem política de uso justo — limite aproximado de 1 requisição por
  segundo. Volume alto de geocodificação exige alternativa.
- **A posição do 📍 vem do navegador de quem aperta o botão**, e a precisão é
  a do aparelho — nenhum código de página consegue mais do que isso.
  Confirmado em 10/09/2026:
  - **celular** (v3.7): posição real, a poucas quadras de onde o usuário estava.
    O botão funciona;
  - **os dois computadores do usuário** (Edge e Brave, permissões do Windows e
    do site todas liberadas): o próprio navegador, testado direto no console
    sem nenhum código do app, devolve o **centro de Criciúma com 50 km de
    margem**. É localização por cidade, não posição — o ponto caiu a 30 m do
    centroide da cidade no OpenStreetMap e a 2,8 km de onde o usuário estava.

  ⚠️ Correção de uma conclusão minha anterior: cheguei a escrever aqui que a
  margem de 50 km era "só o pior caso" e que o ponto servia. Estava errado — o
  erro era real. E o primeiro relato do problema aconteceu com a **v3.4**, cujo
  botão era idêntico ao da v2.5: a precisão caiu antes de qualquer mudança no
  código. Causa provável nos computadores: o Windows se localiza pelas redes
  Wi-Fi em volta; sem Wi-Fi ligado, sobra a estimativa pela internet.

  ⚠️ Lições das v3.5–v3.7, para não repetir:
  - `enableHighAccuracy: true` num computador sem GPS faz o navegador esperar
    um GPS que não existe e devolver ERRO no tempo limite (v3.5).
  - Enquadrar o círculo de uma margem de 50 km afasta o mapa até ~100 km de
    largura e, com a mensagem em vermelho, parece falha mesmo com o pino no
    lugar certo (v3.6).
  - Recusar posições acima de um limite de margem é recusar exatamente o que
    funcionava (v3.7 na primeira forma, nunca publicada).

  Regra: **não mexer nesse botão sem testar numa máquina real do usuário** —
  a aba de testes do Claude não tem permissão de localização, então todo teste
  lá é com leitura simulada.
- **Número de casa não funciona na busca de endereço.** Não é limitação do código:
  o OpenStreetMap tem pouquíssimos endereços numerados na região (verificado em
  06/09/2026 — apenas 122 em toda a área central de Criciúma). Digitar
  "Rua X, 178" cai na rua, não na porta. O app avisa quando isso acontece e
  oferece o clique no mapa (🎯) para marcar o ponto exato. Resolver de verdade
  exigiria trocar o Nominatim por um serviço com base própria de endereços
  brasileiros (Google, Mapbox), com cadastro e chave de acesso.
- **Persistência parcial.** Ficam salvos no navegador: a base de clientes, a
  origem padrão, a lista de tipos de serviço, a preferência de formato do link,
  a opção de voltar para a origem, o zoom ao clicar na parada,
  a largura do painel e das colunas, o arranjo dos módulos (só quando salvo pelo
  botão), o tema (claro/escuro) e o progresso do modo campo. **Não** ficam salvos:
  a seleção de paradas do dia, a ordem da viagem, a origem e a rota traçada —
  recarregar a página zera essa parte, de propósito (é o roteiro do dia, não
  configuração). Nada disso sai da máquina de quem usa.
- **Sem busca no checklist.** Com dezenas ou centenas de clientes, rolar a lista
  fica impraticável.
- **Otimização puramente geográfica.** Considera apenas distância/tempo de carro.
  Não trata janelas de horário nem duração da visita. Prioridade de parada é
  resolvida manualmente (★), fixando as primeiras — o otimizador não decide isso.
- **Sem navegação por voz dentro do app** — resolvido de propósito (ADR-02):
  o app não compete com Waze/Google Maps, só abre um deles por parada.

---

## 8. Direções em aberto

Decididas em 22/08/2026, registradas em detalhe no documento de arquitetura
(`ARQUITETURA-E-REQUISITOS.html`, fora do repositório público por descrever a
operação — ver `.gitignore`):

- **Quem usa:** dois papéis — escritório planeja, campo (2-3 pessoas) executa
  e marca progresso. Operação: entrega de toner + manutenção.
- **Origem da base de clientes:** uMap exportado como backup completo (`.umap`), volume
  pequeno (dezenas de pontos). Sem integração com ERP/planilha por ora.
- **Navegação:** app + botão para Waze/Google Maps por parada (ADR-02) —
  ✅ implementado na Fase 1.
- **Hospedagem:** GitHub Pages — ✅ feito, ver seção 10.
- **Como o roteiro chega ao campo:** link com o roteiro codificado no
  fragmento da URL (ADR-01) — ✅ implementado na Fase 1.

Ainda em aberto:
- Troca do OSRM público antes do uso diário sério (Fase 4)
- Revisar a decisão de manter arquivo único (Fase 3, ponto de decisão, não iniciado)

### Pendências para a próxima sessão (atualizadas em 18/09/2026)

**Revisão de design (17–18/09/2026).** O usuário pediu que eu olhasse o app como
designer e comparasse versões: "me sugira e compare as versões". Saíram **7
pontos**, montados lado a lado (hoje × proposta, no CSS e nas cores reais) na
página `COMPARACAO-DESIGN.html`, que fica fora do repositório.

Já feitos:
- **1 · Barra de progresso do campo invisível** → v7.5.0 (era defeito, não gosto;
  ver seção 5).
- **2 · Parada da vez em destaque no campo** → v7.7.0.
- **3 · Botões da parada** → v7.8.0. O usuário não quis nenhuma das duas
  variantes oferecidas: pediu navegação **sem preenchimento, com contorno
  âmbar**, e com isso o de concluir virou o único cheio.
- **6 · Numeração dos módulos** → v7.5.1, variante "tirar os números".
- **7 · Cabeçalho** → v7.6.0, ações agrupadas e tema como ícone.

**Em aberto — retomar perguntando quais liberar:**

- **4 · Módulo Rota: dois botões disputando.** "Traçar nesta ordem" e "Otimizar
  ordem" têm a mesma largura e pesos parecidos, mas o uso normal é otimizar.
  Proposta: **"Otimizar e traçar"** como principal (o botão diz o resultado, não
  a mecânica) e "Nesta ordem" como alternativa estreita.
- **5 · Um só idioma de ícones.** O app mistura emoji (🎯 📍 ★ ✕ ▲ ▼) com ícones
  de traço em SVG. Emoji muda de desenho entre o Windows do escritório e o
  Android da equipe, não acompanha o tema e tem peso visual diferente. Proposta:
  tudo em traço, 1,8px de espessura, 15–17px. É barato e aparece em todas as
  telas — seria o próximo que eu levaria.

**Sugestões de layout (14–15/09/2026), lista à parte e já encerrada.** Das 7, a
única que sobrou foi a **4 · rolagens dentro de rolagem**, ⚠️ **TENTADA E
REVERTIDA**: a v6.4 fez o painel funcionar como as colunas; o usuário testou e
não gostou (com rota traçada os três módulos com lista dividiam a tela, ~131px
cada numa janela de 1000px, contra 320/240 fixos). Desfeita na v6.4.1. Se o
assunto voltar, o caminho é o outro que foi oferecido: mexer só no Roteiro, que
deixaria de rolar sozinho, de 4 barras para 3.

**Também em aberto, de antes:**
- **Trocar o OSRM público** (Fase 4) — o usuário ficou de decidir onde hospedar.
- **Arquivo único** (Fase 3, ponto de decisão, não iniciado).
- **Nome do programa.** Em 17/09 ele pediu sugestões; foram dadas (Haga Rotas,
  Parada Certa, Percurso, Rota Viva, Farol, entre outras) e **nenhuma foi
  escolhida** — a marca segue `hagamorfis/rotas`. ⚠️ Se um dia trocar, mudar só o
  nome que aparece: **renomear o repositório muda o endereço do GitHub Pages e
  quebra todos os links de roteiro já enviados à equipe**.

**Material de trabalho na pasta, fora do repositório** (padrão `COMPARACAO-*.html`
no `.gitignore`), para apagar quando não servirem mais:
`COMPARACAO-MODULO1.html` (as 5 propostas do módulo 1; a v7.4.0 saiu da "D") e
`COMPARACAO-DESIGN.html` (os 7 pontos acima).

Regra de trabalho vigente: implementar e testar, mas **perguntar antes de fazer
commit/push** — o usuário testa antes de publicar; `.haga` dele significa "pode
commitar e publicar agora".

Concluído:
- Tipo de serviço por parada — ✅ implementado na v1.9
- Busca/filtro no checklist — ✅ implementado na v2.1
- Base de clientes guardada no navegador — ✅ implementado na v2.2
- Endereço de origem memorizado — ✅ implementado na v2.5

Adiados a pedido do usuário em 29/08/2026 (continuam descritos no documento
de arquitetura, para retomar quando fizer sentido):
- Campo de observação livre por parada (Fase 2)
- Dividir clientes entre as 2-3 pessoas da equipe, um link por pessoa (Fase 2)

---

## 9. Histórico

| Passo | O que foi feito |
|---|---|
| 1 | Protótipo inicial: destino único, filiais fixas no código (Rio Maina / Próspera) |
| 2 | Reconstrução: upload de GeoJSON, múltiplas paradas, otimização de ordem, filiais de exemplo removidas |
| 3 | Correção do "Failed to fetch" — `safeFetchJSON()` com mensagem explicativa |
| 4 | **Fase 1 do roadmap:** modo campo — gerar link do roteiro, tela separada para celular, navegação via Waze/Maps, marcar parada concluída com progresso salvo no aparelho |
| 5 | **v1.1** — Troca do provedor de tiles: CARTO (passou a carimbar o mapa) → Esri Dark Gray Canvas, sem cadastro. Adotado o versionamento numerado (seção 11) |
| 6 | **v1.2** — Clientes separados por camada: app passa a ler o backup completo do uMap (`.umap`) e o checklist vira uma lista agrupada por cliente, com cascata |
| 7 | **v1.3** — Correção: a instalação usada (`umap.hotosm.org`) guarda o nome da camada em `properties.name`, forma que a v1.2 não lia |
| 8 | **v1.4** — Correção: o checklist agrupado não rolava (filhos de flex sendo espremidos em vez de gerar rolagem) |
| 9 | **v1.5** — Nome do cliente passa a aparecer no modo campo, acima do nome da filial |
| 10 | **v1.6** — Ícones do Waze/Maps nos botões de navegação; "Marcar como concluída" maior e com confirmação em duas etapas |
| 11 | **v1.7** — Ícones reais do Waze e do Google Maps (recortados pelo usuário); parada concluída esconde os botões de navegação |
| 12 | **v1.8** — Reabrir uma parada concluída também passa a pedir confirmação em duas etapas |
| 13 | **v1.9** — Fase 2 (parcial, a pedido do usuário): tipo de serviço por parada, com lista configurável |
| 14 | **v2.0** — 5 ajustes leves acumulados desde a v1.9 fecharam a versão; o último trocou o `<button>` "Adicionar" por um elemento genérico, eliminando de vez a divergência de altura entre navegadores |
| 15 | **v2.1** — Fase 3 (parcial, a pedido do usuário): busca/filtro no checklist |
| 16 | **v2.2** — Fase 3: base de clientes guardada no navegador, com link para esquecê-la |
| 17 | **v2.3** — Correção: busca de origem falhava fora de Criciúma (a cidade era grudada à força em tudo que se digitava) |
| 18 | **v2.4** — Paradas prioritárias fixas antes da otimização; escolha entre endereços quando a busca de origem é ambígua |
| 19 | **v2.5** — Origem por clique no mapa, aviso quando o número da casa não existe no mapa, e origem padrão salva no navegador |
| 20 | **v2.6** — Formato compacto do link do roteiro (37% menor), retrocompatível com os links já enviados |
| 21 | **v2.7** — Compressão gzip nativa do navegador no link (530 caracteres), com formato compatível para celular antigo |
| 22 | **v2.8** — Marcação de prioridade passa a aparecer também na tela do campo |
| 23 | **v2.9** — Grupos do uMap: leitura das camadas aninhadas (que sumiam em silêncio) e cascata de três níveis no checklist |
| 24 | **v3.0** — Base ordenada alfabeticamente ao carregar, nos três níveis, com ordenação ciente de acentos |
| 25 | **v3.1** — Divisória arrastável entre mapa e painel, com a largura guardada; nomes longos passam a quebrar linha em vez de serem cortados |
| 26 | **v3.2** — Mapa escurecido por filtro CSS sobre os ladrilhos do Esri, com os rótulos clareados à parte |
| 27 | **v3.3** — Modo noturno para o app inteiro (interruptor nas duas telas), tema claro completo e véu no lugar do contraste que manchava o mapa |
| 28 | **v3.4** — Definição do mapa escuro: clarear → contrastar → escurecer, batendo com a referência do usuário. Corrige a medição errada que guiou a v3.3 |
| 29 | **v3.5** — Origem passa a enquadrar o mapa; margem de erro da geolocalização informada, com círculo de incerteza e endereço de conferência |
| 30 | **v3.6** — Correção: a v3.5 quebrou o botão de localização em computador de mesa ao exigir GPS. Passa a tentar duas vezes, caindo para a rede |
| 31 | **v3.7** — Botão de localização volta à lógica da v2.5 (aceita a posição do navegador); ficam só o enquadramento no pino e o endereço de conferência. Desfaz as v3.5–v3.6 |
| 32 | **v3.8** — Arrastar para reordenar as paradas, como no celular; soltar no outro bloco muda a prioridade |
| 33 | **v3.9** — Parada comum arrastada para as prioritárias passa a ser incluída no fim delas, em vez de só conseguir entrar à frente de todas |
| 34 | **v4.0** — Correção: a tela do campo ficava desenhada embaixo do escritório desde a v1.0 (display do CSS vencia o hidden). Subtítulo do cabeçalho removido |
| 35 | **v4.1** — Revisão de código completa: 6 correções (marcar todos e prioridades, balão com busca, rota/link desatualizados, marcadores velhos ao trocar base, símbolos nos balões, novo roteiro na mesma aba). Inclui a remoção do rodapé |
| 36 | **v4.2** — Altura do módulo 4 (Ordem da viagem) ajustável por puxador, guardada no navegador. Primeiro passo dos módulos customizáveis |
| 37 | **v4.3** — Divisória entre os módulos 3 e 4: o 4 cresce para cima e o 3 encolre junto, sem empurrar o resto. Substitui o puxador da v4.2. Alvo do projeto passa a ser Full HD |
| 38 | **v4.4** — Módulo 4 pode ser puxado para o lado e virar guia ao lado do mapa, com volta ao painel pela âncora |
| 39 | **v4.5** — A guia passa a ficar encostada no painel (estava na borda da tela) e o módulo 3 ganha puxador próprio quando o 4 está na guia |
| 40 | **v4.6** — Módulo 4 volta a abrir sempre no painel (o lugar deixa de ser guardado) e ganha puxador próprio, ao lado da divisória 3↔4 |
| 41 | **v4.7** — Módulo 3 também pode ir para a guia; o mecanismo vira uma tabela de módulos, pronta para os demais |
| 42 | **v4.8** — Guia ganha divisória de largura; corrigido o campo de busca que esticava na vertical dentro da coluna |
| 43 | **v4.9** — Divisória entre os dois módulos dentro da guia, repartindo a altura da coluna |
| 44 | **v5.0** — Cascata do cliente fecha ao selecionar uma filial, inclusive durante a busca |
| 45 | **v5.1** — Botões ★ ▲ ▼ ✕ da lista de paradas com alvo de toque maior (32×32, e 44×44 no toque) |
| 46 | **v5.2** — Desfaz a v5.1 e aumenta o que o usuário realmente pediu: a área de pegada da alça de arrastar |
| 47 | **v5.3** — Largura da alça dobrada (27 → 54; 37 → 74 no toque) |
| 48 | **v5.4** — Módulo 3 ajustável desde a abertura, antes de escolher paradas; corrigidas as linhas fantasmas da lista de paradas ao trocar de base |
| 49 | **v5.5** — Correção do aperto na lista de paradas: botões deixam de encolher e a alça acompanha a largura da coluna |
| 50 | **v5.6** — Limite de largura das colunas (320px) e mínimo para o nome da parada: fim da quebra letra a letra |
| 51 | **v5.7** — Terceira coluna; todos os módulos móveis entre painel e duas colunas; salvar arranjo e voltar ao normal |
| 52 | **v5.8** — Correção: divisória e puxadores dos módulos 3 e 4 travavam durante uma busca de poucos resultados |
| 53 | **v5.9** — Correção: módulo com lista sozinho numa coluna não tinha controle de altura; puxador e modo de altura fixa |
| 54 | **v6.0** — Posição livre dos módulos: soltar entre dois módulos, com linha âmbar de destino e rolagem automática |
| 55 | **v6.1** — Vão de encaixe do tamanho do módulo arrastado, abrindo espaço real no destino (substitui a linha âmbar) |
| 56 | **v6.1.1** — Contorno da coluna só ao mudar de coluna. Adotada a regra MAIOR.MENOR.AJUSTE |
| 57 | **v6.1.2** — Contorno da coluna só com a borda, sem preenchimento |
| 58 | **v6.1.3** — Área de pegada da alça dos módulos de 15×13 para 30×32px (44×44 no toque) |
| 59 | **v6.1.4** — Alça dos módulos com 60×64px (88×88 no toque), conteúdo com prioridade no clique; "3 · Selecionar paradas" alinhado aos outros títulos |
| 60 | **v6.1.5** — Alça como retângulo em volta dos pontinhos e do número (39×30), no lugar da área invisível da v6.1.4 |
| 61 | **v6.1.6** — Destaque âmbar translúcido também na alça das paradas (módulo 4) |
| 62 | **v6.2** — Painel enxuto (sugestões de layout 1–3): parada em uma linha, distância/tempo junto dos botões, módulo 1 compacto |
| 63 | **v6.2.1** — Módulo 1 compacto diz "✓ Base carregada", com espaçamento |
| 64 | **v6.2.2** — Fonte do "✓ Base carregada" 4px maior (15,5px) |
| 65 | **v6.2.3** — Fonte do "✓ Base carregada" 1px menor (14,5px) |
| 66 | **v6.3** — Retorno para a origem: no traçado, na otimização, no Roteiro e no link do campo (formato v7) |
| 67 | **v6.4** — Painel funciona como as colunas: sem rolagem própria, com os mesmos controles de altura (sugestão de layout 4) |
| 68 | **v6.4.1** — v6.4 desfeita (o usuário preferiu o painel de antes); versão passa a mostrar sempre três números |
| 69 | **v6.5.0** — Módulo Rota grudado na coluna em que estiver, compacto enquanto flutua (sugestão de layout 5) |
| 70 | **v6.6.0** — Ações do módulo 3 viram ícones ao lado da busca; título volta a caber em uma linha (sugestão de layout 6) |
| 71 | **v6.7.0** — Realce entre mapa e lista de paradas, nos dois sentidos (sugestão de layout 7) |
| 72 | **v6.8.0** — Retomar roteiro pelo link (formato v8 com id do roteiro e origem) e progresso do campo por parada |
| 73 | **v6.8.1** — Campo de retomar também no módulo 1, com o link esperando a base |
| 74 | **v6.9.0** — Clicar no nome ou no número da parada centraliza o mapa nela |
| 75 | **v7.0.0** — Parada avulsa: ponto fora da base por coordenada colada ou clique no mapa; retomada traz avulsas de volta |
| 76 | **v7.1.0** — Correção: módulo Rota grudado alternava entre compacto e inteiro, fazendo o painel piscar |
| 77 | **v7.2.0** — Correção: bolinha da parada avulsa continuava no mapa depois de excluída |
| 78 | **v7.3.0** — Botão que bloqueia e libera o zoom ao clicar na parada |
| 79 | **v7.4.0** — Módulo 1 enxuto: estado no título, upload em uma linha e retomar dobrável |
| 80 | **v7.5.0** — Correção: barra de progresso do campo estava com largura zero desde a Fase 1 |
| 81 | **v7.5.1** — Títulos dos módulos sem numeração |
| 82 | **v7.6.0** — Cabeçalho: ações agrupadas e tema como ícone sol/lua |
| 83 | **v7.7.0** — Tela do campo: parada da vez em cartão, as outras em linha compacta |
| 84 | **v7.8.0** — Botões da parada: navegação em contorno âmbar, concluir como único cheio |

---

## 10. Estrutura do projeto e hospedagem

O projeto é versionado em Git e publicado no GitHub Pages.

**Estrutura (a partir de 22/08/2026):**

```
PROJETO APP LOGISTICA/        ← raiz do repositório Git
├── index.html                ← o app (servido pelo GitHub Pages)
├── CLAUDE.md                 ← este arquivo (contexto do projeto)
├── LOG-ALTERACOES.txt        ← log de todas as alterações
├── README.md
├── .gitignore
├── Exemplos/
│   └── exemplo.umap          ← 3 clientes fictícios, só para demonstrar o formato
├── .claude/launch.json       ← config do servidor local de testes
└── backups/                  ← pontos de restauração (NÃO versionado)
```

- **Repositório:** github.com/antoniocmp97/roteador-clientes (público)
- **Site publicado:** https://antoniocmp97.github.io/roteador-clientes/
- Todo `git push` para o branch `main` republica o site automaticamente.

**Regra de dados:** o repositório é público, então dados reais de clientes
**nunca** são versionados. O app carrega o `.umap` por upload — os dados
ficam só na máquina de quem usa. O `.gitignore` bloqueia `clientes*.geojson`
e a pasta `dados-reais/` por precaução.

**Para testar localmente:** abrir o `index.html` no navegador, ou usar o
servidor local (`.claude/launch.json`). Não usar a pré-visualização embutida
do chat — ela bloqueia as chamadas de rede ao OSRM e ao Nominatim.

---

## 11. Versionamento e backups

Adotado em 29/08/2026, a pedido do usuário.

### Como o número é contado

**Regra atual (desde 14/09/2026, a pedido do usuário):** formato
`MAIOR.MENOR.AJUSTE`.

1. **Cada alteração leve** acrescenta 1 no terceiro número, na hora:
   6.1.0 → 6.1.1 → 6.1.2 → …
2. **Uma alteração mediana ou grande** sobe o número do meio e zera o terceiro:
   6.1.3 → **6.2.0** (e 6.9.x → 7.0.0).
3. O terceiro número **aparece sempre** (pedido do usuário em 15/09/2026, que
   sentiu falta dele no selo da v6.4).

A regra anterior ("cinco leves acumuladas fecham uma versão") foi **substituída**
— não há mais contador de leves. Versões até a 6.1 seguem a regra antiga.

**O que é alteração leve:** não muda o comportamento do app. Texto, cor,
espaçamento, comentário no código, documentação, renomear variável.

**O que é mediana ou grande:** muda o que o app faz ou o que o usuário vê.
Funcionalidade nova, troca de serviço externo, mudança de fluxo, correção de
bug que atrapalhava o uso.

### Versão atual

```
7.8.0
```

### Onde o número aparece

Na constante `VERSAO` no início do script do `index.html`, e a partir dela num
selo discreto ao lado da marca **nas duas telas** — escritório e campo. A ideia
é que quem está na rua consiga informar a versão ao relatar um problema.

Ao fechar versão nova, atualizar a constante `VERSAO` junto com o resto.

### Backups

Cada versão fechada ganha uma cópia em `backups/vX.Y_AAAA-MM-DD_apelido/`,
com o `index.html` daquela versão e um `LEIA-ME.txt` explicando o que mudou e
por quê. Ajustes leves (`X.Y.Z`) também ganham backup, na mesma forma
(`backups/vX.Y.Z_AAAA-MM-DD_apelido/`).

⚠️ **A pasta `backups/` não é versionada** (bloqueada no `.gitignore`), então
essas cópias existem **apenas nesta máquina**. Numa próxima troca de computador
elas se perdem se a pasta não for copiada junto. A rede de segurança real
continua sendo o Git, que guarda todas as versões no GitHub de qualquer forma —
os backups locais são conveniência, não garantia.

### Versões existentes

| Versão | Data | O que é |
|---|---|---|
| 1.0 | 22/08/2026 | Fase 1 — modo campo. Validada em celular real. Resgatada do commit `2c87046` |
| 1.1 | 29/08/2026 | Troca CARTO → Esri e adoção do versionamento |
| 1.2 | 29/08/2026 | Leitura do `.umap` e checklist agrupado por cliente |
| 1.3 | 29/08/2026 | Correção do nome da camada no formato do `umap.hotosm.org`. **Validada com a base real** |
| 1.4 | 29/08/2026 | Correção da rolagem do checklist agrupado |
| 1.5 | 29/08/2026 | Nome do cliente no modo campo (formato do link v2, retrocompatível com v1) |
| 1.6 | 29/08/2026 | Ícones dos apps de navegação e confirmação em duas etapas ao concluir |
| 1.7 | 29/08/2026 | Ícones reais (Waze/Maps) e trava de navegação na parada concluída |
| 1.8 | 29/08/2026 | Confirmação em duas etapas também para reabrir uma parada concluída |
| 1.9 | 29/08/2026 | Fase 2 (parcial): tipo de serviço por parada, lista configurável |
| 2.0 | 29/08/2026 | Fecha 5 ajustes leves; correção definitiva do alinhamento do botão "Adicionar" (deixou de ser um `<button>` nativo) |
| 2.1 | 29/08/2026 | Fase 3 (parcial): busca/filtro no checklist por cliente ou filial |
| 2.2 | 29/08/2026 | Fase 3: base de clientes guardada no navegador |
| 2.3 | 06/09/2026 | Correção da geocodificação: endereços fora de Criciúma voltaram a funcionar |
| 2.4 | 06/09/2026 | Paradas prioritárias (★) e escolha entre endereços ambíguos |
| 2.5 | 06/09/2026 | Origem: clique no mapa, aviso de número inexistente e origem padrão salva |
| 2.6 | 06/09/2026 | Link do roteiro 37% menor (tabelas, formato compacto e coordenadas por diferença) |
| 2.7 | 06/09/2026 | Compressão nativa no link: 1204 → 530 caracteres no total |
| 2.8 | 06/09/2026 | Estrela de prioridade também no modo campo (formato do link v6) |
| 2.9 | 08/09/2026 | Grupos do uMap: Grupo → Camada → Unidades no checklist |
| 3.0 | 08/09/2026 | Ordenação alfabética da base nos três níveis |
| 3.1 | 09/09/2026 | Painel de largura ajustável (medido: nome de 438px numa caixa de 207px) |
| 3.2 | 09/09/2026 | Mapa escurecido (opção "cinza escuro", escolhida entre três em comparação lado a lado) |
| 3.3 | 09/09/2026 | Modo noturno no app inteiro + véu no mapa (definição caiu para 8 — corrigido na v3.4) |
| 3.4 | 09/09/2026 | Definição do mapa escuro igual à da referência (8 → 45) |
| 3.5 | 09/09/2026 | Origem enquadra o mapa; margem de erro da localização à mostra |
| 3.6 | 09/09/2026 | Correção da v3.5: localização em duas tentativas (exigir GPS quebrava o botão) |
| 3.7 | 10/09/2026 | Botão 📍 volta à lógica da v2.5, que funcionava nas máquinas do usuário |
| 3.8 | 10/09/2026 | Arrastar para reordenar paradas; soltar no outro bloco muda a prioridade |
| 3.9 | 10/09/2026 | Arrastar para as prioritárias inclui a parada no fim delas (a divisa segue o título "Demais") |
| 4.0 | 10/09/2026 | Tela do campo parou de vazar embaixo do escritório (bug da v1.0); subtítulo do cabeçalho removido |
| 4.1 | 10/09/2026 | 6 correções da revisão de código completa (ver seção 5) |
| 4.2 | 11/09/2026 | Altura do módulo 4 ajustável (puxador), guardada no navegador |
| 4.3 | 11/09/2026 | Divisória entre os módulos 3 e 4 (o 4 cresce para cima); alvo Full HD |
| 4.4 | 11/09/2026 | Módulo 4 vira guia ao lado do mapa (arrastar para o lado ou duplo clique) |
| 4.5 | 12/09/2026 | Guia encostada no painel; módulo 3 com puxador próprio |
| 4.6 | 12/09/2026 | Módulo 4 abre sempre no painel e ganha puxador próprio |
| 4.7 | 12/09/2026 | Módulo 3 também vai para a guia; mecanismo generalizado |
| 4.8 | 12/09/2026 | Guia com largura ajustável; campo de busca não estica mais na coluna |
| 4.9 | 12/09/2026 | Divisória entre os dois módulos da guia |
| 5.0 | 12/09/2026 | Cascata fecha ao selecionar uma filial |
| 5.1 | 12/09/2026 | Alvo de toque maior nos botões da lista de paradas (revertida na v5.2) |
| 5.2 | 12/09/2026 | Área de pegada maior na alça de arrastar paradas |
| 5.3 | 12/09/2026 | Largura da alça dobrada (54px; 74px no toque) |
| 5.4 | 12/09/2026 | Módulo 3 ajustável desde a abertura; linhas fantasmas corrigidas |
| 5.5 | 12/09/2026 | Alça acompanha a largura da coluna; botões não encolhem mais |
| 5.6 | 12/09/2026 | Piso de 320px nas colunas e mínimo de 120px para o nome da parada |
| 5.7 | 14/09/2026 | Terceira coluna, os 7 módulos móveis e botões de salvar/voltar ao normal o arranjo |
| 5.8 | 14/09/2026 | Listas dos módulos 3 e 4 com altura fixa: controles de altura não travam mais na busca |
| 5.9 | 14/09/2026 | Puxador de altura nas colunas para módulo com lista sozinho (modo de altura fixa) |
| 6.0 | 14/09/2026 | Posição livre dos módulos (ex.: o 4 acima do 1), com a ordem no arranjo salvo |
| 6.1 | 14/09/2026 | Vão de encaixe proporcional ao módulo arrastado |
| 6.1.1 | 14/09/2026 | Contorno da coluna só quando o módulo muda de coluna (primeira versão na regra de três números) |
| 6.1.2 | 14/09/2026 | Contorno da coluna só com a borda, sem preenchimento |
| 6.1.3 | 14/09/2026 | Área de pegada maior na alça dos módulos |
| 6.1.4 | 14/09/2026 | Alça com o dobro da área e números dos títulos alinhados |
| 6.1.5 | 14/09/2026 | Alça em volta dos pontinhos e do número, proporcional ao que se vê |
| 6.1.6 | 14/09/2026 | Destaque âmbar na alça das paradas |
| 6.2 | 15/09/2026 | Paradas em uma linha, resultado junto dos botões e módulo 1 compacto |
| 6.2.1 | 15/09/2026 | "✓ Base carregada" no módulo 1 compacto |
| 6.2.2 | 15/09/2026 | Fonte maior no "✓ Base carregada" |
| 6.2.3 | 15/09/2026 | Fonte do "✓ Base carregada" em 14,5px |
| 6.3 | 15/09/2026 | Retorno para a origem (interruptor ligado por padrão; link v7) |
| 6.4 | 15/09/2026 | Painel como as colunas: fim das rolagens empilhadas — **desfeita na v6.4.1** |
| 6.4.1 | 15/09/2026 | Volta ao painel da v6.3; três números no selo de versão |
| 6.5.0 | 15/09/2026 | Módulo Rota sempre visível (grudado, compacto enquanto flutua) |
| 6.6.0 | 15/09/2026 | Ícones de tipos de serviço e marcar todos na linha da busca |
| 6.7.0 | 15/09/2026 | Realce entre mapa e lista de paradas |
| 6.8.0 | 15/09/2026 | Retomar roteiro pelo link; progresso do campo sobrevive ao link atualizado |
| 6.8.1 | 15/09/2026 | Campo de retomar no módulo 1, sempre visível |
| 6.9.0 | 15/09/2026 | Clicar na parada leva o mapa até ela |
| 7.0.0 | 15/09/2026 | Parada avulsa (coordenada colada ou clique no mapa) |
| 7.1.0 | 15/09/2026 | Fim do painel piscando (folga na decisão do módulo grudado) |
| 7.2.0 | 15/09/2026 | Parada avulsa excluída sai do mapa |
| 7.3.0 | 17/09/2026 | Botão que bloqueia e libera o zoom ao clicar na parada |
| 7.4.0 | 17/09/2026 | Módulo 1 enxuto (estado no título, upload em uma linha, retomar dobrável) |
| 7.5.0 | 17/09/2026 | Barra de progresso do campo volta a aparecer |
| 7.5.1 | 18/09/2026 | Títulos dos módulos sem numeração |
| 7.6.0 | 18/09/2026 | Cabeçalho: ações agrupadas e tema como ícone |
| 7.7.0 | 18/09/2026 | Tela do campo: a parada da vez em destaque |
| 7.8.0 | 19/09/2026 | Botões da parada: navegação em contorno âmbar |
