![Desafio técnico — Product Engineer](assets/cover.jpg)

# sinkfi — fechar a segunda à noite

A sinkfi antecipa e concilia recebíveis de cartão e Pix para pequenas e médias empresas. O cliente conecta as adquirentes e a conta bancária. A sinkfi mostra o que vai cair, quando, e o que já caiu. A conciliação cruza o arquivo das adquirentes com o extrato do banco, em lote, a cada 6 horas.

Marina tem duas lojas de roupa em Belo Horizonte. A Loja Centro recebe pela Stone. A Loja Savassi recebe pelo PagSeguro. As duas liquidam na mesma conta PJ do Inter. Ela faz o fechamento sozinha, toda segunda à noite, e contratou a sinkfi para parar de conferir o extrato linha a linha. Ela não sabe o que é MDR. Sabe quanto tinha que cair na conta.

## O pedido

A pessoa de suporte abriu isto ontem:

> Precisamos de um dashboard de divergências. A tela lista tudo que não bate, o cliente se perde e o suporte não dá conta. Queremos gráficos do que conciliou e do que ficou pendente para a reunião de segunda.

Os dados de uma semana estão em `dados/`. São sintéticos. CNPJ e CPF não passam em validador de dígito.

## Política que já está escrita

Isto é o que já está combinado com a Marina. O que não está aqui não tem dono. Você decide e registra.

- Fuso do negócio: `America/Sao_Paulo`.
- O fechamento desta semana é a noite de segunda, 10 de novembro de 2026.
- Entra neste fechamento o recebível com liquidação prevista até esse dia, cujo crédito no banco está confirmado até 18:00 de segunda.
- `pending` não é dinheiro na conta.
- Chargeback desfaz o crédito original. Vale mesmo que o lançamento do chargeback passe das 18:00, se o crédito desfeito entrou neste fechamento. O `original_transaction_id` aponta para o crédito.
- O Open Finance repete lançamento. A chave de idempotência é `transaction_id`.
- MDR e tarifa de antecipação não são venda faltando. O líquido esperado é `gross_cents - mdr_cents - anticipation_cents`.
- Parcela com `expected_settlement_at` depois deste fechamento não está atrasada.
- Tarifa da conta e transferência da própria titular não são recebível.
- Valores estão em centavos, inteiros.
- Não existe política escrita de materialidade. Ninguém definiu a partir de qual diferença o fechamento trava.

## O que as pessoas disseram

**Marina, dona das duas lojas.** "Eu fecho toda segunda à noite. A maquininha falou R$ 4.812 e no Inter tem R$ 4.790. Cadê os R$ 22? Não me lista quarenta linhas vermelhas. Me fala o que eu posso fechar e o que está faltando de verdade, nas duas lojas, na mesma conta."

**Rafa, suporte.** "No mês, trinta por cento dos chamados é conciliação que não bate. Quem fica reabrindo a tela de divergência cancela. Um cliente me disse: cancelei o plano, voltei pra planilha, pelo menos lá eu entendo."

**Caio, outro cliente.** "O parcelado aparece como várias vendas e não bate com o total da maquininha. E o que liquidou na sexta só apareceu no domingo, eu já tinha fechado o caixa."

## Dados

### `dados/lojas.csv`

| Coluna | Significado |
| --- | --- |
| `id` | Identificador da loja |
| `name` | Nome da loja |
| `document` | CNPJ da titular, o mesmo nas duas lojas |
| `acquirer` | Adquirente que liquida essa loja |

### `dados/recebiveis.csv`

O que as adquirentes reportaram.

| Coluna | Significado |
| --- | --- |
| `id` | Identificador do recebível |
| `loja_id` | Loja |
| `acquirer` | Quem vai liquidar |
| `rail` | `card` ou `pix` |
| `sold_at` | Dia da venda |
| `expected_settlement_at` | Dia previsto para o dinheiro cair |
| `gross_cents` | Valor bruto, o que a maquininha mostra |
| `mdr_cents` | MDR |
| `anticipation_cents` | Tarifa de antecipação, quando houve |
| `net_cents` | Líquido esperado na conta |
| `nsu` | Código da venda na adquirente |
| `installment_number` | Número desta parcela |
| `installment_count` | Total de parcelas da venda |
| `status` | `open` ou `cancelled` |

### `dados/extrato.csv`

O que o Open Finance leu no Inter. Uma linha é um lançamento, não necessariamente um crédito novo.

| Coluna | Significado |
| --- | --- |
| `id` | Identificador do lançamento |
| `transaction_id` | Identificador no banco. Pode repetir quando a sincronização repete |
| `original_transaction_id` | Preenchido no chargeback. Aponta para o crédito original |
| `booked_at` | ISO-8601. Pode vir com offset `-03:00` ou em `Z` |
| `amount_cents` | Valor do lançamento, sempre positivo. A direção está em `kind` |
| `counterparty_name` | Nome como o banco mandou |
| `counterparty_document` | Documento como o banco mandou |
| `description` | Texto livre |
| `status` | `confirmed` ou `pending` |
| `kind` | `settlement`, `pix_in`, `chargeback`, `fee`, `internal_transfer` |

## Entrega

Um produto que outra pessoa sobe e usa para fazer o fechamento, com instrução curta de como chegar nele. Stack livre.

No `DECISOES.md`, as escolhas de produto e o desenho do que está no repositório. Responda sobre a solução que você entregou.

- Quem usa, e qual trabalho essa pessoa termina.
- O que entrou e o que ficou de fora.
- A regra de fechamento, numa frase que a Marina consiga aplicar.
- O que você perguntaria para ela amanhã.
- **Desenho.** Quais peças existem, o que cada uma faz, onde a regra mora e o que fica gravado. Qual trade-off você aceitou para esta solução caber.
- **Monitorar.** O que você observa para saber que o fechamento rodou e que a Marina conseguiu terminar. O que mudaria nesse sinal antes de ela abrir um chamado.
- **Debugar.** Ela diz que o número não bate. Qual caminho vai da frase dela até o recebível, o lançamento e a decisão da regra. O que precisa existir para outra pessoa do time fazer isso sem você.
- **Responder.** Na segunda à noite o fechamento quebra, ou sai um número errado. O que acontece nas primeiras horas, como a Marina fica sabendo, e o que impede o mesmo erro de passar na semana seguinte.
- **Evoluir.** Qual é a próxima mudança se esta aposta estiver certa. Que evidência faria você desfazer ou trocar a aposta.
