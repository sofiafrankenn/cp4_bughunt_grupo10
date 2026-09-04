# Checkpoint 4 — Bug Hunt StreamFIAP

> Copie este arquivo para a raiz do seu repositório com o nome **README.md**
> e preencha todas as seções.

## Identificação

**Grupo:** 10

| Integrante | RM | Turma |
|---|---|---|
| Ana Luiza Bertão | 563171 | 2CCPG |
| Larissa Machado | 564168 | 2CCPG |
| Raira | 564850 | 2CCPG |
| Sofia Franken | 562767 | 2CCPG |
| Anny | 565055 | 2CCPG |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 6 / 6 |

---

## Parte 1 — Bugs encontrados

> Uma linha por bug, na ordem em que você os encontrou. Use a numeração dos seus
> commits (`fix: bug01 ...`). Preencha TODAS as colunas — metade da nota está aqui.

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | `POST /api/usuarios` salvava o usuário mas o `id` voltava `null`/repetido | `Usuario.java` — campo `id` só tinha `@Id`, sem `@GeneratedValue` | Adicionado `@GeneratedValue(strategy = GenerationType.IDENTITY)` no `id` | Mapeamento JPA (Aula 13) — geração automática de chave primária |
| bug02 | Usuário cadastrado voltava com `nome` vazio, mesmo enviando o campo | `Usuario.java`, construtor — `nome = nome;` (faltava `this.`) | Trocado para `this.nome = nome;` | Encapsulamento / escopo de variável (Aula 3) — shadowing entre parâmetro e atributo |
| bug03 | Documentário era cobrado a R$ 9,90 em vez de ser gratuito | `Documentario.java` — não sobrescrevia `calcularPrecoAluguel()` | Adicionado `@Override public double calcularPrecoAluguel() { return 0.0; }` | Herança e polimorfismo (Aula 7) — subclasse precisa sobrescrever o comportamento padrão da superclasse |
| bug04 | Preço promocional do Filme ficava **maior** que o preço cheio | `Filme.java` — `aplicarPromocao` retornava `preco * 1.2` | Trocado para `preco * 0.8` (20% de desconto, conforme `Promocionavel`) | Interfaces / contrato de comportamento (Aulas 8 e 9) |
| bug05 | Série cadastrada voltava com título, categoria e classificação `null`/0 | `Serie.java`, construtor — não chamava `super(...)` | Adicionado `super(titulo, categoria, duracaoMinutos, classificacaoEtaria, true);` | Herança — encadeamento de construtores (Aula 7) |
| bug06 | Preço da série sempre caía no valor padrão (R$ 9,90), nunca em 4,90×temporadas | `Serie.java` — `calcularPrecoAluguel(double desconto)` tinha assinatura diferente da superclasse (sobrecarga, não sobrescrita) | Corrigida a assinatura para `calcularPrecoAluguel()` (sem parâmetro) + `@Override` | Override vs Overload (Aula 7) |
| bug07 | Usuário com 100 créditos era recusado por "créditos insuficientes" ao alugar algo de R$ 9,90 | `Usuario.java` — `temCreditosSuficientes` comparava `preco >= creditos` (invertido) | Corrigido para `this.creditos >= preco` | Lógica booleana / regra de negócio (Aula 4) |
| bug08 | `GET /api/conteudos/999` retornava corpo vazio com status 200, como se tivesse dado certo | `ConteudoController.buscarPorId` — `try/catch (Exception e)` engolia a exceção e retornava `null` | Removido o `try/catch`; a exceção `ConteudoNaoEncontradoException` agora propaga para o `GlobalExceptionHandler` | Tratamento de exceções (Aula 11) — nunca capturar e silenciar genericamente |
| bug09 | `GET /api/conteudos/categoria/FICCAO` sempre retornava lista vazia, mesmo com conteúdos dessa categoria | `ConteudoController.listarPorCategoria` — comparava String com `==` | Trocado para `.equals()` | Comparação de objetos vs `==` em Strings (Aula 3) |
| bug10 | Conteúdo marcado como indisponível era alugado normalmente | `Usuario.alugar` — nunca checava `c.isDisponivel()`; a exceção `ConteudoIndisponivelException` existia mas nunca era lançada | Adicionada checagem no início de `alugar()`, lançando `ConteudoIndisponivelException` | Regras de negócio no domínio / exceções customizadas (Aula 11) |
| bug11 | Era possível cadastrar filme/série/documentário com `duracaoMinutos` 0 ou negativo | `Conteudo.java`, construtor — nenhuma validação | Adicionado `if (duracaoMinutos <= 0) throw new IllegalArgumentException(...)` no construtor da superclasse | Blindagem de invariantes no construtor (Aulas 3 e 4) |
| bug12 | Aluguel de menor de idade e cadastro com duração inválida devolviam erro 500 genérico do Spring, sem mensagem clara | `GlobalExceptionHandler.java` — não tinha `@ExceptionHandler` para `ClassificacaoIndicativaException` nem para `IllegalArgumentException` | Adicionados os dois handlers, retornando JSON com a mensagem e status apropriado (403 e 400) | Exceções checked vs unchecked / `@RestControllerAdvice` (Aula 11) |

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | `Usuario.debitarCreditos` | Comentário enganoso (dizia "adiciona" enquanto o código subtrai) | Corrigido o texto do comentário para refletir o que o código faz |
| clean02 | `Usuario.alugar` | Nome de variável sem significado (`p`) | Renomeada para `preco` |
| clean03 | `ConteudoController` | Código morto: método `calcularDescontoAntigo` nunca chamado + bloco de comentário de código desativado (`TODO cupom`) | Removidos ambos — código morto/comentado não deve ficar no repositório |
| clean04 | `Conteudo`, `Filme`, `Serie` | Números mágicos (`9.90`, `5.00`, `4.90`, `0.8`, `1.2`) espalhados direto no código | Extraídos para constantes nomeadas (`PRECO_BASE`, `TAXA_ESTREIA`, `PRECO_POR_TEMPORADA`, `FATOR_DESCONTO_PROMOCAO`, `PRECO_PADRAO`) |
| clean05 | `ConteudoController.listarPorCategoria` | Duplicação de lógica: filtro reimplementado na mão enquanto o `ConteudoRepository` já tinha `findByCategoria` pronto | Método simplificado para `return conteudoRepository.findByCategoria(categoria);` |
| clean06 | `Usuario.alugar` | Mistura de responsabilidades: regra de negócio (domínio) fazendo `System.out.println` de "recibo" | Removidos os `println` do model — impressão/apresentação não é responsabilidade da entidade de domínio |

