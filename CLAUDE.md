# Roteador de Clientes — Hagamorfis

> Documento de contexto do projeto. Mantido atualizado a cada passo para permitir
> migração do chat para o Claude Code sem perda de contexto.
>
> **Última atualização:** 10/10/2026 — v8.26.0 (varredura completa atrás de bugs: nove correções, quatro pontos em aberto)

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
- **O checklist é uma LISTA, e não uma pilha de cartões** (v8.22.0, ponto 4 da
  revisão de layout de 26/09/2026). Cada cliente era um cartão com borda, fundo
  e 6px de respiro, e cada unidade dentro dele era outro. Medido: o nome de uma
  unidade ficava dentro de **duas caixas com borda** (cliente e unidade), dentro
  do cartão de vidro. Agora o cliente é uma **linha de lista separada por um
  fio**, e a cascata aberta se marca por um **filete à esquerda**.
  ⚠️ **Borda é uma afirmação** — "isto aqui é uma coisa separada". Quando tudo
  tem borda, nada está agrupado; e agrupar é justamente o trabalho deste módulo,
  que existe para mostrar cliente → filial.
  ⚠️ **O filete não é invenção: é o recurso que o nível do GRUPO do uMap já
  usava desde a v2.9**, com a decisão escrita no CSS ("sem caixa dentro de
  caixa... lê melhor do que bordas aninhadas num painel de 340px"). Esta versão
  só levou a mesma decisão um nível abaixo, para a cascata de unidades.
  ⚠️ **O `gap` do checklist foi a ZERO**, e isso é parte da mudança, não um
  descuido: quem separa agora é o fio. Com gap e fio ao mesmo tempo a lista
  ficaria mais frouxa do que era com cartão.
  ⚠️ **Quem marca a área de clique passou a ser o `:hover`**, que antes era
  coadjuvante da borda. As três linhas clicáveis ganharam a regra
  (`.grupo-head`, `.grupo-umap-head`, `.check-row`). ⚠️ E o sinal de "tem
  parada escolhida aqui dentro" **não mudou**: continua sendo a seta e o
  contador acendendo em âmbar, da v8.4.2 — conferido depois.
  ⚠️ **A `.check-row` teve de sair das duas listas de "caixas de dentro" do
  vidro** (`.console .check-row` a 80% de `--panel-2`, e a variante do vidro
  desligado). Elas têm especificidade maior que a regra nova e teriam devolvido
  o fundo do cartão. Conferido nos dois estados do vidro: fundo transparente.
  ⚠️ **A "Ordem da viagem" tinha ficado de fora, e o usuário desfez isso na
  v8.22.1**: *"ficou bom, mas para ficar melhor o módulo de ordem de viagem
  precisa seguir a mesma lógica"*. Meu argumento era que uma parada ali é um
  **objeto que se arrasta** e que o cartão é o que a faz parecer pegável. Ele
  tem razão: **consistência no painel inteiro pesa mais** que essa distinção, e
  quem diz "isto se arrasta" é a **alça**, que tem destaque âmbar próprio desde
  a v6.1.6 — não a borda. O módulo virou lista igual, com uma diferença que
  aproveita o meu argumento em vez de descartá-lo: **o cartão volta enquanto a
  parada está sendo arrastada**, que é quando ela é mesmo um objeto na mão.
  ⚠️ **O cartão da parada carregava TRÊS estados pela cor da borda**, e cada um
  precisou de destino antes de ele sair:
  · **arrastando** (borda âmbar + sombra) → o cartão **volta**, com fundo,
    borda âmbar e raio. Uma linha sem caixa flutuando com sombra pareceria
    defeito, e é também o que separa a linha que se move das que ficaram.
  · **prioritária** (borda âmbar-escura) → **saiu e não foi substituída**: era
    o terceiro sinal da mesma coisa. A parada já está dentro do bloco
    "Prioritárias · nesta ordem" e já tem a ★ acesa em âmbar. É a mesma lição
    da v8.4.2, quando a bolinha da calha saiu por repetir o contador.
  · **realçada pelo mapa** (v6.7.0, borda âmbar + fundo) → virou uma **barra
    âmbar por dentro** (`inset box-shadow`), porque o fundo sozinho é fraco
    demais numa lista que rola. Inset não ocupa espaço: não empurra o texto nem
    muda a altura da linha — e a altura é medida durante o arraste.
  ⚠️ **O fio vai em `border-top`, e não embaixo**: é o que permite apagá-lo na
  primeira linha e na linha logo depois de um cabeçalho de bloco com um seletor
  só (`.stop-grupo + .stop-row`) — o CSS não sabe selecionar "o irmão
  anterior". O respiro que o `gap` dava passou para o `padding` dos cabeçalhos.
  ⚠️ **O ganho de altura aqui é pequeno, e isto não era sobre altura**: 5
  paradas sem prioritárias foram de **253,5 para 233,5px** (4px por parada). O
  pedido era consistência.
  ⚠️ **O cabeçalho do grupo do uMap continua sendo uma caixa**: ele é o topo de
  um bloco, não uma linha de lista. Quem ganhou o fio e o respiro foi o bloco
  inteiro.
  ⚠️ Medido na base de teste (13 clientes, 27 unidades, tudo aberto): a lista
  foi de **1845px para 1591px — 254px a menos, 14%**. Molduras fechadas visíveis
  no escritório: **35 → 26**. Com os 39 clientes da base real o ganho é maior,
  porque ele é por linha (7px de borda e respiro que saíram de cada uma).
- **A calha do checklist tem uma marca só** (v8.4.2, relatado pelo usuário com
  print: "está me parecendo redundante o ponto e a seta"). ⚠️ O instinto estava
  certo, o alvo não: a seta e a bolinha **não** faziam a mesma coisa — a seta
  diz se a cascata abre, a bolinha dizia que há filial selecionada ali dentro.
  A redundância era da **bolinha com o contador**, que já virava "2/9" e diz
  *quantas de quantas*, no mesmo lugar. E em repouso a bolinha era teal a **35%
  de opacidade**: uma segunda marca apagada, sem informação nenhuma. Ela saiu
  dos dois níveis, e agora quem acende em âmbar é a **seta e o contador** —
  os dois, escolha do usuário entre as opções comparadas na
  `COMPARACAO-CHECKLIST.html`: o contador é o preciso, a seta é a que dá para
  varrer de relance numa lista de 39 clientes. Medido num painel de 340px: a
  calha até o nome caiu de **42px para 31px**, o que tira uma linha dos nomes
  que quebram em duas. ⚠️ O **▶ virou chevron SVG** na mesma leva (constante
  `CHEVRON`, usada nos dois níveis): ele tinha ficado para trás na v7.9.0, que
  trocou os outros controles justamente porque caractere depende da fonte do
  sistema. Por isso o ganho real é **11px** e não 15 — o chevron é 4px mais
  largo que o ▶; os 15px são só da bolinha
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
  grupos com resultado abrem sozinhos durante a busca. **Ignora acento e
  cedilha** (v8.24.0, relatado pelo usuário depois de um dia de uso: "o filtro é
  muito específico, não consegue localizar caso o nome tenha acento"): digitar
  `educacao` acha `EDUCAÇÃO` e o contrário também, nos três níveis (grupo,
  camada, unidade). `semAcento()` = NFD + apagar a faixa U+0300–U+036F +
  minúscula. ⚠️ **Escopo decidido pelo usuário: só acento.** Continuam **não**
  achando palavras em outra ordem (`educacao exemplo`) nem erro de digitação
  (`acuo` não acha `AÇUÃO`). ⚠️ **Não é o `Intl.Collator` de `ordenarBase()`**:
  ordenar precisa do acento, buscar precisa ignorá-lo — não tocar no Collator.
  ⚠️ **A forma normalizada é calculada na hora, NUNCA guardada no objeto**
  (`g._busca`/`pt._busca`): a parada avulsa nasce em execução e, sem o campo,
  sumiria da busca sem erro. Custo medido: **0,07ms** por tecla com 117 nomes
  (39 clientes × 2 filiais + grupos), então cache não se paga. A mensagem de
  "nenhum resultado" continua ecoando o texto **com** o acento digitado. Ao lado do campo, dois
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
  (Nominatim). ⚠️ **Entre a v8.9.0 e a v8.25.0 esse rótulo não chegava à "Ordem
  da viagem" nem ao link** (a `stops` passou a guardar cópia, e o nome era
  trocado só no ponto da base); corrigido na v8.26.0 por `renomearAvulsa()`. ⚠️ Vive só na sessão: recarregar limpa, como a seleção do dia; para
  recuperar, retoma-se pelo link
- **A cascata fecha ao selecionar** (v5.0): marcar uma filial fecha a cascata do
  cliente, para a lista não ficar poluída. O **contador** no cabeçalho
  (ex.: "1/9") continua mostrando que há seleção ali dentro — **em âmbar e
  negrito**, junto com a seta, desde a v8.4.2. Desmarcar
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
- **Ícones em traço, não emoji** (v7.9.0, ponto 5 da revisão de design): o alvo
  de "escolher no mapa" (🎯), o alfinete da localização (📍), a estrela da parada
  prioritária (★/☆, cheia quando ligada), os chevrons de mover uma casa (▲▼) e o
  ✕ de remover viraram **SVG de traço** (1,8–1,9px, 13–17px), que acompanham a
  cor do tema. A estrela da tela do campo também. Motivo: emoji muda de desenho
  entre o Windows do escritório e o Android da equipe, e ★ ☆ ▲ ▼ ✕ são
  caracteres, que dependem da fonte do sistema. ⚠️ Ficaram de propósito como
  texto: o **✓** (base carregada, confirmar, parada concluída), o **↩** do
  retorno e o **🎉** do fim do roteiro — são glifos dentro de frases e rótulos,
  não controles.
  ⚠️ **Os dois botões da Origem trocaram de DESENHO na v8.25.0**: a mira passou
  para "usar minha localização" e "marcar no mapa" virou um mapa dobrado (ver o
  item do cabeçalho com nome, abaixo). A posição e a função de cada botão são as
  mesmas
- **Modo noturno** (v3.3): botão no cabeçalho das **duas telas**
  (escritório e campo) troca o app inteiro entre tema escuro e claro. A escolha
  fica guardada no navegador e é aplicada antes da primeira pintura, para a tela
  não piscar no tema errado ao abrir. Padrão: escuro.
  **Desde a v7.6.0 é um ícone de 32px** (ponto 7 da revisão de design) que mostra
  o **estado atual** — lua no escuro, sol no claro —, no lugar do interruptor de
  trilho com "MODO NOTURNO" escrito ao lado (~110px em caixa alta). Continua
  sendo `<button role="switch">` com `aria-checked`: o CSS escolhe o ícone a
  partir dele e o leitor de tela anuncia a partir dele, num lugar só. A dica do
  mouse diz para onde o clique leva.
  ⚠️ **No escritório, desde a v8.25.0, é a linha "Tema escuro" do menu
  "Exibição"** — mesmo `data-tema-btn`, mesmo `aria-checked`. O ícone sozinho
  continua existindo **só na tela do campo**, e é por causa dele que a classe
  `.icone-btn` não pode ser apagada do CSS
- **Ações do cabeçalho num bloco só** (v7.6.0): `[disquete] [lixeira] | [tema]`
  encostados à direita, com um divisor de 1px entre o arranjo e o tema — 121px no
  total. Antes o disquete e a lixeira ficavam colados na marca, como se fossem
  parte dela. Em tela estreita (<760px) o arranjo some, como já era, e o divisor
  some junto. ⚠️ **Este bloco de ícones deixou de existir na v8.25.0** — ver o
  item seguinte
- **Cabeçalho com NOME nos botões** (v8.25.0, a pedido do usuário: "com um olhar
  mais de usuário, visualize os botões do cabeçalho e do restante do site... quero
  verificar se está acessível para pessoas com menos conhecimento de tecnologia").
  O bloco `[disquete] [lixeira] | [foco] [leve] [janelas] [ímã] [vidro] [tema]`
  virou **dois botões com palavra**: `[Mapa: todos]` e `[Exibição ▾]`. O primeiro
  é o filtro de clientes da v8.15.0, agora com o **estado escrito** ("Mapa:
  todos" / "Mapa: destaque" / "Mapa: só a viagem"). O segundo abre um **menu**
  em que cada comando é uma linha com **nome e uma frase**: Modo leve, Tema
  escuro, Transparência do painel · Janelas soltas, Alinhar janelas, Guardar esta
  organização, Voltar à organização padrão. Escolhido pelo usuário entre três
  propostas na `COMPARACAO-CABECALHO.html` (a "A"; as outras eram o nome embaixo
  de cada ícone e os mesmos ícones com dica imediata).
  ⚠️ **O que foi medido antes**: 27 dos 33 botões visíveis do escritório eram só
  ícone, e no cabeçalho eram sete de sete. A única explicação era a dica do
  navegador, que demora perto de um segundo, **não existe em tela de toque** e
  pede que a pessoa saiba parar o ponteiro em cima. E nenhum dos sete fazia
  parte do trabalho do dia — eram aparência e arrumação, no lugar mais nobre.
  ⚠️ **Três desenhos diziam outra coisa**, e é por isso que não bastava pôr
  nome: o **disquete** é o símbolo de salvar o *trabalho* (quem clicava achava
  que tinha guardado o planejamento do dia — que se guarda sozinho desde a
  v8.5.0); a **lixeira** parece "apagar" e assusta quem tem medo de estragar; e
  o do **vidro** eram dois quadrados sobrepostos, o ícone universal de
  **copiar**. Viraram: painel com ✓, seta de voltar e vidraça com reflexo.
  ⚠️ **O vocabulário mudou junto**: "arranjo" → "organização da tela", "voltar
  ao normal" → "voltar à organização padrão", "vidro" → "transparência do
  painel", "janelas livres" → "janelas soltas", "encaixe" → "alinhar janelas".
  **No código os nomes são os de sempre** (`arranjo`, `aplicarVidro`,
  `data-vidro`, `hg_arranjo_modulos`...), e este documento continua falando em
  vidro, janelas livres e arranjo — é o vocabulário interno, como "módulo 3".
  ⚠️ **As dicas do filtro do mapa diziam "APAGAR"**, e para quem não conhece o
  app apagar é **excluir**. Agora dizem "os outros ficam mais fracos".
  ⚠️ **Os ids e os `data-*-btn` são OS MESMOS**, e isso é o que fez a mudança
  ser de marcação e não de lógica: `aplicarTema`, `aplicarVidro`,
  `aplicarModoLeve`, `aplicarModoJanelas`, `aplicarIma` e
  `atualizarBotaoFoco` continuam achando o seu botão pelo mesmo seletor. O que
  saiu de dentro delas foi o `b.title`, que era a explicação — ela agora está
  escrita na linha.
  ⚠️ **Medido**: bloco de **265px → 252px** (275 com "Mapa: destaque", 292 com
  "Mapa: só a viagem"; o de antes chegava a 301 com o ímã). Cabeçalho continua
  com **65px**. Fica mais estreito apesar das palavras, porque sete botões
  viraram dois. Menu de 356×440px.
  ⚠️ **O PREÇO, assumido**: ligar o modo leve passou de **um clique para dois**,
  e o que está ligado **saiu da vista**. Sobra um **ponto âmbar** no botão
  "Exibição" (`atualizarPontoExibicao()`), que acende com modo leve ligado,
  transparência desligada, janelas soltas ou arrumação da tela ainda não
  guardada — e a dica do botão diz qual. ⚠️ Em máquina que vive no modo leve o
  ponto fica aceso sempre; é o comportamento aprovado na comparação.
  ⚠️ **Com o modo leve ligado, as linhas de transparência e de janelas NÃO
  somem mais**: ficam **travadas**, com "Desligado pelo modo leve." no lugar da
  frase. Antes os dois botões desapareciam do cabeçalho e nada dizia por quê.
  ⚠️ **O menu não tem animação nenhuma**: abre e fecha pelo atributo `hidden`,
  no mesmo quadro. É igual com o modo leve ligado e desligado por construção
  (medido: zero animações, `transition-duration: 0s`).
  ⚠️ **O `z-index: 2000` mora na caixa do menu (`.exibicao`), e não no
  `<header>`**: as janelas livres sobem o próprio z-index a cada clique
  (`frenteJanela`), os controles do Leaflet vão a 1000 e a etiqueta do módulo
  arrastado é 1001. Pondo ali, só o menu vira contexto de empilhamento.
  ⚠️ **`atualizarPontoExibicao()` é chamada na CARGA**, de dentro de
  `aplicarVidro`, antes de `LARGURA_MINIMA_GUIA` existir — por isso ela pergunta
  a tela estreita ao `matchMedia`, e não a `telaComportaGuia()`, que estouraria
  na zona morta do `const`.
  ⚠️ **Clique fora fecha, clique dentro não** (dá para ligar duas coisas sem
  reabrir), Esc fecha e devolve o foco ao botão. O clique de fora é escutado na
  **fase de captura**: o mapa e os arrastes tratam o próprio `pointerdown`.
  ⚠️ **Em tela estreita (≤760px) o menu fica com duas linhas** — Modo leve e
  Tema escuro —, que são as únicas que valem ali (classe `.exib-so-largo`).
  ⚠️ **Na mesma versão, os dois botões da Origem trocaram de desenho.** No
  Google Maps, no Waze e em todo celular a **mira é "onde eu estou"** e o
  alfinete é "um ponto no mapa"; aqui estava ao contrário, herança dos emojis
  🎯 e 📍 de antes da v7.9.0. Agora a mira é "usar minha localização" e "marcar
  o ponto no mapa" é um **mapa dobrado** — que também deixa de se confundir com
  o alfinete-com-mais da parada avulsa (o botão de marcar da avulsa trocou
  junto). **Só o desenho mudou**: a posição dos botões, a função de cada um e o
  comportamento da geolocalização são os mesmos. ⚠️ Quem usa há semanas vai ver
  a mira **no botão da direita**, onde antes estava o alfinete
- **Vidro: o mapa passa por baixo do painel** (v8.1.0, pedido do usuário; ele
  escolheu o modo "Apple" numa comparação com quatro modos e cinco controles).
  O mapa é **absoluto e ocupa a tela inteira**; o painel e as colunas flutuam
  sobre ele como **cartões** (margem de 12px, cantos de 18px), com tinta em
  gradiente (84%→94%), `backdrop-filter: blur(16px) saturate(140%)`, **véu**
  escuro de 18% atrás do conteúdo, fio de luz de 1px e sombra. No tema claro o
  véu e o fio são claros. As caixas de dentro vão a 80% — sólidas, virariam
  blocos boiando no vidro. Medido numa janela de 1400px: o mapa passou de
  **~580px para 1400px** de largura.
  ⚠️ **A saturação é o que faz parecer vidro**, não só o desfoque — sem ela o
  fundo vira um borrão cinza. ⚠️ O usuário pediu "transparente, **mas não ao
  ponto de sumir**": é para isso que existe o véu, que segura a leitura quando
  embaixo passa um bairro cheio de ruas claras.
  ⚠️ **O enquadramento do mapa teve de mudar junto**, senão a rota fica debaixo
  do painel: `folgaDasColunas()` mede o quanto os cartões cobrem, `paddingDoMapa()`
  entra nos `fitBounds`/`flyToBounds` e `centralizarVisivel()` no clique da parada
  e na origem. Nesta última, o centro deslocado é **calculado** — com um `panBy`
  depois do `setView` o resultado dependia de a animação ter rodado, e a parada
  ia parar na borda (medido: x=8 numa área visível de 676px; calculando, x=338).
  ⚠️ Em tela estreita (≤760px) **o vidro sai de cena**: volta o empilhado de
  antes, mapa em cima e painel embaixo, e `folgaDasColunas()` devolve 0.
  ⚠️ A divisória de largura perdeu o fundo sólido: sobre o mapa, ela cortaria a
  imagem em duas. ⚠️ O módulo Rota também perdeu o fundo fixo aqui, e o assunto voltou mais
  três vezes até a **v8.12.0 tirar o sticky inteiro**. O histórico fica no
  `LOG-ALTERACOES.txt`; nada disso está mais no código.
  ⚠️ **E a folga de 120px da v7.1.0 passou a valer só para a borda de baixo**:
  em cima ela fazia o módulo **nascer grudado** quando estava na primeira
  posição e a coluna não tinha rolado (a sentinela fica em `c.top + h - 16`, por
  causa do padding/margem negativa que cobre o respiro da coluna, e a conta
  exigia `c.top + h + 120`). Até a v8.0.0 o efeito era só o modo compacto ligado
  à toa; com o vidro virou o bloco escuro do print. A oscilação que a folga
  resolve é só de baixo, onde o módulo encolhe ao grudar e a sentinela reaparece.
  ⚠️ Não tente medir "está deslocado?" com `offsetTop`: num elemento sticky ele
  **já vem com o deslocamento embutido** e a conta dá zero sempre (medido:
  offsetTop subindo 16→216→516→771 junto com o scroll, com o rect parado).
  ⚠️ **O `padding:16px 0` com `margin:-16px 0` também virou coisa do estado
  grudado**: ele existe para cobrir o respiro da coluna enquanto o módulo
  flutua, e parado no lugar a margem negativa puxava o módulo de baixo 16px
  para cima — com a linha de status vazia fora, o de baixo passou a quase
  encostar (relatado pelo usuário). ⚠️ E a **sentinela é um item de layout do
  painel** (a section é `display:contents`): ela comia um segundo `gap` de 14px,
  deixando o módulo abaixo da Rota com 28px de respiro contra os 14 dos outros.
  Resolvido com `margin-bottom:-14px` **na sentinela** — mover a sentinela em si
  deslocaria o limiar do "grudado", que depende da posição dela. Medido no fim:
  14px entre módulos comuns e 14px da Rota para o de baixo
- **Vidro que sai de cena quando atrapalha** (v8.2.0, relatado pelo usuário:
  "testei no trabalho e ficou meio travado, como se estivesse rodando a 20-30
  fps"; na máquina dele de casa o mesmo arquivo roda liso).
  ⚠️ **O `backdrop-filter` não é desenhado uma vez só**: o navegador refaz o
  borrão a cada quadro em que muda alguma coisa **atrás** do painel — e o que
  está atrás, desde a v8.1.0, é o mapa inteiro. Três agravantes somados: o mapa
  passou de ~580px para a largura toda, os ladrilhos carregam o filtro de quatro
  estágios da v3.4 (então o borrão lê uma saída **já filtrada**) e o módulo Rota
  grudado tinha borrão próprio **dentro** do cartão que já tinha o seu. Numa
  janela de 1280 são ~724×640px de borrão por quadro. Com vídeo dedicado passa
  despercebido; com vídeo integrado, não. Três frentes:
  · **Enquanto o mapa se mexe, o borrão sai** (`body.vidro-parado`, ligada no
    `movestart`/`zoomstart`, desligada 150ms depois do `moveend`) e entra uma
    tinta opaca **da mesma cor**: a mistura de 84%/94% passa a ser com
    `--map-bg` no lugar da transparência. Não é aproximação — o véu já deixa
    passar só ~13% do mapa, e a conta fecha no mesmo pixel (medido: srgb 0.0800
    nos dois casos no escuro, 0.9862 nos dois no claro). O desenho da linha da
    v8.0.0 congela junto, que é o mesmo tipo de movimento.
    ⚠️ **O atraso de 150ms para voltar não é enfeite**: o zoom pela rodinha
    dispara um `moveend` por degrau, e devolver o borrão entre um degrau e outro
    custaria mais caro que tê-lo deixado ligado. ⚠️ E há rede de segurança de
    2,5s, pela armadilha de sempre (v6.5/v6.9/v8.0.0): em aba sem pintura o
    `moveend` pode não chegar, e o vidro ficaria congelado para sempre.
  · **Interruptor no cabeçalho** (`hg_vidro`, ícone de camadas ao lado do tema,
    riscado e em âmbar quando desligado — o mesmo idioma da lupa do módulo 4):
    desligado, o app volta ao **layout de antes da v8.1.0**, com o painel opaco
    e o mapa dividindo a linha. Aí não há borrão nenhum **e** o mapa volta a
    ~metade da largura, então o filtro dos ladrilhos também custa menos — os
    dois custos de uma vez (medido numa janela de 1280: mapa de 1280px para
    918px, e 551px com uma coluna aberta). ⚠️ Mora em `data-vidro` no `<html>` e
    é aplicado pelo script do `<head>`, **antes da primeira pintura**, pelo
    mesmo motivo do tema. Sem escolha guardada, quem manda é o
    `prefers-reduced-transparency` do sistema — o "efeitos de transparência" do
    Windows. ⚠️ A regra vive dentro de `@media (min-width:761px)`: em tela
    estreita quem manda é o empilhado, e os dois modos dão o mesmo layout.
  ⚠️ **Lição que sobreviveu à v8.10.0**, mesmo com o sticky fora: o fundo de um
  filho pinta **acima** do véu do cartão (`inset 0 0 0 2000px rgba(0,0,0,.18)`),
  então uma tinta de 94% de `--panel` sai com a cor **sem** o véu — medido,
  rgb(22,35,42) contra os rgb(18,28,34) do cartão em volta. Quem precisar casar
  com o cartão tem de pôr o véu na conta: tinta sobre `--map-bg`, e o resultado
  a 82% sobre preto (65% sobre branco no tema claro).
  ⚠️ **O `#map` nunca pode virar `position:static`** (v8.3.0, relatado com
  print: "com o vidro desligado o mapa fica por cima"). Os painéis do Leaflet —
  ladrilhos, marcadores, traçado — são `position:absolute` e se ancoram no
  primeiro ancestral **posicionado**; com o mapa static esse ancestral vira o
  corpo da página, os painéis saem da caixa dele (o `overflow:hidden` não
  recorta um absoluto cujo ancestral é outro) e pintam **por cima do painel**,
  esticando a janela (medido: `offsetParent` do `.leaflet-map-pane` de `map`
  para o corpo, e o documento de 1280px para 1338px). Tanto a regra do vidro
  desligado quanto a de tela estreita usam `position:relative; inset:auto`.
  ⚠️ Até a v8.0.0 o `#map` não tinha `position` no CSS e o **próprio Leaflet**
  escrevia `relative` inline ao nascer — por isso isto nunca tinha aparecido. A
  v8.1.0 pôs `absolute`, e aí o Leaflet deixa de escrever o inline: quem abre
  com vidro e desliga depois fica sem a rede. ⚠️ E o teste que não pega isso é
  **recarregar já desligado** — aí o mapa nasce static, o Leaflet se salva
  sozinho e tudo parece certo. O caminho quebrado é abrir com vidro e desligar.
  ⚠️ `folgaDasColunas()` devolve **0** com o vidro desligado — o mapa não passa
  mais por baixo de nada. ⚠️ **A melhora não foi medida**: na máquina de casa o
  problema não reproduz. O que dá para afirmar é o que saiu do caminho —
  durante um arraste, de até **três borrões** para **nenhum**
- **Modo leve** (v8.7.0; ✅ **validado pelo usuário na máquina do trabalho em
  25/09/2026** — "utilizei o modo desempenho no trabalho e deu certo" — e
  **revalidado em 28/09/2026**, já com as seis versões da revisão de layout por
  cima: "modo leve com bom desempenho"; relatado
  depois de testar a v8.2.0 no trabalho: "ficou menos travado, mas notei que antes ficava mais fluido. Tem
  como criar um botão onde desativa tudo que possa deixar o site mais travado
  para computadores mais antigos?"). Um interruptor no cabeçalho (`hg_leve`,
  ícone de velocímetro, âmbar quando ligado) que **manda nos outros**: força o
  **vidro** e as **janelas livres** desligados — e com isso o mapa volta a
  ~metade da largura (medido a 1400px: de 1400 para 1053) — e ainda tira
  **transições, animações, sombras desfocadas** e o **efeito ao traçar**. Os
  botões de vidro e de janelas **somem** enquanto ele está ligado, porque a
  escolha deles não vale nada ali; o do ímã some junto, pela regra dele.
  ⚠️ **Desde a v8.25.0 eles não somem**: são linhas do menu "Exibição", que
  ficam travadas com o motivo escrito. O interruptor do modo leve também mora lá.
  ⚠️ **Não apaga a escolha dos outros dois**: força as duas desligadas sem tocar
  no que está guardado, e ao sair elas voltam exatamente como estavam. É
  aplicado pelo script do `<head>` **depois** dos outros dois, porque manda
  neles.
  ⚠️ **A exceção que não pode faltar**: matar `transition` em tudo quebraria o
  zoom do Leaflet, que termina a animação no evento de **fim da transição** —
  sem ela o `transitionend` não chega e a vista pode ficar presa no meio do
  zoom (a mesma família das armadilhas de relógio parado das v6.5/v6.9/v8.0.0).
  Uma regra devolve exatamente a declaração do próprio Leaflet para
  `.leaflet-zoom-anim .leaflet-zoom-animated`.
  ⚠️ **O que ele NÃO desliga**: o **filtro de cor dos ladrilhos** (v3.4). Ele
  existe desde a v3.2 e o app era fluido com ele — o que mudou na v8.1.0 foi o
  mapa virar tela cheia e ganhar o borrão por cima, e é isso que o modo leve
  desfaz. Tirar o filtro devolveria o cinza médio do Esri e quebraria a
  identidade do tema escuro, que custou três versões para acertar. Fica como a
  **próxima alavanca** se ainda estiver pesado
- **Janelas livres** (v8.4.0, pedido do usuário: "poder apertar no topo ao
  centro, e poder colocar a coluna inteira para qualquer parte da tela,
  parecendo bastante como um aplicativo"; depois: "baseado nesse mesmo caminho
  do botão de vidro, adicione um que permita a customização das colunas").
  **Botão no cabeçalho**, ao lado do vidro (`hg_janelas`, ícone mostrando o
  estado atual como o do tema: três colunas em fila, ou duas janelas soltas em
  âmbar). **Desligado por padrão — aí nada muda.** Ligado, cada coluna vira uma
  janela com **x, y, largura, altura e ordem de frente**: puxador no topo ao
  centro para mover, canto embaixo à direita para largura **e** altura, e tocar
  em qualquer lugar traz a janela para a frente. As bordas se **imantam** às da
  tela e às das outras janelas a menos de 10px, com uma linha âmbar mostrando o
  encontro (escolha do usuário: "totalmente livre, porém um leve auxílio para
  ficar alinhado quando soltado") — e esse ímã **tem botão próprio**
  (`hg_ima`), que só aparece com as janelas ligadas, porque fora delas não há o
  que alinhar. Desligado, a janela fica exatamente onde for solta; foi pedido
  depois de o modo ficar pronto ("para poder soltar aonde quiser"), e o usuário
  escolheu o botão entre três formas — as outras eram segurar Alt durante o
  arraste, e as duas juntas. O cabeçalho ficou
  `[disquete] [lixeira] | [janelas] [ímã] [vidro] [tema]`.
  ⚠️ Com o ímã desligado, `linhasDeApoio()` **nem é chamada**: o arraste recebe
  listas vazias, e com isso somem a atração e as linhas de apoio de uma vez —
  sem um segundo caminho no código do arraste. **Soltar um módulo no mapa** abre a coluna
  nova **no ponto onde foi solto**. A geometria entra no arranjo: o disquete
  salva, a lixeira devolve tudo.
  ⚠️ **Só a janela que foi arrastada é guardada** (marca `livre`); as outras são
  recalculadas pela geometria padrão a cada `aplicarJanelas()` — encostadas à
  direita, painel · coluna 2 · coluna 3. Sem isso o modo ligado abriria
  diferente do desligado, e o **disquete acenderia sozinho** ao abrir, porque a
  geometria padrão depende do tamanho da tela e dois computadores nunca dariam
  o mesmo JSON. ⚠️ O **z fica fora** do que se salva e do que se compara: ele
  muda a cada clique, e "trouxe para a frente" não é arranjo.
  ⚠️ **O modo exige o mapa por baixo**, então vale mesmo com o vidro desligado:
  sem vidro muda só o **material** da janela (opaca, com sombra própria), não o
  layout — por isso as regras do vidro desligado ganharam
  `:not([data-janelas="1"])` na parte de layout, e o material virou regra à
  parte. ⚠️ **O puxador e o canto não moram dentro da janela**: ela rola, e eles
  sumiriam junto com a rolagem; ficam em `main`, posicionados pelo mesmo código.
  ⚠️ **Posição de emergência no CSS** (`top/right/bottom`): entre o `<head>`
  marcar `data-janelas` e o JS posicionar as janelas há um quadro em que elas
  são absolutas **sem coordenada**, e aí ficariam onde calhar, com a altura do
  conteúdo. Encostadas à direita, esse quadro fica idêntico ao layout de sempre
  (medido: 1048/77, 340×731 nos dois casos).
  ⚠️ **`folgaDasColunas()` teve de ser reescrita**: somar as larguras só valia
  com as colunas enfileiradas à direita. Agora ela mede a **faixa colada na
  borda direita** que está coberta, encadeando janela a janela; janela solta no
  meio não conta, e o mapa enquadra na tela inteira. ⚠️ A tolerância do
  encadeamento é **24px** — entre duas colunas encostadas há a margem de 12 do
  cartão mais a divisória de 7 (19px de vão); com 13 a conta parava na primeira
  e devolvia 352 onde o certo eram 731
- **Um roteiro por técnico, em abas** (v8.5.0, pedido do usuário: "abrir uma
  segunda tela, como se fosse uma guia de navegador... ao traçar a rota consigo
  ver no mapa a rota de ambos, cada um em sua cor"). É a **Fase 2** adiada em
  29/08/2026. Uma barra de pílulas acima do mapa: cada aba tem a **sua**
  seleção, ordem, origem, rota e link, e o mapa mostra **todas ao mesmo tempo**
  — a ativa com opacidade .9 e traço 5, as outras .28 e traço 3, a ativa na
  frente. Abas se **adicionam e removem**, com nome editável (duplo clique), e
  seis cores cicladas: âmbar `#F2A93C` (o técnico 1 fica com a cor de sempre),
  `#A78BFA`, `#5BC0F8`, `#F97EB9`, `#B8E04A`, `#FF9A6B` — todas claras o
  bastante para o número escuro do marcador continuar legível, e nenhuma perto
  do teal dos não selecionados. **A origem é por técnico**, começando na padrão
  salva (escolha do usuário).
  ⚠️ **A tela do campo não mudou nada**: o link já carrega o roteiro inteiro com
  id próprio e o progresso já é por `rid` desde a v6.8.0 — cada técnico gera o
  dele e abre o dele.
  ⚠️ **Por que isto não virou uma refatoração gigante**: o estado de um roteiro
  são sete variáveis, mas lidas em **~150 lugares** (só `stops` aparece 83
  vezes). Em vez de trocar tudo para `tecnico.stops`, elas continuam sendo as do
  técnico **ativo**: trocar de aba guarda as atuais no objeto da aba e carrega
  as da outra. Nenhuma das ~150 chamadas mudou. A exceção são as **camadas do
  mapa**, que ficam no mapa o tempo todo — é justamente o que se quer ver junto.
  ⚠️ O que o traçado escreveu no painel (roteiro, km/min, link) é guardado como
  **texto já pronto**: refazer o traçado ao trocar de aba redesenharia o mapa e
  rodaria a animação da v8.0.0 de novo. ⚠️ `realcarCamadas()` começa chamando
  `salvarNoTecnico()` — as camadas do ativo vivem nas globais, e sem sincronizar
  antes ela leria a rota anterior.
  ⚠️ **Dois defeitos que a mudança criaria, resolvidos junto**: a **parada
  avulsa perdia o dono** (`limparAvulsasSoltas()` apagava toda avulsa fora da
  `stops`, então a do técnico 2 sumiria ao mexer na lista do técnico 1 — agora
  "solta" é a que não está na viagem de *nenhum*), e o **realce do mapa cruzava
  as abas** (o mouse num marcador do técnico 2 acendia outra parada na lista do
  técnico 1 — os manipuladores guardam de quem é a rota).
  ⚠️ **As abas moram dentro do cabeçalho**, ao lado do selo de versão (pedido do
  usuário; antes eram uma barra própria que comia 40px de altura mesmo com um
  técnico só). `align-self:center` porque o header é `align-items:baseline`; com
  o `flex-wrap` que ele já tinha, em tela estreita elas caem sozinhas para uma
  linha só delas (medido a 375px: header de 102px). **Um clique no nome da aba
  aberta** entra na edição com o texto inteiro selecionado, então digitar troca
  "Técnico 1" pelo nome da pessoa
- **A troca de técnico tem passagem, em dois tempos** (v8.14.0, pedido do
  usuário: "implemente um efeito fade ao trocar de técnico"; e, depois de ver a
  primeira forma rodando, "pensei em algo como a opção D, porém quando abrir o
  outro técnico, aplicar um efeito estilo cascata, que vai aparecendo aos
  poucos"). Até aqui a troca acontecia num **quadro só**: origem, lista de
  clientes, ordem da viagem, km/min, link, roteiro e a rota acesa no mapa
  mudavam todos de uma vez, sem nada dizendo que o painel passou a ser de outra
  pessoa. Agora: **(1)** o painel de quem sai **apaga junto, em 100ms**;
  **(2)** o de quem entra volta **módulo a módulo, de cima para baixo** — 55ms
  de um para o outro, cada um com **380ms** de fade e 7px de subida (o fade
  nasceu em 190ms e o usuário pediu o dobro depois de ver rodando); **(3)** no mapa
  a **linha da aba que sai apaga em 260ms** (`stroke-opacity` .9→.28 e
  `stroke-width` 5→3), com os marcadores dela junto (1→.38). Vale também ao
  **abrir e ao fechar** uma aba. A saída é o "limpou"; a cascata é o "encheu de
  novo com outra coisa".
  ⚠️ **A troca de verdade acontece DEPOIS da saída**, dentro do `setTimeout`. É
  o preço do primeiro tempo, e é por isso que ele é curto: **100ms é o limite em
  que um clique ainda parece instantâneo**. Alongar a saída é alongar o atraso
  do clique — não mexer nesse número sem saber disso.
  ⚠️ **`animation-fill-mode: both` é o que faz a cascata existir**: sem ele, o
  módulo com atraso de 275ms ficaria visível durante a espera e só então
  piscaria para o começo da animação.
  ⚠️ **A duração é quase sete vezes o passo**, e é isso que faz a coisa ser uma
  **onda** subindo pelo painel e não uma escada de blocos acendendo um a um: os
  módulos entram muito sobrepostos. Mexer só no passo muda o desenho — mais
  passo, mais escada.
  ⚠️ **O atraso é contado por contêiner**: painel e cada coluna começam do zero
  e cascateiam ao mesmo tempo. Uma cascata só, atravessando as três colunas,
  deixaria a última esperando meio segundo. ⚠️ E **módulo escondido não gasta
  uma casa** da cascata, senão abre um buraco no meio dela — por isso a ordem
  sai do DOM e a visibilidade de `offsetParent`, e não de um `nth-child`.
  ⚠️ **Com o modo leve não some só a animação: some o relógio.**
  `passagemLigada()` é consultado antes de qualquer coisa, e ali a troca é a de
  sempre, no mesmo quadro. A espera de 100ms sem a animação seria atraso puro,
  sem nada em troca. O mesmo vale para `prefers-reduced-motion`.
  ⚠️ **Duas redes de segurança, as duas obrigatórias.** A classe de saída é
  tirada **antes** de escrever o conteúdo novo, no mesmo quadro — se algo
  estourar no meio, o painel não fica invisível. E o `setTimeout` que tira a
  classe de entrada é o que salva a **aba que não está pintando** (armadilha
  das v6.5/v6.9/v8.0.0): ali as animações não correm e o `both` seguraria o
  painel em opacidade 0 para sempre. Testado: com as animações paradas, o
  painel volta inteiro assim que a classe sai.
  ⚠️ **`abaPedida` existe por causa de um bug pego no teste**: clicar na aba B
  e, dentro dos 100ms, voltar para a A deixava a pessoa na B — a guarda "já
  estou nela" comparava com `tecnicoAtivo`, que ainda era o antigo enquanto a
  troca estava pendente. A guarda passou a olhar para o **destino da troca em
  curso**.
  ⚠️ **Os atrasos são zerados antes da saída**, e isso não é zelo: eles ficam
  **inline** no elemento desde a cascata anterior, e sem zerar a saída desfila
  junto — e torta, porque um módulo que estava escondido na troca passada não
  tem atraso e os outros têm. Só apareceu com os **seis módulos à vista**, na
  base real; com poucos módulos abertos passa despercebido.
  ⚠️ **Anima o `.modulo-corpo`, não a `section`**: no painel ela é
  `display:contents` e não gera caixa. Assim o efeito vale igual no painel, nas
  colunas, nas janelas livres e no empilhado de tela estreita (testados os
  quatro). ⚠️ O módulo **Clientes fica de fora**: a base é a mesma para todas as
  abas, e piscá-la seria dizer que mudou algo que não mudou — é também o que
  mantém o painel ancorado enquanto o resto vai e volta.
  ⚠️ **A transição da linha vai INLINE**, em `realcarCamadas()`, e não numa
  regra de folha de estilo: o efeito de traçar (v8.0.0) escreve `transition`
  inline no mesmo path, e a partir daí o inline ganharia de qualquer regra
  externa. `stroke-opacity` e `stroke-width` vêm de `setStyle()` como
  **atributo**, e atributo de apresentação de SVG é propriedade CSS — por isso
  transiciona (confirmado por `getAnimations()`, que devolve as duas). A
  `opacity` do elemento fica de fora de propósito: é a que o efeito de traçar
  usa para esconder a linha durante o voo.
  ⚠️ Trocar de aba **no meio de um traçado** apaga a transição do desenho e a
  linha aparece inteira — comportamento desejado: quem trocou de aba não está
  mais olhando aquela rota
- **O que o mapa mostra dos clientes, em três estados** (v8.15.0, pedido do
  usuário em duas partes: primeiro "esconder as bolinhas no mapa que
  representam as paradas, deixando somente as que pertencerem à rota traçada,
  para uma visualização mais limpa da rota em si"; depois, vendo aquilo rodando,
  "realmente a opção a) faz muito sentido" — apagar em vez de sumir — com o
  esclarecimento de que "apareceriam os pontos de todos os técnicos, como é
  hoje, porém os que não fazem parte do que foi planejado ficariam como na
  opção a)"). Um botão no cabeçalho, **primeiro do bloco depois do divisor** —
  `[disquete] [lixeira] | [foco] [leve] [janelas] [ímã] [vidro] [tema]` — que
  **cicla em três**: **0** tudo à vista (padrão, nada muda), **1** os clientes
  fora da viagem **apagados** (opacidade .18, ainda clicáveis), **2** os
  clientes fora da viagem **escondidos**. Em todos, o que fica inteiro é a
  linha, os números, a origem e as paradas de **todos** os técnicos. Com a base
  real são 39 bolinhas no mapa e 5 ou 6 na viagem: o traçado nascia dentro de um
  campo de pontos que não têm nada a ver com ele (medido: 34 de 39 saem de
  cena). O ícone mostra o **estado atual**, como o do tema — pontos espalhados,
  a rota com os vizinhos apagados em volta, a rota sozinha —, discreto no 0 e
  âmbar nos dois filtrados. A escolha fica guardada (`hg_foco_rota`).
  ⚠️ **É o único botão do cabeçalho com três estados**, e por isso **não é
  `role="switch"`**: switch é de dois, e `aria-checked` mentiria no do meio.
  O `aria-label` diz o estado atual e a dica do mouse diz para onde o clique
  leva, como no botão de tema.
  ⚠️ **Por que três e não dois**: apagar e esconder não respondem à mesma coisa.
  Apagado, o cliente vizinho continua ali **para ser clicado e entrar na
  viagem** — é isso que ataca a desconfiança do "passei na porta de um cliente
  do roteiro" medida em 19/09/2026, porque o vizinho fica visível sem competir
  com a rota. Escondido, a rota fica sozinha na tela, que foi o pedido original.
  ⚠️ **Quanto apaga é decisão do CSS, não do JS**: o JS só marca quem está fora
  com `.ponto-fora-da-rota`, e `data-foco` no `<html>` escolhe entre .18 e 0.
  Trocar do apagado para o escondido é trocar um atributo, e a passagem entre os
  dois continua suave.
  ⚠️ **O recorte é "está na viagem de ALGUM técnico"**, e não "da aba aberta".
  O mapa mostra todas as rotas ao mesmo tempo desde a v8.5.0 — esconder os
  pontos do técnico 2 porque a aba do 1 está aberta apagaria metade do dia a
  cada clique numa aba. É a mesma regra de dono de `limparAvulsasSoltas()`.
  ⚠️ **E o recorte é a VIAGEM, não a rota traçada**, apesar de o pedido falar em
  rota: assim o botão já serve enquanto se escolhe as paradas, e a lista vai
  ficando limpa junto. Recortar pela rota faria o botão não fazer nada até o
  traçado — e depois **desfazer sozinho** o efeito a cada parada acrescentada,
  porque acrescentar desfaz a rota (v4.1).
  ⚠️ **Sem nenhuma parada escolhida o foco não vale**, e isso não é detalhe:
  seria um mapa vazio, sem nada explicando o que aconteceu, logo depois de um
  clique num botão. Aí não há o que limpar, a base fica inteira à vista e a dica
  do botão diz isso. É o que faz abrir o app com a chave ligada ser seguro.
  ⚠️ **O ponto escondido continua no mapa**, apagado (`opacity:0` +
  `pointer-events:none`), e não é removido da camada: removê-lo e recolocá-lo a
  cada clique no checklist custaria 30 e tantas operações do Leaflet por
  marcação, e quebraria o realce mapa↔lista da v6.7.0, que conta com
  `marcadorDaParada()` achando a bolinha do cliente **antes** de a rota existir.
  ⚠️ O `pointer-events:none` vale **só no estado escondido**, e é obrigatório
  lá: um marcador a opacidade 0 continua clicável, e o balão de um cliente
  invisível abriria do nada. No **apagado** é o contrário — o ponto continua
  clicável de propósito, e o `:hover` devolve a opacidade inteira, senão se
  estaria apontando para algo que não se vê. ⚠️ O realce mapa↔lista não acende
  num ponto fora da viagem, e **nunca acendeu**: `realcarPeloMarcador` só age
  quando o ponto está em `stops`. Não confundir com defeito do apagado.
  ⚠️ **A suavidade é de graça e o modo leve é seguro por construção**: quem
  apaga é a transição que `.leaflet-marker-icon` já tem desde a v8.14.0. Não há
  classe de saída nem espera de `transitionend` — com o modo leve a transição
  morre e o ponto simplesmente muda (medido: 1,2ms, zero animações). Por isso o
  botão **continua visível no modo leve**, ao contrário do vidro e das janelas:
  ele não custa desenho, tira.
  ⚠️ O gancho é `renderStopsList()`, que é o ponto por onde **toda** mudança na
  viagem passa — marcar, desmarcar, remover, reordenar, trocar/abrir/fechar aba,
  retomar por link, restaurar o dia, trocar de base.
  ⚠️ Custo medido no cabeçalho: o bloco de ações foi de 229 para 265px, e a
  altura continua em 65px (uma linha) a 1400, 1366, 1200 e 1024. **Abaixo de
  ~430px** o bloco passa a cair para uma linha só dele (medido a 420px: 102 →
  144px) — largura que já não é caso de uso do escritório
- **A parada é do técnico, não da base** (v8.9.0, bug achado na revisão de
  24/09/2026). `stops` guarda uma **cópia rasa** do ponto de `clientPoints`, e
  não o próprio objeto. ⚠️ Antes disso, com o mesmo cliente na rota de **dois
  técnicos**, os dois apontavam para o mesmo objeto: pôr ★ ou trocar o tipo num
  mudava no outro, sem aviso — justamente o caso que justifica permitir o
  repetido ("entrega de manhã, manutenção à tarde"). ⚠️ O preço é que `stops`
  **não pode mais ser comparado por identidade** com `clientPoints`: as quatro
  comparações que faziam isso (`toggleStop`, "marcar todos",
  `limparAvulsasSoltas` e o índice da avulsa recém-criada) viraram comparação
  de `id`. Se aparecer outra, o sintoma é uma parada que não é encontrada.
  ⚠️ A `marker` vai junto na cópia de propósito: é o mesmo desenho no mapa para
  todo mundo, e `marcadorDaParada()` conta com ela antes de a rota ser traçada
- **Aviso de parada já tomada por outro técnico** (v8.5.0): uma bolinha na
  **cor do outro técnico** na unidade ("Já está na rota de Jonas"), na linha do
  cliente e na do grupo do uMap ("Jonas tem parada aqui dentro"). **Avisa, mas
  não bloqueia** — há caso legítimo, como entrega de manhã e manutenção à
  tarde. ⚠️ Fica do lado **direito**, junto do contador: a calha da esquerda
  acabou de ser limpa na v8.4.2 e não volta a ter duas marcas. Só aparece
  quando há o que dizer, então nunca é enfeite — que foi o problema da bolinha
  antiga
- **Quilometragem somada do dia** (v8.8.0, pedido do usuário depois de o modo
  de vários técnicos ficar pronto): uma **pílula no cabeçalho**, logo depois das
  abas — `QUILOMETRAGEM APROXIMADA` e o total em km. Soma as rotas
  **traçadas** de todas as abas;
  rota desfeita não conta, porque o número dela deixou de valer. Só aparece com
  **duas ou mais**: com uma só, o total é exatamente o que o módulo Rota já
  mostra a meio metro dali.
  ⚠️ **Ela nasceu no módulo Rota e o usuário pediu para mudar** — com razão, e
  por duas: aquele módulo é o de **uma** rota, e ele pode ser arrastado para
  qualquer coluna, então o número do dia mudaria de lugar junto. O cabeçalho é
  fixo, e é onde o dia mora. Medido a 1400px: pílula de 109×28 entre as abas
  (que terminam em 1016) e os botões (que começam em 1149).
  ⚠️ **"Aproximada" é a parte que importa**: o número vem do cálculo da rota no
  OSRM, não de odômetro — é o que se espera rodar, antes do trânsito e dos
  desvios do dia.
  ⚠️ **PRIMEIRA MEDIÇÃO DE CAMPO, 28/09/2026**: o app previu **27 km** e o
  usuário rodou **31 km** — **15% a mais**. Ele mesmo apontou a causa principal:
  *"precisei fazer pequenos quadrados para estacionar"*. São três coisas
  somadas: (1) as manobras, que o traçado porta-a-porta não conhece; (2) o
  caminho real diferir do calculado, que é o que a análise de 19/09/2026 já
  tinha medido entre Waze e OSRM; e (3) o próprio **odômetro**, que em carro de
  série costuma ler 2 a 5% alto. Para uma estimativa feita antes de sair, 15% é
  erro normal.
  ⚠️ **Um ponto não faz média.** Se a diferença se repetir perto de 15% ao longo
  de semanas, aí é fator sistemático e cabe um **ajuste configurável** — mas só
  vale a pena se o número for usado para combustível ou reembolso. Não
  implementar antes de ter a série. O rótulo levou três tentativas: nasceu "DIA" (curto demais),
  passou por "total do dia / a percorrer / percorrido" e o usuário trouxe o que
  ficou. ⚠️ Em português é **quilometragem**, com "qu" — só a abreviação "km"
  leva k. ⚠️ Quantas rotas são fica na dica do mouse: as abas ao lado já dizem.
  ⚠️ **O número é âmbar**, a mesma cor em que o módulo Rota escreve o km/min de
  cada rota — os dois números do app que dizem distância falam a mesma língua.
  ⚠️ O rótulo longo levou a pílula de 109px para **249px**: cabe no 1366 da TV
  com três abas numa linha só, mas **abaixo de ~1100px o cabeçalho passa a
  quebrar em duas linhas** (medido a 1024: 107px). Não é resolução-alvo, mas
  com o rótulo curto isso não acontecia.
  ⚠️ **O tempo não é somado**, e fica só na dica do mouse: os técnicos saem ao
  mesmo tempo, então somar as durações daria um número grande e errado bem ao
  lado de um número certo. Quem responde "quando o dia acaba" é a **rota mais
  longa**, e é isso que a dica diz.
  ⚠️ A soma lê o resumo do técnico ativo da variável global e o das outras abas
  do objeto de cada uma, em vez de chamar `salvarNoTecnico()` — que mexeria no
  DOM no meio da montagem do DOM
- **O planejamento do dia sobrevive ao recarregar** (v8.5.0, pedido do usuário).
  Até aqui só a **configuração** era guardada; o roteiro do dia morria no F5, de
  propósito. Com as abas a conta mudou de tamanho: um F5 sem querer passou a
  custar o planejamento dos **três** técnicos de uma vez. Agora ele vai para
  `hg_dia_planejado` e, ao abrir, uma faixa **oferece** restaurar — não restaura
  sozinho, senão a seleção de ontem voltaria sem ninguém pedir. "Restaurar" ou
  "Começar do zero".
  ⚠️ **A rota traçada não é guardada**, só o planejamento (paradas, ordem, ★,
  tipo, origem, nome e cor de cada aba): a geometria do OSRM é grande, envelhece,
  e retraçar é um clique — o trabalho de verdade é escolher e ordenar.
  ⚠️ **A armadilha**, que apareceu na primeira tentativa: ao abrir,
  `renderStopsList()` roda com a lista vazia e gravaria um dia **vazio** por cima
  do que se quer restaurar, antes de a pessoa ver a faixa. A gravação só começa
  depois que ela decide, ou quando o planejamento deixa de estar vazio — este
  último caso cobre quem ignora a faixa e começa a trabalhar.
  ⚠️ **A parada é reencontrada pela coordenada**, com o id só como reserva: o id
  de um ponto da base é `c<índice da camada>_<id da feature>`, então basta o uMap
  reordenar ou renomear uma camada para todos mudarem e o planejamento voltar
  vazio (pego no teste). É a mesma regra da retomada por link desde a v6.8.0, e
  a avulsa é recriada das coordenadas como lá. ⚠️ A faixa só aparece com **base
  carregada**: sem ela as paradas não têm como ser reencontradas
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
  na tela **mapa · coluna 3 · coluna 2 · painel**. Os **seis** módulos —
  Clientes, Origem, Selecionar paradas, Ordem da viagem, Rota (botões, status e
  o envio para o campo) e Roteiro — têm alça no título.
  ⚠️ **Eram SETE até a v8.17.1**: o "Enviar para o campo" era o módulo `m6` e
  virou parte do módulo Rota na v8.18.0 (ver o item próprio, mais abaixo). O
  apelido `m6` **não foi reaproveitado** — o Roteiro continua sendo `m7`, como
  em todo arranjo já salvo por aí.
  ⚠️ **A ordem em que eles nascem no painel não é a dos apelidos** desde a
  v8.13.0: é `ORDEM_PADRAO` = **m1, m2, m5, m3, m4, m7** — Clientes, Origem,
  **Rota**, Selecionar paradas, Ordem da viagem, Roteiro. `ORDEM_MODULOS`
  continua sendo só a lista de apelidos, para iterar. O **markup do `<body>`
  segue a mesma ordem**, para a primeira pintura já sair certa. E o **duplo
  clique** (que joga o módulo "na posição da numeração" ao trocar de coluna) usa
  `ORDEM_PADRAO` também — senão devolveria a Rota para baixo do módulo 4.
  ⚠️ Arranjo salvo antes disso abre **como foi salvo**, com a ordem antiga, e o
  disquete não acende; só a lixeira devolve a ordem nova.
  ⚠️ **O PADRÃO DE ABERTURA depende do tamanho da tela desde a v8.19.0**
  (ponto 5 da revisão de layout de 26/09/2026). Acima de **1600px** de largura,
  **Selecionar paradas** e **Ordem da viagem** nascem na **coluna 2**; abaixo
  disso, tudo no painel como sempre. Medido na v8.18.0, a 1920×1080 com rota
  traçada e link gerado: o painel pedia **1442px** e tinha **991px** — **451px
  (31%) fora da vista** — enquanto ocupava **17,7% da largura**, com 1568px de
  mapa parado ao lado. Depois: **nada rola**, nem painel nem coluna, e o mapa
  fica com **1189px**.
  ⚠️ **A solução já existia desde a v5.7; o que faltava era o padrão.** As
  colunas sempre funcionaram — quem abria numa tela de 1920 é que recebia um
  layout de tela estreita e rolava metade do painel sem descobrir que bastava
  arrastar dois módulos para o lado. Um padrão é uma opinião sobre o caso mais
  comum, e a opinião estava errada para tela grande.
  ⚠️ **A decisão é tomada UMA VEZ, na abertura, e congelada em
  `abreEmDuasColunas`.** Não é zelo: `aplicarArranjo()` roda a cada `resize` e
  chama `atualizarBotaoArranjo()`, que compara a tela com
  `arranjoGuardado() || arranjoNormal()`. Lendo a largura ao vivo, arrastar a
  janela por cima de 1600px mudaria a referência e o **disquete acenderia
  sozinho** — a mesma armadilha que a v8.4.0 documenta para as janelas livres.
  Testado: 1920 → 1400 → 1920 não reorganiza nada e não acende nada.
  ⚠️ **`arranjoNormal()` passou a canonizar o que devolve**, e isso é
  obrigatório: `descreverArranjo()` compara o JSON da tela (sempre canonizado)
  com o do normal. Sem canonizar, a ordem sairia `m1 m2 m5 m3 m4 m7` contra
  `m1 m2 m5 m7 m3 m4` e o disquete **nasceria aceso**.
  ⚠️ **Quem salvou arranjo continua mandando** — isto é só o fallback. E quem
  não gostar do padrão faz o de sempre: arrasta e salva. Testado.
  ⚠️ **A lixeira mudou de texto, não de comportamento**: "voltar ao normal"
  sempre significou "o arranjo padrão", e acima de 1600px o padrão agora inclui
  a coluna 2. Prometer "tudo no painel" ali seria mentira, então o título do
  botão e o `confirm()` são montados a partir de `textoDoArranjoNormal()`.
  ⚠️ **1600 e não 1366**: a 1366 o mapa cairia para ~626px, e a TV deixou de
  guiar o layout em 11/09/2026 (decisão 6). A 1600 sobram 869px de mapa
  (medido). **Isto não resolve os 53% de painel fora da vista da TV** — lá
  continua valendo arrastar à mão e salvar. O número é um só:
  `LARGURA_DUAS_COLUNAS`.
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
  (v6.1.1). O contorno é **só a borda**, sem preenchimento (v6.1.2) — e **o vão
  também**, desde a v7.9.1 (pedido do usuário, com print: "apenas o contorno em
  amarelo indicando a posição, e que a textura dentro da marcação saísse"). O
  nome do módulo continua no meio do vão, que é o que diz qual módulo está sendo
  movido. A alça do
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
  esvazia, o conteúdo da 3 passa para ela. Módulo escondido (4 sem paradas,
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
  (v4.6, crescia empurrando o resto; a v4.2 fazia isso e foi substituída).
  ⚠️ **Correção de 24/09/2026:** este documento afirmava que a chave
  `hg_alturas_modulos` também tinha saído. **Não saiu** — ela continua viva e em
  uso (`alturasGuardadas()`), guardando as alturas do checklist e da lista de
  paradas no painel. O resto saiu quando o
  painel passou a repartir altura como as colunas: hoje o ajuste é a **divisória
  entre módulos com lista** e o **puxador do último**, iguais nos três lugares.
  ⚠️ **Os puxadores ganharam TETO na v8.11.0** (relatado pelo usuário com
  print: "o módulo de clientes pode ser expandido passando pelo módulo de
  Rota"). Os três — checklist, lista de paradas e o das colunas — só tinham
  piso; arrastar para baixo crescia sem limite. Medido num contêiner de 741px
  visíveis: **960px**, **940px** e **1321px**. As **divisórias** nunca tiveram
  o problema, porque conservam uma soma e já limitavam os dois lados.
  A regra: **nenhum módulo mais alto que o contêiner que o mostra, menos o
  mínimo de um módulo** (741 → 621). Uma lista maior que o painel é inútil de
  qualquer forma — rola-se o painel para ver uma lista que também rola, que é a
  dupla rolagem rejeitada na v6.4.1.
  ⚠️ A regra **não** promete que tudo caiba sem rolar, de propósito: o painel
  foi feito para rolar e o módulo Rota flutuar sobre a lista é a v6.5
  funcionando. Ela impede o caso absurdo, não o projeto.
  ⚠️ **Uma tentativa que não serve, para ninguém refazer**: medir o espaço livre
  com `clientHeight - acima - abaixo`, tirando `abaixo` do `scrollHeight`.
  Quando o conteúdo **cabe**, o `scrollHeight` fica preso na altura da caixa e a
  conta degenera para "o teto é a altura atual" — o módulo nunca mais cresce
  (pego no teste: o checklist foi a 120px sozinho no arranque). Somar os filhos
  à mão também não fecha (medido 964 contra 932), porque as sections do painel
  são `display:contents` e o módulo Rota é `sticky`.
  ⚠️ `corrigirAlturasDoPainel()` aperta o módulo quando a **janela encolhe**,
  comparando sempre contra o valor **guardado** e sem gravar — alargando a
  janela de volta, ele volta ao tamanho escolhido (medido: guardado 600, tela
  431 numa janela de 560, e 600 de novo a 830).
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
- **Rota otimizada** — OSRM Trip API recalcula a melhor sequência, reordena a
  lista e traça (`source=first`, `roundtrip=false`). É o **botão principal** do
  módulo Rota desde a v7.9.0 (ponto 4 da revisão de design). O nome diz o
  **resultado**, não a mecânica — chamava-se "Otimizar e traçar" até a v8.12.0,
  quando o usuário pediu o nome curto e **sem o preenchimento âmbar** ("tire a
  cor de background, deixando o botão mais limpo").
  ⚠️ A hierarquia continua de pé por dois outros caminhos: ele é o **dobro da
  largura** do vizinho (medido: 185px contra 105px) e usa o **âmbar**, a cor
  forte do app, contra o teal do alternativo. É a mesma forma que o usuário
  escolheu para os botões de navegação da tela do campo na v7.8.0
- **Nesta ordem** — rota respeitando a ordem que está na lista (OSRM Route API),
  sem reordenar. Virou a **alternativa** (contorno teal, estreita): traçar na
  ordem escolhida à mão é o caso especial, não o uso normal.
  ⚠️ O **Enter no campo de origem** continua disparando este, e não o principal:
  quem digita um endereço e aperta Enter não espera que a ordem das paradas mude
  sozinha
- **O módulo Rota rola como os outros** (v8.12.0, pedido do usuário: "a questão
  do scroll no módulo Rota está me incomodando, aquele background não me agrada,
  precisa ficar fluido igual os outros"). ⚠️ **Isto desfez a v6.5.0**, que o
  deixava `sticky` — preso na base da coluna com o resto rolando por baixo.
  Saíram juntos: a regra sticky, o **fundo do estado grudado nas quatro
  variantes** que ele acumulou (vidro escuro, tema claro, sem vidro e o fio do
  modo leve), o `padding:16px 0`/`margin:-16px 0`, o modo compacto, a sentinela
  com a margem de -14px, e `atualizarRotaGrudada()` com toda a escuta dela.
  **159 linhas a menos.** Medido depois: zero posições de rolagem com fundo ou
  sombra, e 14px de respiro em todos os seis vãos do painel.
  ⚠️ **O custo disso durou uma versão.** A v6.5 existia porque os botões de
  traçar ficavam abaixo da dobra com a base carregada, e tirar o sticky trouxe
  isso de volta. O usuário resolveu na **v8.13.0** com uma ideia melhor do que
  a original: em vez de um módulo flutuando sobre a lista, a **Rota passou a
  nascer logo abaixo da Origem**. Medido: o botão a 142px do topo do painel,
  visível sem rolar mesmo com todos os módulos abertos.
  ⚠️ **De brinde, o suspeito de desempenho foi junto**: a revisão de 24/09 tinha
  deixado em aberto o `scroll` em `main` na fase de captura, que disparava duas
  leituras de caixa a cada rolagem de qualquer lista. Não há mais nada
  escutando rolagem no app. Continua **sem medição** — aqui o problema não
  reproduz; o que dá para afirmar é que o código saiu
- **Voltar para a origem no fim** (v6.3): interruptor no módulo Rota, guardado no
  navegador (`hg_voltar_origem`), **ligado por padrão**. A origem entra como último
  ponto do traçado, o otimizador fecha o círculo (`destination=last`, com a origem
  nas coordenadas e o waypoint dela descartado ao remontar as paradas), o Roteiro
  ganha "Retorno · <origem>" e o link leva o retorno (formato v7). Trocar o
  interruptor desfaz a rota traçada — km, tempo e ordem mudam. Medido na base de
  teste: 16,2 km sem volta, 27,1 km com volta na mesma ordem, e 24,1 km quando o
  otimizador trabalha sabendo que precisa voltar
- Marcadores numerados conforme a ordem final de visita
- **Efeito ao traçar a rota** (v8.0.0, pedido do usuário; ele escolheu o conjunto
  entre quatro efeitos comparados lado a lado em mapas de verdade): o mapa **voa**
  até o enquadramento (1,2 s), a **linha se desenha** da origem até o fim (900 ms,
  por `stroke-dasharray`/`stroke-dashoffset` no path do Leaflet), cada **número
  acende quando a linha chega nele** — a fração vem das `legs` do OSRM, distância
  acumulada sobre o total, então não é intervalo fixo — e o **km/min conta** de
  zero até o valor (600 ms). ~2,1 s no total, com o mapa usável o tempo inteiro.
  Antes a rota aparecia inteira de uma vez e nada indicava o sentido da viagem.
  **Desligado** quando o sistema pede `prefers-reduced-motion` (aí o enquadramento
  é instantâneo, sem nem o deslize) ou quando o Leaflet não devolve o elemento do
  path. ⚠️ **Duas armadilhas resolvidas, as duas da mesma família das v6.5/v6.9:**
  em aba de segundo plano o navegador pausa o relógio de quadros, o voo não
  termina, o `moveend` não chega e o Leaflet **recorta o traçado ao que está
  visível** — medir o path ali devolve comprimento **zero** e a rota sumiria
  (reproduzido: mapa travado no zoom 19, `d` do path com 4 caracteres). Um
  `setTimeout` de 1,35 s enquadra na marra e mostra tudo pronto nesse caso. O
  contador de km/min tem o mesmo tipo de rede. E traçar de novo no meio da
  animação não faz a linha antiga voltar: cada traçado tem um número e só o mais
  recente desenha
- **Clicar na parada leva o mapa até ela** (v6.9.0): clique no nome ou no número
  centraliza o mapa na parada com zoom 16, abre o balão e acende o marcador. O
  zoom **nunca afasta** (se já estiver mais perto, só centraliza). Alça e botões
  mantêm a função deles.
  ⚠️ **CORRIGIDO NA v8.17.0** (relatado pelo usuário: "ao dar o zoom em qualquer
  parada que esteja dentro da ordem de viagem parece que a tela não acompanha
  muito bem. Preciso que o zoom e foco passe a ser o ponto clicado"). O centro
  calculado sempre esteve certo — o que falhava era o **`setView` animado não
  chegar ao fim**, e ele era o **único** caminho até o resultado. Medido na base
  real, seis paradas clicadas em sequência: com `animate:false` o ponto cai no
  alvo exato e o zoom vai a 16 nas seis; com `animate:true` (como o app fazia), o
  mapa **não se mexe** — o zoom fica em 15 e a parada para entre **69 e 250px**
  do alvo. Depois da correção: **erro de 1 pixel** nas seis, zoom 16 em todas.
  ⚠️ **A lição é a regra do projeto aplicada a um lugar que tinha escapado**: a
  animação não pode ser o único caminho até o resultado. `centralizarVisivel()`
  agora confere o destino 420ms depois e aplica de uma vez se não chegou — a
  mesma rede de `setTimeout` das v6.5/v6.9/v8.0.0/v8.1.0.
  ⚠️ **A conferência desiste se a pessoa arrastou o mapa** nesse meio tempo
  (`arrastouDepois`, ligado no `dragstart`, que o Leaflet só dispara em gesto de
  verdade — `setView` não dispara). Sem isso a correção puxaria o mapa de volta
  debaixo da mão de quem está olhando outra coisa.
  ⚠️ **A v8.17.0 nasceu com um defeito próprio, corrigido na v8.17.1** (relatado
  na hora: "tem uma animação que parece um delay meio travado"). A rede comparava
  o centro **em graus, com 1e-7** (~1cm). Medido: o Leaflet erra **0,42 pixel**
  ao aplicar um centro mesmo **sem animação nenhuma** — ou seja, "chegou" dava
  falso SEMPRE e a rede disparava um `setView` seco no fim de toda ida. A
  animação rodava e, um tempo depois, vinha o solavanco. Agora a comparação é
  **em pixel, com folga de 2**: medido no mesmo ponto, a regra antiga dá `false`
  e a nova dá `true`, e a rede não dispara nenhuma vez com o mapa já no lugar.
  **Lição: tolerância em unidade de mundo para julgar coisa de tela é armadilha
  — o número parece pequeno e é quinze vezes menor que o erro do próprio
  Leaflet.**
  ⚠️ **E a ida virou VOO (`flyTo`), não `setView` animado.** O `setView` animado
  tenta um zoom de 0,25s e **desiste quando o destino está longe**, caindo num
  salto instantâneo — o "nada, nada, PULO" que também pesava na sensação de
  travado. O `flyTo` sempre anima, em arco, perto ou longe; é o mesmo recurso do
  efeito ao traçar (v8.0.0), só que curto (`DUR_VOO_PARADA`, 0,55s).
  ⚠️ **São dois relógios, e os dois precisam existir**: o de 140ms pega "a
  animação nem começou" e resolve na hora, sem fazer ninguém esperar o voo
  inteiro; o de 770ms pega "começou e parou no meio". Um só teria de escolher
  entre demorar à toa e cortar o voo pela metade.
  ⚠️ **No modo leve nem se tenta animar**, e o clique fica instantâneo (medido:
  1,8ms). É o comportamento certo na máquina fraca — lá o borrão do vidro entra e
  sai a cada `movestart`/`moveend`, e é isso que faz a tela "não acompanhar".
  ⚠️ O que **não** mudou: com o zoom bloqueado (v7.3.0) o mapa continua parado e
  só o balão abre; e a parada avulsa recém-criada continua levando o mapa até
  ela mesmo bloqueado (testados os dois)
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
  Rota (v6.2; antes ficavam no topo do Roteiro, no fim do painel).
  ⚠️ **A linha de status some quando está vazia** (v8.1.0, pelo print do usuário):
  `.status` tem `min-height:16px` e `#status` 14px de margem, então ela reservava
  **30px** embaixo dos botões mesmo sem recado nenhum — dentro do cartão de vidro
  essa sobra ficou à vista. `#status:empty{display:none}`, a mesma regra que a
  linha de avisos do módulo Clientes usa desde a v7.4.0. Instruções passo
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
  **O nome do técnico aparece no cabeçalho** (v8.6.0, pedido do usuário): uma
  pílula com o nome da aba que gerou o link, **ao lado do selo de versão**, na
  mesma forma dele mas em **âmbar** — o selo de versão é informação de suporte,
  e este é de quem o roteiro é. Link gerado antes disso não traz nome e a pílula
  simplesmente não aparece.
  **Formato v8** (v6.8.0): 7º grupo = retorno ("1"/vazio), 8º = **id do roteiro**
  (rid), 9º = **origem** "lat*lng*rótulo". No v7 a coordenada do retorno vinha no
  7º grupo; a leitura entende os dois, e links v5/v6/v7 continuam abrindo.
  **Formato v9** (v8.6.0): 10º grupo = **nome do técnico**. Os nove anteriores
  não mudaram de posição nem de significado, e a leitura trata v8 e v9 juntos
  onde eles são iguais (rid, retorno e origem moram nos mesmos grupos).
  **Formato v10** (v8.16.0): 11º grupo = **o trajeto**, para o mapa da tela do
  campo. Os dez anteriores seguem iguais; v5 a v9 continuam abrindo, sem a
  linha.
- **O trajeto do dia na tela do campo** (v8.16.0, pedido do usuário: "um preview
  do mapa para a tela externa, no caso da rota feita para aquele dia — assim o
  técnico teria uma noção do trajeto que fará"). Até aqui a tela do campo era,
  de propósito, "sem mapa": a ordem das paradas, navegação e marcação. Faltava a
  **visão do dia** — se o roteiro sobe para Nova Veneza e volta, ou se fecha um
  laço dentro de Criciúma, a lista não diz. Agora uma **faixa de 160px** entre o
  cabeçalho e a lista mostra a rota desenhada, os números das paradas e a
  origem; **toque amplia para tela cheia**; "Ocultar" troca a faixa por uma
  barra e a escolha fica guardada no aparelho (`hg_campo_mapa`). Os pinos
  **acompanham o dia**: a parada da vez acende, a concluída apaga e fica teal.
  ⚠️ **O Leaflet já estava carregado ali** — é o mesmo arquivo único, e a
  biblioteca entra antes da divisão entre as duas telas; o que o modo campo
  pulava era só criar o mapa do escritório. Então isto **não acrescenta
  download de biblioteca** ao celular: o custo novo são os ladrilhos (12 na
  faixa, 20 na tela cheia, na rota medida).
  ⚠️ **A faixa fica ENTRE o cabeçalho e a lista**, e isso é decisão, não acaso.
  Dentro da lista não pode: ela é reescrita inteira (`innerHTML`) a cada toque,
  e o mapa seria destruído e recriado a cada parada marcada. Dentro do
  cabeçalho também não: ele é `sticky`, e a faixa comeria 160px o dia inteiro.
  Onde está, ela **rola para fora** quando o técnico desce o roteiro. Medido a
  375×812: cabeçalho 116 + faixa 160, e a etiqueta "Agora" com o cartão da
  parada da vez **ainda cabem sem rolar** — que é o que a v7.7.0 conquistou.
  ⚠️ **O mapa da faixa nasce com TODA a interação desligada** (arrastar, pinça,
  rodinha, duplo toque, teclado). Mapa arrastável dentro de lista que rola rouba
  o gesto vertical e trava a tela — é o defeito clássico de mapa embutido em
  celular. Quem quer mexer toca e vai para a tela cheia, onde tudo religa (e
  desliga de novo ao fechar).
  ⚠️ **`zoomSnap: 0`** (zoom fracionário) não é luxo: com os degraus inteiros o
  `fitBounds` escolhe o degrau que **cabe**, e sobra tela vazia — medido na tela
  cheia de 375×812, a rota ocupava 199×166 centrada, com **323px de folga** em
  cima e embaixo. Sem degraus ela encosta na borda (331×276).
  ⚠️ **`maxZoom:16` no enquadramento**: com uma parada só, a caixa tem tamanho
  zero e o Leaflet iria ao zoom máximo — o técnico abriria o link e veria uma
  calçada ampliada, sem referência nenhuma. 16 é também o último zoom com
  imagem de verdade do Esri nesta região.
  ⚠️ **Nenhum `fitBounds` anima**, pelos dois motivos de sempre: o modo leve
  mata transições, e tela que não pinta nunca termina animação.
  ⚠️ **O crédito ao Esri e ao OpenStreetMap é exigência de licença e fica** —
  mas vira etiqueta de canto, porque a faixa clara padrão do Leaflet tomava a
  base inteira de uma caixa de 160px. O selo "Leaflet" sai (`setPrefix(false)`):
  esse é autopromoção da biblioteca, não obrigação.
  ⚠️ **As camadas de ladrilho do mapa do campo são declaradas junto das do
  escritório**, lá no topo, e não na seção do mapa do campo: `aplicarTema()`
  roda na **carga** e lê as quatro, e com o `let` depois dele a zona morta
  derrubaria o app inteiro ao abrir, nas duas telas
- **O trajeto viaja dentro do link** (v8.16.0, formato v10): 11º grupo, a
  geometria da rota em **polyline** (precisão 5), **simplificada** por
  Douglas-Peucker a **30 m**. Os dez grupos anteriores não mudaram de posição
  nem de significado, e link v5–v9 continua abrindo — só mostra o mapa **sem a
  linha**, com os pontos do dia.
  ⚠️ **A geometria crua não cabe.** Medido numa rota real de 10 paradas e
  52,8 km: 1417 pontos, e o link iria de **518 para 4160** caracteres. A 30 m
  sobram **139 pontos** e o link fica em **1186** (o compatível, sem compressão,
  em 1553). A 15 m seriam 1316; a 120 m, 838.
  ⚠️ **30 m foi escolhido pela tela, não pelo gosto**: numa faixa de 340px
  mostrando uma rota de 50 km, um pixel vale ~150 m, então o desvio fica abaixo
  de meio pixel. O ida-e-volta pelo link erra **0,56 m** no pior ponto — só o
  arredondamento das 5 casas.
  ⚠️ **A codificação polyline gera caracteres de 63 a 126, e essa faixa inclui
  `~` e `|`** — justamente os separadores do link. Sem escapar (`escSep`), um
  trajeto com o caractere errado **partiria o link ao meio** e o roteiro
  chegaria corrompido ao celular. Medido nesta rota: 17 ocorrências, 34
  caracteres a mais.
  ⚠️ **O trajeto mora em `ultimoResumo.g`**, e não numa variável nova: o resumo
  já é guardado por técnico e já morre em `invalidarRota()`. Um trajeto que
  sobrevivesse à rota desfeita mandaria para o campo o desenho de um caminho que
  não é mais o do dia.
  ⚠️ O simplificador usa **geometria plana** e tolerância em graus. Em 50 km de
  Criciúma isso erra fração de metro; **não usar para distância**
- **O Roteiro nasce fechado, e "Enviar para o campo" virou parte da Rota**
  (v8.18.0, dois pedidos do usuário na mesma mensagem: "o módulo do roteiro vem
  por padrão minimizado, sendo opcional a abertura" e "pensei na possibilidade
  de mesclar o módulo Rota com o módulo Enviar para o campo... acaba poluindo a
  tela").
  **(1) O Roteiro** ganhou um chevron no título e abre e fecha por ele. Fechado,
  o módulo é **uma barra de 58px** em vez dos 278px da lista (medido no painel,
  numa rota de 6 paradas com 60 instruções). A escolha fica guardada
  (`hg_roteiro_aberto`) e o padrão, sem escolha nenhuma, é **fechado**.
  ⚠️ **Por que fechado faz sentido**: o passo a passo é a parte mais longa do
  painel e a menos consultada no escritório — quem planeja olha a ordem das
  paradas e o km/min. A instrução rua a rua é do Waze, na mão de quem dirige.
  ⚠️ **Fechado, o módulo sai da divisão de altura da coluna**
  (`elasticosVisiveis()` passou a descartar `.recolhido`). Sem isso um Roteiro
  fechado seguraria metade da coluna com uma lista que ninguém vê — medido: com
  ele e o checklist na mesma coluna, fechado o checklist vai a 511px e some a
  divisória; aberto, os dois voltam a 277/277.
  ⚠️ **A margem do título saiu do markup para o CSS**: recolhido ela tem de ir a
  zero, e um `style=` inline venceria a regra.
  **(2) O "Enviar para o campo"** deixou de ser módulo (`m6`) e virou um bloco
  dentro do módulo **Rota**, separado por um fio de 1px. São o mesmo passo do
  dia — traçou, manda — e o módulo próprio custava um título, uma alça e uma
  linha de "aparece depois" em cada coluna. **Junto saiu o campo de retomar
  repetido** (ver o item de retomar, abaixo).
  ⚠️ **O apelido `m6` não foi reaproveitado**, e isso não é preguiça: todo
  arranjo salvo por aí ainda cita `m6`, e `arranjoGuardado()` monta o arranjo a
  partir de `ORDEM_MODULOS` — a chave desconhecida é simplesmente ignorada.
  Testado com um arranjo da v8.17.1 (m3+m4 na coluna 2, m6+m7 na coluna 3): o
  m6 some, o resto fica onde estava e **o disquete não acende sozinho**.
  ⚠️ **`#envioSection` continua sendo o mesmo id, mostrado e escondido por
  `style.display`** — `prepararEnvio()`, `invalidarRota()`, `salvarNoTecnico()` e
  `carregarDoTecnico()` não mudaram uma linha. O que saiu foi a `section`
  em volta e a classe `.modulo-corpo`.
  ⚠️ **Sem segundo título dentro do cartão**: um "ENVIAR PARA O CAMPO" embaixo
  de "ROTA" seria justamente a poluição que o pedido queria tirar. O fio basta,
  e o texto do botão já diz o que ele faz.
  **(3) Embaixo do link ficou só o "?"** (v8.20.0, ponto 6 da revisão de
  layout). O bloco de envio tinha **290px**, e **122px (42%)** eram texto:
  "mande este link para quem vai fazer as visitas…", a saída "não abriu no
  celular de alguém?" e a opção "sempre gerar link compatível". As três foram
  para um painel atrás do **"?"**, na mesma caixa dos painéis de tipos, avulsa e
  retomar. Medido: bloco de envio **290px → 169px** e o módulo Rota inteiro
  **510px → 390px**.
  ⚠️ **Duas são instrução de PRIMEIRA VEZ e a terceira é CONFIGURAÇÃO.** Quem
  manda roteiro todo dia já sabe o que o link faz; e configuração de aparelho da
  equipe não pertence ao lado do botão que se aperta todo dia.
  ⚠️ **NENHUMA frase sobrou embaixo do link**, e isso foi correção do usuário
  dentro da própria versão: a primeira forma deixava ali uma linha de estado
  ("Mande para quem vai fazer as visitas."), e ele cortou na hora — *"remova a
  escrita… deixe somente o '?' e a janela que ele abre"*. Estava certo: o botão
  logo acima já diz "Gerar link do roteiro", e Compartilhar/Copiar dizem o
  resto. Era o mesmo vício que a revisão estava consertando, repetido por mim
  no conserto dele.
  ⚠️ **Mas o ESTADO do formato não podia sumir junto, e foi para o ícone.**
  Esconder a opção "sempre gerar link compatível" **ligada** sem dizer isso em
  lugar nenhum seria pior do que o problema que a versão resolve: a pessoa
  geraria links longos sem saber por quê. Então o próprio **"?" acende em
  âmbar** quando o link na tela é o compatível, com a explicação na dica do
  mouse — o mesmo idioma de todo estado de ícone no app (a lupa do zoom, o ímã,
  o vidro, o foco do mapa).
  ⚠️ **O aceso lê o LINK QUE ESTÁ NA TELA, e não a preferência guardada.** Foi
  defeito meu, pego no teste desta própria versão: com a preferência desligada e
  a saída avulsa "gerar link compatível" usada uma vez, o aviso ficaria errado.
  O prefixo manda — `#r=` é o compatível, `#z=` o comprimido. De brinde, isto
  acerta também o navegador sem compressão nenhuma, em que o link sai compatível
  sem ninguém ter pedido. Por isso `atualizarResumoDoEnvio()` é chamada em
  `gerarLinkDoRoteiro()` **e** em `carregarDoTecnico()` (trocar de aba restaura
  o link de outro técnico, que pode ser de outro formato).
  ⚠️ **O aceso NÃO é "painel aberto"**: abrir o painel já se vê sozinho, logo
  abaixo, e gastar a mesma cor nas duas coisas apagaria o recado que importa.
  ⚠️ **O risco assumido é de DESCOBERTA**: quem precisar do "gerar link
  compatível" num aparelho antigo tem de achar o "?". Aceitável porque a falha
  **nunca é descoberta aqui** — ela chega horas depois, pela pessoa que está na
  rua, e a essa altura o link já saiu da tela de qualquer forma. O que resolve
  de verdade é a opção guardada, que passa a valer para todo roteiro seguinte.
  ⚠️ O painel abre e fecha **só na sessão**, como os de tipos, avulsa e retomar:
  é ajuda, e ajuda não deve reabrir sozinha todo dia. Medido no modo leve: 0ms,
  zero animações.

- **Retomar um roteiro pelo link** (v6.8.0, pedido do usuário: "meia hora depois
  surge mais uma parada"): colar o link no **módulo 1** (v6.8.1, sempre à mão;
  desde a v7.4.0 fica num **painel dobrável**, atrás do ícone de corrente do
  título, que abre com o foco no campo e fecha sozinho quando a retomada dá
  certo). Volta ordem, ★, tipo, origem e o interruptor de retorno;
  ⚠️ **Havia um segundo campo, igual, no "Enviar para o campo", e ele saiu na
  v8.18.0** (o usuário apontou: "posso colar o link de um roteiro para editá-lo,
  mas também é possível fazer isso apertando no botão do módulo Cliente — acaba
  poluindo a tela"). Ficou o do módulo 1, que está sempre à mão: o outro só
  existia **depois de uma rota traçada**, que é exatamente quando não se precisa
  retomar nada;
  ⚠️ Havia um terceiro caminho até a v7.9.2: o atalho **"editar no escritório"**,
  no rodapé da tela do roteiro, que guardava o link no `sessionStorage`
  (`hg_retomar_roteiro`) e recarregava sem o `#`. Saiu junto com o rodapé, a
  pedido do usuário — junto com a chave e com o bloco que a lia na abertura do
  escritório. A funcionalidade não se perdeu: mudou o caminho, que agora é colar
  o link. o **id do
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
--barra:#334955       --barra-forte:#4B6776
```

As **barras de rolagem** saem dessas duas últimas desde a v8.4.1 (relatado
pelo usuário: "o scroll com fundo branco não está na mesma sintonia do
restante"). Nunca tinham estilo: o navegador desenhava a dele, clara, e dentro
de um cartão escuro virava um risco branco atravessando a coluna. A **pista é
transparente**, senão viraria uma faixa dentro do vidro. Estão declaradas nas
duas formas — `scrollbar-color`/`scrollbar-width` (padronizada) e
`::-webkit-scrollbar` (Chrome/Edge antigo); onde a primeira vale, o navegador
ignora a segunda. ⚠️ `scrollbar-color` **é herdada**, `scrollbar-width` **não**:
declarando as duas só em `:root`, o checklist, a lista de paradas e o roteiro
saíam com a cor certa e a barra larga do sistema.

Tema claro em `:root[data-tema="claro"]`. **Não é o escuro invertido**: o âmbar
e o teal precisaram escurecer (`#C9821A`, `#12796E`) para continuarem legíveis
sobre branco — os tons do tema escuro sobre fundo claro ficam lavados. Já os
textos que ficam *sobre* preenchimento âmbar/teal (`#241705`, `#062420`) valem
nos dois temas, porque o preenchimento continua sendo a cor forte.

Âmbar = origem, rota traçada e paradas numeradas. Teal = clientes não
selecionados e estados de sucesso.

**A monoespaçada é dos NÚMEROS** (v8.21.0, ponto 3 da revisão de layout de
26/09/2026). A regra passou a ser uma frase: **número, código e endereço em
JetBrains Mono; palavra em Inter.** Medido antes: **36 elementos em mono contra
33 em Inter** numa tela só — a mono tinha virado a fonte principal do app, sem
ninguém ter decidido isso. Depois: **23 em mono**, e todos os 23 são o selo de
versão, os números dos marcadores, os contadores do checklist, os números das
paradas e o km/min.
⚠️ **O que a mono resolve é alinhamento de dígito**, e por isso ela fica onde
há dígito. O que ela não faz bem é *palavra*: todas as letras ocupam a mesma
largura, e o desenho da palavra — que é o que se reconhece ao bater o olho —
desaparece. Os títulos de módulo ainda somavam as **quatro** coisas que mais
atrasam a leitura, todas juntas: mono + CAIXA ALTA + entreletra de 1,5px +
11px, no elemento **mais repetido do painel**.
⚠️ **De brinde, todo rótulo encolheu** (medido no mesmo texto, ajuste antigo ×
novo): "Selecionar paradas" **146 → 112px**, "Ordem da viagem" 122 → 102,
"Salvar como padrão" 146 → 116, "Quilometragem aproximada" 194 → 161 (a pílula
do cabeçalho foi de 249 para 241px) e "Prioritárias · nesta ordem" **211 →
148px**. Num painel de 340px isso é espaço de verdade.
⚠️ **Dois textos precisaram de maiúscula inicial no código**, porque até aqui
era o `text-transform` do CSS que os punha em caixa alta e sem ele ficariam
todos minúsculos: `'Salvar como padrão'`/`'Esquecer padrão'` e
`'Quilometragem aproximada'` (mais o `'Nova coluna'` do alvo de arraste). O
**"gerar link compatível" ficou minúsculo de propósito** — ele vive no meio de
uma frase.
⚠️ **A COR não mudou junto.** O âmbar dos títulos é o **ponto 2** da revisão,
que é escolha de identidade e espera decisão à parte. ⚠️ Ficaram **fora**: a
etiqueta de tipo de serviço (ali a caixa alta normaliza um texto que o próprio
usuário digita, e a gêmea dela mora na tela do campo) e a **tela do campo
inteira**, que teve revisão própria em 17–19/09 e não foi medida nesta.

**Os botões de ícone falam UMA língua só** (v8.23.1, relatado pelo usuário
depois de um dia de uso: *"os botões do módulo selecionar paradas está com a
coloração diferente dos botões do módulo origem"*). Ele viu a cor, mas eram
**três** diferenças: cor do ícone (`--text` no Origem contra `--muted` no
Selecionar paradas), **largura** (43px contra 38) e o desenho (17px/traço 1,8
contra 18px/1,7). Agora `.btn-icon` e `.btn-lista` têm os mesmos valores:
**34×36px, ícone de 16px com traço 1,7, `--text` em repouso**, borda teal com
fundo `--hover` ao passar o mouse, e âmbar quando ligados.
⚠️ **A v8.23.1 unificou para BAIXO — os dois apagados — e o usuário corrigiu na
v8.23.2**: *"se ficar com os traços do ícone dentro do botão na cor branca vai
ficar bom"*. Ele tem razão: apagados, os ícones ficavam discretos demais para
controles que se usam todo dia. O argumento de que **o campo de texto é o
carro-chefe continua de pé** — só que quem passou a carregá-lo é o **tamanho**
(34px, contra os 43 e 38 de antes), e não a cor.
⚠️ **Por isso o `:hover` trocou de portador.** Com o ícone já claro em repouso,
clarear mais não diz nada; o hover passou a ser a **borda teal + o fundo
`--hover`**, o mesmo idioma das linhas do checklist desde a v8.22.0.
⚠️ **Medido**: os três botões comiam 138px dos 328px da linha de busca. Depois,
o campo de busca foi de **190 para 202px** (58% → 62% da linha) e o de origem
de **206 para 224px**.
⚠️ **O ícone acompanha o tema**, porque é `--text` e não uma cor fixa: claro no
tema escuro (`#E7EEF2`) e escuro no tema claro (`#152229`). Conferido nos dois.
⚠️ **ARMADILHA NO TESTE, de novo a nº 1**: medindo logo depois de trocar o tema,
`getComputedStyle` devolveu a cor do tema ANTERIOR — o painel não estava
pintando e a transição de `color` ficou parada no meio. Um clone sem transição
em curso, e `getAnimations().forEach(a => a.finish())`, deram o valor certo.
⚠️ **No toque eles voltam a 40px** (`@media (pointer:coarse)`), como a alça da
parada já fazia desde a v5.2: 34px é confortável com mouse e pequeno demais com
o dedo.
⚠️ **O comportamento do 📍 NÃO foi tocado** — só cor e tamanho. A regra de não
mexer naquele botão sem testar na máquina real do usuário vale para a
**geolocalização**, que continua exatamente como estava desde a v3.7.

**A escala do escritório tem TRÊS degraus de texto** (v8.23.0, ponto 1 da
revisão de layout, recalculado depois das quatro versões do dia):

| degrau | para quê |
|---|---|
| **10,5px** | dado miúdo: contador do checklist, cliente sob o nome da parada, número do marcador, etiqueta de tipo |
| **11,5px** | rótulo: títulos de módulo, links do título, dicas, legendas, URL do link, selo de versão |
| **13px** | conteúdo: tudo que se lê e se clica — nomes, botões, campos, status, passo a passo |

Mais **20px** para o número do resultado (km/min) e **19px** para a marca, que
é logotipo. Com tudo aberto no escritório — Roteiro, os três painéis dobráveis,
a ajuda do envio, dois técnicos e o bloco de prioritárias — **nenhum elemento
fica fora desses cinco valores** (medido: 18 em 10,5 · 23 em 11,5 · 125 em 13).
⚠️ **Antes eram oito degraus de texto num vão de 3,5px** (10 · 10,5 · 11 · 11,5
· 12 · 12,5 · 13 · 13,5), com três saltos de **4%** — invisíveis. Agora os
saltos são de **9,5% e 13%**, e **nenhum par de meio pixel sobrou**. Meio pixel
dá altura de linha **idêntica** e 0,3px por caractere: custava um valor a mais
para manter coerente e não entregava hierarquia nenhuma.
⚠️ **A inversão que a revisão apontou morreu aqui**: a legenda "distância
total" era **13px**, maior que o título do módulo acima dela. Ela é rótulo, e
foi para 11,5.
⚠️ **O risco que eu previ na revisão não se confirmou.** Eu tinha escrito que
este era "o mais arriscado na prática: altura de letra mexe em altura de
módulo, que mexe nos pesos guardados". Medido antes de aplicar, injetando a
escala no app rodando: **checklist 0, linha do cliente 0, cabeçalho 0, módulo
Origem 0**; a lista de paradas ganhou 3,2px e o módulo Rota perdeu 3,7px. O
motivo é o próprio trabalho do dia — os degraus que sobraram estavam quase
vazios, então quase não havia o que mover. A promessa da v8.19.0 continua de
pé: a 1920 nada rola e o mapa fica com 1189px.
⚠️ **Dois achados de faxina, no caminho**: o `.btn-icon` carregava
`font-size:15px` desde antes da v7.9.0, quando o conteúdo dele era **emoji** —
hoje é um `<svg>` com tamanho próprio, e aquele corpo não pintava nada. E o
`.stop-row .nm-tipo` estava em **9,5px**, o menor corpo do app, num rótulo.
⚠️ **A tela do campo continua fora**, com escala própria e maior (o nome da
parada tem 17px): ela é lida em celular e no sol. Conferido depois: selo 11px,
técnico 11px, contagem 12,5px, nome 17px — nada mudou lá.

**O ícone da aba é uma rota, sem letra** (v8.24.1, pedido do usuário: "uma logo
para colocar na Aba, hoje está um globinho. Crie algo como HF"). Um
`<link rel="icon">` com **SVG em data-URI** (399 caracteres) logo abaixo do
`<title>`: curva âmbar com uma bolinha pequena na origem e uma grande no destino,
sobre o crachá `#0C1418` — **as mesmas cores e o mesmo sentido do mapa** (âmbar =
origem, rota e paradas). Nenhum arquivo novo; vale nas duas telas, que dividem o
`<head>`. ⚠️ **É a primeira peça da marca que mostra o que o app FAZ, e não como
ele se chama.** O pedido andou HF → HR → estudo de 14 propostas em 4 famílias
(`COMPARACAO-ICONE.html`), e o usuário escolheu o `rota` ("da origem ao destino")
vendo o 16px real.
⚠️ **O que o estudo de 16×16 (256 pixels) descobriu, para ninguém repropor:**
sigla de **duas letras não cabe** (o buraco do R fecha em 1,5px); o **R de asfalto
com faixa central não cabe** (a fita pede ≥4px e a faixa ≥1,5px, e sobram <2px
para o buraco); **asfalto em perspectiva lê como a letra A**; a **rota em S lia
como "?"** até ganhar uma parada cheia em cada ponta. O que funcionou entre as
letras foi o `r-via` (núcleo âmbar com borda escura sobre crachá cor de asfalto) —
ponto de partida se a marca voltar a querer letra. O `rota` é o mais vago dos
finalistas e **ele aceitou isso de olhos abertos**: não "consertar" sem pedido.
Variantes prontas, se pedir: `rota-b` (traço 3,0 e pontos 3,4/4,6) e `rota-f`
(crachá âmbar, a de silhueta mais forte) — as linhas estão no
`PLANO-BUSCA-E-ICONE.md`.
**O mesmo ícone também está ao lado do nome da marca** (v8.24.2, pedido do usuário
depois de ver o da aba no ar: "gostei do ícone"), nas duas telas: um `<symbol
id="logoMarca">` no topo do `<body>`, usado por `<use>`. ⚠️ **O desenho mora em
dois lugares** (o data-URI do `<link>` e o `<symbol>`): mudar um exige mudar o
outro. ⚠️ O crachá é `#0C1418`, **igual ao fundo do tema escuro**: o fio de 1px em
`--line` é o que o mantém à vista; sem ele sobraria só a curva âmbar.
⚠️ **Armadilhas do data-URI**: o `#` das cores vai como `%23` (senão cortaria o SVG
no meio **sem erro no console**); atributos do SVG com aspas **simples** (a linha
mora num atributo HTML de aspas duplas); `<text>` não serve, porque favicon não
tem webfont. ⚠️ O ícone **não acompanha o tema do app** (a aba tem o tema do
navegador); numa aba escura o crachá quase some e sobra o percurso âmbar — é o
desenho. ⚠️ O navegador guarda favicon em cache próprio: após publicar, o globo
pode continuar até um Ctrl+Shift+R. ⚠️ O iOS ignora SVG/data-URI no
`apple-touch-icon` ("adicionar à tela de início" não pega este ícone) — aceito; não
acrescentar PNG sem o usuário decidir, é arquivo versionado novo.

O escritório **não tem rodapé** (removido em 10/09/2026, a pedido do usuário).
O crédito obrigatório ao OpenStreetMap e à Esri fica no canto do próprio mapa,
no controle de atribuição do Leaflet (opção attribution da camada base) — é
exigência das licenças dos mapas, não remover. **A tela do campo também não tem
rodapé** desde a v7.9.2, a pedido do usuário: saíram a frase "Roteiro recebido
por link · nada é enviado para servidor" — explicação de como o app funciona por
dentro, que quem está na rua não precisa — e o atalho "editar no escritório". A
lista de paradas vai até o fim da tela.

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
      "_umap_options": { "name": "LABORATORIO EXEMPLO" },
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
  "properties": { "name": "PMEX", "group": true },
  "layers": [ { "type": "FeatureCollection", "features": [...],
                "properties": { "name": "PREFEITURA EXEMPLO - EDUCACAO" } } ] }
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
  a opção de voltar para a origem, o zoom ao clicar na parada, **o que o mapa
  mostra dos clientes** (v8.15.0), **o Roteiro aberto ou fechado** (v8.18.0),
  o vidro ligado
  ou desligado, as janelas livres ligadas ou desligadas, o ímã de alinhamento
  delas, **a faixa de mapa da tela do campo** (v8.16.0, no aparelho de quem
  abriu o link), **o modo leve** (v8.7.0), **o planejamento do dia** (v8.5.0, restaurado
  só se a pessoa aceitar), a largura do painel e das colunas, o arranjo dos módulos (só quando salvo pelo
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
- **MÉTRICAS E RELATÓRIOS — a próxima fase, planejada em 28/09/2026.** O plano
  inteiro está em **`PLANO-METRICAS.md`** (versionado): as três fases, o que já
  foi medido, a escolha do backend (**Cloudflare Workers + D1**) e as decisões
  que travam cada etapa. ⚠️ **Ler aquele arquivo antes de começar qualquer coisa
  desta frente** — ele guarda números que não precisam ser remedidos e duas
  armadilhas conhecidas. A **Fase B (o servidor)** está detalhada passo a passo
  em **`PLANO-SERVIDOR.md`**, pronta para implementar. ⚠️ A Fase B **revisa o ADR-01** (dado de cliente passa
  a sair da máquina) e derruba a **decisão 5** da seção 4 (arquivo único). Ponto
  de retorno: tag `v8.23.2` no commit `e789131` e
  `backups/MARCO_v8.23.2_2026-09-28_antes-das-metricas/`
- Troca do OSRM público antes do uso diário sério (Fase 4)
- Revisar a decisão de manter arquivo único (Fase 3, ponto de decisão, não iniciado)

### Pendências para a próxima sessão (atualizadas em 19/09/2026)

**Revisão de design (17–19/09/2026) — encerrada.** O usuário pediu que eu olhasse o app como
designer e comparasse versões: "me sugira e compare as versões". Saíram **7
pontos**, montados lado a lado (hoje × proposta, no CSS e nas cores reais) na
página `COMPARACAO-DESIGN.html`, que fica fora do repositório.

Os sete, na ordem em que foram propostos:
1. **Barra de progresso do campo invisível** → v7.5.0 (era defeito, não gosto;
   ver seção 5).
2. **Parada da vez em destaque no campo** → v7.7.0.
3. **Botões da parada** → v7.8.0. O usuário não quis nenhuma das duas variantes
   oferecidas: pediu navegação **sem preenchimento, com contorno âmbar**, e com
   isso o de concluir virou o único cheio.
4. **Módulo Rota** → v7.9.0: "Otimizar e traçar" como principal, "Nesta ordem"
   como alternativa.
5. **Um só idioma de ícones** → v7.9.0, junto com o 4.
6. **Numeração dos módulos** → v7.5.1, variante "tirar os números".
7. **Cabeçalho** → v7.6.0, ações agrupadas e tema como ícone.

**Nada ficou em aberto nessa lista.**

**Sugestões de layout (14–15/09/2026), lista à parte e já encerrada.** Das 7, a
única que sobrou foi a **4 · rolagens dentro de rolagem**, ⚠️ **TENTADA E
REVERTIDA**: a v6.4 fez o painel funcionar como as colunas; o usuário testou e
não gostou (com rota traçada os três módulos com lista dividiam a tela, ~131px
cada numa janela de 1000px, contra 320/240 fixos). Desfeita na v6.4.1. Se o
assunto voltar, o caminho é o outro que foi oferecido: mexer só no Roteiro, que
deixaria de rolar sozinho, de 4 barras para 3.

**Conflito entre a ordem do app e o caminho do Waze (medido em 19/09/2026 — o
usuário decidiu NÃO mexer por enquanto).** Ele relatou: traçou a rota, e o Waze
levou por outro caminho, que passava perto de um cliente que também estava no
roteiro — "gerando desconfiança". Mandou o link do roteiro real (10 paradas +
retorno em Criciúma/Içara, 29,9 km / 44 min) e eu medi, no próprio motor do app:

1. **A ordem do app não é ótima, mas erra pouco.** Rodando o cálculo **exato**
   (Held-Karp sobre a matriz de tempos do OSRM), a melhor ordem seria
   `1,10,9,5,6,7,8,4,3,2` — o mesmo laço ao contrário — com **42 min / 29,0 km**.
   O app entregou 44 min / 29,9 km: **~5% pior**, porque a Trip API resolve o
   caixeiro-viajante por aproximação.
2. **O "passei na porta de outro cliente" é geografia, não erro de ordem.**
   Medindo a distância de cada parada ainda não visitada até o traçado: o trecho
   2→3 passa a **109 m** da parada 4; o 4→5 a **90 m** da 6; e o 5→6 a **13 m**
   da 7. Mas os clientes de Içara estão a **179 m (3↔4)**, **235 m (5↔6)** e
   **251 m (6↔7)** uns dos outros — no mesmo quarteirão. Rodando a mesma análise
   na ordem **ótima**, os casos **aumentam** (4, um deles a 8 m). Ou seja:
   **otimizar melhor não resolve a desconfiança**.
3. **Waze × OSRM vão discordar sempre**: o Waze escolhe o *caminho* com trânsito
   ao vivo; o app escolhe a *ordem* com tempo livre de trânsito.

Opções levantadas, na ordem que eu recomendaria: **(a)** agrupar paradas a menos
de ~300 m num "bloco", mostrando no cartão do campo "mais N paradas aqui perto" —
é o que ataca a desconfiança; **(b)** aviso de "passa perto" no escritório (o
cálculo de ponto-até-traçado já foi escrito e validado nesta análise);
**(c)** otimização exata até ~12 paradas (uma chamada a mais, ao `/table` do
OSRM) e 2-opt acima disso; **(d)** km/min por trecho na tela do campo;
**(e)** abrir a viagem inteira no Google Maps, que aceita vários pontos numa URL
(o Waze só aceita um destino). Nada disso foi implementado.

**Revisão completa do código (24/09/2026, a pedido do usuário).** Feita com
análise mecânica do arquivo (funções declaradas x chamadas, ids x uso, classes
x uso, seletores repetidos, variáveis de topo, temporizadores e observadores) e
leitura dirigida aos pontos suspeitos. O mapa que saiu junto está em
`MAPA-DO-CODIGO.md`, versionado.

**O que está são** (verificado, não suposto): nenhuma função declarada sem ser
chamada; nenhuma variável de topo sem leitura; nenhum `addEventListener`
acumulando (os que estão dentro de `render*` são sempre em elementos recém
-criados); o `setInterval` do arraste e o laço de `requestAnimationFrame` da
rolagem automática têm parada garantida; **nenhuma dependência de
`transitionend`/`animationend`** no código do app — que é o que faz o modo leve
ser seguro.

**1. BUG — ★ e tipo de serviço eram do PONTO, não do técnico.**
✅ **Corrigido na v8.9.0.**
Com o mesmo cliente na rota de dois técnicos (caso que o app permite e só
avisa), os dois `stops` guardam **o mesmo objeto**: marcar ★ ou trocar o tipo
num técnico muda no outro, sem aviso. Reproduzido: técnico 1 com ★ e
"Manutenção"; o técnico 2 entrou já com ★ e "Manutenção" herdados, tirou a ★ e
pôs "Entrega de toner" — e o técnico 1 ficou com "Entrega de toner" e sem ★.
⚠️ É justamente o caso que a v8.5.0 usa para justificar permitir o repetido
("entrega de manhã, manutenção à tarde") — e é o único em que não funciona.
Conserto: `stops` passar a guardar uma **cópia rasa** do ponto, e trocar as
**quatro** comparações por identidade que existem (`stops.includes(ponto)` em
`toggleStop`, `!stops.includes(c)` no marcar todos, as duas de
`limparAvulsasSoltas` e o `stops.indexOf(ponto)` da avulsa) por comparação de
`id`. Contido, mas pede teste cuidadoso do realce mapa↔lista.

**2. Peso morto (pequeno, sem efeito em desempenho).** ✅ **Limpo na v8.9.0**,
menos os ids `versaoEscritorio`/`versaoCampo`, que ficaram de propósito: são a
única forma de dizer qual dos dois selos se está olhando ao inspecionar.
O texto abaixo fica como registro do que foi achado. A classe `.parada-obs`
está no CSS e nunca é aplicada (sobra do campo de observação adiado); seis ids
no HTML não são usados por ninguém (`arranjoAcoes`, `envioDica`, `modulo3`,
`sempreCompatLinha`, `versaoCampo`, `versaoEscritorio` — os dois últimos são
alcançados pela classe `.versao`); e duas linhas apagam chaves de
`localStorage` aposentadas na v4.4/v4.9, que já não existem em máquina nenhuma
há meses.

**3. Ponto a vigiar.** ✅ **Resolvido na v8.12.0**, por tabela: o módulo Rota
deixou de ser sticky a pedido do usuário, e com ele saiu a escuta de `scroll`
em `main` na fase de captura — que disparava duas leituras de caixa a cada
rolagem de qualquer lista. Não há mais nada escutando rolagem no app.
⚠️ Continua **sem medição**: aqui o problema não reproduz. O que dá para
afirmar é que o código saiu.

**4. Comentários são 36% do arquivo** (130 KB de 365 KB). Fica registrado como
fato, não como problema: o GitHub Pages serve comprimido e o navegador descarta
comentário no parse. Eles são a memória do projeto.

**Também em aberto, de antes:**
- **Trocar o OSRM público** (Fase 4) — o usuário ficou de decidir onde hospedar.
  ⚠️ A análise acima reforça: um motor com trânsito aproximaria o plano do que o
  Waze faz, e o `/table` usado no item (c) também depende desse servidor.
- **Arquivo único** (Fase 3, ponto de decisão, não iniciado).
- **Nome do programa.** Em 17/09 ele pediu sugestões; foram dadas (Haga Rotas,
  Parada Certa, Percurso, Rota Viva, Farol, entre outras) e **nenhuma foi
  escolhida** — a marca segue `hagamorfis/rotas`. ⚠️ Se um dia trocar, mudar só o
  nome que aparece: **renomear o repositório muda o endereço do GitHub Pages e
  quebra todos os links de roteiro já enviados à equipe**.

**Varredura de bugs (10/10/2026), a pedido do usuário: "elabore uma varredura
completa no site, procurando por bugs".** Análise mecânica do `index.html`,
leitura do script inteiro e — o que separa achado de palpite — **cada suspeito
reproduzido no app rodando**, com a rede de verdade e, nos que dependem de
tempo, a rede atrasada de propósito. Tela do campo por link de verdade, tela
estreita, e tudo de novo com o modo leve. A cópia publicada era byte a byte a
do `HEAD`. O detalhe e os números estão no `LOG-ALTERACOES.txt`.

| # | O que foi achado | Situação |
|---|---|---|
| 1 | **A resposta do OSRM era aplicada ao que estivesse na tela quando chegasse**, e não ao que foi pedido. Marcar um cliente durante "Rota otimizada" fazia a parada nova **sumir**; trocar de aba durante o cálculo **trocava as paradas do outro técnico** pelas do primeiro | ✅ v8.26.0 — `retratoDoPedido()` / `pedidoMudou()` |
| 2 | **Mudar a origem não desfazia a rota**: o link saía com a origem nova e o trajeto/km velhos (medido: 12 km entre o pino e o começo do desenho) | ✅ v8.26.0 — `setOrigin()` chama `invalidarRota()` |
| 3 | **Digitar no campo de origem** deixava a rota sem origem: link sem origem e sem retorno, e "Salvar como padrão" respondendo com um erro que não era o dele | ✅ v8.26.0 |
| 4 | **A linha da rota sumia pela metade ao aproximar o mapa**, com o efeito de traçar ligado: o `stroke-dasharray` do desenho (v8.0.0) ficava no path para sempre | ✅ v8.26.0 — sai por relógio |
| 5 | **O endereço da avulsa não chegava à "Ordem da viagem"** nem ao link (regressão da v8.9.0: `stops` guarda cópia) | ✅ v8.26.0 — `renomearAvulsa()` |
| 6 | **Restaurar o dia** só devolvia o pino de origem da aba aberta | ✅ v8.26.0 |
| 7 | **Aba nova de técnico** ignorava a preferência de retorno guardada | ✅ v8.26.0 — `voltarGuardado()` |
| 8 | O "atrás" dos marcadores da aba apagada era desfeito pelo Leaflet a cada zoom (cosmético) | ✅ v8.26.0 — `setZIndexOffset` |
| 9 | **Dois clientes na mesma coordenada** voltavam como um só ao restaurar o dia ou retomar um link. ⚠️ Latente: nenhuma base local tem coordenada repetida | ✅ v8.26.0 — `indicePorCoordenada()` / `acharNaBase()` |
| A | **Trocar de base com várias abas zera só a aba aberta**; as outras ficam com paradas que não existem na base nova. Duas saídas (zerar todas, ou reencontrar pela coordenada) | **em aberto — decisão do usuário** |
| B | Na **tela do campo** o progresso é por coordenada: dois clientes no mesmo prédio seriam marcados juntos. Latente, como o 9. ⚠️ Mudar a chave quebraria o progresso de quem já está na rua | em aberto |
| C | "Marcar todos" com a busca ativa marca a base inteira, não o que a busca mostra | em aberto |
| D | Números com ponto ("61.2 km") numa interface em português | em aberto |

⚠️ **As lições, para a próxima mudança:**
- **Depois de um `await`, a tela não é mais a de antes.** Todo trecho que espera
  a rede e depois escreve em `stops`, na origem ou no mapa precisa conferir se o
  pedido ainda vale. É a mesma família do relógio parado, só que no outro
  sentido: lá a tela não anda, aqui ela anda sem avisar.
- **`stops` é cópia desde a v8.9.0** — quem muda um dado do ponto da base (nome)
  tem de mudar nas cópias de **todas** as abas. O defeito 5 passou três semanas
  sem ser visto porque o checklist, que lê a base, mostrava o nome certo.
- **Estilo inline deixado num elemento do Leaflet sobrevive ao zoom; a geometria
  dele não.** Valor em pixel escrito num path (`stroke-dasharray`) envelhece no
  primeiro zoom — tem de sair quando o efeito acaba.
- **O mapa do código dizia que origem invalidava a rota, e não invalidava.**
  Afirmação de documento se confere rodando, não lendo.

⚠️ **A pasta `TUTORIAL/`** está na raiz sem versionar **e sem estar no
`.gitignore`**: entra no próximo `git add -A` de um repositório PÚBLICO. Antes de
decidir, varrer por nome de cliente real (as imagens são capturas da tela).

**Revisão de usabilidade (10/10/2026), a pedido do usuário: "verificar se está
acessível para pessoas com menos conhecimento de tecnologia".** Feita percorrendo
o app como alguém que nunca o viu (base de teste, paradas, origem, rota, link) e
inventariando todo botão visível — rótulo, dica, tamanho. **O caminho principal
é fácil** (toda ação principal tem palavra: Rota otimizada, Nesta ordem, Gerar
link, Compartilhar, Copiar) e as mensagens de status dizem o próximo passo em
frase inteira; o problema estava em volta dele.

| # | O que foi achado | Situação |
|---|---|---|
| 1 | Cabeçalho com 7 ícones sem nome, 3 deles dizendo outra coisa | ✅ **feito na v8.25.0** |
| 2 | Mira e alfinete da Origem trocados em relação ao que o celular ensinou | ✅ **feito na v8.25.0** (só o desenho) |
| 3 | Os botões de traçar vêm **antes** de escolher as paradas (ordem da v8.13.0): quem lê de cima para baixo aperta "Rota otimizada" primeiro e recebe "Selecione ao menos um cliente". Proposta: o módulo Rota dizer "Escolha as paradas abaixo e depois trace a rota" enquanto não há parada — **sem** desfazer a v8.13.0 | em aberto |
| 4 | **14 botões de 22–24×18px** (★, ✕ de remover, ícones dos títulos). O de esquecer a base fica colado no de trocar a base | em aberto |
| 5 | "Retomar roteiro pelo link" atrás de um ícone de corrente, que ninguém adivinha; "base", "backup (.umap)" e "parada avulsa" pedem explicação para um cliente novo | em aberto |
| 6 | Dicas do zoom do mapa em inglês ("Zoom in"/"Zoom out") | em aberto |
| 7 | Pelo teclado não se chega às linhas de cliente do checklist, ao nome da parada nem às abas de técnico (`tabindex` −1) | em aberto |

⚠️ Os itens 3, 4 e 6 são pequenos e não mexem em identidade. **A tela do campo
não foi reaberta nesta revisão.**

**Revisão de layout (26/09/2026), a pedido do usuário: "elabore um review com
um olhar de um designer sênior".** Seis pontos, todos medidos no app rodando
(`getComputedStyle`/`getBoundingClientRect`) a 1920×1080 e 1366×768, montados
lado a lado em `COMPARACAO-LAYOUT.html` (fora do repositório). O que a revisão
mostra não é defeito de funcionamento: é o efeito **somado** de decisões que
tinham razão uma a uma — cada versão acrescentou um tamanho de letra, um uso do
âmbar, uma caixa e uma explicação, e ninguém olhava o total.

| # | O que foi medido | Situação |
|---|---|---|
| 1 | **11 tamanhos de letra** num vão de 3,5px (10 · 10,5 · 11 · 11,5 · 12 · 12,5 · 13 · 13,5), e nada entre 13,5 e 19. Meio pixel dá **altura de linha idêntica** e 0,3px por caractere. E "distância total" (13px) é maior que o título do módulo (11px) | ✅ **feito na v8.23.0** — três degraus de texto (10,5 · 11,5 · 13). ⚠️ O risco que eu previ NÃO se confirmou: medido, o maior deslocamento foi de 3,7px |
| 2 | **22 elementos de texto em âmbar** contra 4 em teal, fazendo cinco trabalhos (título, contador, número, resultado, ação, link). Contraste do título 4,66:1; o do contador **4,36:1, abaixo do mínimo da AA**, no menor corpo do app | em aberto — barato em código, mas é escolha de identidade |
| 3 | **36 elementos em JetBrains Mono** contra 33 em Inter. Os 6 títulos acumulam mono + CAIXA ALTA + entreletra 1,5px + 11px, tudo junto, no elemento mais repetido do painel | ✅ **feito na v8.21.0** — 36 elementos em mono viram 23, e todos são número ou código |
| 4 | **34 caixas com borda** numa coluna de 340px, aninhadas até 4 molduras em volta de um nome | ✅ **feito na v8.22.0** — o nome da unidade sai de dentro de 2 molduras, e a lista encolhe 14%. ⚠️ A conta "4 molduras" da revisão somava o cartão de vidro e o módulo, que não têm borda de verdade; medindo só as bordas do DOM eram **2** |
| 5 | **31% do painel fora da vista a 1920×1080** (1442px de conteúdo em 991px), com 17,7% da largura em painel e 1568px de mapa parado; **53% fora da vista na TV de 1366** | ✅ **feito na v8.19.0** (só a parte de 1920; a TV continua em aberto) |
| 6 | **Módulo Rota com 510px**, dos quais **122px (42% do bloco de envio)** são três explicações lidas uma vez na vida | ✅ **feito na v8.20.0** — envio de 290 para 169px, módulo Rota de 510 para 390px |

**Sobra só o ponto 2**, que o usuário pediu para deixar de lado em 27/09/2026,
sem descartar. Os outros cinco estão publicados.

**Material de trabalho na pasta, fora do repositório** (padrão `COMPARACAO-*.html`
no `.gitignore`), para apagar quando não servirem mais:
`COMPARACAO-MODULO1.html` (as 5 propostas do módulo 1; a v7.4.0 saiu da "D"),
`COMPARACAO-DESIGN.html` (os 7 pontos acima), `COMPARACAO-EFEITOS.html` (os
quatro efeitos ao traçar a rota, em seis mapas de verdade; a v8.0.0 saiu do
"conjunto") e `COMPARACAO-VIDRO.html` (quatro modos de transparência com cinco
controles; a v8.1.0 saiu do modo "Apple") e `COMPARACAO-CHECKLIST.html` (a
bolinha da calha: como era, contador acendendo e seta acendendo, na largura
real do painel; a v8.4.2 saiu das duas últimas combinadas) e
`COMPARACAO-TROCA-ABA.html` (banco de provas das passagens ao trocar de
técnico: um painel só, um efeito de cada vez, em câmera lenta; a v8.14.0 saiu
do "saída + cascata") e `COMPARACAO-LAYOUT.html` (os 6 pontos da revisão de
layout acima, hoje × proposta na largura real do painel, com controle de
ampliação — várias diferenças são de meio pixel e não se julgam a 1×) e
`COMPARACAO-ICONE.html` (14 propostas de ícone da aba em quatro famílias, com o
**16×16 real** de cada uma, os pixels à mostra, e barras de aba falsas clara e
escura; clicar no cartão manda o candidato para a aba da própria página e imprime
a linha do `<link>`; a v8.24.1 saiu do `rota`). Lição que ela registra: **16×16
são 256 pixels — comparar em tamanho real, com os pixels à mostra, ou não
comparar**. E o veredito dela, que ranqueou por *legibilidade* e errou, porque o
usuário decidiu por *significado*.
E `COMPARACAO-CABECALHO.html` (o cabeçalho de hoje × três propostas, cada uma o
cabeçalho inteiro em tamanho real, clicável, com as **medidas tiradas na hora**
embaixo — largura do bloco, altura, se quebra em duas linhas, quantos botões
sem nome; e os cinco desenhos que enganavam, hoje → proposta; a v8.25.0 saiu da
"A"). ⚠️ O que ela registra: as quatro versões **dividem o mesmo estado** (ligar
o modo leve numa liga em todas), que é o que deixa comparar a mesma situação; e
o usuário disse nesta sessão que **assimila melhor por imagem** — proposta de
layout já deve nascer como página de comparação, não como texto.
⚠️ **A primeira versão dessa página não servia**, e o
defeito era meu: os quatro efeitos disparavam **ao mesmo tempo**, em quatro
painéis — ninguém consegue olhar para quatro lugares em 180ms ("não consegui
notar com nitidez a diferença real entre uma e outra"). Lição para as
próximas: **efeito curto se compara em sequência e no mesmo lugar**, com a
opção "como é hoje" na lista e um controle de câmera lenta. ⚠️ A de efeitos tem a rota de exemplo embutida, então funciona sem
rede — só os ladrilhos do mapa é que precisam de internet.

Regras de trabalho vigentes:

1. Implementar e testar, mas **perguntar antes de fazer commit/push** — o
   usuário testa antes de publicar; `.haga` dele significa "pode commitar e
   publicar agora".
2. **Toda alteração precisa funcionar com o MODO LEVE ligado** (decidido em
   24/09/2026). O usuário pensa no app como produto para outros clientes além
   da Hagamorfis, e "computador mais antigo" é a realidade esperada nesses
   lugares — um recurso que só funciona com vidro e animações seria inútil
   justamente para quem mais precisa do modo leve. Na prática: nada pode
   depender de `transitionend`, de `animationend`, de uma animação terminar,
   nem do mapa estar em tela cheia. **Testar nos dois estados.**
3. **`.Verify` do usuário pede uma revisão completa** (combinado em
   26/09/2026, nos mesmos moldes do `.haga`).
   ⚠️ **O escopo é o REPOSITÓRIO INTEIRO e o que está no ar — não só o
   `index.html`** (o usuário fez questão de deixar isso claro). Ou seja, os dez
   arquivos versionados: `index.html`, `CLAUDE.md`, `MAPA-DO-CODIGO.md`,
   `LOG-ALTERACOES.txt`, `README.md`, `.gitignore`, `.claude/launch.json`,
   `Exemplos/exemplo.umap` e os dois ícones em `Exemplos/icones/`. Mais a
   **cópia publicada** no GitHub Pages.
   As frentes:
   - **Bugs e falhas no código**: análise mecânica do `index.html` — funções
     declaradas × chamadas, ids × uso, classes × uso, variáveis de topo ×
     leitura, temporizadores e observadores com parada garantida,
     `addEventListener` acumulando — e leitura dirigida ao que ela apontar.
   - **`MAPA-DO-CODIGO.md`**: linha a linha, se a seção citada ainda existe com
     aquele nome e se as funções listadas ainda são as que fazem o trabalho.
   - **Documentação**: se o que este documento afirma ainda é verdade no
     código; se o `README.md` ainda descreve o app como ele é; se
     `LOG-ALTERACOES.txt`, a constante `VERSAO`, a tabela de versões e o backup
     da versão atual estão coerentes entre si.
   - **Os arquivos de apoio**: se o `.gitignore` ainda barra tudo que é dado
     real (o repositório é PÚBLICO — esta é a conferência que não pode falhar),
     se o `Exemplos/exemplo.umap` ainda cobre as três variantes de nome de
     camada e a camada vazia, e se o `.claude/launch.json` ainda sobe o
     servidor que este documento manda usar.
   - **O que está publicado**: se a `VERSAO` servida pelo GitHub Pages é a do
     `main`, se o repositório está limpo (nada de dado real esperando commit) e
     se o site abre sem erro de console nas **duas** telas.
   - **Funcionalidade**: se o que está documentado como funcionando de fato
     funciona, testado nos dois estados do modo leve (regra 2) e nas duas telas
     — escritório e campo, esta última por um link de roteiro de verdade.
   ⚠️ **Conferir a documentação não é zelo, é a parte que já pegou erro.** A
   revisão de 24/09/2026 achou um bug real (★ e tipo de serviço compartilhados
   entre técnicos, corrigido na v8.9.0) **e uma afirmação errada neste próprio
   arquivo** — a chave `hg_alturas_modulos` estava descrita como removida e
   continuava em uso. Este documento é a memória do projeto: quando ele mente, a
   mentira vira decisão errada na sessão seguinte.
   ⚠️ **`.Verify` não autoriza publicar.** Defasagem de documentação e do mapa
   do código é corrigida direto (é texto, risco baixo); defeito de comportamento
   é corrigido e relatado, mas o commit continua esperando o `.haga` — vale a
   regra 1. O resultado da revisão vai para o `LOG-ALTERACOES.txt`, como o da
   de 24/09.

Concluído:
- Tipo de serviço por parada — ✅ implementado na v1.9
- Busca/filtro no checklist — ✅ implementado na v2.1
- Base de clientes guardada no navegador — ✅ implementado na v2.2
- Endereço de origem memorizado — ✅ implementado na v2.5

Adiados a pedido do usuário em 29/08/2026 (continuam descritos no documento
de arquitetura, para retomar quando fizer sentido):
- Campo de observação livre por parada (Fase 2)
- ~~Dividir clientes entre as 2-3 pessoas da equipe, um link por pessoa~~ —
  ✅ **feito na v8.5.0**, em abas de técnico, com aviso no checklist quando um
  cliente já está na rota de outro.

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
| 85 | **v7.9.0** — Módulo Rota com um botão principal e um só idioma de ícones |
| 86 | **v7.9.1** — Vão de encaixe só com a borda, sem preenchimento |
| 87 | **v7.9.2** — Rodapé da tela do campo removido (frase e atalho do escritório) |
| 88 | **v8.0.0** — Efeito ao traçar a rota: voo, linha se desenhando, números em sequência e km/min contando |
| 89 | **v8.1.0** — Vidro: o mapa passa por baixo do painel e das colunas |
| 90 | **v8.2.0** — Fluidez: o borrão sai enquanto o mapa se mexe, e um interruptor desliga o vidro |
| 91 | **v8.3.0** — Correção: o mapa pintava por cima do painel com o vidro desligado |
| 92 | **v8.4.0** — Janelas livres: cada coluna solta pela tela, atrás de um botão |
| 93 | **v8.4.1** — Barras de rolagem no tema, com a pista transparente |
| 94 | **v8.4.2** — Checklist sem a bolinha: a seta e o contador acendem |
| 95 | **v8.5.0** — Um roteiro por técnico, em abas, com as rotas juntas no mapa |
| 96 | **v8.6.0** — O nome do técnico viaja no link e aparece na tela do campo |
| 97 | **v8.7.0** — Modo leve: um botão desliga de uma vez tudo que custa desenho |
| 98 | **v8.8.0** — Quilometragem somada do dia, com todas as abas |
| 99 | **v8.9.0** — A parada é do técnico, não da base (★ e tipo deixam de ser compartilhados) |
| 100 | **v8.10.0** — O módulo Rota grudado deixa de virar uma laje mais clara |
| 101 | **v8.11.0** — Os puxadores de altura ganham teto (nenhum módulo maior que o painel) |
| 102 | **v8.12.0** — O módulo Rota rola como os outros (fim do sticky) e o botão principal fica sem preenchimento |
| 103 | **v8.13.0** — A Rota nasce logo abaixo da Origem: os botões de traçar ficam à vista sem rolar |
| 104 | **v8.14.0** — Passagem ao trocar de técnico: o painel sai junto e volta em cascata |
| 105 | **v8.15.0** — O que o mapa mostra dos clientes: um botão apaga ou esconde as bolinhas de fora da viagem |
| 106 | **v8.16.0** — O trajeto do dia na tela do campo (link v10 com a geometria simplificada) |
| 107 | **v8.16.1** — Primeira revisão `.Verify`: nomes reais fora dos arquivos versionados, mapa do código e README em dia |
| 108 | **v8.17.0** — Correção: o clique na parada nem sempre chegava ao ponto (a animação era o único caminho) |
| 109 | **v8.17.1** — A ida até a parada vira voo em arco, e a rede de segurança para de dar solavanco |
| 110 | **v8.18.0** — O Roteiro nasce fechado, e "Enviar para o campo" entra no módulo Rota (de sete módulos para seis) |
| 111 | **v8.19.0** — Ponto 5 da revisão de layout: acima de 1600px a coluna 2 já nasce aberta, e o painel para de rolar |
| 112 | **v8.20.0** — Ponto 6: embaixo do link fica só o "?", e as três explicações vão para o painel que ele abre |
| 113 | **v8.21.0** — Ponto 3: a monoespaçada volta a ser dos números; todo rótulo do escritório passa para Inter, em caixa normal |
| 114 | **v8.22.0** — Ponto 4: o checklist vira lista — fio entre as linhas e filete na cascata, no lugar de um cartão por cliente |
| 115 | **v8.22.1** — A Ordem da viagem segue a mesma lógica (pedido do usuário); o cartão volta só enquanto a parada é arrastada |
| 116 | **v8.23.0** — Ponto 1: a escala do escritório vai de oito degraus de texto para três (10,5 · 11,5 · 13) |
| 117 | **v8.23.1** — Os botões de ícone do Origem e do Selecionar paradas passam a ter a mesma cor e o mesmo tamanho, menores |
| 118 | **v8.23.2** — O traço do ícone volta a ser claro nos dois (pedido do usuário); quem carrega a hierarquia passa a ser o tamanho |
| 119 | **v8.24.0** — A busca do checklist ignora acento e cedilha (`educacao` acha `EDUCAÇÃO`) |
| 120 | **v8.24.1** — Ícone da aba: uma rota em âmbar (origem pequena, destino grande), SVG em data-URI, sem letra |
| 121 | **v8.24.2** — O mesmo ícone ao lado do nome "hagamorfis/rotas", nas duas telas |
| 122 | **v8.25.0** — Cabeçalho com nome: sete ícones viram dois botões ("Mapa: …" e o menu "Exibição"); disquete, lixeira, vidro e os dois da Origem trocam de desenho |
| 123 | **v8.26.0** — Varredura de bugs: nove correções (resposta do OSRM conferida antes de aplicar, origem desfaz a rota, tracejado do efeito sai, nome da avulsa chega à viagem, entre outras) |

---

## 10. Estrutura do projeto e hospedagem

O projeto é versionado em Git e publicado no GitHub Pages.

**Estrutura (a partir de 22/08/2026):**

```
PROJETO APP LOGISTICA/        ← raiz do repositório Git
├── index.html                ← o app (servido pelo GitHub Pages)
├── CLAUDE.md                 ← este arquivo (contexto do projeto)
├── MAPA-DO-CODIGO.md         ← onde cada coisa mora no index.html (v8.8.0)
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
8.26.0
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

⚠️ **A pasta `backups/` é ignorada por padrão, e só sobem as pastas liberadas
no `.gitignore`** (desde 10/10/2026, a pedido do usuário: "todo backup na máquina
local e no repositório também"). Sobem **v1.0, v1.1 e da v8.16.1 em diante**.
**Ficam só na máquina** as outras 102 pastas e o `MARCO_v8.23.2`: trazem nomes de
clientes reais em comentários do código e nos `LEIA-ME`, e o repositório é
PÚBLICO — a limpeza de 26/09/2026 saneou os arquivos da raiz, **não os backups**.
Conferido por varredura dos nomes dos `.umap` reais locais contra cada pasta
(o `.zip` da v1 foi excluído: comprimido, a varredura não o enxerga).
⚠️ **Backup novo NÃO sobe sozinho**: varrer por nome real e só então acrescentar a
linha `!backups/<pasta>/` ao `.gitignore`. A pasta inteira está dentro do
OneDrive, o que é uma segunda cópia, e as versões ficam também no histórico do
Git; mas o `MARCO` e as 102 pastas antigas **não estão no GitHub** — numa troca
de máquina, copiar `backups/` junto.

**Antes de uma mudança estrutural, o marco é diferente** (criado em 28/09/2026,
antes da fase de métricas, a pedido do usuário: *"faça uma marcação do projeto
atual, pois acredito que será realizada uma alteração importante"*). São duas
coisas, e as duas juntas:

1. uma **tag anotada** no commit (`git tag -a v8.23.2`), que é a única das duas
   que **sobrevive a troca de máquina**, porque vai para o GitHub;
2. uma pasta `backups/MARCO_vX.Y.Z_AAAA-MM-DD_apelido/` com **os dez arquivos
   versionados**, e não só o `index.html`.

⚠️ **Os dez e não um**: numa mudança estrutural o projeto pode **ganhar
arquivos** (a fase de métricas pode acrescentar uma função de servidor, por
exemplo), e aí um backup do `index.html` sozinho não reconstrói nada. O
`LEIA-ME.txt` do marco traz o hash do commit, o nome da tag e os três caminhos
de volta — `git checkout`, `git revert` e `git reset --hard`.

⚠️ **Marco não substitui o backup de versão**: ele é raro, por fase, e o de
versão continua sendo a cada número fechado.

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
| 7.9.0 | 19/09/2026 | Módulo Rota com um botão principal e ícones de traço |
| 7.9.1 | 19/09/2026 | Vão de encaixe só com a borda |
| 7.9.2 | 19/09/2026 | Rodapé da tela do campo removido |
| 8.0.0 | 19/09/2026 | Efeito ao traçar a rota |
| 8.1.0 | 22/09/2026 | Vidro: o mapa por baixo do painel e das colunas |
| 8.2.0 | 23/09/2026 | Fluidez do vidro (travava em máquina com vídeo integrado) |
| 8.3.0 | 23/09/2026 | Correção: o mapa pintava por cima do painel sem o vidro |
| 8.4.0 | 23/09/2026 | Janelas livres (posição, tamanho e ordem de frente por coluna) |
| 8.4.1 | 23/09/2026 | Barras de rolagem acompanhando o tema |
| 8.4.2 | 23/09/2026 | Checklist sem a bolinha da calha; chevron de traço |
| 8.5.0 | 23/09/2026 | Um roteiro por técnico, em abas (Fase 2) |
| 8.6.0 | 23/09/2026 | Nome do técnico no cabeçalho da tela do campo (link v9) |
| 8.7.0 | 24/09/2026 | Modo leve (para computador mais antigo) |
| 8.8.0 | 24/09/2026 | Quilometragem somada do dia |
| 8.9.0 | 24/09/2026 | A parada é do técnico, não da base; limpeza de peso morto |
| 8.10.0 | 24/09/2026 | Correção: o fundo do módulo Rota grudado não batia com o cartão |
| 8.11.0 | 25/09/2026 | Correção: puxadores de altura sem teto deixavam o módulo passar do painel |
| 8.12.0 | 25/09/2026 | Fim do módulo Rota grudado; botão "Rota otimizada" sem preenchimento |
| 8.13.0 | 25/09/2026 | Ordem padrão do painel com a Rota logo abaixo da Origem |
| 8.14.0 | 25/09/2026 | Saída + cascata ao trocar de técnico, no painel e no mapa |
| 8.15.0 | 26/09/2026 | O que o mapa mostra dos clientes: tudo / fora da viagem apagado / fora da viagem escondido |
| 8.16.0 | 26/09/2026 | O trajeto do dia na tela do campo, com o desenho da rota dentro do link |
| 8.16.1 | 26/09/2026 | Revisão `.Verify`: saneamento dos nomes reais, correções no mapa do código e no README |
| 8.17.0 | 26/09/2026 | Correção: o clique na parada passa a chegar sempre ao ponto clicado, com rede de segurança |
| 8.17.1 | 26/09/2026 | A ida até a parada em voo suave; a rede deixa de disparar a toa (tolerância em pixel) |
| 8.18.0 | 26/09/2026 | O Roteiro nasce fechado; "Enviar para o campo" vira parte do módulo Rota e o campo de retomar repetido sai |
| 8.19.0 | 26/09/2026 | Acima de 1600px o padrão passa a ser duas colunas: 31% de painel fora da vista viram 0% |
| 8.20.0 | 27/09/2026 | O bloco de envio devolve 121px ao painel: embaixo do link sobra só o "?" |
| 8.21.0 | 27/09/2026 | Rótulos em Inter e caixa normal; a mono fica só nos números (36 elementos em mono viram 23) |
| 8.22.0 | 27/09/2026 | Checklist sem cartão por linha: a lista encolhe 14% e o nome da unidade sai de dentro de duas molduras |
| 8.22.1 | 27/09/2026 | A Ordem da viagem também vira lista; o cartão só existe enquanto a parada está sendo arrastada |
| 8.23.0 | 27/09/2026 | Escala tipográfica do escritório em três degraus de texto; some a inversão da legenda do km |
| 8.23.1 | 28/09/2026 | Botões de ícone unificados (cor, largura e desenho) e menores; o campo de texto ganha espaço |
| 8.23.2 | 28/09/2026 | O ícone volta ao claro em repouso; o hover passa a ser borda teal + fundo, já que clarear não diria nada |
| 8.24.0 | 10/10/2026 | A busca do checklist ignora acento e cedilha, nos três níveis |
| 8.24.1 | 10/10/2026 | Ícone da aba: rota âmbar sobre crachá escuro, em data-URI, sem letra |
| 8.24.2 | 10/10/2026 | O mesmo ícone ao lado do nome da marca, nas duas telas |
| 8.25.0 | 10/10/2026 | Cabeçalho com dois botões com nome e menu "Exibição"; cinco desenhos que diziam outra coisa, trocados |
| 8.26.0 | 10/10/2026 | Nove correções da varredura de bugs; quatro pontos ficaram em aberto |
