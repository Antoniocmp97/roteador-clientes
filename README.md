# Roteador de Clientes — Hagamorfis

Aplicativo web para montar rotas de visita com múltiplas paradas a partir de
uma lista de clientes em GeoJSON. Feito para uso em Criciúma/SC.

**➡️ Acesse: https://antoniocmp97.github.io/roteador-clientes/**

---

## O que faz

- Carrega os clientes de um arquivo `.geojson` (clique ou arraste o arquivo)
- Mostra todos os pontos no mapa
- Define o ponto de partida por endereço digitado ou pela sua localização
- Permite escolher quais clientes visitar na viagem
- **Traçar nesta ordem** — calcula a rota respeitando a ordem que você definiu
- **Otimizar ordem** — encontra automaticamente a melhor sequência de visitas
- Mostra distância total, tempo estimado e o roteiro passo a passo

## Como usar

1. Abra o link acima (funciona no computador e no celular)
2. Carregue seu arquivo `.geojson` de clientes
3. Digite o endereço de partida ou toque em 📍 para usar sua localização
4. Marque os clientes que vai visitar
5. Clique em **Otimizar ordem** (ou **Traçar nesta ordem**, se já sabe a sequência)

## Formato do arquivo de clientes

GeoJSON do tipo `FeatureCollection`, com pontos. É o formato que o
[uMap](https://umap.openstreetmap.fr/) exporta:

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

Coordenadas na ordem `[longitude, latitude]`. O nome vem de `properties.name`.
Há um arquivo de exemplo em [`Exemplos/exemplo.geojson`](Exemplos/exemplo.geojson),
com pontos fictícios na região de Criciúma apenas para demonstrar o formato.

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
