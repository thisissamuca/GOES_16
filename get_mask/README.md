# Manual de Uso - RECLASSIFIED_MASK.ipynb

Este manual explica como usar o algoritmo do notebook `RECLASSIFIED_MASK.ipynb` para reclassificar mascaras de nuvem do produto **ABI-L2-ACMF (GOES-16)** e exportar os resultados em **GeoTIFF georreferenciado**.

## 1) Objetivo do algoritmo

O pipeline principal faz:

1. Leitura dos arquivos NetCDF do produto ACMF.
2. Reclassificacao da mascara binaria.
3. Conversao para `float32` e redimensionamento.
4. Extracao de georreferenciamento (transformacao afim + CRS).
5. Exportacao para GeoTIFF com compressao.
6. Processamento em lote com paralelismo por chunks.

## 2) Reclassificacao aplicada

Mapa padrao:
- `1 -> 0`
- `0 -> 1`

No contexto descrito no codigo:
- a inversao e intencional para que `1` represente pixel valido (ceu claro), conforme a logica de mascara do fluxo.

Constante usada:
- `DEFAULT_RECLASS_MAP = {1: 0, 0: 1}`

## 3) Dependencias

Bibliotecas principais do algoritmo:
- `affine`
- `numpy`
- `xarray`
- `rasterio`
- `opencv-python`

Instalacao sugerida:

```bash
pip install affine numpy xarray rasterio opencv-python
```

Observacao:
- O notebook tambem cita `cartopy`, `netCDF4` e `satpy`, mas o codigo principal usa leitura com `xarray` e georreferenciamento direto dos atributos do NetCDF.

## 4) Organizacao esperada dos dados

Estrutura de entrada:

`<base_path>/<product>/netCDF/*.nc`

Exemplo:
- `base_path = "Dados"`
- `product = "ABI-L2-ACMF"`
- arquivos em `Dados/ABI-L2-ACMF/netCDF`

Saida:
- GeoTIFFs em `output_dir`
- cada arquivo recebe sufixo `_RECLASSIFIED.tif`

## 5) Componentes principais

### `NetCDFBatchProcessor`
- valida o diretorio base
- lista arquivos `.nc` do produto
- carrega datasets com `xarray`

### `ImageBatchProcessor`
- `reclassify_array`: aplica o dicionario de mapeamento
- `to_float32`: garante tipo numerico compativel com raster
- `resize_image`: redimensiona com `INTER_NEAREST` (preserva classes)

### `GeoTIFFBatchExporter`
- `create_georef`: extrai georreferenciamento a partir da variavel `goes_imager_projection`
- `export_to_geotiff`: grava GeoTIFF com:
  - `dtype=float32`
  - `compress=lzw`
  - `tiled=True`
  - valor `nodata=np.nan`

### `batch_process_files(...)`
- orquestra tudo em lote
- divide arquivos em chunks para controlar memoria
- usa `ThreadPoolExecutor` para processar em paralelo

## 6) Parametros principais de execucao

Na funcao `batch_process_files`:
- `product`: produto GOES (ex.: `"ABI-L2-ACMF"`)
- `output_dir`: pasta dos GeoTIFFs
- `base_path`: raiz dos dados NetCDF
- `reclass_map`: mapa de reclassificacao (padrao invertido)
- `target_shape`: resolucao alvo (padrao `5424 x 5424`)
- `max_workers`: numero de threads paralelas
- `chunk_size`: arquivos por rodada

Dica:
- mantenha `chunk_size >= max_workers` para melhor uso das threads.

## 7) Exemplo de uso rapido

```python
import os
import time

PRODUCT    = "ABI-L2-ACMF"
BASE_PATH  = "Dados"
OUTPUT_DIR = "MASCARA_RECLASSIFICADA"

inicio = time.time()

batch_process_files(
    product=PRODUCT,
    output_dir=OUTPUT_DIR,
    base_path=BASE_PATH,
    max_workers=4,
    chunk_size=16,
)

print(f"Tempo total: {time.time() - inicio:.2f}s")
```

## 8) Saidas geradas

Para cada `.nc`, o algoritmo gera:
- `<nome_original>_RECLASSIFIED.tif`

Os rasters sao salvos com georreferenciamento do proprio arquivo GOES, adequados para uso em SIG (QGIS, ArcGIS, etc.).

## 9) Boas praticas

- Teste primeiro com poucos arquivos.
- Ajuste `max_workers` conforme CPU/RAM.
- Para lotes muito grandes, mantenha `chunk_size` moderado para evitar alto uso de memoria.
- Use `INTER_NEAREST` (ja aplicado) para nao criar classes intermediarias em mascara binaria.

## 10) Erros comuns

- **Diretorio base inexistente**: `FileNotFoundError`.
- **Diretorio do produto inexistente**: caminho `<base_path>/<product>/netCDF` nao encontrado.
- **Sem arquivos `.nc`**: o processamento e interrompido com erro.
- **Falhas de leitura NetCDF**: validar integridade dos arquivos e versoes das bibliotecas.

