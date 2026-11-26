# FidelidadeScriptFMU

**Integrantes (4 a 6 alunos):** _Preencher com o nome de todos os integrantes._

Projeto acadêmico de um compilador para **FidelidadeScript**, uma linguagem específica de domínio (DSL) voltada à definição de regras de pontos e cashback em programas de fidelidade. A proposta é permitir descrever operações de forma simples e legível, com variáveis, expressões, condições e comandos próprios do domínio.

## Exemplo

```text
decimal compra = 250.0;
inteiro pontos = 0;
se compra >= 200.0 {
    pontos = pontos + 50;
}
creditar "cliente_42", pontos;
resgatar "cliente_42", "cupom_10", 100;
```

Nesse exemplo, a condição da compra determina quantos pontos serão creditados ao cliente, e depois parte do saldo desse mesmo cliente é usada em um cupom.

## Comandos e regras de negócio

A linguagem é planejada em torno dos seguintes comandos:

- `creditar`: adicionar pontos à conta de um cliente.
- `resgatar`: aplicar um cupom ao cliente informado e debitar pontos de sua conta.
- `expirar`: expirar pontos conforme as regras do programa.
- `extrato`: consultar o histórico ou saldo de pontos.

O compilador deverá validar regras do domínio, incluindo o limite de 100% para o percentual de cashback, a proibição de pontos negativos e erros de execução ao tentar resgatar ou expirar mais pontos do que o saldo disponível.

## Requisitos do projeto final

O projeto deve construir, em Python, um compilador completo para uma DSL criada pelo grupo, do código-fonte até a execução e passando pelas fases estudadas durante o semestre. A linguagem precisa refletir o domínio escolhido pelo grupo.

### Formato de entrega

- Grupo de **4 a 6 alunos**.
- Entrega por meio do link de um repositório **público** no GitHub.
- O README deve explicar o projeto e começar com o nome de todos os integrantes.
- Apenas um membro do grupo realiza a entrega.
- O compilador deve estar em um notebook `.ipynb` que execute do início ao fim usando **Reiniciar e executar tudo**, sem erros. A entrega somente em arquivos `.py` tem desconto de **0,5 ponto**.
- O prazo é o informado pelo professor.

Como ponto de partida, a **Célula 12** do notebook de referência apresenta a ficha da linguagem, uma função para conferir os requisitos mínimos e indica o que adaptar em cada passo.

## Requisitos de qualidade — ficha da linguagem

Preencham a ficha **antes de implementar o compilador** e executem `conferir_ficha(MINHA_FICHA)`. A ficha deve conter:

- nome da linguagem e domínio;
- pelo menos **3 tipos**;
- pelo menos **2 comandos do domínio**;
- pelo menos **2 regras de negócio**, que serão tratadas como erros ou avisos semânticos;
- pelo menos **3 programas-exemplo**, escritos antes do compilador.

Use este modelo no projeto e substitua os itens `TODO` e exemplos pelos dados da DSL do grupo:

```python
MINHA_FICHA = {
    "nome": "TODO_Script",  # ex.: "ReceitaScript"
    "dominio": "TODO",  # ex.: "receitas de cozinha"
    "tipos": ["inteiro", "decimal", "texto", "logico"],  # mínimo 3
    "comandos_dominio": [  # mínimo 2
        # "adicionar <ingrediente>, <quantidade>;",
        # "assar <minutos>, <temperatura>;",
    ],
    "regras_de_negocio": [  # mínimo 2
        # "temperatura do forno entre 100 e 300 (erro)",
        # "quantidade > 0 (erro)",
    ],
    "programas_exemplo": [  # mínimo 3, definidos antes do compilador
        # """inteiro porcoes = 4; ...""",
    ],
}

def conferir_ficha(f):
    checagens = [
        ("nome e domínio preenchidos", "TODO" not in f["nome"] + f["dominio"]),
        ("pelo menos 3 tipos", len(f["tipos"]) >= 3),
        ("pelo menos 2 comandos do domínio", len(f["comandos_dominio"]) >= 2),
        ("pelo menos 2 regras de negócio", len(f["regras_de_negocio"]) >= 2),
        ("pelo menos 3 programas-exemplo", len(f["programas_exemplo"]) >= 3),
    ]
    for texto, ok in checagens:
        print(("✅" if ok else "⬜"), texto)
    faltam = sum(not ok for _, ok in checagens)
    print(
        "\n🚀 Ficha completa! Agora adapte os Passos 1 a 9."
        if not faltam
        else f"\nFaltam {faltam} item(ns) na ficha."
    )

conferir_ficha(MINHA_FICHA)
```

