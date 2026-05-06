# Manual de Uso - APPLIED_MASK_HIDRIC.ipynb

Este manual explica como usar o algoritmo do notebook `APPLIED_MASK_HIDRIC.ipynb` para aplicar a **mascara hidrica** sobre os rasters recortados e gerar o produto final.

## 1) Objetivo do algoritmo

O modulo combina:
- mascaras hidricas (geradas por `MASK_HIDRIC.ipynb`)
- cenas recortadas (geradas por `CUT.ipynb`)

E produz rasters finais onde:
- agua e nodata da mascara viram `NaN`
- pixels validos permanecem com os valores da cena

## 2) Fluxo principal

1. Pareamento automatico entre arquivos de mascara e cena pelo identificador temporal GOES-16.
2. Leitura da mascara hidrica (`0`, `1`, `255`).
3. Conversao dos valores invalidos (`0` e `255`) para `NaN`.
4. Multiplicacao elementar `scene * mask`.
5. Exportacao em GeoTIFF `float32` com compressao `LZW`.

## 3) Convencao da mascara hidrica

Valores esperados:
- `0` -> agua / nao-vegetado -> `NaN` no resultado
- `1` -> vegetacao / solo valido -> mantido
- `255` -> sem dado (`nodata`) -> `NaN` no resultado

Constante usada:
- `MASK_INVALID_VALUES = (0, 255)`

## 4) Dependencias

Bibliotecas principais:
- `numpy`
- `rasterio`

Instalacao sugerida:

```bash
pip install numpy rasterio
```

## 5) Pareamento temporal

O pareamento usa regex no nome dos arquivos:
- padrao: `G16_s<CHAVE>_e`

A funcao `match_files(mask_files, scene_files)` retorna pares:
- `(mask_path, scene_path)`

Somente arquivos com a mesma chave temporal sao processados.

## 6) Funcoes principais

### `process_and_export(mask_path, scene_path, output_dir)`
Processa um par mascara+cena:
- valida existencia dos arquivos
- le mascara e cena
- aplica mascara hidrica
- escreve GeoTIFF de saida

### `batch_process(mask_files, scene_files, output_dir, start_index=0, verbose=True)`
Executa processamento em lote:
- encontra todos os pares correspondentes
- permite retomada por indice (`start_index`)
- imprime progresso por arquivo
- retorna total de sucessos

## 7) Nome dos arquivos de saida

A funcao `_build_output_filename`:
- usa nome da cena como base
- remove prefixo `clip_` se existir
- preserva timestamp para evitar sobrescrita entre datas

Exemplo:
- entrada: `clip_ABI-L2-CMIPF_... .tif`
- saida: `ABI-L2-CMIPF_... .tif`

## 8) Exemplo de uso rapido

```python
from pathlib import Path

MASK_DIR   = Path("MASCARA_HIDRICA")
SCENE_DIR  = Path("CUT")
OUTPUT_DIR = Path("FINAL")

mask_files = sorted(str(f) for f in MASK_DIR.iterdir() if f.is_file() and f.suffix.lower() == ".tif")
scene_files = sorted(str(f) for f in SCENE_DIR.iterdir() if f.is_file() and f.suffix.lower() == ".tif")

batch_process(
    mask_files=mask_files,
    scene_files=scene_files,
    output_dir=str(OUTPUT_DIR),
    start_index=0,
)
```

## 9) Metadados da saida

GeoTIFF final com:
- `driver = GTiff`
- `count = 1`
- `dtype = float32`
- `nodata = NaN`
- `compress = lzw`
- `tiled = True`

O georreferenciamento e herdado do raster de cena.

## 10) Erros comuns e diagnostico

- **Nenhum par correspondente**: nomes sem chave temporal compativel.
- **Diretorio/arquivo ausente**: `FileNotFoundError`.
- **Listas vazias**: sem arquivos `.tif` em uma ou ambas as pastas.

Dicas:
- mantenha padrao de nomes GOES-16.
- use `start_index` para retomar processamentos interrompidos.
- valide visualmente alguns outputs da pasta `FINAL`.

## 11) Posicao no pipeline

Esse modulo costuma ser a etapa final da cadeia:
1. `RECLASSIFIED_MASK` / `APPLIED_MASK`
2. `CUT`
3. `MASK_HIDRIC`
4. `APPLIED_MASK_HIDRIC` (saida final)

