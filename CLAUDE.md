# Roteador de Clientes — Hagamorfis

> Documento de contexto do projeto. Mantido atualizado a cada passo para permitir
> migração do chat para o Claude Code sem perda de contexto.
>
> **Última atualização:** 22/08/2026 — Passo 3

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

- Upload de arquivo `.geojson` por clique ou arrastar-e-soltar
- Parse de `FeatureCollection`, extraindo features do tipo `Point`
- Plot dos pontos no mapa (Leaflet) com popup por cliente
- Definição de origem: geolocalização do navegador **ou** endereço digitado
  (geocodificação via Nominatim, com viés para Criciúma/SC)
- Checklist de seleção de quais clientes visitar na viagem
- Lista de paradas reordenável manualmente (setas ▲▼ e remoção)
- **Traçar nesta ordem** — rota respeitando a ordem escolhida (OSRM Route API)
- **Otimizar ordem** — OSRM Trip API recalcula a melhor sequência e reordena a
  lista automaticamente (`source=first`, `roundtrip=false`)
- Marcadores numerados conforme a ordem final de visita
- Distância total, tempo estimado e instruções passo a passo agrupadas por parada

---

## 3. Stack técnica

| Camada | Escolha | Observação |
|---|---|---|
| Mapa | Leaflet 1.9.4 (CDN cdnjs) | Mesma base que o uMap usa |
| Tiles | CARTO dark_all | Combina com o tema escuro da UI |
| Rotas | OSRM — `router.project-osrm.org` | **Servidor público de demonstração** |
| Otimização | OSRM Trip API | Resolve TSP aproximado |
| Geocodificação | Nominatim (OpenStreetMap) | Sujeito a política de uso justo |
| Fontes | Space Grotesk, JetBrains Mono, Inter | Google Fonts |

### Identidade visual

Tema escuro. Variáveis CSS definidas em `:root`:

```
--ink:#0C1418  --panel:#16232B  --panel-2:#1D2E37  --line:#2A3D47
--text:#E7EEF2 --muted:#7C93A0  --amber:#F2A93C   --amber-dim:#B87F26
--teal:#2FB6A6 --danger:#E2604F
```

Âmbar = origem, rota traçada e paradas numeradas. Teal = clientes não
selecionados e estados de sucesso.

---

## 4. Decisões tomadas

1. **uMap não faz roteamento interno.** Ele apenas abre um link externo para o
   OSRM ao pedir direções. Por isso construímos app próprio sobre Leaflet + OSRM,
   usando o uMap apenas como ferramenta de criação/exportação dos pontos.
2. **Dados carregados por upload**, não fixos no código nem buscados de URL.
   Usuário carrega o `.geojson` quando quiser.
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

---

## 6. Formato de dados esperado

GeoJSON `FeatureCollection` com features do tipo `Point`:

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": { "type": "Point", "coordinates": [-49.3697, -28.6775] },
      "properties": { "name": "Nome do Cliente" }
    }
  ]
}
```

Coordenadas em ordem GeoJSON: `[longitude, latitude]`. O nome é lido de
`properties.name`; sem ele, o ponto recebe rótulo genérico "Ponto N".

Arquivo de exemplo versionado: `Exemplos/exemplo.geojson` — 5 pontos fictícios
na região de Criciúma, apenas para demonstrar o formato.

O arquivo com os pontos reais usados nos testes fica **fora do repositório**
(`clientes-teste.geojson` na raiz, bloqueado pelo `.gitignore`), porque o
repositório é público.

---

## 7. Limitações conhecidas

- **OSRM público** é servidor de demonstração, sem garantia de disponibilidade
  nem uso comercial. Para produção: instância própria de OSRM ou GraphHopper
  (auto-hospedadas, gratuitas) ou API paga (Mapbox Directions, Google Directions).
- **Nominatim** tem política de uso justo — limite aproximado de 1 requisição por
  segundo. Volume alto de geocodificação exige alternativa.
- **Sem persistência.** Nada é salvo: recarregar a página zera tudo.
- **Sem busca no checklist.** Com dezenas ou centenas de clientes, rolar a lista
  fica impraticável.
- **Otimização puramente geográfica.** Considera apenas distância/tempo de carro.
  Não trata prioridade de cliente, janelas de horário ou duração da visita.
- **Sem navegação por voz** ou trânsito em tempo real.

---

## 8. Direções em aberto

Levantadas mas ainda **não decididas** — pendente de alinhamento com o usuário:

- Quem usa o app (o próprio usuário, vendedores em campo, entregadores?) e em
  qual dispositivo
- Origem real da base de clientes (uMap, planilha, sistema interno?) e volume
- Distribuição de visitas entre múltiplos vendedores
- Botão "abrir no Google Maps / Waze" para navegação por voz
- Hospedagem (GitHub Pages, Netlify) para acesso pela equipe sem passar arquivo
- Busca/filtro no checklist quando a lista crescer

---

## 9. Histórico

| Passo | O que foi feito |
|---|---|
| 1 | Protótipo inicial: destino único, filiais fixas no código (Rio Maina / Próspera) |
| 2 | Reconstrução: upload de GeoJSON, múltiplas paradas, otimização de ordem, filiais de exemplo removidas |
| 3 | Correção do "Failed to fetch" — `safeFetchJSON()` com mensagem explicativa |

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
│   └── exemplo.geojson       ← pontos fictícios, só para demonstrar o formato
├── .claude/launch.json       ← config do servidor local de testes
└── backups/                  ← pontos de restauração (NÃO versionado)
```

- **Repositório:** github.com/antoniocmp97/roteador-clientes (público)
- **Site publicado:** https://antoniocmp97.github.io/roteador-clientes/
- Todo `git push` para o branch `main` republica o site automaticamente.

**Regra de dados:** o repositório é público, então dados reais de clientes
**nunca** são versionados. O app carrega o `.geojson` por upload — os dados
ficam só na máquina de quem usa. O `.gitignore` bloqueia `clientes*.geojson`
e a pasta `dados-reais/` por precaução.

**Para testar localmente:** abrir o `index.html` no navegador, ou usar o
servidor local (`.claude/launch.json`). Não usar a pré-visualização embutida
do chat — ela bloqueia as chamadas de rede ao OSRM e ao Nominatim.
