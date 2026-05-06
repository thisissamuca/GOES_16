# Manual de Uso - APPLIED_MASK.ipynb

Este manual explica como usar o algoritmo do notebook `APPLIED_MASK.ipynb` para aplicar mascaras de nuvem reclassificadas sobre cenas do produto **ABI-L2-CMIPF (GOES-16)** e exportar os resultados em **GeoTIFF**.

## 1) Objetivo do algoritmo

O pipeline faz:

1. Pareamento automatico entre arquivos de mascara (`.tif`) e cenas (`.nc`) pelo identificador temporal GOES-16.
2. Leitura da mascara reclassificada (onde `0` vira `NaN`).
3. Leitura da variavel principal da cena NetCDF.
4. Redimensionamento da cena para a grade da mascara.
5. Aplicacao da mascara por multiplicacao elementar.
6. Exportacao georreferenciada em GeoTIFF (`float32`, `LZW`, `tiled`).

## 2) Convencao da mascara

No algoritmo:
- `1` = pixel valido (ceu claro), mantido no resultado
- `0` = pixel invalido (nuvem), convertido para `NaN`

Assim, ao multiplicar `scene * mask`:
- areas com nuvem viram `NaN`
- areas validas preservam o valor da cena

## 3) Dependencias

Bibliotecas principais:
- `numpy`
- `rasterio`
- `xarray`
- `opencv-python`

Instalacao sugerida:

```bash
pip install numpy rasterio xarray opencv-python
```

## 4) Estrutura esperada de entradas

O exemplo do notebook usa:
- mascaras em `MASCARA_RECLASSIFICADA` (`.tif`)
- cenas em `Dados/ABI-L2-CMIPF/netCDF` (`.nc`)

Saida:
- `MASCARA_APLICADA` com os GeoTIFFs finais

## 5) Funcoes principais

### `match_files(mask_files, scene_files)`
Emparelha mascara e cena com base na chave temporal extraida por regex:
- padrao: `G16_s<CHAVE>_e`

Retorna lista de pares:
- `(mask_path, scene_path)`

### `process_and_export(mask_path, scene_path, output_dir, interpolation=...)`
Processa um par mascara+cena:
- le mascara e converte `0 -> NaN`
- le cena NetCDF
- redimensiona para o shape da mascara
- aplica mascara
- exporta GeoTIFF mantendo o perfil geoespacial da mascara

### `batch_process(mask_files, scene_files, output_dir, start_index=0, ...)`
Executa em lote:
- faz pareamento automatico
- permite retomar por indice (`start_index`)
- imprime progresso e contabiliza sucessos

## 6) Parametros importantes

Na funcao `batch_process`:
- `mask_files`: lista de caminhos de mascaras `.tif`
- `scene_files`: lista de caminhos de cenas `.nc`
- `output_dir`: pasta de saida
- `start_index`: indice para retomar processamento
- `interpolation`: metodo do OpenCV no resize (padrao `INTER_LINEAR`)
- `verbose`: exibir ou nao logs

Observacao:
- `INTER_LINEAR` e adequado para dados continuos (radiancia/reflectancia).
- Para dados categoricos, prefira `INTER_NEAREST`.

## 7) Nome dos arquivos de saida

O nome final vem da cena NetCDF:
- remove prefixo `OR_` e extensao `.nc`
- gera `<nome_base>.tif`
- se o nome estiver fora do padrao GOES, usa fallback com aviso

## 8) Exemplo de uso rapido

```python
from pathlib import Path

MASK_DIR   = Path("MASCARA_RECLASSIFICADA")
SCENE_DIR  = Path("Dados") / "ABI-L2-CMIPF" / "netCDF"
OUTPUT_DIR = Path("MASCARA_APLICADA")

mask_files = sorted(str(f) for f in MASK_DIR.iterdir() if f.is_file() and f.suffix == ".tif")
scene_files = sorted(str(f) for f in SCENE_DIR.iterdir() if f.is_file() and f.suffix == ".nc")

if not mask_files or not scene_files:
    print("Nenhum arquivo encontrado.")
else:
    batch_process(
        mask_files=mask_files,
        scene_files=scene_files,
        output_dir=str(OUTPUT_DIR),
        start_index=0,
    )
```

## 9) Saidas geradas

Para cada par correspondente mascara+cena:
- um GeoTIFF com mascara aplicada em `output_dir`

Cada raster de saida:
- mantem georreferenciamento da mascara
- usa `float32`
- usa compressao `LZW`
- marca pixels invalidos com `NaN` (`nodata`)

## 10) Problemas comuns e diagnostico

- **Nenhum par encontrado**: nomes de mascara/cena sem chave temporal compativel.
- **Arquivo nao encontrado**: caminho de entrada incorreto.
- **Diretorios vazios**: sem `.tif` ou sem `.nc`.
- **Erro de leitura da variavel da cena**: validar integridade do NetCDF e variaveis disponiveis.

Dicas:
- valide primeiro alguns pares com `start_index` pequeno.
- mantenha listas ordenadas para reproducibilidade.

