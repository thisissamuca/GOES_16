# Manual de Uso - GET_FILE.ipynb

Este manual explica como usar o algoritmo do notebook `GET_FILE.ipynb` para listar, filtrar e baixar arquivos NetCDF do satelite **GOES-16** a partir do bucket publico da NOAA (AWS S3).

## 1) O que o algoritmo faz

O fluxo principal possui 4 etapas:

1. `available_products` - mostra ou retorna os produtos GOES-16 disponiveis.
2. `get_netcdf_data` - lista arquivos no S3 por intervalo de ano/dia juliano/hora.
3. `filter_files` - filtra bandas de interesse (quando o produto tem bandas `M6C##`).
4. `download_files` - baixa os arquivos filtrados para o disco local.

## 2) Requisitos

- Python 3.9+ (recomendado)
- Biblioteca `s3fs`
- Acesso a internet (para ler o bucket publico da NOAA)

Instalacao:

```bash
pip install s3fs
```

## 3) Estrutura de funcoes

### `available_products(operation: str)`
- `operation='codes'`: retorna uma lista com os codigos dos produtos.
- `operation='description'`: imprime tabela com indice, codigo e descricao.

Exemplo:

```python
codes = available_products("codes")
available_products("description")
print(codes[20])  # Exemplo: ABI-L2-CMIPF
```

### `get_netcdf_data(product, year_start, year_end, day_start, day_end, hour)`
Lista arquivos no bucket:

`noaa-goes16/<produto>/<ano>/<dia_juliano>/<hora>/`

Parametros:
- `product`: codigo do produto (ex.: `"ABI-L2-CMIPF"`).
- `year_start`, `year_end`: intervalo de anos (inclusive).
- `day_start`, `day_end`: intervalo de dias julianos (1 a 366).
- `hour`: hora UTC no formato `"HH"` (de `"00"` ate `"23"`).

Retorno:
- `files`: lista de listas com caminhos S3 de arquivos encontrados.
- `errors`: pares `(ano, dia)` com falha de acesso/listagem.

### `filter_files(file_list)`
- Se os nomes dos arquivos contem bandas (`M6C##`), mantem apenas:
  - `M6C01`, `M6C02`, `M6C03`, `M6C05`, `M6C06`
- Se o produto nao tiver bandas no nome, retorna todos os arquivos.

### `download_files(product, file_list, local_base_dir)`
Baixa cada arquivo S3 para:

`<local_base_dir>/<product>/netCDF/<nome_arquivo>.nc`

Retorna a lista de caminhos locais baixados com sucesso.

## 4) Exemplo completo de uso

```python
import s3fs

# 1) Escolher produto
product = available_products("codes")[20]  # ABI-L2-CMIPF

# 2) Listar arquivos no periodo
files, errors = get_netcdf_data(
    product=product,
    year_start=2020,
    year_end=2020,
    day_start=130,
    day_end=240,
    hour="13",
)

print("Dias encontrados:", len(files))
print("Dias com erro:", len(errors))

# 3) Filtrar arquivos
filtered = filter_files(files)
print("Arquivos filtrados:", len(filtered))

# 4) Baixar
downloaded = download_files(product, filtered, local_base_dir="Dados")
print("Arquivos baixados:", len(downloaded))
```

## 5) Como interpretar os dados de tempo

O algoritmo usa **dia juliano**:
- `1` = 1 de janeiro
- `32` = 1 de fevereiro (em ano nao bissexto), etc.

Se voce quer um periodo de datas convencionais (dd/mm/aaaa), converta para dia juliano antes de chamar a funcao.

## 6) Dicas e boas praticas

- Comece com um intervalo pequeno (ex.: 1 ou 2 dias) para validar.
- Verifique `errors` apos a busca para identificar datas sem dados.
- Use um diretorio base dedicado (ex.: `"Dados_GOES16"`).
- Para grandes volumes, monitore espaco em disco.

## 7) Erros comuns

- **Hora invalida**: a funcao aceita somente `"00"` a `"23"`.
- **Produto invalido**: use `available_products("description")` para confirmar o codigo.
- **Sem internet**: o acesso ao bucket publico depende de conectividade.

## 8) Resultado esperado

Ao final, os arquivos `.nc` ficam organizados em pasta local por produto, prontos para processamento posterior (indices espectrais, analises ambientais, etc.).

