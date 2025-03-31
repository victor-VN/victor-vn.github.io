---
title: "Como fazer um endpoint com multiplos filtros usando Spring e Hibernate (JPA)"
categories:
  - Java
  - Banco
  - DB
  - JPA
  - Hibernate
  - SQL
tags:
  - java
  - banco
  - hibernate
  - jpa
  - sql
toc: true
toc_label: "Conteúdo"
toc_icon: "file"
---

Como eu crio um endpoint com filtros dinâmicos utilizando Spring e Hibernate (JPA)

### Cenário

Se você já precisou construir um _endpoint_ que recebe _n_ parâmetros mas precisa que sua aplicação consulte o banco dinamicamente de acordo com os parâmetros informados e ignore aqueles que não foram informados, sabe como pode ser desafiador escolher uma boa estratégia.

Imagine que, no nosso banco de dados, temos uma tabela com a seguinte estrutura

```sql
CREATE TABLE empresa (
    cpf VARCHAR(14) NOT NULL PRIMARY KEY,
    nome VARCHAR(255) NOT NULL,
    estado VARCHAR(255) NOT NULL,
    uf VARCHAR(2) NOT NULL,
    numero_funcionarios INT NOT NULL,
    qtd_filiais INT NOT NULL
);

```

Nos foi solicitado que criássemos um endpoint que vai filtrar os parceiros comerciais com base nos campos CPF, nome e estado. A questão é que temos aqui 7 combinações diferentes, se consideramos que todos os parâmetro nulos é uma combinação possível (afinal talvez a gente não queira aplicar parâmetro nenhum) 

### Utilizando um método para cada

A primeira vez que me deparei com esse problema, a minha solução foi criar uma query JPA para cada combinação e validar quais dos parâmetros haviam sido informados. Dessa forma eu podia chamar a query específica para cada cenário. Meu código ficou mais ou menos assim:

Repository com cada combinação de filtros

```java
public interface ParceiroRepository extends JpaRepository<ParceiroEntity, String> {

    public ParceiroEntity findByCpf(String cpf);

    public ParceiroEntity findByCpfAndNome(String cpf, String nome);

    // demais combinações abaixo
}
```

Service com as combinações de filtro e validação das combinações: 
```java
@Service
public class ParceiroService {

    private final ParceiroRepository repository;

    public ParceiroService(ParceiroRepository repository) {
        this.repository = repository;
    }

    public ParceiroEntity findByParams(ParceiroRequestParams params){
        if (params.getCpf() != null && params.getEstado() == null && params.getNome() == null)
            return repository.findByCpf(params.getCpf());

        if (params.getCpf() == null && params.getNome() != null && params.getEstado() != null)
            return repository.findByCpfAndNome(params.getCpf(), params.getNome());

        throw new IllegalArgumentException("Parâmetros informados não correspondem a nenhuma combinação permitida.");
    }

    public ParceiroEntity findByCpf(String cpf){
        return repository.findByCpf(cpf);
    }

    public ParceiroEntity findByCpfAndNome(String cpf, String nome){
        return repository.findByCpfAndNome(cpf, nome);
    }

    // demais combinações abaixo
}

```

Controller para chamar o service: 
```java
@RestController
public class ParceirosController {

    private final ParceiroService service;

    public ParceirosController(ParceiroService service) {
        this.service = service;
    }

    @GetMapping("/parceiros")
    public ResponseEntity<ParceiroEntity> getParceiros(ParceiroRequestParams params){
        return ResponseEntity.ok(service.findByParams(params));
    }
}
```

O problema é que, sempre que surgia um novo parâmetro, a quantidade de métodos e de validações aumentava. 

### Query com parâmetros dinâmicos

A implementação que mais costumo utilizar hoje em dia consiste em encarregar a minha query no banco de dados de fazer todo o tratamento necessário. 

Primeiro, crio um objeto com os filtros que vou quere aplicar na minha consulta. Cada campo representa uma coluna na tabela.

```java
public class ParceiroRequestParams {
    String cpf;
    String nome;
    String estado;
   
   // getters e setters
}
```

E passo esse mesmo objeto como parâmetro na minha query JPA

```java
@Query("SELECT e FROM Empresa e " +
            "WHERE ((:#{#params.cpf} IS NULL OR :#{#params.cpf} = '') OR e.cpf = :#{#params.cpf}) " +
            "AND ((:#{#params.nome} IS NULL OR :#{#params.nome} = '') OR LOWER(e.nome) LIKE LOWER(CONCAT('%', :#{#params.nome}, '%'))) " +
            "AND ((:#{#params.estado} IS NULL OR :#{#params.estado} = '') OR LOWER(e.estado) LIKE LOWER(CONCAT('%', :#{#params.estado}, '%'))) ")
    List<ParceiroEntity> searchEmpresas(@Param("params") ParceiroRequestParams params);
```

No meu _controller_ o mesmo objeto que eu informo ao banco representa os filtros do meu _endpoint_

```java
@GetMapping("/parceiros")
    public ResponseEntity<List<ParceiroEntity>> getParceiros(ParceiroRequestParams params){
        return ResponseEntity.ok(service.findByParams(params));
    }
```

Agora, temos uma query totalmente dinâmica. Ao rodar o código, vemos que, se informarmos diferentes parâmetros, nossa API trará os valores com base nesses parâmetros, com diferentes combinações.

### Como funciona

Para cada parâmetro na nossa query, primeiro verificamos se o valor é NULL (ou uma string vazia '' no caso de varchar), se o valor for nulo a condição automaticamente retorna TRUE, o que faz aquela condição seja automaticamente ignorada.

Se, por outro lado, o parâmetro tiver um valor, essa condição será considerada na busca no banco de dados.

A melhor parte é que caso seja necessário adicionar um novo filtro, basta adicionar uma nova propriedade no nosso objeto e uma nova condição na nossa query

```java
public class ParceiroRequestParams {
    String cpf;
    String nome;
    String estado;
    String numero;

// getters e setters
}
```

```java
@Query("SELECT e FROM Empresa e " +
            "WHERE ((:#{#params.cpf} IS NULL OR :#{#params.cpf} = '') OR e.cpf = :#{#params.cpf}) " +
            "AND ((:#{#params.nome} IS NULL OR :#{#params.nome} = '') OR LOWER(e.nome) LIKE LOWER(CONCAT('%', :#{#params.nome}, '%'))) " +
            "AND ((:#{#params.estado} IS NULL OR :#{#params.estado} = '') OR LOWER(e.estado) LIKE LOWER(CONCAT('%', :#{#params.estado}, '%'))) " +
            "AND ((:#{#params.numero} IS NULL) OR e.numeroFuncionarios = :#{#params.numero})")
    List<ParceiroEntity> searchEmpresas(@Param("params") ParceiroRequestParams params);
```

### Conclusão

Concentrando toda a lógica dos filtros na query, evitamos sobrecarregar o código com validações e, ao mesmo tempo, permitimos maior flexibilidade para adicionar novos parâmetros.

Uma contrapartida desta abordagem é que a sintaxe da query acaba ficando muito complexa o que pode causar dificuldade pra quem for ler seu código. 

Código da API no github: <a href="https://github.com/victor-VN/jpa-query-article-test" target="_blank">link</a>