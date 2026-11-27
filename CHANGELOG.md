# Changelog

Histórico das etapas de implementação do compilador FidelidadeScript. Este arquivo registra a evolução técnica do projeto; a visão de entrega e uso está no [README](./README.md).

## Preparação

- Centralizadas as dependências Python em `requirements.txt` para instalação com um único comando.
- Ajustado `.gitignore` para não ignorar `main.ipynb` junto com outros notebooks.
- Definido `main.ipynb` como notebook principal de implementação; `Projeto-final.ipynb` foi usado apenas como referência.

## R1 — Ficha e programas-exemplo

- Definida a ficha FidelidadeScript para o domínio de pontos e cashback.
- Incluídos quatro tipos, quatro comandos de domínio, regras de negócio e três programas-exemplo.
- A validação da ficha exige exemplos com `se` ou `enquanto`.
- Registradas regras para cashback acima de 100%, pontos negativos e operações acima do saldo.

## R2 — Análise léxica

- Implementado lexer Lark com 38 tipos de token e 17 palavras reservadas.
- Comentários `//` e espaços em branco são ignorados.
- A tabela de tokens apresenta tipo, lexema, linha e coluna.
- Erros léxicos incluem localização e dicas específicas, por exemplo para aspas simples, operadores lógicos inválidos e identificadores com acento.

## R3 — Análise sintática e AST

- Criada gramática Lark com parser LALR para declarações, atribuições, expressões, blocos, condicionais, laços e comandos de domínio.
- Definidos nós de AST com `dataclass` e exibição por `mostrar_ast`.
- A precedência das expressões cobre lógica, comparação, soma, multiplicação e operadores unários.
- Erros sintáticos informam linha, coluna e terminais esperados.
- A forma de resgate definida é `resgatar "<cliente>", "<cupom>", <pontos>;`.

## R4 — Análise semântica

- Implementada tabela de símbolos com escopo de bloco e suporte a sombra de nomes.
- Verificados os tipos `inteiro`, `decimal`, `texto` e `logico`, incluindo coerção de inteiro para decimal.
- Adicionadas regras semânticas para cashback e pontos, além de avisos.
- Criada galeria com oito erros semânticos que passam pelas fases léxica e sintática.

## R5 — Código de Três Endereços

- Implementado gerador de TAC com temporários, rótulos e desvios condicionais invertidos.
- Preservada a resolução de variáveis em escopos e nomes sombreados.
- Criadas instruções TAC próprias para `CREDITAR`, `RESGATAR`, `EXPIRAR` e `EXTRATO`.
- Exibido TAC de exemplo com `se`/`senao`, laço e comandos do domínio.

## R6 — Otimização

- Implementados folding de constantes, propagação de constantes, simplificação algébrica e eliminação de temporários mortos.
- Mantidas operações com possíveis efeitos de erro durante a eliminação de código morto.
- Demonstrada equivalência de saída e redução de instruções executadas em dois programas: 4 → 3 e 5 → 4.

## R7 — Execução

- Implementada máquina virtual para as instruções TAC, com memória, saldos por cliente, saída e estatísticas.
- Os quatro comandos de domínio produzem saída e atualizam contadores e totais.
- Tratados divisão por zero, saldo insuficiente, operandos inválidos, rótulos inválidos e limite de instruções para laços potencialmente infinitos.
- `resgatar` recebe cliente, cupom e pontos; `expirar` acima do saldo também gera erro de execução.

## R8 — Testes automáticos

- Adicionada `compilar()` para integrar léxico, sintaxe, semântica, geração de TAC, otimização opcional e execução.
- Criados 16 casos de integração: quatro programas válidos e erros léxicos, sintáticos, semânticos e de execução.
- Acrescentados 12 testes de regressão cobrindo os contratos implementados de R1 a R7.
- A suíte conjunta contém 28 casos/verificações e interrompe a execução se uma asserção falhar.

## R9 — Interface e documentação

- Criado o painel FidelidadeScript Studio com `ipywidgets`, seleção de exemplos, editor e opção de execução com TAC otimizado.
- Disponibilizadas as visões Execução, Tokens, TAC e AST.
- Erros de compilação e execução são apresentados no painel.
- Organizada a documentação: o README contém as informações de entrega e uso previstas em R9; este changelog concentra o histórico das etapas.
