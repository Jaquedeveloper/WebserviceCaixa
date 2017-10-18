# Cobran�a registrada da Caixa Econ�mica Federal

Webservice de acesso �s opera��es b�sicas de consulta, inclus�o e altera��o
de cobran�as registradas segundo o [manual fornecido pela CEF](doc/MO38239.pdf).
Existem algumas diverg�ncias devido �s frequentes modifica��es no servi�o pela
CEF, mas encontra-se funcional e est�vel na data de publica��o deste c�digo.

Trata-se de uma implementa��o gen�rica. Regras espec�ficas devem
ser implementadas � parte como � o caso do m�todo `GeraBoleto`.

Compat�vel com PHP 5.1+

## Modo de uso

1. O atendimento t�cnico da Caixa Econ�mica deve fornecer a voc� os seguintes
   arquivos para serem colocados na pasta `xml`:

    * `Consulta_Cobranca_Bancaria_Boleto.wsdl`
    * `Manutencao_Cobranca_Bancaria_Externo.wsdl`

   Se voc� n�o os tiver, pode acess�-los nas urls e salvar nestes arquivos.
   
    * https://des.barramento.caixa.gov.br/sibar/ConsultaCobrancaBancaria/Boleto?wsdl
    * https://des.barramento.caixa.gov.br/sibar/ManutencaoCobrancaBancaria/Boleto?wsdl

2. Modifique as constantes definidas no in�cio do arquivo
   `WebserviceCaixa.php`.

3. Verifique os m�todos `Consulta`, `Inclui` e `Altera` para verificar quais
   par�metros devem ser inseridos. Para a rela��o completa de op��es Verifique
   o manual da Caixa Econ�mica Federal.

```php
include('./WebServiceCaixa/WebserviceCaixa.php');
$parametros = array(
    'CODIGO_BENEFICIARIO' => '951955',
    'NOSSO_NUMERO' => '1947658325871322',
    'NUMERO_DOCUMENTO' => '674389152',
    'DATA_VENCIMENTO' => '2017-10-18',
    'VALOR' => '81.53',
    'FLAG_ACEITE' => 'N',
    'DATA_EMISSAO' => '2017-10-18'
    'NUMERO_DIAS' =>  '30'
    'PAGADOR' => array(
        'CPF' => '0036893461927'
        'NOME' => 'CARLOS FERNANDO ROSA'
        'ENDERECO' => array(
            'LOGRADOURO' => 'ROD ADMAR GONZAGA, 1823',
            'BAIRRO' => 'ITACORUBI',
            'CIDADE' => 'FLORIANOPOLIS',
            'UF' => 'SC'
            'CEP' => '88034000'
    ),
    'FICHA_COMPENSACAO' => array(
        'MENSAGENS' => array(
            'MENSAGEM1' => 'PRIMEIRA LINHA DA MENSAGEM',
            'MENSAGEM2' => 'SEGUNDA LINHA DA MENSAGEM'
        )
    )
);
$ws = new WebserviceCaixa($parametros);
$ws->Gera();
```

## Gera��o de boletos

### PDF gerado pela Caixa

O retorno do webservice cont�m uma URL para um PDF que pode ser utilizada,
embora nem sempre ele esteja dispon�vel.

```html
<a href="<?php echo $ws->GetUrlBoleto($boleto) ?>">Link para o boleto</a>
```

```php
header('Location:' . $ws->GetUrlBoleto($boleto));
exit();
```

### BoletoPHP

Recomenda-se gerar o seu pr�prio boleto ap�s a realiza��o das opera��es usando
o [BoletoPHP](https://github.com/CobreGratis/boletophp), por exemplo:

```php
$dias_de_prazo_para_pagamento = floor((strtotime($ws->GetDataVencimento()) - time()) / 60 * 60 * 24);
$taxa_boleto = 0;
$data_venc = date('d/m/Y', strtotime($ws->GetDataVencimento()));

$nn = $ws->GetNossoNumero();

$dadosboleto["nosso_numero_const1"] = substr($nn, 0, 1);
$dadosboleto["nosso_numero_const2"] = substr($nn, 1, 1);
$dadosboleto["nosso_numero1"] = substr($nn, 2, 3);
$dadosboleto["nosso_numero2"] = substr($nn, 5, 3);
$dadosboleto["nosso_numero3"] = substr($nn, 8, 9);

$valor_cobrado = $ws->GetValor();
$valor_boleto = number_format($valor_cobrado+$taxa_boleto, 2, ',', '');
$dadosboleto["numero_documento"] = $ws->GetNumeroDocumento();
$dadosboleto["data_vencimento"] = date('d/m/Y', strtotime($ws->GetDataVencimento()));
$dadosboleto["data_documento"] = date('d/m/Y', strtotime($ws->GetDataEmissao()));
$dadosboleto["data_processamento"] = date('d/m/Y', strtotime($ws->GetDataEmissao()));
$dadosboleto["valor_boleto"] = $valor_boleto;

$dadosboleto["sacado"] = $ws->GetPagadorNome();
$dadosboleto["endereco1"] = $ws->GetPagadorLogradouro() . ' - ' . $ws->GetPagadorBairro();
$dadosboleto["endereco2"] = $ws->GetPagadorCidade() . ' - ' . $ws->GetPagadorUf() . ' CEP: ' . $ws->GetPagadorCep();

$dadosboleto["demonstrativo1"] = $ws->GetMensagem1();
$dadosboleto["demonstrativo2"] = $ws->GetMensagem2();
$dadosboleto["demonstrativo3"] = '';

$dadosboleto["instrucoes1"] = $ws->GetMensagem1();
$dadosboleto["instrucoes2"] = $ws->GetMensagem2();
$dadosboleto["instrucoes3"] = '';
$dadosboleto["instrucoes4"] = '';

$dadosboleto["quantidade"] = "";
$dadosboleto["valor_unitario"] = "";
$dadosboleto["aceite"] = $ws->GetFlagAceite();
$dadosboleto["especie"] = "R$";
$dadosboleto["especie_doc"] = "";

$dadosboleto["agencia"] = $ws->GetUnidade();
$dadosboleto["conta"] = $ws->GetCodigoBeneficiario();
$dadosboleto["conta_dv"] = '0';

$dadosboleto["conta_cedente"] = $ws->GetCodigoBeneficiario();
$dadosboleto["carteira"] = 'RG';

$cnpj = $ws->GetCnpj();
$dadosboleto["identificacao"] = $ws->GetIdentificacao();
$dadosboleto["cpf_cnpj"] = substr($cnpj, 1, 2) . '.' . substr($cnpj, 3, 3) . '.' . substr($cnpj, 6, 3) . '/' . substr($cnpj, 9, 4) . '-' . substr($cnpj, 13, 2);
$dadosboleto["endereco"] = $ws->GetEndereco1();
$dadosboleto["cidade_uf"] = $ws->GetEndereco2();
$dadosboleto["cedente"] = $ws->GetCedente();

include('./boletophp/funcoes_cef_sigcb.php');
include('./boletophp/layout_cef.php');

if ($ws->dev)
    $ws->ExibeErro('Consulta realizada');

exit();
```

## Debug

Informa��es da consulta podem ser verificadas acessando a vari�vel GET `DEBUG`
e um `HASH_DEBUG` secreto na URL:

    http://seusite.com/seuscript.php?DEBUG=8a7478ca4d29abecb82118cc089fc7c057b0d0872a34d0ee1400b2076258f67
