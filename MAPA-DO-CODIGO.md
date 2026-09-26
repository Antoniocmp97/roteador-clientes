# Mapa do código — `index.html`

> Onde cada coisa mora e o que liga o quê. Feito em 24/09/2026, na v8.8.0
> (6.446 linhas). Os números de linha envelhecem; os **títulos de seção** não —
> procure pelo título quando a linha não bater.
>
> Este documento responde "quero mexer em X, onde encosto?". O **porquê** de
> cada decisão está no `CLAUDE.md`, e o histórico no `LOG-ALTERACOES.txt`.

---

## 1. As três camadas do arquivo

| Camada | Tamanho | Onde |
|---|---|---|
| CSS | 1.426 linhas | um `<style>` no `<head>` |
| Markup | 473 linhas | `<body>`: duas telas inteiras |
| JS | 4.548 linhas | um `<script>` no fim do `<body>` |

**36% do arquivo são comentários** (130 KB de 365 KB). É proposital: eles são a
memória do projeto. Não afetam o desempenho — o GitHub Pages serve comprimido, e
o navegador descarta comentário no parse.

⚠️ **Duas telas no mesmo arquivo.** `.app` é o escritório; `#telaCampo` é a tela
do técnico. Quem decide qual aparece é `modoCampoAtivo`, calculado uma vez a
partir do `#` da URL. Quase todo o JS está dentro de `if (!modoCampoAtivo){...}`.

---

## 2. Sequência de arranque (a ordem importa)

```
1. <head> script curto        → data-tema, data-vidro, data-janelas, data-leve
                                ANTES da primeira pintura (senão a tela salta)
2. CSS                        → lê esses quatro atributos do <html>
3. <script> principal:
   3.1 const VERSAO           → preenche os selos .versao das duas telas
   3.2 map = L.map(...)       → só se !modoCampoAtivo
   3.3 iniciarTecnicos()      → cria a aba 1; o estado dela SÃO as globais
   3.4 aplicarTema(...)
   3.5 arranjo = arranjoGuardado() || arranjoNormal()
   3.6 aplicarArranjo()       → põe os módulos nas colunas ou nas janelas
   3.7 aplicarModoJanelas / aplicarIma / aplicarModoLeve  → acertam os ícones
   3.8 base guardada volta    → importClients() → oferecerRetomarDia()
```

⚠️ **O `<head>` script é o único fora do bloco principal, e é de propósito.** Os
quatro atributos precisam existir antes de o navegador pintar. O modo leve é
aplicado **por último** ali dentro, porque ele manda no vidro e nas janelas.

---

## 3. "Quero mexer em…" → onde encostar

| Se você quer mexer em | Procure a seção | Função que faz o trabalho |
|---|---|---|
| tema claro/escuro | `Modo noturno (v3.3)` | `aplicarTema(tema, salvar)` |
| vidro ligado/desligado | `Vidro ligado ou desligado (v8.2.0)` | `aplicarVidro(lig, salvar)` |
| **modo leve** | `Modo leve (v8.7.0)` | `aplicarModoLeve(lig, salvar)` |
| janelas livres | `Janelas livres (v8.4.0)` | `aplicarModoJanelas`, `aplicarJanelas` |
| ímã de alinhamento | dentro de `Janelas livres` | `aplicarIma(lig, salvar)` |
| abas de técnico | `Um roteiro por técnico, em abas (v8.5.0)` | `renderAbas`, `trocarTecnico` |
| efeito ao trocar de aba | `A passagem ao trocar de técnico (v8.14.0)` | `passagemLigada`, `animarTrocaDeAba`, `escalonarCascata` |
| km do dia | `Quilometragem do dia (v8.8.0)` | `resumoDoDia`, `atualizarResumoDia` |
| restaurar o planejamento | `O planejamento do dia sobrevive…` | `salvarDia`, `restaurarDia` |
| carregar o `.umap` | `Upload / parse GeoJSON` | `lerUmap`, `importClients` |
| checklist de clientes | `Checklist + stops UI` | `renderChecklist` |
| lista de paradas (ordem) | `Arrastar para reordenar (v3.8)` | `renderStopsList` |
| tipos de serviço | `Configuração dos tipos de serviço` | `renderTiposPainel` |
| parada avulsa | `Parada avulsa (v7.0.0)` | `criarParadaAvulsa`, `novoPontoAvulso` |
| origem | `Origin` / `Origem padrão salva` | `setOrigin`, `lerOrigemPadrao` |
| traçar / otimizar | `Routing` | `drawResult`, `prepararEnvio` |
| efeito ao traçar | `Efeito ao traçar a rota (v8.0.0)` | `efeitoLigado`, `animarNumero` |
| mover módulos entre colunas | `Arrastar um módulo pela alça…` | `destinoDoModulo`, `moverModulo` |
| ordem em que os módulos nascem | `MODULOS` / `ORDEM_PADRAO` | `arranjoNormal` |
| alturas dentro das colunas | `Espaço dividido entre os módulos 3 e 4` | `alturasGuardadas`, `atualizarModulosNasColunas` |
| link do roteiro | `Identificador do roteiro` / `Compressão do link` | `montarRoteiro`, `lerRoteiroCompacto` |
| retomar pelo link | `Retomar um roteiro a partir do link` | `retomarRoteiro` |
| tela do técnico | fim do arquivo | `desenharCampo` |

