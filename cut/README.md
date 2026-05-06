# Manual de Uso - CUT.ipynb

Este manual explica como usar o algoritmo do notebook `CUT.ipynb` para recortar, em lote, rasters GeoTIFF com base em um shapefile da area de estudo (ex.: limite do Pantanal).

## 1) Objetivo do algoritmo

O pipeline realiza:

1. Leitura dos GeoTIFFs de entrada.
2. Carregamento do shapefile de limite.
3. Reprojecao do shapefile para o CRS dos rasters (se necessario).
4. Recorte espacial (`clip`) de cada raster pela geometria do shapefile.
5. Conversao para `float32` e substituicao de `nodata` original por `NaN`.
6. Exportacao dos rasters recortados em GeoTIFF com compressao `LZW`.

## 2) Dependencias

Bibliotecas principais:
- `rasterio`
- `geopandas`
- `numpy`

Biblioteca opcional:
- `tqdm` (barra de progresso; se nao existir, o codigo usa fallback simples)

Instalacao sugerida:

```bash
pip install rasterio geopandas numpy tqdm
```

## 3) Entradas e saidas

Entradas:
- `input_dir`: pasta com arquivos `.tif`
- `shapefile_path`: caminho do shapefile de recorte

Saida:
- `output_dir`: pasta dos rasters recortados
- nome de saida por padrao: `clip_<nome_original>.tif`

## 4) Funcao principal

### `process_raster_batch(input_dir, shapefile_path, output_dir, output_prefix="clip_")`

Parametros:
- `input_dir`: diretorio com os GeoTIFFs de entrada
- `shapefile_path`: shapefile com o limite da area de estudo
- `output_dir`: diretorio de saida
- `output_prefix`: prefixo para os arquivos recortados (padrao `clip_`)

Retorno:
- `int`: numero de arquivos processados com sucesso

Comportamento importante:
- se nao houver `.tif`, a funcao retorna `0`
- em caso de erro em um arquivo, o processamento continua para os demais

## 5) Logica de CRS (projecao)

O algoritmo:
- le o CRS do primeiro raster da pasta
- carrega o shapefile
- reprojeta o shapefile para esse CRS, se necessario

Isso evita erro de sobreposicao entre geometrias e rasters em sistemas de referencia diferentes.

## 6) Tratamento de nodata

Regra aplicada:
- apenas o valor `nodata` declarado no raster e trocado por `NaN`
- valores `0` validos de dado nao sao alterados automaticamente

Isso e importante para preservar informacao real da cena.

## 7) Exemplo de uso rapido

```python
import os

process_raster_batch(
    input_dir="MASCARA_APLICADA",
    shapefile_path=os.path.join("Limite_Pantanal", "biome_border.shp"),
    output_dir="CUT",
)
```

## 8) Metadados de saida

Cada raster de saida e salvo com:
- `driver = GTiff`
- `dtype = float32`
- `nodata = NaN`
- `compress = lzw`
- `tiled = True`
- `transform`, `height` e `width` atualizados para o recorte

## 9) Erros comuns e diagnostico

- **Diretorio de entrada inexistente**: gera `FileNotFoundError`.
- **Shapefile inexistente**: gera `FileNotFoundError`.
- **Nenhum `.tif` na pasta**: processamento encerra com aviso.
- **Falha em arquivo especifico**: arquivo e listado no resumo final de erros.

Dicas:
- valide primeiro com poucos arquivos.
- confira se o shapefile representa exatamente a area de estudo desejada.

## 10) Fluxo recomendado no pipeline

Este modulo normalmente vem depois da aplicacao de mascara (`MASCARA_APLICADA`) para produzir produtos finais recortados na area de interesse.

