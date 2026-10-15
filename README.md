# Case – Analista de Inteligência de Mercado e Precificação

## Como executar
1. `pip install pandas openpyxl jupyter`
2. Deixar os dois `.xlsx` de entrada na mesma pasta do notebook.
3. Rodar `analise.ipynb` de cima a baixo.

Saídas: `base_consolidada.xlsx` (item 4.4), `analises.xlsx` (tabelas 5.1–5.4) e `analises_case.html` (notebook executado, com tabelas, gráficos e a resposta dissertativa do 5.4; abre no navegador, sem executar código).

## Bibliotecas
`pandas`, `openpyxl` (leitura/escrita de xlsx).

## Premissas

**Carga**
1. COTAÇÕES: cabeçalho na linha 3; linhas 1–2 ignoradas.
2. SalesForce: o arquivo tem 23.227 linhas reais; o restante (~1 milhão) é resto de formatação com só Frete=0/Chapa=0. Removidas as linhas sem `ID da Cotação do SAP`, o que também descarta 2 linhas reais sem ID (uma PrecoFixo, ID Cotação 00232246, e uma quase vazia), pois não cruzam pela chave.
3. Registros do SalesForce cujo ID SAP não existe em COTAÇÕES foram descartados (fora do escopo).
4. COTAÇÕES não tem linhas duplicadas; `Última Ação em` tem texto "Vazio" (122 linhas), convertido para nulo. Não é usada em nenhum cálculo.

**Cruzamento (3.1)**
5. Chave: ID da Cotação do SAP + Material + Cód. Expedição (não usa "ID da Cotação").
6. Duplicatas no SalesForce: no escopo há 4 chaves duplicadas (8 linhas), com frete idêntico (as divergentes, 8 chaves, estão só fora do escopo). Regra preventiva: fica a linha de maior |Frete Comercial| (0 tende a ser placeholder) e a Chapa vem da mesma linha. O merge é `left` e há `assert` de que o nº de linhas não muda.
7. Sem match (90 cotações): Frete e Chapa ficam **nulos** (não 0) e a coluna extra `Match SalesForce` = "Não". Essa coluna é a única adicionada além das 4 pedidas.

**Janela de 30 dias (4.3)**
8. Elegíveis: `Status da Cotação` = "Aprovada" e `TipoCotação` em Banda/PrecoFixo (TipoCotação vazio fica fora). `Status FLUXO SAP*` não é usado.
9. Janela: datas de `Dt. Criação Sol.` no intervalo [data − 30 dias, data), mesmo cliente + material + expedição. Estrita à esquerda da data: exclui a própria linha e outras cotações do mesmo dia.
10. Início do período (01/06/2026): usa só o histórico disponível, sem extrapolar.
11. Sem cotação elegível → 0. `Var. Preço Proposto` nulo conta como 0 na soma (os nulos são só de Prazo/FreteComercial, fora do escopo).
12. A soma da variação (4.3.2) mantém o sinal original (negativo = desconto, positivo = aumento).

**Análises**
13. 5.1: concessão = soma dos valores **negativos** de `Var. Preço Proposto`, em módulo (positivo é aumento de preço, não desconto; nulo = 0). Universo: todas as solicitações do período (a definição do case chama o campo de "desconto concedido" sem filtrar status). Quantidade = nº de solicitações do cliente. Cliente repetido em mais de uma região/gestão aparece em cada uma com os valores daquele recorte.
14. 5.2: a métrica 4.3.1 é por linha; por cliente uso o **máximo** (pico de recorrência). A soma aparece como coluna de referência. Empates são ordenados arbitrariamente.
15. 5.3: só cotações aprovadas; percentual sobre o total de aprovadas.
16. 5.4: universo = Incoterm "CIF". "Frete zero" = há match e Frete = 0; "Sem informação" = sem match (Frete nulo). Incoterm "Sem informação do Sales Force" (238) não é CIF e fica fora da tabela; é comentado na dissertativa.