---

## 4. Estado global (o que existe e quem manda nele)

### 4.1 Estado de UM roteiro — trocado ao mudar de aba

```
stops[]          paradas escolhidas, NA ORDEM da viagem
originLatLng     [lat, lng] da origem
originMarker     o pino da origem no mapa
routeLine        a polyline traçada
stopMarkers[]    os marcadores numerados
ultimoResumo     { d: metros, t: segundos }
idRoteiro        o rid, que o link leva
```

⚠️ **Essas sete são as do técnico ATIVO.** Elas são lidas em ~150 lugares (só
`stops` aparece 83 vezes), e foi por isso que as abas não viraram uma
refatoração: `salvarNoTecnico()` copia as sete para o objeto da aba e
`carregarDoTecnico()` traz as da outra. Nenhuma das ~150 chamadas mudou.

⚠️ **A exceção são as camadas do mapa**: `linha`, `marcadores` e `origemMarker`
de cada técnico ficam no mapa o tempo todo — é o que permite ver as rotas
juntas. `realcarCamadas()` só muda qual está acesa.

### 4.2 Estado que não troca de aba

```
clientPoints[]   todos os pontos da base + as avulsas (cada um com .marker)
gruposClientes[] a mesma base agrupada por camada, só para o checklist
tecnicos[]       as abas; tecnicoAtivo é o índice da aberta
arranjo          { local, ordem, pesos, fixas, janelas } — onde cada módulo está
map              o Leaflet
```

### 4.3 Preferências no navegador

| Chave | O que guarda |
|---|---|
| `hg_tema` | claro / escuro |
| `hg_vidro` | vidro ligado |
| `hg_janelas` | janelas livres ligadas |
| `hg_ima` | ímã de alinhamento |
| `hg_leve` | **modo leve** |
| `hg_base_clientes` | a última base `.umap` carregada |
| `hg_origem_padrao` | a origem que volta pronta |
| `hg_tipos_servico` | a lista de tipos |
| `hg_voltar_origem` | o interruptor de retorno |
| `hg_zoom_parada` | zoom ao clicar na parada |
| `hg_largura_painel`, `hg_largura_guia`, `hg_largura_guia2` | larguras |
| `hg_alturas_modulos` | alturas do checklist e das paradas no painel |
| `hg_arranjo_modulos` | o arranjo salvo pelo disquete |
| `hg_dia_planejado` | o planejamento do dia (oferecido ao abrir) |
| `hg_link_compativel` | formato do link (legível x comprimido) |
| `hg_prog_r_<rid>` | progresso do campo, por roteiro |

---

## 5. Os quatro atributos do `<html>` e o que cada um liga

O CSS inteiro dos modos pendura nesses quatro. Nenhum deles tem JS decidindo
layout — quem desenha é o CSS.

