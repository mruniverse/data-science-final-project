# Projeto Final — Ciência de Dados: Análise de Dados Aplicada

**Prof. Dr. Eduardo Pena**

---

## Identificação

| Campo         | Informação              |
| ------------- | ----------------------- |
| **Aluno**     | Yuri Lucas Luz da Silva |
| **Matrícula** | 2223481                 |

---

## 1. Descrição do Dataset

### Dataset Principal — Sinistros de Trânsito em Fortaleza

| Atributo                | Detalhe                                                                        |
| ----------------------- | ------------------------------------------------------------------------------ |
| **Fonte**               | Dados Abertos de Fortaleza — Autarquia Municipal de Trânsito e Cidadania (AMC) |
| **URL**                 | https://dados.fortaleza.ce.gov.br/dataset/sinistros-transito                   |
| **Período**             | 2015 a 2024 (10 anos)                                                          |
| **Formato**             | GeoJSON                                                                        |
| **Registros estimados** | > 100.000 ocorrências                                                          |
| **Atualização**         | Julho de 2025                                                                  |

O dataset registra todos os sinistros de trânsito ocorridos em Fortaleza ao longo de dez anos, contendo variáveis espaciais (coordenadas geográficas do local do acidente), temporais (data e hora) e descritivas (tipo de sinistro, gravidade, veículos e vítimas envolvidos). Por estar no formato GeoJSON, cada registro possui geometria de ponto georreferenciada, o que permite análises espaciais diretamente.

### Dataset Complementar — Parque Semafórico de Fortaleza

| Atributo        | Detalhe                                                            |
| --------------- | ------------------------------------------------------------------ |
| **Fonte**       | Dados Abertos de Fortaleza — CTAFOR/AMC                            |
| **URL**         | https://dados.fortaleza.ce.gov.br/dataset/semaforo                 |
| **Formato**     | GeoJSON / CSV                                                      |
| **Conteúdo**    | Posições geográficas e informações de todos os semáforos da cidade |
| **Atualização** | Julho de 2025                                                      |

Esse dataset complementar traz a localização georreferenciada de todos os semáforos instalados em Fortaleza, possibilitando a integração espacial com os registros de sinistros por meio de cálculo de distâncias e análise de densidade semafórica por região.

### Integração e Feature Engineering

A integração entre os dois datasets será realizada via operações espaciais (spatial join), calculando para cada sinistro:

- **Distância ao semáforo mais próximo** (em metros);
- **Número de semáforos em raio de 100 m e 250 m** do local do acidente (densidade semafórica);
- **Indicador binário** de presença de semáforo na interseção.

Após essa integração e engenharia de atributos (variáveis de hora do dia, dia da semana, mês, estação do ano, tipo de via, etc.), o dataset resultante deverá superar os 15 preditores exigidos, combinando variáveis numéricas e categóricas.

---

## 2. Contexto e Justificativa

Os acidentes de trânsito representam um dos maiores problemas de saúde pública do Brasil. Segundo o Ministério da Saúde, eles figuram entre as principais causas de morte e invalidez permanente no país, com impacto expressivo em vítimas jovens e em idade produtiva. Fortaleza, capital do Ceará, historicamente apresenta altos índices de sinistros fatais, o que torna a cidade um caso especialmente relevante para estudos de segurança viária.

A sinalização semafórica é um dos instrumentos mais utilizados para a regulação do tráfego e redução de conflitos em interseções. No entanto, existem poucos estudos que quantificam empiricamente — e com base em dados reais de uma cidade brasileira — o quanto as características específicas dos semáforos (número de estágios, modo de controle, tecnologia empregada) influenciam a frequência e a gravidade dos acidentes. A maior parte das análises disponíveis na literatura trata o semáforo como variável binária (presente ou ausente), sem considerar que semáforos de 2 estágios, 3 estágios ou 4 estágios, ou os que operam em modo adaptativo centralizado (como o SCOOT), podem ter impactos muito distintos sobre a segurança viária.

Esta análise se propõe a preencher essa lacuna: cruzando o registro histórico de dez anos de sinistros em Fortaleza com o mapeamento completo do parque semafórico da cidade — que inclui tipo, número de estágios e modo de controle de cada equipamento —, será possível identificar quais configurações semafóricas estão mais associadas à redução da gravidade dos acidentes e construir modelos preditivos que incorporem essa dimensão de infraestrutura de forma granular.

Os resultados obtidos têm relevância prática direta para a AMC e para gestores de mobilidade urbana, pois podem orientar prioridades de instalação e manutenção de semáforos nos pontos de maior risco, além de subsidiar políticas de segurança viária baseadas em evidências.

---

## 3. Perguntas de Pesquisa

**P1.** O tipo de semáforo presente na interseção — definido pelo número de estágios (2, 3 ou 4 fases) e pelo modo de controle (LOCAL, ECOTRAFIX, SCOOT) — influencia a gravidade e a frequência dos sinistros de trânsito em Fortaleza?

**P2.** É possível predizer a gravidade de um sinistro de trânsito em Fortaleza com base em variáveis temporais, espaciais e de configuração semafórica, e quais dessas variáveis têm maior poder preditivo?

---

## 4. Hipóteses Testáveis

**H1:** Semáforos com maior número de estágios (3 ou 4 fases) estão associados a sinistros de menor gravidade média em comparação com semáforos de 2 estágios, devido à organização mais granular dos fluxos de tráfego nas interseções.

**H2:** Interseções com semáforos operando em modo de controle adaptativo centralizado (SCOOT) apresentam menor frequência de sinistros graves do que interseções com controle local (LOCAL ou ECOTRAFIX), pois o controle adaptativo responde dinamicamente ao volume de tráfego.

**H3:** Modelos de classificação que incluem variáveis de configuração semafórica (número de estágios, modo de controle, presença de sinalização para pedestres e ciclistas) superam, em acurácia e AUC-ROC, modelos treinados apenas com variáveis temporais e categóricas.

---

_Documento referente à Etapa 1 do Projeto Final — Ciência de Dados: Análise de Dados Aplicada._
