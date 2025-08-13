# Documentação - Pipeline Taxi Amarelo (Trusted Zone)

## O que faz
Este pipeline processa arquivos parquet de corridas de taxi amarelo, limpa e padroniza os dados, salvando na trusted zone.

## Tabelas
- **Entrada**: Arquivos parquet em `/Volumes/workspace/raw-zone/taxi_amarelo/`
- **Saída**: `trusted-zone.tb_corrida_taxi_amarelo`

## Como funciona

### 1. Lê arquivos parquet
```python
path = "/Volumes/workspace/raw-zone/taxi_amarelo/"
arquivos = dbutils.fs.ls(path)
```
- Processa todos os arquivos `.parquet` da pasta
- Cada arquivo é processado individualmente

### 2. Padroniza nomes das colunas
Renomeia colunas do formato original para nomes mais claros:

| Coluna Original | Nova Coluna | O que é |
|-----------------|-------------|---------|
| `vendorid` | `cod_motorista` | Código do motorista |
| `tpep_pickup_datetime` | `dt_hr_inicio` | Data/hora início da corrida |
| `tpep_dropoff_datetime` | `dt_hr_fim` | Data/hora fim da corrida |
| `passenger_count` | `qtd_pessoas` | Quantidade de passageiros |
| `trip_distance` | `dist_percorrida` | Distância percorrida |
| `total_amount` | `vlr_total` | Valor total da corrida |
| `fare_amount` | `vlr_taxa_corrida` | Valor da taxa da corrida |
| `tip_amount` | `vlr_troco` | Valor do troco/gorjeta |

### 3. Limpa os dados
- **Converte tudo para string** primeiro para facilitar limpeza
- **Substitui vírgulas por pontos** nos valores decimais
- **Remove caracteres inválidos** usando regex

### 4. Converte tipos de dados
Define tipos específicos para cada coluna:
- **Datas**: `TimestampType`
- **Números inteiros**: `IntegerType` 
- **Valores monetários**: `DecimalType(10, 2)` - até 10 dígitos com 2 casas decimais
- **Textos**: `StringType`

### 5. Tratamento de erros
- **Valores inválidos** viram `NULL`
- **Números decimais** são arredondados para 2 casas
- **Inteiros** aceitam formato "1.0" (converte via double primeiro)

### 6. Adiciona identificação
```python
df = df.withColumn("origem_taxi", lit("taxi_amarelo"))
```
Adiciona coluna para identificar que os dados são do taxi amarelo.

### 7. Une todos os arquivos
- Processa cada arquivo parquet separadamente
- Une todos usando `unionByName`
- Salva tudo em uma única tabela

## Resultado
Uma tabela limpa e padronizada com todas as corridas de taxi amarelo de todos os arquivos parquet processados.

## Colunas da tabela final
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `cod_motorista` | String | Código do motorista |
| `dt_hr_inicio` | Timestamp | Data e hora de início |
| `dt_hr_fim` | Timestamp | Data e hora de fim |
| `qtd_pessoas` | Integer | Quantidade de passageiros |
| `dist_percorrida` | Decimal(10,2) | Distância da corrida |
| `vlr_total` | Decimal(10,2) | Valor total |
| `vlr_taxa_corrida` | Decimal(10,2) | Taxa da corrida |
| `vlr_troco` | Decimal(10,2) | Troco/gorjeta |
| `origem_taxi` | String | Sempre "taxi_amarelo" |

## Para executar
1. Certifique-se que existem arquivos `.parquet` em `/Volumes/workspace/raw-zone/taxi_amarelo/`
2. Execute o script Python
3. A tabela será recriada (modo overwrite) na trusted zone

## Observações
- **Reprocessa todos os dados** a cada execução (drop table + overwrite)
- **Aceita schemas diferentes** entre arquivos (mergeSchema = true)
- **Trata dados sujos** convertendo valores inválidos para NULL
- **Mantém histórico completo** de todas as corridas processadas