| Atributo | Valores | Liga |
|---|---|---|
| `data-tema` | `escuro` / `claro` | a paleta inteira e o conjunto de ladrilhos |
| `data-vidro` | `1` / `0` | mapa em tela cheia + `backdrop-filter`, ou colunas em fila |
| `data-janelas` | `1` / `0` | colunas absolutas com puxador, ou fila |
| `data-leve` | `1` / `0` | **manda nos dois acima** e mata transições, animações e sombras |

⚠️ **`data-leve` não apaga a escolha dos outros.** Ligado, ele força
`data-vidro=0` e `data-janelas=0` sem tocar no `localStorage`; ao sair,
`vidroGuardado()` e `janelasGuardado()` devolvem o que estava lá.

⚠️ **Regra permanente (24/09/2026):** toda alteração nova precisa funcionar com
`data-leve="1"`. Nada pode depender de `transitionend`, `animationend`, de uma
animação terminar, nem do mapa estar em tela cheia.

---

## 6. Os caminhos que cruzam o app inteiro

### 6.1 Da base à rota

```
arquivo .umap
  → lerUmap()          quebra em camadas (= clientes) e pontos (= filiais)
  → importClients()    monta clientPoints[] e gruposClientes[], põe no mapa,
                       salva a base, e chama oferecerRetomarDia()
  → renderChecklist()  desenha a cascata Grupo → Camada → Unidades
  → [checkbox]         entra/sai de stops[]
  → renderStopsList()  desenha a ordem da viagem (e agenda salvarDia)
  → routeBtn / optimizeBtn
  → drawResult()       polyline + marcadores numerados, na cor do técnico
  → prepararEnvio()    guarda ultimoResumo e revela "Enviar para o campo"
  → atualizarResumoDia()  soma os km de todas as abas
```

### 6.2 Do escritório ao campo

```
montarRoteiro()        monta o texto do formato v9 (10 grupos separados por ~)
  → comprime (gzip nativo) ou deixa legível, conforme hg_link_compativel
  → link "…/#z=" ou "…/#r="
  → [WhatsApp]
  → outra máquina abre: modoCampoAtivo = true
  → lerRoteiroCompacto()  entende v5 a v9
  → desenharCampo()       cartão da vez, faixas das concluídas, linhas das outras
  → progresso em hg_prog_r_<rid>, por COORDENADA da parada
```

⚠️ **O link é a única coisa que sai da máquina, e ele viaja depois do `#`** —
o que vem depois do `#` não é enviado ao servidor. É o que sustenta a regra de
"nenhum dado de cliente em servidor".

### 6.3 Qualquer mexida que invalide a rota

```
mudou stops, origem, retorno, ou trocou a base
  → invalidarRota()    tira a linha e os marcadores DO TÉCNICO ATIVO,
                       zera ultimoResumo, esconde o link
  → atualizarResumoDia()
```

⚠️ `invalidarRota()` mexe só no técnico ativo — as rotas das outras abas ficam.

---

## 7. Armadilhas que já morderam (não repetir)

1. **Relógio de quadros parado.** Em aba que não está pintando, o navegador
   pausa `requestAnimationFrame` e as transições/animações. `moveend` pode não
   chegar, `setView` animado não aplica, `getComputedStyle` devolve o último
   valor confirmado. Toda animação do app tem rede de `setTimeout`.
   *Já mordeu nas v6.5, v6.9, v8.0.0, v8.1.0, v8.4.2 e v8.7.0.*
2. **`#map` nunca pode ser `position:static`.** Os painéis do Leaflet se ancoram
   no primeiro ancestral posicionado; static faz eles vazarem por cima do painel.
3. **`display:contents` não muda a árvore do HTML** — o seletor precisa descer
   pela `section`.
4. **Regra de elemento vence em silêncio.** `header{align-items:baseline}`
   chegou na tela do campo e zerou a barra de progresso por meses.
5. **`offsetTop` num elemento sticky já vem com o deslocamento embutido.**
6. **O id de um ponto da base é `c<camada>_<feature>`** — reordenar uma camada
   no uMap muda todos. Casar parada guardada é sempre por **coordenada**.

---

## 8. O que NÃO está no arquivo

- Nenhuma dependência instalada. Leaflet vem do cdnjs; as fontes, do Google.
- Nenhum build, nenhum bundler, nenhum teste automatizado.
- Nenhum dado de cliente. A base real entra por upload e fica no navegador.
