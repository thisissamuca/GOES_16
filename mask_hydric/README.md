# Manual de Uso - MASK_HIDRIC.ipynb

Este manual explica como usar o algoritmo do notebook `MASK_HIDRIC.ipynb` para gerar uma **mascara hidrica** a partir da banda NIR (`M6C03`) do GOES-16.

## 1) Objetivo do algoritmo

O modulo classifica pixels de rasters NIR em:
- agua/superficie nao vegetada
- vegetacao/solo valido
- sem dado

Fluxo principal:

1. Filtra arquivos `.tif` da banda NIR (`M6C03`).
2. Le os dados de reflectancia.
3. Preserva pixels `NaN` (sem dado).
4. Aplica limiarizacao NIR.
5. Exporta mascara binaria em GeoTIFF `uint8`.

## 2) Principio fisico

Corpos d'agua tendem a baixa reflectancia no NIR.  
Por isso, com limiar padrao `0.08`:

- `reflectancia < 0.08` -> agua / nao vegetado
- `reflectancia >= 0.08` -> vegetacao / solo valido

## 3) Convencao de classes na saida

Valores da mascara gerada:
- `0` -> agua / superficie nao-vegetada (`MASK_WATER`)
- `1` -> vegetacao / solo valido (`MASK_VALID`)
- `255` -> sem dado (`MASK_NODATA`, vindo de `NaN` na entrada)

Importante:
- `NaN` da entrada nao e forçado para 0 ou 1; ele vira `255`.

## 4) Dependencias

Bibliotecas principais:
- `rasterio`
- `numpy`

Opcional:
- `tqdm` (barra de progresso)

Instalacao sugerida:

```bash
pip install rasterio numpy tqdm
```

## 5) Funcao principal

### `batch_reclassify(input_dir, output_dir, threshold=0.08, band_id="M6C03", output_prefix="reclass_")`

Parametros:
- `input_dir`: pasta de entrada com GeoTIFFs
- `output_dir`: pasta de saida
- `threshold`: limiar NIR da classificacao
- `band_id`: identificador da banda no nome do arquivo (filtro case-insensitive)
- `output_prefix`: prefixo dos arquivos de saida

Retorno:
- `int`: quantidade de arquivos processados com sucesso

## 6) Entradas e saidas

Entrada esperada:
- rasters `.tif` em `input_dir` contendo `M6C03` no nome

Saida:
- arquivos em `output_dir` com nome:
  - `reclass_<arquivo_original>.tif` (por padrao)

Metadados de saida:
- `dtype = uint8`
- `nodata = 255`
- `compress = lzw`
- `tiled = True`

## 7) Exemplo de uso rapido

```python
batch_reclassify(
    input_dir="CUT",
    output_dir="MASCARA_HIDRICA",
)
```

Exemplo com parametros customizados:

```python
batch_reclassify(
    input_dir="CUT",
    output_dir="MASCARA_HIDRICA",
    threshold=0.10,
    band_id="M6C03",
    output_prefix="hidric_",
)
```

## 8) Ajuste do limiar

O valor `0.08` e empirico e pode variar conforme:
- sazonalidade
- condicao atmosferica
- area de estudo

Recomendacao:
- testar diferentes limiares (ex.: 0.06, 0.08, 0.10)
- validar visualmente e/ou com referencia externa

## 9) Erros comuns e diagnostico

- **Diretorio de entrada inexistente**: `FileNotFoundError`.
- **Nenhum `.tif` com `M6C03`**: processamento retorna `0` com aviso.
- **Falha em arquivo especifico**: o arquivo aparece no resumo de erros, e o lote continua.

Dicas:
- confirme se os rasters de entrada sao realmente da banda NIR.
- mantenha padrao de nomes para facilitar o filtro por `band_id`.

## 10) Posicao no pipeline

Esse modulo normalmente vem apos o recorte (`CUT`) e antes da aplicacao da mascara hidrica sobre os indices/produtos finais.

