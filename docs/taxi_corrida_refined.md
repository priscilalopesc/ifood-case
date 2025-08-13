
# Documentação - Construção da Tabela Unificada Refined

## Objetivo
Construir a tabela `refined-zone.tb_corrida_taxi_unificado` a partir das tabelas processadas na camada *trusted* (`tb_corrida_taxi_verde` e `tb_corrida_taxi_amarelo`), mantendo apenas colunas relevantes e filtrando os registros entre janeiro e maio de 2023.

---

## Passos do Código

1. **Importa funções necessárias**
   ```python
   from pyspark.sql.functions import col
   ```
   - `col`: Usada para referenciar colunas no DataFrame de forma segura.

2. **Lê as tabelas da camada *trusted***
   ```python
   df_verde = spark.read.table("`trusted-zone`.tb_corrida_taxi_verde")
   df_amarelo = spark.read.table("`trusted-zone`.tb_corrida_taxi_amarelo")
   ```
   - Carrega os dados processados previamente, um para cada tipo de táxi.

3. **Define colunas que serão mantidas**
   ```python
   colunas_selecionadas = [
       "cod_motorista",
       "dt_hr_inicio",
       "dt_hr_fim",
       "qtd_pessoas",
       "vlr_total",
       "origem_taxi"
   ]
   ```
   - Apenas campos essenciais para análise e integração.

4. **Define intervalo de datas**
   ```python
   data_inicio = "2023-01-01T00:00:00.000+00:00"
   data_fim = "2023-05-31T23:59:59.000+00:00"
   ```
   - Limita a extração aos meses de **janeiro a maio de 2023**.

5. **Filtra e seleciona colunas no táxi verde**
   ```python
   df_verde_sel = (
       df_verde
       .filter((col("dt_hr_inicio") >= data_inicio) & (col("dt_hr_inicio") <= data_fim))
       .select(colunas_selecionadas)
   )
   ```
   - Mantém apenas registros no período definido e seleciona as colunas de interesse.

6. **Filtra e seleciona colunas no táxi amarelo**
   ```python
   df_amarelo_sel = (
       df_amarelo
       .filter((col("dt_hr_inicio") >= data_inicio) & (col("dt_hr_inicio") <= data_fim))
       .select(colunas_selecionadas)
   )
   ```
   - Mesma lógica aplicada ao dataset do táxi amarelo.

7. **Une os dois datasets**
   ```python
   df_union = df_verde_sel.unionByName(df_amarelo_sel)
   ```
   - Junta os dados do táxi verde e amarelo mantendo o mesmo nome de colunas (`unionByName`).

8. **Salva na camada *refined***
   ```python
   df_union.write.mode("overwrite").saveAsTable("`refined-zone`.tb_corrida_taxi_unificado")
   ```
   - Salva como tabela no *Hive Metastore*, sobrescrevendo se já existir.

---

## Resultado Final
Tabela: **`refined-zone.tb_corrida_taxi_unificado`**

Contendo:
- Apenas corridas entre **01/01/2023 e 31/05/2023**
- Campos: `cod_motorista`, `dt_hr_inicio`, `dt_hr_fim`, `qtd_pessoas`, `vlr_total`, `origem_taxi`
- Dados combinados de táxi verde e táxi amarelo.
