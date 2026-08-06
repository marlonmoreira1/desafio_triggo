# Teste Técnico - Programa Trainee triggo.ai de Excelência em Engenharia de Dados e DataOps 2025

Este repositório contém uma análise completa do dataset público da Olist, um marketplace brasileiro. O projeto foi desenvolvido como parte de um processo seletivo para vaga de trainee na Triggo.

## Artigo Completo

Além do notebook, escrevi um artigo no Medium detalhando o raciocínio utilizado durante a análise, os principais insights encontrados e as recomendações de negócio propostas ao longo do projeto.

**Medium:**  
**[Artigo](https://medium.com/@marlonm.almeida/o-que-mais-de-100-mil-pedidos-podem-ensinar-sobre-um-marketplace-4ae05a161cd2)**

## Estrutura do Projeto

- `Desafio_Triggo.ipynb`: Notebook Jupyter com todo o código e análises
- `README.md`: Este arquivo, com instruções e resultados principais
- `dados/`: Pasta contendo os arquivos CSV do dataset da Olist

## Como Executar

Clone este repositório:

```bash
git clone https://github.com/seu-usuario/Desafio_Triggo.git
cd olist-analysis
```
Instale as dependências necessárias:

pip install pandas numpy matplotlib seaborn scikit-learn duckdb plotly

Baixe os dados da Olist do Kaggle e coloque os arquivos CSV na pasta dados/.
Execute o notebook Desafio_Triggo.ipynb usando Jupyter Notebook ou Google Colab.

## Análise Realizada

O projeto foi dividido em 4 etapas principais:

### 1. Preparação dos Dados

#### 1.1 Importação dos arquivos

- Leitura de todos os arquivos CSV do dataset público da Olist  
- Organização dos DataFrames em um dicionário para facilitar o acesso

#### 1.2 Limpeza

- **Duplicatas**: Apenas o `geolocation` tinha duplicatas exatas; foram removidas, pois não agregavam valor.

- **Valores nulos**:
  - `order_reviews`: colunas de comentários e títulos têm muitos nulos, clientes frequentemente não deixam texto.
  - `orders`: colunas de datas têm nulos, possivelmente pedidos cancelados ou não entregues.
  - `products`: campos importantes faltando para alguns produtos, mas esses produtos tinham pedidos associados, então foram mantidos e classificados como “outros”.

#### 1.3 Normalização

- Conversão de colunas de datas para o tipo `datetime`, facilitando análises temporais.
- Colunas de cidades com erros/variações não foram tratadas, pois não seriam utilizadas na análise.

#### 1.4 Modelo Relacional no DuckDB

- Estruturação das conexões entre tabelas baseada em chaves como `order_id`, `product_id`, `seller_id`, `customer_id` e `zip_code_prefix`.

**Exemplo de conexões:**
- `df_orders` conecta a `pagamentos`, `avaliações`, `itens` e `clientes`
- `itens` conectam a `produtos` e `vendedores`
- `localização` conecta-se por prefixos de CEP


### 2. Análise Exploratória de Dados

#### a) Volume de pedidos por mês e sazonalidade

- Crescimento contínuo de pedidos entre set/2016 e nov/2017.
- Estabilidade em 2018, com queda em set/out 2018 possivelmente por dados incompletos.
- Sem sazonalidade clara.

#### b) Distribuição do tempo de entrega

- 75% das entregas concluídas em até duas semanas.
- 96% das entregas concluídas em até um mês.
- Presença de casos extremos com entregas superiores a 3 meses.
  
  <img width="1589" height="590" alt="insight 1" src="https://github.com/user-attachments/assets/66517974-a5ab-4e4c-9742-6ce80b95f9d5" />

---

#### c) Relação entre valor do frete e distância de entrega

- Correlação positiva, mas distância não explica totalmente o preço do frete.

#### d) Categorias de produtos mais vendidas em termos de faturamento

- 10 categorias (14% do total) são responsáveis por 63% do faturamento.
- As 5 principais categorias (7%) representam 40% de todo o faturamento.
- Alta concentração de receita em poucas categorias, sugerindo oportunidades e riscos.
- Alinhado à regra 80/20 — foco estratégico e risco concentrado.

  <img width="1987" height="989" alt="pareto" src="https://github.com/user-attachments/assets/b761ee90-c8f2-4bbf-9f65-d99e6da56414" />

---


### 3. Solução de Problemas de Negócio

#### 3.1 Análise de Retenção

- Taxa de retenção de apenas 3%.
- Predominância de clientes que realizam apenas uma compra durante o período analisado.

#### 3.2 Predição de Atraso

Desenvolvi dois modelos para prever se um pedido será entregue com atraso. Ambos tiveram desempenho similar, com diferenças importantes:

- **Modelo 1:** cobre apenas parte dos atrasos, deixando de sinalizar cerca de 5 pedidos atrasados por dia.
- **Modelo 2:** identifica 87% dos pedidos atrasados (~12 de 13 por dia), mas com menor precisão (abaixo de 45%), resultando em 27 alertas diários.

  <img width="691" height="468" alt="atraso_ml" src="https://github.com/user-attachments/assets/9e8da513-4041-4741-aca7-79b95cee86c1" />

Apesar do aumento de falsos positivos, o segundo modelo permite uma atuação proativa que pode reduzir em até 80% os atrasos não identificados, melhorando a experiência do cliente.

**Fatores relacionados ao atraso:**

- Tempo de entrega: quanto maior, maior o risco.
- Localização do cliente e vendedor: SP tem mais atrasos; RS menos.
- Categoria do produto: itens como ferramentas e relógios atrasam menos.
- Frete: fretes mais caros tendem a ter menos atrasos.

---

#### 3.3 Segmentação de Clientes

- **Grupo 0 – Insatisfeitos:** menor recorrência e maior tempo de entrega.
- **Grupo 1 – Compra Única:** mais satisfeitos, gastam mais e recebem rápido.
- **Grupo 2 – Parceladores:** alta recorrência e segundo maior gasto.
- **Grupo 3 – Fiéis:** poucos clientes, mas muito recorrentes e engajados.
- **Grupo 4 – Premium:** maior gasto médio e fidelização, foco em retenção.
- **Grupo 5 – Emergentes:** bom potencial de fidelização.

  <img width="853" height="545" alt="cluster" src="https://github.com/user-attachments/assets/0c19fa4b-59d5-4764-bb88-481555802320" />

**Estratégias sugeridas:**

- Incentivar parcelamento, que está associado à recorrência.
- Melhorar a logística, principalmente para o Grupo 0 de Insatisfeitos.
- Investir em retenção nos grupos 2, 4, 3 e 5, que apresentam bom retorno e retenção acima da média geral.

---
#### 3.4 Análise de Satisfação

Avaliando a nota dos clientes em relação a diversas variáveis, identifiquei que:

- Tempo de entrega é o fator com maior impacto negativo na satisfação (correlação -0.33).
- Preço do produto, valor pago e frete têm pouca influência na nota.
- Conclusão: clientes penalizam mais pelos atrasos do que pelo preço.

  <img width="654" height="562" alt="satisfação" src="https://github.com/user-attachments/assets/d8f6eab2-d27e-4843-8f25-e751a2033c6b" />


---

### 4. Visualização e Dashboards

- A Black Friday de 2017 teve um pico expressivo de vendas.
- A região Sudeste lidera nas vendas, com destaque para o estado de São Paulo (mais de 5 milhões).
- Boxplots mostram que avaliações mais altas estão ligadas a entregas mais rápidas e consistentes.
- Produtos com nota 5 são entregues quase duas vezes mais rápido que os com nota 1.

---

## Principais Insights

- **Logística:** O tempo de entrega é crucial para a satisfação do cliente e retenção.
- **Concentração de Receita:** Poucas categorias representam grande parte do faturamento.
- **Retenção:** Taxa muito baixa (3%) indica oportunidade de melhoria.
- **Segmentação:** Identificação de grupos com características distintas que permitem ações de marketing direcionadas.
- **Parcelamento:** Forte relação com recorrência e volume de gastos.

## Recomendações de Negócio

Com base nos resultados obtidos, foram propostas algumas iniciativas que poderiam ser avaliadas pelo marketplace.

### 1. Monitoramento Preventivo de Pedidos com Risco de Atraso

**Hipótese**

A identificação antecipada de pedidos com maior probabilidade de atraso permite reduzir problemas operacionais e minimizar impactos na experiência do cliente.

**Ação**

Integrar o modelo preditivo ao processo logístico para gerar alertas automáticos sobre pedidos com alto risco de atraso, permitindo ações preventivas antes do vencimento do prazo de entrega.

**Critério de sucesso**

- Redução da taxa de pedidos entregues com atraso;
- Aumento do percentual de entregas dentro do prazo;
- Redução das avaliações negativas relacionadas à entrega;
- Aumento da nota média das avaliações.

---

### 2. Priorizar as Categorias de Maior Faturamento

**Hipótese**

Como poucas categorias concentram grande parte da receita, melhorias operacionais e comerciais nesses segmentos tendem a gerar maior impacto financeiro.

**Ação**

Priorizar investimentos em gestão de estoque, campanhas promocionais e otimização logística para as categorias responsáveis pela maior parcela do faturamento.

**Critério de sucesso**

- Aumento do faturamento das categorias priorizadas;
- Crescimento da participação dessas categorias na receita total;
- Redução do tempo médio de entrega dessas categorias;
- Aumento da taxa de conversão das campanhas.

---

### 3. Personalizar Estratégias para Cada Perfil de Cliente

**Hipótese**

Clientes com comportamentos distintos respondem melhor a ações direcionadas do que a campanhas genéricas.

**Ação**

Utilizar os segmentos identificados para personalizar campanhas de CRM, programas de fidelidade e ações de recuperação para clientes insatisfeitos.

**Critério de sucesso**

- Aumento da taxa de recompra por segmento;
- Aumento da frequência média de compra;
- Crescimento do ticket médio dos segmentos priorizados;
- Redução da proporção de clientes classificados como insatisfeitos.

---

### 4. Redução do Tempo Médio de Entrega

**Hipótese**

Como o tempo de entrega apresentou a maior relação com a satisfação dos clientes, melhorias logísticas tendem a gerar ganhos diretos na experiência do consumidor.

**Ação**

Mapear regiões, vendedores e rotas com maior tempo médio de entrega para priorizar melhorias operacionais e revisão dos parceiros logísticos.

**Critério de sucesso**

- Redução do tempo médio de entrega;
- Redução da variabilidade do prazo de entrega;
- Aumento da nota média das avaliações;
- Redução das reclamações relacionadas à logística.