## Requisitos funcionais e etapas de desenvolvimento

Cada requisito R1–R9 corresponde a um estágio do projeto. O desenvolvimento começa pelo **R1 — ficha e programas-exemplo** e avança pelas fases do compilador.

### R1 — Ficha e programas-exemplo

- `MINHA_FICHA` deve estar preenchida e passar por todos os itens de `conferir_ficha`.
- Incluir **3 programas-exemplo** na DSL; cada programa deve conter pelo menos um `se` ou um `enquanto`.

### R2 — Análise léxica

- Reconhecer pelo menos **20 tipos de token** e **8 palavras reservadas**, incluindo os comandos do domínio.
- Ignorar comentários.
- Exibir a tabela de tokens de um programa.
- Para erros léxicos, informar linha, coluna e uma dica; implementar pelo menos **3 dicas específicas**.

#### Implementação atual

O Passo 1 está implementado em `main.ipynb`. O lexer Lark reconhece **38 tipos de token** e **17 palavras reservadas**, incluindo os comandos do domínio:

`creditar`, `resgatar`, `expirar` e `extrato`.

As demais palavras reservadas são `inteiro`, `decimal`, `texto`, `logico`, `mostrar`, `se`, `senao`, `enquanto`, `e`, `ou`, `nao`, `verdadeiro` e `falso`. Os tipos também cobrem identificadores, literais inteiros, decimais e de texto, operadores de comparação e aritméticos, e pontuação como vírgulas, ponto e vírgula, parênteses e chaves.

Espaços em branco e comentários iniciados por `//` são ignorados. A função `mostrar_tokens(PROGRAMA_FIDELIDADE)` exibe a tabela completa do programa de exemplo, com tipo do token, lexema, linha e coluna.

Erros léxicos são encapsulados em `ErroCompilacao` e exibem fase, linha, coluna e dica. Há dicas específicas para aspas simples, textos sem fechamento, os operadores `&`, `|` e `!`, comentários iniciados por `#`, o símbolo `%` e identificadores com acentos.

### R3 — Análise sintática e AST

- Usar gramática Lark com parser **LALR**.
- Suportar declarações, atribuições, expressões com pelo menos **4 níveis de precedência**, `se`/`senao`, `enquanto` e pelo menos **2 comandos do domínio**.
- Construir a AST com `dataclass` e exibi-la por uma função como `mostrar_ast`.
- Erros sintáticos devem informar linha, coluna e o que era esperado.

#### Implementação atual

O Passo 2 está implementado em `main.ipynb`. A gramática Lark usa o parser **LALR** e reutiliza os terminais do Passo 1. Ela reconhece declarações, atribuições, `mostrar`, blocos, `se`/`senao`, `enquanto` e os comandos `creditar`, `resgatar`, `expirar` e `extrato`.

O comando `resgatar` recebe cliente, cupom e quantidade nessa ordem: `resgatar "<cliente>", "<cupom>", <pontos>;`.

As expressões têm precedência, da menor para a maior, para `ou`, `e`, `nao`, comparações (`>`, `<`, `>=`, `<=`, `==`, `!=`), soma/subtração, multiplicação/divisão e operadores unários/parênteses.

