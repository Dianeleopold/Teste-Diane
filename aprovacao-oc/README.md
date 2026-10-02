# Pré-aprovação de Ordem de Compra (OC) — San Jose

Tela única (`index.html`, sem dependências externas) para a diretoria decidir se uma OC se justifica pelo histórico comercial (Chipa) e pelos dados de compras e estoque (relatório SECE109). Valores em guaranis, no formato `G$ 1.234.567`.

> **Acesso restrito a Compras e Diretoria**: a tela exibe custo de **compra**. O custo de venda e a margem do Chipa (`CUSVEN`, `CUSDEV`, qualquer campo com `CUS` ou `MARGEM`/`MARGEN`) são descartados ao receber e nunca aparecem.

## Como abrir
- **Modo exemplo**: abra `index.html` no navegador. A OC **31716** abre por padrão; a fila tem outras 4 OCs fictícias. Uma faixa amarela "DADOS DE EXEMPLO" fica visível enquanto houver dado fictício.
- **Modo API Chipa**: a página precisa ser servida **na origem `chipa.sanjosesa.com.py`** (a API não aceita chamadas de outra origem por CORS e só é acessível pela rede interna). Em "Fonte de dados e configuração": escolha *API Chipa*, confirme a URL base (padrão `/api/v1`), cole o token e clique em *Aplicar e recarregar*. O token fica **só na memória da aba**: não vai para o código nem para o `localStorage`.

## O que a tela mostra
1. **Fila de OCs em aberto** e campo **"Abrir OC nº"** (ex.: 31716).
2. **Resumo executivo**: recomendação automática (*Aprovar*, *Aprovar com ressalva* ou *Revisar*) e os 3 principais motivos.
3. **Cabeçalho e KPIs**: número, fornecedor, comprador, data, valor total, itens, status, modo de dados; semáforo geral ("X itens ok, Y em alerta, Z críticos").
4. **Recorte**: últimos 12 meses fechados (o mês corrente fica de fora), com período e filtros.
5. **Itens** (tabela com semáforo) e **detalhe** ao clicar: vendas 12 meses (gráficos), estoque e giro mês a mês, custo de compra mês a mês, estoque futuro na chegada, top 3 motivos de devolução.
6. **OCs aprovadas pendentes de recebimento** dos mesmos itens ou do mesmo fornecedor, com atraso e alerta de compra duplicada.
7. **Fornecedor (Chipa)**: venda 12 meses, devolução sobre venda, bonificação, crédito por acordo comercial e acordos com valor (`USU_NUMCLA`).
8. **Decisão**: Aprovar, Reprovar, Devolver para ajuste (comentário obrigatório nos dois últimos). Histórico salvo no `localStorage` do navegador (por OC) e exportação do parecer em JSON.

## Consultas ao Chipa (comercial)
`q/comercial/fattotdim` com `data_ini`, `data_fim` e:
| Bloco | `col` | Filtro |
|---|---|---|
| Série mensal do item | `ANO_MES` | `codpro` |
| Motivos de devolução | `CODMOT` | `codpro` |
| Totais do fornecedor | `TODO` | `proveedor` |
| Acordos comerciais | `USU_NUMCLA` | `proveedor` |

Nomes dos motivos: `q/comercial/devncfacc` (se falhar, mostra o código). Paginação, 502/503 com nova tentativa em 5, 15 e 30 segundos, 504 sem repetição, no máximo 2 requisições simultâneas. Métricas chegam como texto e são convertidas para número; campos `COD*`, `NUM*`, `USU_COD*`, `USU_NUM*` ficam texto.

## Compras e estoque (SECE109) — endpoints a confirmar
O Chipa documentado só tem dados comerciais. Compras e estoque vêm do **relatório 109 de compras (SECE109, ERP Senior)**, cujo endpoint e formato ainda não conhecemos. Há duas formas de alimentar a tela:

**a) Ponte Chipa (mesmo token e URL base)**: ajuste o objeto `ENDPOINTS_COMPRAS` no topo do script. Enquanto um caminho contiver `TODO`, a tela não chama a API e avisa no bloco.

| Chave | Parâmetros enviados (a confirmar) |
|---|---|
| `ocsAbertas` | nenhum |
| `ocItens` | `numero` |
| `ocsPendentes` | `codpro` (lista separada por vírgula) e, em outra chamada, `proveedor` |
| `estoqueMensal` | `codpro`, `data_ini`, `data_fim` (desde o mês anterior ao recorte) |
| `estoqueAtual` | `codpro` (opcional; sem ele usa o último mês) |
| `custoMensal` | `codpro`, `data_ini`, `data_fim` |

**b) Importar o SECE109** exportado em CSV (separador `;`, `,` ou tabulação) ou JSON (lista de linhas, ou objeto com listas). Pode enviar vários arquivos. O tipo de cada linha é reconhecido pelas colunas:
- **Linha de OC**: tem `numero_oc` e `codpro`.
- **Estoque**: tem `estoque_final` ou `estoque_atual` (e não tem `numero_oc`).
- **Custo**: tem `custo_unitario` (e não tem `numero_oc`). Sem tabela de custo, o custo sai das linhas de OC recebidas (`data_entrada` e `qtd_recebida` > 0, usando `preco_unitario`).

