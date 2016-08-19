Sumário

Introdução        2

Fluxo de Funções        2

URLs        2

Headers        2

Autenticação        2

Lista de Bandeiras        3

Lista de Parcelas        4

Criação de Transação        5

Fazer Pagamento        7

Confirmar Pagamento        9

Cancelar Pagamento        9

Consultar Transação        10

Logo de Bandeira        11

Formas de tarifação        11



# Introdução

Documentação da API de pagamento com cartões de crédito em venda digitada.

Destinado a orientar profissionais da área de sistemas.

# Fluxo de Funções

1. Inicialmente deve-se chamar a lista de bandeiras para obter os códigos.
2. Listar parcelas para obter os valores para cada opção de parcelamento.
3. Criar uma transação para obter o id da transação a ser feita.
4. Fazer o pagamento para obter a autorização do emissor do cartão.
5. Confirmar ou cancelar transação.

Em caso de eventual erro, como erro de conexão ou timeout, na função de pagamento ou confirmação/cancelamento, deve-se chamar a função de consulta para verificar o estado da transação.

# URLs

Produção: https://www.pagueveloz.com.br/api/{url da api}

Homologação: https://sandbox.pagueveloz.com.br/api/{url da api}

# Headers

São necessários 2 headers na requisição de qualquer função.

Content-Type: application/json

Authorization: Basic ZW1haWxAZG9taW5pby5jb206YWJjZGVm

# Autenticação

A autenticação ocorre através do header &quot;Authorization&quot; que é composto pela palavra Basic sucedida pelo email e token separados por dois pontos codificado com UTF8 em base64.

Exemplo: email@dominio.com:abcdef

Authorization: Basic ZW1haWxAZG9taW5pby5jb206YWJjZGVm

# Lista de Bandeiras

Primeira função a ser chamada para obter a lista de bandeiras disponíveis.

**Método** : GET

**URL** : Cartao/VendaDigitada/v1/Bandeiras

**Resultado** :

Status 200.

Conteúdo: Coleção de BandeirasDTO

| Id | Inteiro longo | Código da Bandeira |
| --- | --- | --- |
| Nome | Texto | Nome da Bandeira |

**Exemplo** :

| [  {    &quot;Id&quot;: &quot;33&quot;,    &quot;Nome&quot;: &quot;Diners&quot;  },  {    &quot;Id&quot;: &quot;2&quot;,    &quot;Nome&quot;: &quot;Master Card&quot;  },  {    &quot;Id&quot;: &quot;1&quot;,    &quot;Nome&quot;: &quot;Visa&quot;  }] |
| --- |

# Lista de Parcelas

Obtém a lista de parcelas disponíveis para o pagamento.

**Método** : GET

**URL** : Cartao/VendaDigitada/v1/Parcelamento

**Parâmetros:**

| Bandeira | Inteiro | Código da Bandeira |
| --- | --- | --- |
| ValorServico | Decimal | Valor do Serviço (ponto como separador decimal) |
| formaTarifacao | Inteiro (opcional) | Forma de tarifação da operação |

FormaTarifacao\*: Ver seção Formas de Tarifação

**Exemplos** :

Cartao/VendaDigitada/v1/Parcelamento?Bandeira=1&amp;ValorServico=500.50

Cartao/VendaDigitada/v1/Parcelamento?Bandeira=1&amp;ValorServico=500.50&amp;formaTarifacao=1

**Resultado** :

Status 200.

Conteúdo: Coleção de ParcelasDTO

| Parcelas | Inteiro | Quantidade de parcelas |
| --- | --- | --- |
| ValorParcela | Decimal | Valor da parcela |
| ValorServico | Decimal | Valor da compra |
| ValorTotal | Decimal | Valor total da transação cobrado do cliente |

**Exemplo de retorno** :

| [  {    &quot;Parcelas&quot;: 1,    &quot;ValorParcela&quot;: 510,    &quot;ValorServico&quot;: 500.50,    &quot;ValorTotal&quot;: 510  },  {    &quot;Parcelas&quot;: 2,    &quot;ValorParcela&quot;: 257,    &quot;ValorServico&quot;: 500.50,    &quot;ValorTotal&quot;: 514  }] |
| --- |

Em caso de inconsistência será retornado status 400 com ReasonPhrase indicando o motivo da inconsistência.

# Criação de Transação

Cria uma transação para fazer o pagamento posteriormente. O resultado desta função é um identificador utilizado para realizar o pagamento junto à autorizadora, e para realizar consultas posteriores.

**Método** : POST

**URL** :  Cartao/VendaDigitada/v1/Transacao

