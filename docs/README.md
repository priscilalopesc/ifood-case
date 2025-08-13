# Documentação dos Pipelines

Este diretório contém a documentação de todos os pipelines de dados do sistema de taxi.

## 📋 Lista de Pipelines

### Camada Raw → Trusted
- **[Pipeline Taxi Amarelo](taxi-amarelo-trusted.md)** - Processa arquivos parquet de taxi amarelo da raw zone para trusted zone
- **[Pipeline Taxi Verde](taxi-verde-trusted.md)** - Processa arquivos parquet de taxi verde da raw zone para trusted zone

### Camada Trusted → Refined  
- **[Pipeline Taxi Unificado](taxi-unificado-refined.md)** - Une dados de taxi verde e amarelo em uma única tabela

## 🏗️ Arquitetura dos Dados

```
Raw Zone                    Trusted Zone                 Refined Zone
├── taxi_amarelo/          ├── tb_corrida_taxi_amarelo  ├── tb_corrida_taxi_unificado
│   └── *.parquet    ────→ │                      ────→ │
└── taxi_verde/            └── tb_corrida_taxi_verde ───┘
    └── *.parquet    ────→
```

## 🔄 Ordem de Execução

1. **Primeiro**: Execute os pipelines da trusted zone
   - Pipeline Taxi Amarelo
   - Pipeline Taxi Verde

2. **Depois**: Execute o pipeline da refined zone
   - Pipeline Taxi Unificado (depende dos dois anteriores)

## 📊 Resumo das Tabelas

| Pipeline | Tabela de Origem | Tabela de Destino | Registros |
|----------|------------------|-------------------|-----------|
| Taxi Amarelo | Arquivos parquet | `trusted-zone.tb_corrida_taxi_amarelo` | Todos os períodos |
| Taxi Verde | Arquivos parquet | `trusted-zone.tb_corrida_taxi_verde` | Todos os períodos |
| Taxi Unificado | Trusted tables | `refined-zone.tb_corrida_taxi_unificado` | Jan-Mai 2023 |

## ⚙️ Como Usar

1. **Verifique os pré-requisitos** de cada pipeline na sua documentação
2. **Execute na ordem correta** (trusted → refined)
3. **Monitore os logs** para verificar se não há erros
4. **Valide os dados** nas tabelas de destino

## 🔧 Padrões Utilizados

### Limpeza de Dados
- Conversão de vírgulas para pontos em decimais
- Tratamento de valores inválidos (conversão para NULL)
- Padronização de tipos de dados

### Nomenclatura
- **Colunas**: Português com prefixos (`cod_`, `dt_hr_`, `vlr_`, `qtd_`)
- **Tabelas**: Formato `tb_[entidade]_[complemento]`
- **Zonas**: `raw-zone`, `trusted-zone`, `refined-zone`

### Processamento
- **Modo Overwrite**: Todos os pipelines recriam as tabelas completamente
- **Union**: Combinação de múltiplos arquivos/tabelas
- **Identificação de Origem**: Coluna `origem_taxi` para rastreabilidade

## 📝 Template para Novos Pipelines

Ao criar documentação para novos pipelines, siga a estrutura:
- **O que faz** - Objetivo do pipeline
- **Tabelas** - Entrada e saída
- **Como funciona** - Etapas principais
- **Resultado** - O que é gerado
- **Para executar** - Instruções práticas
- **Observações** - Pontos importantes

## 🆘 Suporte

Para dúvidas sobre os pipelines:
1. Consulte a documentação específica de cada pipeline
2. Verifique os logs de execução do Spark
3. Entre em contato com a equipe de Engenharia de Dados