O `MontaAST` transforma a árvore sintática em nós `dataclass`, incluindo literais, variáveis, operações, declarações, atribuições, controle de fluxo e os quatro comandos de domínio. `mostrar_ast` apresenta a árvore do programa de demonstração.

Erros sintáticos são convertidos em `ErroCompilacao` com fase, linha, coluna e uma lista legível dos terminais esperados. A célula demonstra programas com `se`/`senao`, `enquanto` e comandos do domínio, além de entradas com erros sintáticos.

### R4 — Análise semântica

- Implementar tabela de símbolos com **escopo de bloco**.
- Suportar pelo menos **3 tipos**, com coerção ou erro de tipo.
- Validar pelo menos **2 regras de negócio** do domínio, sendo pelo menos uma delas um erro, e implementar pelo menos um tipo de aviso.
- Demonstrar uma galeria com pelo menos **6 erros semânticos diferentes**. Todos os casos devem passar pelas fases léxica e sintática.

#### Implementação atual

O Passo 3 está implementado em `main.ipynb`. `TabelaSimbolos` mantém uma pilha de escopos: cada bloco de `se`, `senao` ou `enquanto` cria um escopo local, permitindo sombra de nomes e impedindo o uso de variáveis locais após o bloco.

`AnalisadorSemantico` verifica os tipos `inteiro`, `decimal`, `texto` e `logico`. Ele converte automaticamente `inteiro` para `decimal` por meio do nó `Conv` da AST e registra erros para conversões incompatíveis, operações entre tipos incompatíveis e condições que não sejam lógicas.

As regras de domínio verificadas são percentual de cashback acima de 100% e quantidades de pontos negativas, ambas como erros. Cashback de 0%, crédito de zero pontos e variáveis declaradas mas não usadas geram avisos, sem serem classificados como erros. A validação de resgate acima do saldo disponível é uma regra de execução prevista para o R7.

A `GALERIA_SEMANTICA` contém **8 casos**: variável não declarada, tipo incompatível, declaração duplicada, condição não lógica, operação entre texto e número, variável fora do escopo, cashback acima do limite e pontos negativos. Cada fonte é analisada primeiro por `analisar_sintaxe`; o estágio então confirma que ela produz diagnóstico semântico. Os resultados ficam separados em `analisador.erros` e `analisador.avisos`.

### R5 — Código de Três Endereços

- Gerar rótulos e desvios usando a **condição invertida**.
- Gerar uma instrução própria para cada comando do domínio.
- Exibir o TAC de um programa que contenha um laço e `se`/`senao`.

#### Implementação atual

O Passo 4 está implementado em `main.ipynb`. `gerar_tac` recebe um `Programa` da AST e retorna uma lista estruturada de `InstrucaoTAC`; `mostrar_tac` exibe essa lista. Expressões são reduzidas a operações de três endereços com temporários, incluindo a coerção `inteiro` para `decimal` como `I2F`.

Durante a geração, o gerador resolve nomes pelos escopos de bloco da linguagem e atribui nomes internos distintos às variáveis locais sombreadas, preservando a resolução da análise semântica.

Desvios relacionais são gerados com o operador invertido para alcançar o rótulo falso: por exemplo, `compra >= 200.0` torna-se um salto `IF compra < 200.0 GOTO ...`. `e`, `ou` e `nao` preservam o fluxo lógico usando rótulos e desvios, e o `enquanto` gera rótulo de início, saída e retorno ao teste.

Cada comando do domínio tem sua própria instrução TAC: `CREDITAR`, `RESGATAR`, `EXPIRAR` e `EXTRATO`. A demonstração `PROGRAMA_TAC` inclui `se`/`senao`, um laço e os quatro comandos; sua geração também verifica os desvios invertidos e a presença de rótulos.

### R6 — Otimização