Mapeamento de colunas (`MAPA_SECE109`; ignora maiúsculas, acentos, espaços e `_`):

| Campo | Colunas aceitas |
|---|---|
| numero_oc | numero_oc, NUMOCP, num_ocp, nro_oc, oc, numero |
| data_emissao | data_emissao, DATEMI, fecha_emision, data_oc |
| fornecedor_codigo | fornecedor_codigo, CODFOR, cod_fornecedor, cod_proveedor, proveedor |
| fornecedor_nome | fornecedor_nome, NOMFOR, razao_social, nombre_proveedor, fornecedor |
| comprador | comprador, NOMCPR, usuario_comprador |
| situacao | situacao, SITOCP, situacion, status, estado |
| codpro | codpro, codigo_produto, cod_producto, item |
| descricao | descricao, DESPRO, CPLIPO, descripcion, produto |
| qtd_pedida | qtd_pedida, QTDPED, cantidad_pedida, quantidade |
| qtd_recebida | qtd_recebida, QTDREC, cantidad_recibida |
| qtd_pendente | qtd_pendente, QTDABE, saldo_pendente, cantidad_pendiente |
| preco_unitario | preco_unitario, PREUNI, precio_unitario, preco |
| data_prevista | data_prevista, DATENT, fecha_entrega, previsao_entrega |
| ano_mes | ano_mes, anomes, periodo, mes (AAAAMM ou AAAA-MM) |
| estoque_final | estoque_final, saldo_final, stock_final, QTDEST, estoque |
| estoque_atual | estoque_atual, saldo_atual, stock_actual |
| data_entrada | data_entrada, DATREC, fecha_entrada, data_recebimento |
| custo_unitario | custo_unitario, custo_compra, preco_entrada, precio_compra, VLRUNI |

Datas: `AAAA-MM-DD`, `DD-MM-AAAA` (qualquer separador) ou `AAAAMMDD`. Números: aceita `1.234.567` e `1.234,50`.
Situação (valores a confirmar): contém *aguard*, *aprovacao*, *abert*, *emitid* ou *digit* → **em aberto** (vai para a fila); *aprov*, *liber*, *fech*, *confirm*, *parcial* ou *pend* → **aprovada** (entra em pendentes se faltar receber); *receb*, *conclu*, *total* → recebida; *cancel*, *anul* → cancelada. Sem situação = em aberto.

## Importar uma OC avulsa (JSON)
```json
{"numero":"31716","data":"2026-09-29","comprador":"Rodrigo Benítez","status":"Aguardando aprovação",
 "fornecedor":{"codigo":"0127","nome":"Distribuidora Alimentos del Sur S.A."},
 "itens":[{"codpro":"05040.0075","descricao":"Galletita dulce vainilla 400 g","quantidade":1800,"preco_unitario":50800}]}
```

## Cálculos e sinais (limites editáveis na tela)
- **Média mensal** = unidades vendidas em 12 meses ÷ 12. **Cobertura do pedido** = qtd pedida ÷ média mensal (meses): alerta > 3, crítico > 6.
- **Tendência** = últimos 3 meses sobre os 3 anteriores: alerta abaixo de −20%.
- **Devolução sobre venda** = DEVOLUCION ÷ VENTA: alerta > 3%, crítico > 6%. Também: bonificação sobre venda, desconto sobre bruto (DESC_GS ÷ BRUTO_GS).
- **Sem venda no período** = crítico.
- **Giro do mês** = vendidas no mês ÷ estoque médio (média do estoque final do mês e do anterior). **Cobertura em dias** = estoque final ÷ venda média diária do mês.
- **Custo de compra do mês** = última entrada do mês; variação sobre a entrada anterior e acumulada. Preço da OC sobre o último custo: alerta > +5%, crítico > +10%.
- **Estoque futuro** (prazo padrão 60 dias): estoque atual + pendentes que chegam até a data − venda média diária (12 meses ÷ dias do recorte) × prazo. Sem a OC < 0 → crítico "vai faltar antes de chegar". Com a OC = máximo(0; sem OC) + qtd da OC. Cobertura após a chegada > 120 dias → alerta de excesso.
- **Pedido pendente do mesmo item** → alerta de compra duplicada.
- **Recomendação**: *Revisar* se houver crítico contra a compra; *Aprovar com ressalva* se houver só alertas ou ruptura prevista (urgência); senão *Aprovar*.

## Limitações conhecidas
- Endpoints, parâmetros, nomes de coluna e valores de situação do SECE109 são **suposições a confirmar**.
- A projeção trata todo pendente com data até a chegada (inclusive atrasado) como recebido antes dela, e não modela sazonalidade futura.
- O histórico de decisões fica só no navegador de quem decidiu (não é compartilhado nem auditável centralmente).
- O formato real de `ANO_MES` e o nome da coluna de descrição em `devncfacc` não foram testados contra a API real.
