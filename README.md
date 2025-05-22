# GOES_16
Algoritmos relacionados à aquisição, processamento e tratamento de dados de imagens do GOES-16.

# Aquisição de dados do GOES-16

O algoritmo acessa diretamente o banco de arquivos NOAA-GOES-16 através do AWS S3 e salva os arquivos em um diretório local.

# Aquisição de máscaras

O Algoritmo atráves do banco de arquivos NOAA-GOES atráves do AWS S3 busca e salva os arquivos do produto ABI-L2-ACMF para ser aplicado nas imagens futuras.

# Aplicação de máscaras

O algoritmo aplica uma máscara híbrida (máscara de nuvens e hídrica) nas imagens com objetivo de remover as nuvens e os corpos hídricos da imagem.

## Máscara de nuvens

O algoritmo recebe do diretório local os arquivos do produto ABI-L2-ACMF já adquiridos e faz uma reclassificação.

## Máscara hídrica

O algoritmo recebe do diretório local os arquivos do produto ABI-L2-CMIPF_M6C03, multiplica pela máscara de nuvens e transforma os valores menores que 0.15 em NaN.

# Processamento de imagens através dos índices espectrais

O algoritmo executa uma morfologia matemática através de índices espectrais para obter um produto que será alvo de pesquisas atuais e futuras.

<div align="center">

## NDVI - Normalized Difference Vegetation Index
```math
(NIR - R) / (NIR + R)
```
  
## NBR - Normalized Burn Ratio
```math
(NIR - SWIR2) / (NIR + SWIR2)
```

## NBR2 - Variation of Normalized Burn Ratio
```math
(SWIR1 - SWIR2) / (SWIR1 + SWIR2)
```

## NDMI - Normalized Difference Moisture Index
```math
(NIR - SWIR1) / (NIR + SWIR1)
```

## MIRBI - Mid-Infrared Burn INdex 
```math
((10*SWIR2) - (9.8*SWIR1) + 2)
```

## EVI - Enhanced Vegetation Index
```math
((NIR - R) / (NIR + (6*R) - (7.5*B) + 1)
```

## BAI - Burned Area Index

```math
1 / ((0.1 - R)²+(0.06 - NIR)²)
```

## SAVI - Soil Adjusted Vegetation Index 

```math
(2*[(NIR - R)/(NIR + R + 1))
```
</div>
