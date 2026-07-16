# Manual de Uso - SPECTRAL_INDEX.ipynb

Este manual explica como usar o algoritmo do notebook `SPECTRAL_INDEX.ipynb` para calcular indices espectrais a partir de bandas GOES-16/ABI.

## 1) Objetivo do algoritmo

O modulo:

1. Varre um diretorio com rasters de bandas.
2. Agrupa arquivos por chave temporal GOES-16 (`_sYYYYDDDHHMMSS`).
3. Valida se o grupo possui todas as bandas necessarias.
4. Calcula indices espectrais.
5. Exporta os resultados em GeoTIFF organizados por indice e tempo.

## 2) Indices implementados

Indices calculados:
- `NDVI`
- `NBR`
- `NBR2`
- `NDMI`
- `EVI`
- `SAVI`

## 3) Bandas utilizadas (ABI)

Mapeamento de bandas no nome dos arquivos:
- `M6C01` -> `blue`
- `M6C02` -> `red`
- `M6C03` -> `nir`
- `M6C05` -> `swir`
- `M6C06` -> `swir2`

Um grupo temporal so e processado se tiver as 5 bandas obrigatorias.

## 4) Dependencias

Bibliotecas principais:
- `rasterio`
- `numpy`
- `inspect` (biblioteca padrao Python)

Opcional:
- `tqdm` (progresso visual)

Instalacao sugerida:

```bash
pip install rasterio numpy tqdm
```

## 5) Formula dos indices

- `NDVI = (NIR - RED) / (NIR + RED)`
- `NBR = (NIR - SWIR2) / (NIR + SWIR2)`
- `NBR2 = (SWIR - SWIR2) / (SWIR + SWIR2)`
- `NDMI = (NIR - SWIR) / (NIR + SWIR)`
- `EVI = (NIR - RED) / (NIR + 6*RED - 7.5*BLUE + 1)`
- `SAVI = ((NIR - RED)/(NIR + RED + L))*(1+L)` com `L=0.5`

Observacao:
- o modulo usa divisao segura e retorna `NaN` quando denominador e zero.

## 6) Estrutura de saida

Os arquivos sao gravados em:

`<output_base>/<INDICE>/<ano>/<dia>/<hora_HH>/<horaHHMMSS>.tif`

Exemplo:
- `Arquivos/NDVI/2020/150/13/1300166.tif`

## 7) Funcoes principais

### `encontrar_matches_por_data_hora(diretorio_base)`
- percorre recursivamente os arquivos
- monta grupos por chave temporal
- filtra apenas grupos completos de bandas

### `processar_indice(matches_dict, nome_indice, func_calculo, output_base_dir)`
- identifica automaticamente quais bandas a funcao precisa
- le bandas com validacao de alinhamento geometrico
- calcula o indice
- exporta GeoTIFF

### `INDICES`
Dicionario com os indices disponiveis para iteracao automatica no pipeline.

## 8) Validacoes importantes

Antes de calcular:
- verifica existencia do diretorio base
- valida se o grupo temporal possui todas as bandas
- valida se bandas do grupo tem mesmo `shape` e `transform`

Se houver divergencia, a chave temporal e ignorada com erro no log.

## 9) Exemplo de uso rapido

```python
from pathlib import Path

DIRETORIO_BASE = Path("Final")
OUTPUT_BASE    = Path("Arquivos")

matches = encontrar_matches_por_data_hora(str(DIRETORIO_BASE))
print(f"Grupos completos encontrados: {len(matches)}")

for nome, func in INDICES.items():
    processar_indice(matches, nome, func, str(OUTPUT_BASE))
```

## 10) Metadados de saida

Cada GeoTIFF e salvo com:
- `dtype = float32`
- `count = 1`
- `compress = lzw`
- `tiled = True`
- `nodata = NaN`

## 11) Erros comuns e diagnostico

- **Diretorio base inexistente**: `FileNotFoundError`.
- **Grupos incompletos**: faltam bandas em determinada chave temporal.
- **Desalinhamento espacial**: bandas com resolucao/transform diferente.
- **Chave temporal invalida**: arquivo sem padrao `_sYYYYDDDHHMMSS`.

Dicas:
- garanta padrao de nomenclatura GOES-16.
- mantenha todos os rasters no mesmo grid antes do calculo.

## 12) Posicao no pipeline

Esse modulo costuma rodar apos a geracao dos rasters finais mascarados, para produzir os produtos analiticos de vegetacao, umidade e queimadas.

