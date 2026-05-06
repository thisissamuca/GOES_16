# Manual de Uso - MAXIMUM_VALUE_COMPOSITION.ipynb

Este manual explica como usar o algoritmo do notebook `MAXIMUM_VALUE_COMPOSITION.ipynb` para gerar **MVC (Maximum Value Composite)** dos indices espectrais.

## 1) Objetivo do algoritmo

O modulo calcula composicoes de valor maximo em dois niveis:

- **MVC diario**: maximo de todas as cenas disponiveis no dia
- **MVC horario**: maximo das cenas dentro de cada hora

Ele trabalha sobre os rasters de indices gerados anteriormente (ex.: NDVI, NBR, EVI etc.).

## 2) Estrutura de entrada esperada

Entrada (vinda do `SPECTRAL_INDEX.ipynb`):

`<input_base>/<INDICE>/<ano>/<dia>/<hora_HH>/<horaHHMMSS>.tif`

Exemplo:
- `Arquivos/NDVI/2020/150/13/1300166.tif`

## 3) Estrutura de saida

Saidas geradas:

- **MVC diario**  
  `<output_base>/MVC_<INDICE>/<ano>/<dia>/MVC_<INDICE>_<ano><dia>.tif`

- **MVC horario**  
  `<output_base>/MVC_<INDICE>/<ano>/<dia>/<hora_HH>/MVC_<INDICE>_<ano><dia><hora_HH>.tif`

## 4) Dependencias

Bibliotecas principais:
- `numpy`
- `rasterio`

Opcional:
- `tqdm` (barra de progresso)

Instalacao sugerida:

```bash
pip install numpy rasterio tqdm
```

## 5) Indices processados

Por padrao, o script percorre:
- `NDVI`
- `NBR`
- `NBR2`
- `NDMI`
- `MIRBI`
- `EVI`
- `SAVI`

## 6) Funcao central

### `calcular_mvc(file_list, output_path)`

O que ela faz:
1. Usa o primeiro raster como referencia geometrica (CRS, transform, dimensoes).
2. Reprojeta os demais rasters para essa mesma grade (`Resampling.nearest`).
3. Empilha os arrays e calcula `np.nanmax` (ignora `NaN`).
4. Salva GeoTIFF final.

Comportamentos importantes:
- se todos os pixels de uma posicao forem `NaN`, usa `nodata` do perfil de referencia (ou fallback).
- compressao `LZW` e `tiled=True`.

## 7) Funcao de orquestracao

### `processar_mvc_indice(indice, input_base, output_base, calcular_diario=True, calcular_horario=True)`

Responsabilidades:
- percorre anos, dias e horas do indice informado
- gera MVC diario (recursivo no dia)
- gera MVC horario (arquivos diretos por pasta de hora)
- registra resumo de sucessos e erros

## 8) Exemplo de uso rapido

```python
from pathlib import Path

INPUT_BASE  = Path("Arquivos")
OUTPUT_BASE = Path("Arquivos")

for indice in ["NDVI", "NBR", "NBR2", "NDMI", "MIRBI", "EVI", "SAVI"]:
    processar_mvc_indice(
        indice=indice,
        input_base=str(INPUT_BASE),
        output_base=str(OUTPUT_BASE),
        calcular_diario=True,
        calcular_horario=True,
    )
```

## 9) Metadados de saida

Cada GeoTIFF MVC e salvo com:
- `dtype = float32`
- `count = 1`
- `nodata` herdado da referencia (ou fallback)
- `compress = lzw`
- `tiled = True`

## 10) Erros comuns e diagnostico

- **Diretorio do indice nao encontrado**: indice sem dados na entrada.
- **Sem anos/dias/horas**: estrutura de pastas incompleta.
- **Sem `.tif` no dia/hora**: composicao daquele grupo e ignorada com aviso.
- **Falha de leitura/reprojecao**: erro registrado no resumo do indice.

Dicas:
- rode primeiro para um indice (ex.: NDVI) e valide saida.
- confirme que os rasters de entrada estao consistentes e legiveis.

## 11) Posicao no pipeline

Esse modulo normalmente vem depois de `SPECTRAL_INDEX`, consolidando os indices em produtos compostos diarios/horarios para analise temporal.