**Parâmetros:**

| NSU | Texto | NSU definido pelo cliente |
| --- | --- | --- |
| ValorServico | Decimal | Valor do Serviço. (ponto como separador decimal) |
| ValorTransacao | Decimal | Valor da transação retornado na função de parcelamento |
| Parcelas | Inteiro | Quantidade de parcelas |
| Bandeira | Inteiro | Código da bandeira |
| Descricao | Texto | Descrição do pagamento |
| ProprietarioCartao | ProprietarioCartao\* | Dados do proprietário do cartão |
| FormaTarifacao | Inteiro (opcional)\* | Forma de tarifação da operação |

FormaTarifacao\*: Ver seção Formas de Tarifação

ProprietarioCartao\*

| Nome | Texto | Nome do dono do cartão |
| --- | --- | --- |
| CPF | Texto | CPF |
| RG | Texto | RG |
| TelefoneFixo | Texto | Número do telefone fixo |
| TelefoneCelular | Texto | Número do telefone celular |
| Email | Texto | Email |
| EnviarEmail | Boolean | Indica se o cliente deve receber email sobre a transação |
| EnviarSMS | Boolean | Indica se o cliente deve receber SMS sobre a transação |

\*Todos os campos do proprietário são obrigatórios

**Exemplo** :

| {    &quot;NSU&quot; :&quot;123abc&quot;,    &quot;ValorServico&quot; : 500.50,    &quot;ValorTransacao&quot; : 510,    &quot;Parcelas&quot; : 1,    &quot;Bandeira&quot; : 1,    &quot;Descricao&quot; : &quot;Inscrição de curso&quot;,    &quot;ProprietarioCartao&quot; : {        &quot;Nome&quot; : &quot;João da Silva&quot;,        &quot;CPF&quot; : &quot;12345678909&quot;,        &quot;RG&quot; : &quot;123456789&quot;,        &quot;TelefoneFixo&quot; : &quot;47 21234000&quot;,        &quot;TelefoneCelular&quot; : &quot;47 99887755&quot;,        &quot;Email&quot; : &quot;nome@dominio.com.br&quot;,        &quot;EnviarEmail&quot; : true,        &quot;EnviarSMS&quot; : true    }} |
| --- |

**Resultado** :

Status 200.

Conteúdo

| Sem nome | Inteiro longo | Identificador da transação |
| --- | --- | --- |

**Exemplo de retorno** :

| 740 |
| --- |

Em caso de inconsistência será retornado status 400 com ReasonPhrase indicando o motivo.

# Fazer Pagamento

Realiza o pedido de pagamento junto à autorizadora. Após esta função é necessário confirmar ou cancelar o pagamento.

Em caso de não confirmação nem cancelamento, a transação será cancelada automaticamente em aproximadamente 10 minutos.

**Método** : POST

**URL** : Cartao/VendaDigitada/v1/Pagamento

**Parâmetros:**

| Id | Inteiro longo | Identificador recebido na criação da transação |
| --- | --- | --- |
| NumeroCartao | Texto | Número do cartão |
| CodigoSeguranca | Texto | Código de segurança |
| Validade | Texto | Validade do cartão no formato MMYY |

**Exemplo** :

| {    &quot;Id&quot; : 740    &quot;NumeroCartao&quot; : &quot;400000000000044&quot;,    &quot;CodigoSeguranca&quot; :&quot;123&quot;,    &quot;Validade&quot; : &quot;1016&quot;} |
| --- |

**Resultado** :

ResultadoTransacaoDTO

| Id | Inteiro longo | Identificador da transação |
| --- | --- | --- |
| Sucesso | Boolean | Indica se a transação foi realizada com sucesso |
| Mensagem | Texto | Mensagem do servidor |
| NSU | Texto | NSU definido pelo cliente |
| CupomEstabelecimento | Texto | Cupom do estabelecimento |
| CupomCliente | Texto | Cupom do cliente |

**Exemplos de retorno** :

Com recusa da operadora

| {  &quot;Id&quot;: 740,  &quot;Sucesso&quot;: false,  &quot;Mensagem&quot;: &quot;Houve um problema de transação com o emissor do cartão. Mensagem recebida do emissor do cartão: (denied card)&quot;,  &quot;NSU&quot;: &quot;123abc&quot;,  &quot;CupomEstabelecimento&quot;: null,  &quot;CupomCliente&quot;: null} |
| --- |

Com ok da operadora

