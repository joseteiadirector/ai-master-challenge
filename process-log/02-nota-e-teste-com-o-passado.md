# Etapa 3 — A nota e o teste com o passado

Ferramenta: Claude Code. Código em [`solution/motor/nota.py`](../solution/motor/nota.py) e [`solution/testes/teste_passado.py`](../solution/testes/teste_passado.py).

## Como a chance é medida

A etapa 2 deixou uma suspeita: "vendas que passam de 90 dias fecham 71%" só conta as vendas que já fecharam. Uma venda que fica aberta para sempre nunca entra nessa conta.

Por isso a chance é medida com **fotos do passado**: em cada fim de mês, o motor vê o CRM como estava naquele dia, pega as vendas abertas e confere quantas foram ganhas nos 90 dias seguintes. São 12 fotos. Só entram vendas com cliente (a armadilha da etapa 2).

| Dias aberta | Chance de ganhar em 90 dias |
|---|---|
| 0 a 100 | 44% a 59% |
| 101 a 110 | 45% |
| 111 a 120 | 32% |
| 121 a 130 | 22% |
| 131 a 138 | 10% |
| mais de 138 | 0% (486 casos, nenhum ganho) |

A venda **esfria, mas só depois de 100 dias**. Antes disso, o tempo quase não muda a chance. O "71%" era efeito de olhar só as encerradas.

**Relação vendedor-cliente:** quando o vendedor já ganhou uma venda com aquele cliente (antes da data da foto), a chance é 57%; sem histórico, 48%. A chance da faixa de dias é ajustada por essa proporção.

**Nota** = posição do valor esperado (chance × preço) entre as vendas abertas, de 0 a 100.

## Primeiro erro corrigido: parada antes de cadastro

A primeira rodada mandou 1.425 vendas para "cadastrar cliente" e só 412 para "revisar". Conferindo: 879 vendas estavam **paradas há mais de 138 dias e também sem cliente**. Não faz sentido pedir para cadastrar o cliente de uma venda morta. Correção: "parada" vem antes de "sem cliente". Resultado: 1.291 paradas e 546 sem cliente.

## O teste

Em 4 datas (30/06, 31/07, 31/08 e 30/09/2017), o motor aprende só com o que se sabia naquele dia. Pergunta: se o vendedor focar nos 20% do topo da lista, quanto da receita dos 90 dias seguintes ele pega?

Alternativas comparadas:

- **Nota só pelo tempo aberta** (sem a relação vendedor-cliente).
- **Nota "óbvia":** taxa de ganho do vendedor naquele produto × preço. São os atributos que parecem óbvios (o próprio desafio cita vendedor e produto).
- **Só o maior preço.**
- **Ao acaso:** média de 1.000 sorteios, o "feeling".

## Segundo erro corrigido: empates

O primeiro resultado deu "só o maior preço" = 42,7%, bem abaixo do esperado. A IA investigou a causa: só existem 7 preços, então centenas de vendas empatam, e o resultado dependia da ordem em que elas aparecem no arquivo. Correção: empates são desfeitos por sorteio, com média de 200 sorteios, para todos os métodos. "Só o maior preço" passou para 48,4%.

## Resultado

| Jeito de escolher os 20% do topo | Receita capturada (média) | A nota ganhou em |
|---|---|---|
| **Nota do Lead Scorer** | **59,2%** | — |
| Nota só pelo tempo aberta | 56,4% | 4 de 4 datas |
| Só o maior preço | 48,4% | 4 de 4 datas |
| Nota "óbvia" (vendedor no produto × preço) | 45,3% | 4 de 4 datas |
| Ao acaso | 20,0% | 4 de 4 datas |

- A nota "óbvia" ficou **atrás de ordenar só por preço**: usar a taxa do vendedor no produto atrapalha, porque essa taxa é quase toda acaso (etapa 2).
- A relação vendedor-cliente acrescenta 2,8 pontos sobre a nota só pelo tempo.
- A data mais difícil foi 30/06: 48,6% contra 48,5% do preço. Nas outras três a diferença é grande.

## Previsão de receita

| Data | O CRM mostrava | A ferramenta previu | Entrou em 90 dias |
|---|---|---|---|
| 30/06/2017 | US$ 4,2 mi | US$ 1,6 mi | US$ 1,4 mi |
| 31/07/2017 | US$ 5,5 mi | US$ 2,1 mi | US$ 1,7 mi |
| 31/08/2017 | US$ 5,4 mi | US$ 1,7 mi | US$ 1,3 mi |
| 30/09/2017 | US$ 5,8 mi | US$ 1,6 mi | US$ 1,2 mi |

O CRM mostra 3 a 5 vezes o que entra. A previsão chega muito mais perto, mas erra para cima: de 12% a 35%.

## A chance prevista bate?

Vendas com cliente, nas 4 datas: prevê 45% e acontecem 43%; prevê 57% e acontecem 47%; prevê 66% e acontecem 53%. **A ordem está certa, o número é otimista nas chances altas.** Isso está nas limitações do README.

## O motor não espia o futuro

O teste embaralha tudo o que aconteceu depois da data (ganhou ou perdeu, e a data de fechamento) e confere que o motor aprende exatamente a mesma coisa. Passou.
