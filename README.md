# Roteador de Clientes — Hagamorfis

Aplicativo web para montar rotas de visita com múltiplas paradas a partir de
uma lista de clientes em GeoJSON. Feito para uso em Criciúma/SC.

**➡️ Acesse: https://antoniocmp97.github.io/roteador-clientes/**

---

## O que faz

- Carrega os clientes do backup completo do uMap (`.umap`) — clique ou arraste
- Separa por cliente: cada camada do uMap é um cliente, e os pontos dentro dela
  são as filiais. A lista abre em cascata ao clicar no nome do cliente
- Mostra todos os pontos no mapa
- Define o ponto de partida por endereço digitado ou pela sua localização
- Permite escolher quais clientes visitar na viagem
- **Traçar nesta ordem** — calcula a rota respeitando a ordem que você definiu
- **Otimizar ordem** — encontra automaticamente a melhor sequência de visitas
- Mostra distância total, tempo estimado e o roteiro passo a passo

## Como usar

1. Abra o link acima (funciona no computador e no celular)
2. Carregue o backup completo do uMap (`.umap`) com seus clientes
3. Digite o endereço de partida ou toque em 📍 para usar sua localização
4. Marque os clientes que vai visitar
5. Clique em **Otimizar ordem** (ou **Traçar nesta ordem**, se já sabe a sequência)

## Formato do arquivo de clientes

O app lê o **backup completo do uMap** (`.umap`). No uMap, abra o painel
**"Compartilhar e baixar"** e escolha o backup completo.

É o único formato que preserva as camadas — e no uso deste app **cada camada é
um cliente** e os pontos dentro dela são as **filiais**:

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

Coordenadas na ordem `[longitude, latitude]`. O nome do cliente vem de
`_umap_options.name` (ou `_storage.name`, em versões antigas do uMap) e o da
filial, de `properties.name`.

> **O download simples em `.geojson` não serve.** Ele junta todas as camadas
> numa lista só e descarta os nomes, então a informação de cliente não chega ao
> app. Se você carregar um por engano, o app avisa e explica o caminho certo.
>
> Atenção também: a exportação do uMap **só inclui camadas visíveis**. Camada
> com o "olho" desligado fica de fora do arquivo.

Há um arquivo de exemplo em [`Exemplos/exemplo.umap`](Exemplos/exemplo.umap),
com 3 clientes fictícios na região de Criciúma apenas para demonstrar o formato.

## Privacidade dos dados

Os dados de clientes **não ficam neste repositório**. O aplicativo lê o arquivo
direto no navegador — nada é enviado nem armazenado em servidor. Cada pessoa
carrega o próprio arquivo ao usar.

## Tecnologia

Arquivo HTML único, sem build e sem dependências instaladas.

| Função | Serviço |
|---|---|
| Mapa | [Leaflet](https://leafletjs.com/) + tiles CARTO |
| Rotas e otimização | [OSRM](http://project-osrm.org/) |
| Busca de endereços | [Nominatim](https://nominatim.openstreetmap.org/) (OpenStreetMap) |

> **Nota:** o OSRM usado é o servidor público de demonstração, sem garantia de
> disponibilidade e não indicado para uso comercial intenso. Para produção,
> considere hospedar uma instância própria ou usar uma API paga.

## Documentação

- [`CLAUDE.md`](CLAUDE.md) — contexto completo do projeto, decisões e limitações
- [`LOG-ALTERACOES.txt`](LOG-ALTERACOES.txt) — histórico detalhado de alterações
