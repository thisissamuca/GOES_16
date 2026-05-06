# Manual de Uso - GET_FDCF_DATA.ipynb

Este manual descreve como usar o algoritmo do notebook `GET_FDCF_DATA.ipynb` para processar arquivos NetCDF do produto **ABI-L2-FDCF (GOES-16)** e extrair focos de calor com filtro de qualidade.

## 1) Objetivo do algoritmo

O processamento executa, para cada arquivo `.nc`:

1. Recorte espacial na area de estudo (Pantanal) na projecao original.
2. Reprojecao para **SIRGAS 2000 (EPSG:4674)**.
3. Filtragem de focos com:
   - `Mask == 10` (foco confirmado)
   - `DQF == 0` (dado valido/boa qualidade)
4. Exportacao de resultados por arquivo:
   - CSV com os atributos
   - Shapefile de pontos
5. Geracao de um JSON consolidado com metadados da execucao.

## 2) Dependencias

Bibliotecas principais:
- `geopandas`
- `rioxarray`
- `xarray`
- `shapely`
- `numpy`
- `pandas`

Instalacao sugerida:

```bash
pip install geopandas rioxarray xarray shapely numpy pandas
```

## 3) Estrutura e funcoes principais

### `processar_arquivo(arquivo, pasta_saida, bounds=PANTANAL_BOUNDS)`
Processa um unico arquivo NetCDF e retorna:
- `bool`: sucesso (`True`) ou falha (`False`)
- `dict`: metadados do processamento

Etapas internas:
- extracao de data no nome (`_sYYYYDDDHHMMSS`)
- recorte e reprojecao (`_recortar_e_reprojetar`)
- filtro de qualidade (`_filtrar_focos`)
- salvamento de CSV e Shapefile

### `processar_pasta_completa(pasta_entrada, pasta_saida=None, bounds=..., extensoes=None)`
Processa todos os arquivos da pasta de entrada (padrao: extensao `.nc`) e gera:
- arquivos CSV e SHP por cena
- `metadados_execucao.json` com resumo geral

Se `pasta_saida` nao for informada, usa:
- `<pasta_entrada>/resultados`

### Funcoes auxiliares importantes
- `_extrair_data_do_nome`: le data/hora do padrao GOES no nome do arquivo.
- `_filtrar_focos`: aplica mascara `Mask == 10` e `DQF == 0`.
- `_salvar_shapefile`: converte os pontos para GeoDataFrame e grava `.shp`.
- `testar_extracao_data`: utilitario de diagnostico para nomes de arquivos.

## 4) Variaveis utilizadas do produto FDCF

- `Mask`: classificacao do pixel
- `DQF`: qualidade do dado
- `Area`: area estimada do foco (m2)
- `Temp`: temperatura do foco (K)
- `Power`: potencia radiativa do fogo (MW)

Durante o processamento, algumas colunas sao renomeadas:
- `x` -> `lon`
- `y` -> `lat`
- `Area` -> `Area_m2`
- `Temp` -> `Temp_K`

## 5) Area de recorte padrao (Pantanal)

Limites geograficos padrao:
- `xmin = -60.0`
- `xmax = -53.0`
- `ymin = -22.0`
- `ymax = -15.0`

Voce pode alterar esse `bounds` ao chamar as funcoes.

## 6) Exemplo de uso rapido

```python
from pathlib import Path

PASTA_ENTRADA = Path("Dados") / "ABI-L2-FDCF" / "netCDF"
PASTA_SAIDA   = Path("Arquivos") / "FDCF_DATA"

processar_pasta_completa(
    pasta_entrada=PASTA_ENTRADA,
    pasta_saida=PASTA_SAIDA,
)
```

## 7) Saidas geradas

Para cada arquivo processado:
- `dados_filtrados_<data_id>.csv`
- `focos_<data_id>.shp` (e arquivos auxiliares do shapefile)

Ao final da execucao:
- `metadados_execucao.json`

Esse JSON inclui:
- data da execucao
- pastas de entrada e saida
- bounds utilizados
- total de arquivos
- quantidade com sucesso/erro
- metadados detalhados por arquivo

## 8) Como interpretar o filtro de qualidade

Somente pixels que atendem simultaneamente:
- `Mask == 10` (foco de calor confirmado)
- `DQF == 0` (qualidade adequada)

Isso reduz falsos positivos e melhora a confiabilidade dos pontos exportados.

## 9) Problemas comuns e diagnostico

- **Pasta de entrada inexistente**: gera `FileNotFoundError`.
- **Sem arquivos `.nc`**: o algoritmo informa que nao encontrou arquivos.
- **Variaveis ausentes (`Mask`/`DQF`)**: retorna DataFrame vazio para o arquivo.
- **Falhas de ambiente/dependencias** (ex.: erro de versao): revisar versoes das bibliotecas.

Dica:
- Execute primeiro com poucos arquivos para validar o ambiente e o fluxo.

## 10) Fluxo recomendado

1. Garanta que os arquivos FDCF `.nc` estao em uma pasta local.
2. Defina pasta de saida.
3. Rode `processar_pasta_completa`.
4. Verifique o resumo no terminal.
5. Consulte `metadados_execucao.json` para auditoria completa da execucao.