---

## Parte 3 — Perguntas de reflexão

> Responda com suas palavras, 5 a 10 linhas cada, **usando o código real do projeto
> como exemplo**. Respostas genéricas de tutorial não pontuam.

### 1. Injeção de dependência (Aula 13)
O `ConteudoRepository` é só uma interface, não existe uma classe nossa implementando ela — quem cria essa implementação é o próprio Spring, quando a aplicação sobe. Por isso não dá pra fazer `new ConteudoRepository()`: não existe corpo escrito pra essa interface no nosso código, só quem sabe montar isso é o Spring Data JPA. Quando colocamos `@Autowired` no `ConteudoController`, estamos avisando o Spring que precisamos desse objeto, e ele entrega a implementação pronta. Isso também facilita trocar essa implementação depois (por exemplo, num teste) sem precisar mudar o controller.

### 2. JDBC vs Spring Data JPA (Aulas 12 e 13)
Na Aula 12, no `ProdutoDAO`, a gente escrevia manualmente `Connection`, `PreparedStatement` e `ResultSet`, e ainda tratava `SQLException` na mão. Aqui, o `ConteudoRepository` faz o CRUD inteiro só estendendo `JpaRepository`, sem escrever nenhuma linha de implementação. O `findByCategoria` funciona porque o Spring Data lê o nome do método (`findBy` + `Categoria`) e monta a query sozinho, sem a gente escrever SQL. O JDBC ainda faz sentido quando a consulta é complexa demais pra esse padrão de nome resolver, mas pra CRUD simples o Spring Data poupa bastante código repetitivo.

### 3. Exceções checked vs unchecked (Aula 11)
`ClassificacaoIndicativaException` estende `Exception`, então é uma exceção checked — o compilador obriga a declarar `throws` em quem a usa, e é por isso que `alugar()` e `AluguelController` têm esse `throws` na assinatura. O bug era que, mesmo sendo lançada corretamente, o `GlobalExceptionHandler` não tinha nenhum `@ExceptionHandler` pra ela, então o Spring devolvia o erro 500 genérico em vez da mensagem da regra de negócio. Corrigimos adicionando um handler específico pra essa exceção, do mesmo jeito que já existia pras exceções unchecked (`CreditosInsuficientesException` e `ConteudoIndisponivelException`, que estendem `RuntimeException` e por isso não precisam de `throws`).

### 4. Sobrescrita vs sobrecarga (Aula 7)
Esse foi o bug mais difícil de achar. O método da `Serie` era `calcularPrecoAluguel(double desconto)`, com um parâmetro a mais do que o método da `Conteudo`. Parecia que estava sobrescrevendo, mas na verdade é overload: um método novo, com assinatura diferente, que fica ao lado do herdado em vez de substituí-lo. Como ninguém chamava essa versão com parâmetro, toda série usava o preço padrão da superclasse (9,90) e ignorava o número de temporadas. Se a gente tivesse colocado `@Override` desde o início, o compilador teria acusado erro, porque a assinatura não bate com nenhum método da superclasse — o bug teria aparecido na hora de compilar.

### 5. Onde blindar o objeto? (Aulas 3, 4 e 13)
Encontramos bugs de duração zerada/negativa sendo aceita e de créditos que podiam ficar negativos. A duração é uma regra que vale sempre pra qualquer conteúdo, então colocamos a validação no construtor da `Conteudo` — assim nenhum objeto inválido chega a existir. Já a regra de créditos suficientes depende do preço do conteúdo que está sendo alugado no momento, então não daria pra validar isso num setter isolado, porque o setter não tem essa informação; por isso ela ficou dentro do próprio `alugar()`, que é onde as duas informações se encontram. Ou seja: regra que é sempre verdadeira sobre o objeto fica no construtor; regra que depende de uma interação com outro objeto fica no método que participa dessa interação.

### 6. Abstração e interface (Aulas 8 e 9)
`Conteudo` é abstrata porque nunca deveria existir sozinha — só faz sentido como Filme, Série ou Documentário, e por isso concentra os atributos e métodos comuns aos três. Já `Promocionavel` é interface porque representa uma capacidade separada da hierarquia: só Filme e Série têm promoção, Documentário não, e isso não muda a árvore de herança. Se o Documentário passasse a ter promoção, bastaria fazer ele implementar `Promocionavel` e escrever o próprio `aplicarPromocao` — nenhuma linha de `Conteudo`, `Filme` ou `Serie` precisaria mudar, porque `calcularPrecoPromocional()` já verifica quem é `Promocionavel` de forma genérica. Isso mostra que separar "o que a classe é" de "o que ela consegue fazer" deixa o sistema mais fácil de estender sem mexer no que já funciona.

---

## Parte 4 — Espaço livre (opcional)
O clean code 02 e 06 estão no mesmo commit!
