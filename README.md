# FidelidadeScript

**Integrantes (4 a 6 alunos):** Felipe Ferreira, Igor Xavier, Nicole Vieira, Sthefany Alves, Vitória Santos

## Tema

FidelidadeScript é uma DSL para descrever regras de pontos e cashback em programas de fidelidade. O compilador, implementado em Python no notebook [`main.ipynb`](./main.ipynb), transforma o código da linguagem em instruções TAC e as executa em uma máquina virtual.

## Ficha da linguagem

| Item                | Definição                                                                                                                          |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Nome                | FidelidadeScript                                                                                                                   |
| Domínio             | Programas de pontos e cashback                                                                                                     |
| Tipos               | `inteiro`, `decimal`, `texto`, `logico`                                                                                            |
| Comandos do domínio | `creditar`, `resgatar`, `expirar`, `extrato`                                                                                       |
| Regras de negócio   | Cashback acima de 100% e pontos negativos são erros; resgatar ou expirar mais pontos do que o saldo disponível é erro de execução. |

```text
creditar "<cliente>", <pontos>;
resgatar "<cliente>", "<cupom>", <pontos>;
expirar "<cliente>", <pontos>;
extrato "<cliente>";
```

## Tokens

O lexer reconhece 38 tipos de token e 17 palavras reservadas. Espaços em branco e comentários `//` são ignorados.

| Categoria                    | Tipos de token                                                                                           | Exemplos                               |
| ---------------------------- | -------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| Tipos e comandos reservados  | `T_INTEIRO`, `T_DECIMAL`, `T_TEXTO`, `T_LOGICO`, `MOSTRAR`, `CREDITAR`, `RESGATAR`, `EXPIRAR`, `EXTRATO` | `inteiro`, `decimal`, `creditar`       |
| Controle e lógica reservados | `SE`, `SENAO`, `ENQUANTO`, `E`, `OU`, `NAO`, `VERDADEIRO`, `FALSO`                                       | `se`, `enquanto`, `e`, `verdadeiro`    |
| Literais e identificadores   | `NUM_DECIMAL`, `NUM_INTEIRO`, `TEXTO_LIT`, `NOME`                                                        | `10.5`, `10`, `"cliente_42"`, `pontos` |
| Comparação                   | `MAIOR_IGUAL`, `MENOR_IGUAL`, `IGUAL_IGUAL`, `DIFERENTE`, `MAIOR`, `MENOR`                               | `>=`, `<=`, `==`, `!=`, `>`, `<`       |
| Aritmética e atribuição      | `ATRIB`, `MAIS`, `MENOS`, `VEZES`, `DIVIDE`                                                              | `=`, `+`, `-`, `*`, `/`                |
| Pontuação                    | `PONTO_VIRGULA`, `VIRGULA`, `ABRE_CHAVE`, `FECHA_CHAVE`, `ABRE_PAR`, `FECHA_PAR`                         | `;`, `,`, `{`, `}`, `(`, `)`           |

## Gramática

A gramática é implementada com Lark e parser LALR. A estrutura simplificada abaixo resume as instruções e a precedência dos operadores, da menor para a maior:

```ebnf
programa       = { comando } ;
comando        = declaracao | atribuicao | mostrar | se | enquanto
               | creditar | resgatar | expirar | extrato ;
tipo           = "inteiro" | "decimal" | "texto" | "logico" ;
nome           = identificador ;
declaracao     = tipo, nome, [ "=", expressao ], ";" ;
atribuicao     = nome, "=", expressao, ";" ;
mostrar        = "mostrar", expressao, ";" ;
se             = "se", expressao, bloco, [ "senao", bloco ] ;
enquanto       = "enquanto", expressao, bloco ;
creditar       = "creditar", expressao, ",", expressao, ";" ;
resgatar       = "resgatar", expressao, ",", expressao, ",", expressao, ";" ;
expirar        = "expirar", expressao, ",", expressao, ";" ;
extrato        = "extrato", expressao, ";" ;
bloco          = "{", { comando }, "}" ;

expressao      = logica_ou ;
logica_ou      = logica_e, { "ou", logica_e } ;
logica_e       = logica_nao, { "e", logica_nao } ;
logica_nao     = "nao", logica_nao | comparacao ;
comparacao     = soma, { operador_comparacao, soma } ;
soma           = produto, { ("+" | "-"), produto } ;
produto        = unario, { ("*" | "/"), unario } ;
unario         = ("+" | "-"), unario | atomo ;
atomo          = literal | nome | "(", expressao, ")" ;
literal        = numero_inteiro | numero_decimal | texto | "verdadeiro" | "falso" ;
operador_comparacao = ">" | "<" | ">=" | "<=" | "==" | "!=" ;
```

## Como executar

1. Instale Python e, na pasta do projeto, as dependências:

   ```powershell
   python -m pip install -r requirements.txt
   ```

2. Abra `main.ipynb` no Jupyter ou VS Code e selecione esse ambiente Python.
3. Use **Reiniciar e executar tudo** para executar o compilador, os testes e criar o painel interativo.
4. No painel FidelidadeScript Studio, selecione um exemplo ou edite o programa e clique em **Compilar e executar**. As abas mostram Execução, Tokens, TAC e AST; a opção **Executar TAC otimizado** alterna a execução otimizada.

## Exemplo de programa e saída

```text
decimal compra = 250.0;
decimal percentual_cashback = 10.0;
decimal cashback = compra * percentual_cashback / 100.0;
inteiro pontos = 100;

se compra >= 200.0 {
    pontos = pontos + 50;
}

mostrar "Cashback calculado:";
mostrar cashback;
creditar "cliente_42", pontos;
extrato "cliente_42";
resgatar "cliente_42", "cupom_10", 100;
expirar "cliente_42", 25;
extrato "cliente_42";
```

Saída da máquina virtual:

```text
Cashback calculado:
25.0
Crédito: cliente_42 recebeu 150 pontos (saldo: 150).
Extrato: cliente_42 possui 150 pontos.
Resgate: cupom cupom_10 usado por cliente_42; 100 pontos (saldo: 50).
Expiração: 25 pontos expiraram para cliente_42 (saldo: 25).
Extrato: cliente_42 possui 25 pontos.
```

Resumo: 23 instruções executadas, 1 crédito, 1 resgate, 1 expiração, 2 consultas de extrato e saldo final de 25 pontos.

## Contribuições

| Integrante        | RA      | Atuação                                                                    |
| ----------------- | ------- | -------------------------------------------------------------------------- |
| _Felipe Ferreira_ | 2993620 | _Organização de código, Implementação de commits e criação de repositório_ |
| _Igor Xavier_     | 2352643 | _Atuação na ficha de fidelidade e análise léxica_                          |
| _Nicole Vieira_   | 2438577 | _Execução única e testes automáticos_                                      |
| _Sthefany Alves_  | 2457820 | _Código de três endereços e otimização_                                    |
| _Vitória Santos_  | 2392314 | _Análise sintática e semântica_                                            |

O histórico das etapas de implementação está em [`CHANGELOG.md`](./CHANGELOG.md).
