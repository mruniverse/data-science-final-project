# Projeto Final — Ciência de Dados: Análise de Dados Aplicada

**Prof. Dr. Eduardo Pena**

---

## Identificação

| Campo         | Informação              |
| ------------- | ----------------------- |
| **Aluno**     | Yuri Lucas Luz da Silva |
| **Matrícula** | 2223481                 |

---

## Etapa 2 — Integração e Limpeza de Dados

Esta etapa está implementada na **Seção 2 do notebook** [`notebook.ipynb`](../notebook.ipynb),
que é a fonte executável e documentada deste processo. Este documento resume as decisões tomadas.

---

## 1. Objetivo

Enriquecer o dataset principal de sinistros com uma fonte externa (o parque semafórico de
Fortaleza), resolvendo os problemas de qualidade dos dados e produzindo um **dataset final limpo**,
pronto para a análise exploratória e a modelagem das etapas seguintes.

---

## 2. Integração com Fonte Externa

| Dataset               | Papel                  | Fonte                         | Registros |
| --------------------- | ---------------------- | ----------------------------- | --------- |
| Sinistros de Trânsito | Principal              | AMC / Dados Abertos Fortaleza | 130.823   |
| Parque Semafórico     | Enriquecimento externo | CTAFOR/AMC                    | 1.231     |

A integração é **espacial** e **temporal**:

- **Reprojeção métrica.** Os dois arquivos vêm em **EPSG:4674 (SIRGAS 2000, em graus)** — informação
  declarada no próprio GeoJSON e confirmada no notebook via `.crs`. Para medir distâncias em metros,
  reprojetei as duas camadas para **EPSG:31984 (SIRGAS 2000 / UTM 24S)**, o sistema métrico adequado
  ao fuso de Fortaleza.
- **Reconstrução temporal por ano (tratamento da defasagem).** O parque semafórico reflete o estado
  *atual*, mas os sinistros cobrem 2015–2024. Para cada acidente, considerei apenas os semáforos
  **provavelmente ativos no ano da ocorrência**, usando os campos `DATA_IMPLANTAÇÃO`,
  `DATA_DESATIVAÇÃO`, `DATA_REATIVAÇÃO` e `STATUS`. Efeito real: o nº de semáforos ativos vai de
  **754 (2015)** a **1.106 (2024)**.
  - *Pressupostos:* implantação ausente = já existia; desativação ausente = nunca desativado;
    `STATUS = "PROJETO"` = nunca existiu (excluído); `"DESATIVADO"` sem data = removido (inativo).
- **Variáveis criadas** para cada sinistro (matching geográfico por proximidade):
  - `dist_semaforo_m` — distância ao semáforo ativo mais próximo (via `sjoin_nearest`);
  - `densidade_semaforos_100m` / `densidade_semaforos_250m` — nº de semáforos ativos no raio
    (índice espacial STRtree, `sindex.query(predicate='dwithin')`, sem dependência de `scipy`);
  - `semaforo_proximo_status`, `semaforo_proximo_modo`, `semaforo_proximo_estagios`,
    `semaforo_proximo_codigo` — atributos do semáforo mais próximo;
  - `tem_semaforo_interseccao` — indicador (1 se o semáforo mais próximo está a ≤ 30 m).

---

## 3. Pipeline de Limpeza (reprodutível)

| Variável / item                          | Problema                          | Decisão                                              |
| ---------------------------------------- | --------------------------------- | ---------------------------------------------------- |
| `TIPOCRUZAMENTO`, `CONTROLETRAFEGO`, `NATUREZA` | Variações de caixa/acento (ex.: *Meio De Quadra* / *Meio de quadra*) | Padronização por mapeamento canônico explícito |
| `"Não informado"`                        | Ausência registrada como texto    | Mantido como categoria explícita                     |
| `HORA`                                   | Descartada pelo `read_file` (tipo OGR não suportado) | Recuperada via `json.load` das *properties* |
| `data_hora`                              | Não existia                       | Construída de `DIA/MES/ANO + HORA` (datas inválidas → `NaT`) |
| `NUMERO`                                 | ~50% ausente, baixo valor preditivo | Coluna removida                                    |
| Semáforos — datas                        | Formato `DD/MM/AAAA`              | `to_datetime(dayfirst=True)`                         |
| Semáforos — `MODO_CONTROLE` (11,5% nulo) | Ausência                          | Imputação por **constante** (`"Desconhecido"`)       |
| `semaforo_proximo_estagios` (~8% ausente) | Ausência (estágios desconhecidos) | **Imputação pela moda da rua** (`LOG1`) + *fallback* moda global; flag `estagios_imputado` |
| `dist_semaforo_m` (outliers)             | Distâncias extremas (até ~6.675 m) | Detectados por IQR; **mantidos** (são reais) + versão winsorizada `dist_semaforo_m_cap` (p99 ≈ 2.088 m) |

> **Imputação de `semaforo_proximo_estagios`:** dos 10.609 ausentes, 6.159 foram preenchidos pela
> moda da rua e 4.450 pela moda global (ruas sem nenhum valor conhecido). A flag `estagios_imputado`
> permite, na Etapa 3, testar as hipóteses usando **apenas os valores conhecidos**.

O pipeline é **determinístico e idempotente**: mapeamentos fixos, sem amostragem aleatória nas
transformações; re-executar produz o mesmo resultado.

---

## 4. Dataset Final

- **Saída:** `data/sinistros_semaforos_integrado.csv` (CSV).
- **Dimensões:** 130.823 linhas × 37 colunas (28 originais − `NUMERO` + `data_hora` + 7 variáveis de
  integração + `estagios_imputado` + `dist_semaforo_m_cap`).
- **Distância ao semáforo mais próximo:** mediana ≈ **147 m** (25% a ≤ 56 m), confirmando que a
  malha semafórica concentra-se nas vias principais.

---

## 5. Reprodutibilidade

- Carregamento dos GeoJSON, recuperação da `HORA`, limpeza, integração e exportação estão todos no
  notebook, em blocos numerados e comentados.
- Bibliotecas: `geopandas`, `pandas`, `numpy`, `shapely`, `pyproj` (leitura via `pyogrio`).
- O notebook baixa os dados automaticamente caso não existam localmente.

---

_Documento referente à Etapa 2 do Projeto Final — Ciência de Dados: Análise de Dados Aplicada._