| {  &quot;Id&quot;: 740,  &quot;Sucesso&quot;: true,  &quot;Mensagem&quot;: &quot;&quot;,  &quot;NSU&quot;: &quot;123abc&quot;,  &quot;CupomEstabelecimento&quot;: &quot;S...I...M...U...L...A...D...O....&quot;,  &quot;CupomCliente&quot;: &quot;S...I...M...U...L...A...D...O....&quot;} |
| --- |

Em caso de inconsistência será retornado status 400 com ReasonPhrase indicando o motivo.

# Confirmar Pagamento

Confirma o pagamento finalizando a transação. Em caso de não confirmação nem cancelamento, a transação será cancelada automaticamente em aproximadamente 10 minutos.

**Método** : POST

**URL** : Cartao/VendaDigitada/v1/Confirmar

**Parâmetros:**

| Sem Nome | Inteiro longo | Identificador da transação |
| --- | --- | --- |

**Exemplo** :

| 740 |
| --- |

**Resultado** :

Status 200 para confirmação ok

Em caso de inconsistência será retornado status 400 com ReasonPhrase indicando o motivo.

# Cancelar Pagamento

Cancela um pagamento que aguarda a confirmação ou cancelamento, finalizando a transação. Em caso de não confirmação nem cancelamento, a transação será cancelada automaticamente em aproximadamente 10 minutos.

Esta função não trata de estorno de uma transação já confirmada. Esta função apenas possibilita o rollback de uma transação, caso ocorra algum problema em eventual processamento posterior à etapa de pagamento no sistema cliente do PagueVeloz.

**Método** : POST

**URL** : Cartao/VendaDigitada/v1/Cancelar

**Parâmetros:**

| Sem Nome | Inteiro longo | Identificador da transação |
| --- | --- | --- |

**Exemplo** :

| 740 |
| --- |

**Resultado** :

Status 200 para cancelamento ok

Em caso de inconsistência será retornado status 400 com ReasonPhrase indicando o motivo.

# Consultar Transação

Retorna os dados de uma transação. Utilizado principalmente em casos de timeout ou erros de rede nas etapas de pagamento e confirmação/cancelamento.

**Método** : GET

**URL** : Cartao/VendaDigitada/v1/Consulta/

**Parâmetros:**

| Id | Inteiro longo | Identificador da transação |
| --- | --- | --- |

**Exemplo** :

Cartao/VendaDigitada/v1/Consulta/740

**Resultado** :

Operação

| Id | Inteiro longo | Quantidade de parcelas |
| --- | --- | --- |
| NSU | Texto | NSU definido pelo cliente |
| DataHora | DataHora | Data e hora da transação |
| ValorTotal | Decimal | Valor total da transação cobrado do cliente |
| Mensagem | Texto | Mensagem do servidor |
| CupomEstabelecimento | Texto | Cupom do estabelecimento |
| CupomCliente | Texto | Cupom do cliente |

**Exemplo** :

| {  &quot;Id&quot;: 740,  &quot;NSU&quot;: &quot;123abc&quot;,  &quot;DataHora&quot;: &quot;2016-03-17T15:21:52.0000000-03:00&quot;,  &quot;Status&quot;: &quot;Pago&quot;,  &quot;ValorTotal&quot;: 510,  &quot;Mensagem&quot;: &quot;&quot;,  &quot;CupomEstabelecimento&quot;: &quot;  .S...I...M...U...L...A...D...O....  &quot;,  &quot;CupomCliente&quot;: &quot;.....S...I...M...U...L...A...D...O....} |
| --- |

# Logo de Bandeira

Obtém uma imagem JPEG do logotipo da bandeira.

**Método** : GET

**URL** : Cartao/VendaDigitada/v1/Bandeiras/Logo/

**Parâmetros:**

| Id | Inteiro longo | Código da bandeira |
| --- | --- | --- |

**Exemplo** :

Cartao/VendaDigitada/v1/Bandeiras/Logo/1

**Resultado** :

Binário image/jpeg

# Formas de tarifação

A forma de tarifação informa ao sistema PagueVeloz a política de aplicação de taxas sobre o valor do serviço.

A disponibilidade das diversas políticas de forma de tarifação depende dos planos contratados, podendo não estar todas disponíveis.

Abaixo os valores possíveis.

1 - O valor do serviço é o valor a ser cobrado no cartão. As taxas referentes à operação serão descontados do repasse (antecipação).

2 - O valor do serviço é o valor a ser recebido no repasse (antecipação). As taxas serão adicionadas no valor a ser cobrado no cartão.

3 - Os juros são adicionados ao valor do serviço resultando no valor a ser cobrado no cartão, as taxas administrativas serão descontadas do repasse.
