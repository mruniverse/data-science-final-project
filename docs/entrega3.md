# Projeto Final — Ciência de Dados: Análise de Dados Aplicada

**Prof. Dr. Eduardo Pena**

---

## Identificação

| Campo         | Informação              |
| ------------- | ----------------------- |
| **Aluno**     | Yuri Lucas Luz da Silva |
| **Matrícula** | 2223481                 |

---

## Etapa 3 — Análise Exploratória e Consultas SQL

Implementada na **Seção 3 do notebook** [`notebook.ipynb`](../notebook.ipynb). Este documento resume
o processo e os principais achados.

---

## 1. Objetivo

Compreender os dados por meio de consultas SQL e análises estatísticas que respondam às perguntas de
pesquisa (P1/P2) e verifiquem preliminarmente as hipóteses **H1** (nº de estágios × gravidade) e
**H2** (controle adaptativo SCOOT × gravidade).

---

## 2. Formato Tidy e Exportação em Parquet

- O dataset já está em formato *tidy*: cada linha é um sinistro (observação) e cada coluna é uma
  variável. Padronizei os tipos (categóricas e `data_hora`).
- Mostrei um exemplo de reestruturação **wide → long** (contagem de veículos: 176.081 linhas no
  formato longo, uma por sinistro × tipo de veículo presente).
- Exportei o dataset em **Parquet** (`data/sinistros_semaforos_integrado.parquet`) **via DuckDB**
  (escritor nativo), que preserva os tipos — ao contrário do CSV.

---

## 3. Consultas SQL (DuckDB)

Registrei o dataset como tabela SQL e fiz 6 consultas (agregações, comparações entre grupos, função
de janela e CTE). "Graves" = `Fatal` + `Ferido`, excluindo `Não informado`.

| # | Consulta                                   | Principal achado                                              |
| - | ------------------------------------------ | ------------------------------------------------------------- |
| 1 | Tendência temporal (% graves por ano)      | "Sobe" de 53% (2015) a 96% (2023) — efeito da **queda no registro de _Ileso_**, não piora real |
| 2 | Com vs sem semáforo na interseção (≤30 m)  | **72,0%** graves com semáforo vs **81,1%** sem (≈ 9 p.p. menos) |
| 3 | Taxa de graves por **modo de controle**    | SCOOT **66,4%** (menor) · ECOTRAFIX 77,5% · LOCAL **85,3%** (maior) |
| 4 | Taxa de graves por **nº de estágios**      | 2→78,8% · 3→81,7% · 4→70,3% — relação **não-monotônica**       |
| 5 | Ranking de naturezas (função de janela)    | Colisão (#1), Queda (#2), Colisão Transversal (#3)…           |
| 6 | CTE: diferença de cada modo vs. média geral | SCOOT −13,0 p.p. · SCOOT CONJUGADO −11,1 · LOCAL +5,9         |

---

## 4. Análise Exploratória

- **Univariada:** distribuição de `SEVERIDADE` (predomínio de _Ferido_) e da distância ao semáforo
  (mediana ≈ 147 m, cauda longa).
- **Bivariada:** taxa de graves por presença de semáforo e por nº de estágios (gráficos de barras).
- **Multivariada:** matriz de correlação das variáveis numéricas (distância, densidades, estágios e
  contagens de vítimas).

---

## 5. Teste de Hipóteses (qui-quadrado de independência)

| Hipótese                                   | Resultado                                         | Conclusão preliminar                          |
| ------------------------------------------ | ------------------------------------------------- | --------------------------------------------- |
| **H1** — nº de estágios × gravidade        | qui² = 158,5 · p = 3,8×10⁻³⁵                       | Associação significativa, porém **não-monotônica** → H1 **não confirmada** de forma simples |
| **H2** — SCOOT (adaptativo) × gravidade    | qui² = 2740,8 · p ≈ 0                               | SCOOT 67,1% graves vs 82,4% nos demais → **H2 fortemente sustentada** |

---

## 6. Insights para a Modelagem (Etapa 4)

- Variáveis semafóricas (presença na interseção, **modo de controle**, densidade) são candidatas
  fortes; estágios tem associação significativa mas não-monotônica.
- **Cuidado com `ANO`:** confundido com o alvo pela mudança de critério de registro de _Ileso_.
- **Desbalanceamento:** ~79% de graves e _Fatal_ ~1,5% → usar AUC/recall e tratar o desbalanceamento.
- **Associação ≠ causa:** contrastes podem ter confundidores (ex.: SCOOT em grandes corredores);
  a modelagem multivariada ajudará a isolar efeitos.

---

## 7. Reprodutibilidade

- Bibliotecas adicionais desta etapa: **duckdb** (SQL e escrita do **Parquet**) e **scipy** (qui-quadrado),
  além de matplotlib/seaborn para os gráficos.
- Instalação: `pip install duckdb scipy`.
- Nota: o Parquet é gravado pelo **DuckDB** (`COPY ... FORMAT PARQUET`) para evitar uma
  incompatibilidade entre `pandas 2.3` e `pyarrow 21` no `to_parquet`.
- Todas as consultas e análises estão no notebook, em blocos numerados e comentados.

---

_Documento referente à Etapa 3 do Projeto Final — Ciência de Dados: Análise de Dados Aplicada._