- Implementar pelo menos **3 técnicas** de otimização e mostrar o código antes e depois.
- Em pelo menos **2 programas**, provar que a saída otimizada é igual à original e que o número de instruções executadas diminuiu.

#### Implementação atual

O Passo 5 está implementado em `main.ipynb` com quatro técnicas: folding de constantes, propagação de constantes, simplificação algébrica (por exemplo, `x + 0` e `x * 1`) e eliminação de temporários mortos. A propagação é interrompida em rótulos e desvios para não transportar valores por caminhos de controle incertos. Temporários são identificados separadamente dos nomes da DSL; a eliminação conserva operações que podem produzir erros em execução, como divisão ou conversão.

As demonstrações exibem o TAC antes e depois e executam ambos para comparar a saída e contar as instruções:

| Programa | Saída original e otimizada | Instruções executadas |
|---|---:|---:|
| `inteiro total = 2 + 3; mostrar total;` | `[5]` | 4 → 3 |
| `inteiro pontos = 10; pontos = pontos + 0; mostrar pontos;` | `[10]` | 5 → 4 |

O executor dessa comparação cobre as instruções aritméticas e de saída usadas nesses exemplos; a máquina virtual completa para o TAC será implementada no R7.

### R7 — Execução

- Implementar uma máquina virtual que execute o TAC.
- Cada comando do domínio deve produzir saída e alimentar pelo menos uma estatística exibida no resumo.
- Tratar pelo menos **2 erros de execução**, como divisão por zero, laço infinito ou violação de regra de negócio com valores reais.

#### Implementação atual

O Passo 6 está implementado em `main.ipynb` pela classe `MaquinaVirtual`. Ela executa declarações, movimentações, operações aritméticas e lógicas, desvios, rótulos e as quatro instruções de domínio. `ResultadoExecucao` reúne a saída, a memória, os saldos e o resumo estatístico.

Os saldos são mantidos por cliente. A assinatura de resgate é `RESGATAR cliente, cupom, pontos`, correspondente à sintaxe `resgatar "<cliente>", "<cupom>", <pontos>;`. Crédito, resgate, expiração e extrato geram saída e atualizam contadores e totais próprios. A demonstração exercita os quatro comandos, `se`/`senao` e `enquanto`.

A VM trata divisão por zero, resgate ou expiração acima do saldo, pontos inválidos, operações com operandos inválidos e desvios para rótulos inexistentes. Para detectar laços infinitos, o padrão é limitar a execução a **10.000 instruções**; o limite pode ser ajustado ao instanciar a máquina. A célula demonstra e verifica os diagnósticos de divisão por zero, saldo insuficiente e limite de instruções.

### R8 — Testes automáticos

- Implementar pelo menos **12 casos**, incluindo pelo menos **4 programas válidos**.
- Incluir pelo menos um erro de cada fase: léxica, sintática, semântica e execução.
- Todos os casos devem passar.

### R9 — Interface e README

- Criar um painel com `ipywidgets` que apresente os exemplos da linguagem e ofereça pelo menos **3 visões**, como Execução, Tokens e TAC.
- O README final deve incluir nomes dos integrantes, tema, ficha da linguagem, tabela de tokens, gramática, instruções para executar, exemplos de saída e a contribuição de cada integrante.

## Preparar o ambiente Python

O desenvolvimento e a execução do compilador ficam concentrados no notebook `main.ipynb`, que é o único arquivo executável do projeto. As dependências externas estão centralizadas em `requirements.txt`.

No terminal, a partir da pasta do projeto e com o interpretador Python desejado selecionado:

```powershell
python -m pip install -r requirements.txt
```

No Jupyter, execute em uma célula antes de usar o notebook:

```python
%pip install -r requirements.txt
```

Abra `main.ipynb` no VS Code ou Jupyter e selecione o mesmo ambiente Python em que as dependências foram instaladas. Para Google Colab, disponibilize `requirements.txt` no diretório de trabalho antes de executar o comando.
