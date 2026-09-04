# Checkpoint 4 — Bug Hunt StreamFIAP

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

### 1. Injeção de dependência (Aula 13)
O Spring gerencia o ciclo de vida do `ConteudoRepository` como um *bean*: ele cria a implementação real da interface (proxy do Spring Data JPA), abre/gerencia a conexão com o banco e injeta essa instância pronta no `ConteudoController` via `@Autowired`. Se fizéssemos `new ConteudoRepository()`, não funcionaria porque `ConteudoRepository` é uma *interface* — não existe implementação escrita por nós; quem gera a implementação em tempo de execução é o container do Spring, com base no contrato da interface. Além disso, deixar o Spring injetar permite trocar a implementação (ex.: um mock em teste) sem mudar o controller.

### 2. JDBC vs Spring Data JPA (Aulas 12 e 13)
No `ProdutoDAO` da Aula 12 escrevíamos manualmente `Connection`, `PreparedStatement`, `ResultSet`, tratávamos `SQLException` e convertíamos linha a linha para objeto. O Spring Data JPA automatiza tudo isso: só declarando a interface `ConteudoRepository extends JpaRepository<Conteudo, Long>` já temos `save`, `findAll`, `findById` etc. prontos. O `findByCategoria(String categoria)` funciona sem implementação porque o Spring Data interpreta o **nome do método** (`findBy` + nome do atributo `Categoria`) e gera a query JPQL/SQL automaticamente. O JDBC/DAO na mão ainda vale a pena quando a query é muito específica/complexa e o "query derivation" do Spring Data não dá conta.

### 3. Exceções checked vs unchecked (Aula 11)
`ClassificacaoIndicativaException extends Exception` é uma exceção *checked*: o compilador obriga a declarar `throws` (por isso `Usuario.alugar` e `AluguelController.alugar` têm `throws ClassificacaoIndicativaException`). O bug era que, mesmo sendo lançada corretamente, não existia `@ExceptionHandler` para ela no `GlobalExceptionHandler` — então o Spring devolvia o erro 500 padrão, sem a mensagem da regra de negócio. A correção foi adicionar um handler específico que captura essa exceção e devolve um JSON com a mensagem (`403 Forbidden`), como já era feito para as exceções *unchecked* (`CreditosInsuficientesException`, `ConteudoIndisponivelException`, que estendem `RuntimeException` e por isso não exigem `throws` na assinatura).

### 4. Sobrescrita vs sobrecarga (Aula 7)
A `Serie` tinha `public double calcularPrecoAluguel(double desconto)`, com um parâmetro a mais do que `Conteudo.calcularPrecoAluguel()`. Isso não é *override* — é *overload*: um método novo, com assinatura diferente, que coexiste com o método herdado em vez de substituí-lo. Como ninguém nunca chamava a versão com parâmetro, toda série calculava o preço pelo método padrão da superclasse (R$ 9,90 fixo), ignorando completamente `numeroTemporadas`. Se a anotação `@Override` estivesse presente desde o início, o compilador teria acusado erro (`method does not override a method from its superclass`), pois a assinatura não batia — o bug teria sido pego na hora de compilar, e não em produção.

### 5. Onde blindar o objeto? (Aulas 3, 4 e 13)
Encontramos dois bugs de dados inválidos: duração `<= 0` sendo aceita e créditos podendo ficar negativos. A duração é uma invariante do próprio objeto `Conteudo` — não faz sentido existir um `Conteudo` com duração zero/negativa em nenhum contexto, então a validação foi colocada no **construtor** da superclasse, garantindo que nenhum objeto inválido seja instanciado. Já a regra de créditos suficientes depende de uma *interação* entre dois objetos (`Usuario` e `Conteudo`) no momento do aluguel — por isso ela mora em `Usuario.alugar()`/`temCreditosSuficientes()`, um método de negócio, e não num setter isolado. Validar só no setter não seria suficiente, porque o setter não sabe qual é o preço do conteúdo sendo alugado; e validar só no controller não seria suficiente porque outra parte do sistema poderia criar o objeto sem passar pelo controller (ex.: testes, outro service). A regra tem que estar onde a decisão é tomada.

### 6. Abstração e interface (Aulas 8 e 9)
`Conteudo` é uma classe abstrata porque representa um conceito comum (`titulo`, `duracaoMinutos`, `calcularPrecoAluguel`...) que **nunca existe sozinho** — só existe como Filme, Série ou Documentário; ela também compartilha estado (atributos) com as subclasses via herança. `Promocionavel` é uma interface porque representa uma **capacidade** (poder ter desconto) que é ortogonal à hierarquia de herança — Filme e Série têm essa capacidade, Documentário não, sem que isso mude a árvore de herança. Se o Documentário passasse a ter promoções, bastaria: (1) fazer `Documentario implements Promocionavel` e (2) implementar `aplicarPromocao(double preco)`. Nenhuma linha de `Conteudo`, `Filme` ou `Serie` precisaria mudar, porque `calcularPrecoPromocional()` já verifica `instanceof Promocionavel` de forma genérica. Isso mostra que separar "o que a classe é" (herança) de "o que a classe consegue fazer" (interface) deixa o sistema aberto para extensão sem exigir alteração no código existente.

---

## Parte 4 — Espaço livre (opcional)
Clean code 01 e 06 estão no mesmo commit!
