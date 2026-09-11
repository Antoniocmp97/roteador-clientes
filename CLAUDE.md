# Roteador de Clientes — Hagamorfis

> Documento de contexto do projeto. Mantido atualizado a cada passo para permitir
> migração do chat para o Claude Code sem perda de contexto.
>
> **Última atualização:** 10/09/2026 — v4.0 (tela do campo parou de vazar no escritório; subtítulo removido)

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

- Upload do **backup completo do uMap (`.umap`)** por clique ou arrastar-e-soltar
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
  volta ao padrão). Existe porque o mesmo site é usado numa TV de 1366x768 e
  em monitores Full HD — nenhuma largura fixa serve para as duas
- **Nome do cliente nunca é cortado**: quebra em duas ou três linhas quando o
  painel está estreito, em vez de terminar em "…" (o final do nome é justamente
  o que distingue uma unidade da outra)
- **Busca no checklist** (Fase 3): filtra por nome do cliente ou da filial;
  grupos com resultado abrem sozinhos durante a busca
- **Base guardada no navegador** (Fase 3): depois de carregar o `.umap` uma vez,
  a base volta sozinha na próxima abertura. Também é a rede de segurança se o
  uMap sair do ar. Link "esquecer base" apaga a base guardada (com confirmação)
- **Tipo de serviço por parada** (Fase 2): lista configurável (editável pelo
  próprio escritório, salva no navegador — ex.: "Entrega de toner",
  "Manutenção"). Cada parada selecionada ganha um seletor para escolher o
  tipo; aparece na lista de paradas do escritório e como etiqueta na tela
  do campo
- **Modo noturno** (v3.3): interruptor no cabeçalho das **duas telas**
  (escritório e campo) troca o app inteiro entre tema escuro e claro. A escolha
  fica guardada no navegador e é aplicada antes da primeira pintura, para a tela
  não piscar no tema errado ao abrir. Padrão: escuro
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
  da linha continua rolando a lista
- **Paradas prioritárias** (★): as marcadas ficam fixas no início, na ordem
  escolhida, e "Otimizar ordem" reordena só as demais — para quando é preciso
  passar num lugar antes do resto do roteiro
- **Traçar nesta ordem** — rota respeitando a ordem escolhida (OSRM Route API)
- **Otimizar ordem** — OSRM Trip API recalcula a melhor sequência e reordena a
  lista automaticamente (`source=first`, `roundtrip=false`)
- Marcadores numerados conforme a ordem final de visita
- Distância total, tempo estimado e instruções passo a passo agrupadas por parada
- **Modo campo (Fase 1):** depois de traçar a rota, um botão gera um link com o
  roteiro inteiro. Quem abre esse link (ex.: recebido pelo WhatsApp) cai numa
  tela separada, feita para celular: lista das paradas na ordem certa, botão
  de navegação para Waze ou Google Maps em cada uma, e um botão para marcar a
  parada como concluída. O progresso marcado fica salvo no aparelho de quem
  abriu (sobrevive a fechar o navegador) e nenhum dado de cliente passa pelo
  servidor — o roteiro trafega inteiro dentro do link, no trecho depois do
  `#` (ver ADR-01 no documento de arquitetura).

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
  a largura do painel, o tema (claro/escuro) e o progresso do modo campo. **Não** ficam salvos:
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

Formato `MAIOR.MENOR`, subindo de 0.1 em 0.1 (1.0 → 1.1 → 1.2 … → 1.9 → 2.0).

Uma versão nova é fechada quando acontece **um** destes dois gatilhos:

1. **Uma alteração mediana ou grande** — sobe a versão na hora.
2. **Cinco alterações leves acumuladas** — sobem a versão juntas.

O objetivo do segundo gatilho é não gerar uma versão nova a cada ajuste de
texto ou de cor, o que encheria a pasta de backups sem necessidade.

**O que é alteração leve:** não muda o comportamento do app. Texto, cor,
espaçamento, comentário no código, documentação, renomear variável.

**O que é mediana ou grande:** muda o que o app faz ou o que o usuário vê.
Funcionalidade nova, troca de serviço externo, mudança de fluxo, correção de
bug que atrapalhava o uso.

### Contador de alterações leves

Quando houver alteração leve, incrementar aqui. Ao chegar em 5, fechar versão
nova e zerar o contador.

```
Leves acumuladas desde a v4.0:  0 / 5
```

### Onde o número aparece

Na constante `VERSAO` no início do script do `index.html`, e a partir dela num
selo discreto ao lado da marca **nas duas telas** — escritório e campo. A ideia
é que quem está na rua consiga informar a versão ao relatar um problema.

Ao fechar versão nova, atualizar a constante `VERSAO` junto com o resto.

### Backups

Cada versão fechada ganha uma cópia em `backups/vX.Y_AAAA-MM-DD_apelido/`,
com o `index.html` daquela versão e um `LEIA-ME.txt` explicando o que mudou e
por quê.

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
