---
title: "Aprendizado Superficial: falhei em uma entrevista por não ter lido um livro"
categories:
  - Carreira
tags:
  - carreira
toc: true
toc_label: "Conteúdo"
toc_icon: "file"
---

Um relato sobre um dos meus processos seletivos, o feedback que recebi dos avaliadores e como isso me fez questionar meu método de aprendizado.

### A entrevista

Recentemente, participei de um processo seletivo para uma das grandes empresas de _e-commerce_ do mercado brasileiro. Foi a minha primeira experiência com uma entrevista técnica envolvendo _pair programming_ e _system design_, então eu estava um pouco nervoso. Porém, conforme a entrevista foi passando, fui me sentindo mais confiante, e as respostas foram saindo com mais naturalidade.

O ponto alto da entrevista, para mim, foi justamente na parte teórica sobre os pilares da orientação a objetos e sobre SOLID. Eu havia lido diversos materiais na internet, então estava bastante confiante em responder a todas as perguntas.

Ao final da entrevista, eu não estava 100% satisfeito. Havia cometido alguns erros na parte de _system design_ e tinha consciência disso, por isso já esperava a reprovação — que de fato veio.

### Feedback e surpresa

O que mais me surpreendeu, então, não foi a recusa, mas sim o motivo fornecido no e-mail, que dizia que o candidato (eu mesmo) precisava melhorar, entre outras coisas, o "domínio sobre os princípios SOLID".

Eu havia estudado muito. Todos os conteúdos que li na internet, absorvi; não apenas decorei, mas entendi o que cada um deles dizia e como aplicar nos projetos. Então eu não conseguia entender.

Foi aí que decidi me aprofundar no assunto e ir diretamente a uma das melhores fontes de conhecimento que temos disponível há séculos: livros. Adquiri o livro [Arquitetura Limpa: o Guia do Artesão Para Estrutura e Design de Software](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164) e o devorei de ponta a ponta.

### O conteúdo do livro x o conteúdo de artigos da internet

Enquanto eu lia os capítulos 7 a 11 do livro, que falam sobre os princípios SOLID, uma coisa me saltou aos olhos: a explicação do autor é totalmente diferente das explicações que li na internet!

Ao ler o capítulo 7 — **SRP: The Single Responsibility Principle** — me lembrei da minha entrevista, das perguntas feitas e das respostas que dei. Ficou claro para mim onde errei.

No artigo [The S.O.L.I.D Principles in Pictures](https://medium.com/backticks-tildes/the-s-o-l-i-d-principles-in-pictures-b34ce2f1e898), o seguinte exemplo é dado para o princípio S (Single Responsibility):

> A class should have a single responsibility: If a Class has many responsibilities, it increases the possibility of bugs because making changes to one of its responsibilities could affect the other ones without you knowing.

Não está errado: uma classe deve ter apenas uma responsabilidade e, ao alterá-la, o comportamento não pode impactar outros componentes do código.

Vamos ver agora a explicação do _Uncle Bob_ em seu livro?

> A module should be responsible to one, and only one, actor. — Página 65, Robert C. Martin, *Clean Architecture*

Isso muda tudo, porque não estamos mais falando de classes, estamos falando de **atores**. Não estamos construindo nosso sistema com base nas funcionalidades do código, mas sim com base nas nuances daqueles que vão utilizá-lo — os **atores**.

Isso me leva de volta à minha entrevista e a uma das perguntas feitas:

### A pergunta sobre single responsibility

> Imagine que você possui uma classe com a responsabilidade de enviar e-mails, que recebe uma _String_ com o template do e-mail que deseja enviar e expõe esse comportamento por meio de um _Controller_ REST. Esse endpoint é utilizado pelo departamento de RH para o envio de e-mails. Recentemente, o departamento de _marketing_ sentiu a necessidade de enviar e-mails; eles já têm o template e precisam apenas de um _endpoint_ para fazer o envio.  
> A pergunta é: se o departamento de _marketing_ utilizar o mesmo endpoint, estamos ferindo o princípio SOLID?

Respondi que **"Não, afinal a classe ainda possui apenas uma responsabilidade: enviar e-mail."**

Minha resposta, percebo agora, não foi satisfatória. Isso porque, de acordo com a primeira definição que estudei — e era a que eu conhecia até então —, dizia apenas que **se uma classe tem muitas responsabilidades, a chance de criarmos bugs ao alterar uma funcionalidade e outra ser afetada ao mesmo tempo é grande**.

Novamente, não está errado.

Mas veja como, no cenário da pergunta apresentada na entrevista, a definição do _Uncle Bob_ deixa a resposta muito mais clara: **SIM!**  
Dois departamentos diferentes são atores diferentes, e utilizar o mesmo endpoint fere gravemente o primeiro princípio de SOLID.


O que deveríamos fazer então? Criar um novo _endpoint_ especificamente para o departamento de _marketing_.

Isso, é claro, geraria bastante código duplicado — mas isso é tema para outro capítulo do livro (_SYMPTOM 1: ACCIDENTAL DUPLICATION, páginas 65–66_).

Outros artigos na internet trazem o mesmo senso que, não digo que está errado, mas para um iniciante no assunto pode trazer um entendimento completamente diferente daquele que é, de fato, a proposta do princípio SOLID descrito no livro de Robert C. Martin.

Vejamos o exemplo dado em [A Solid Guide to SOLID Principles](https://www.baeldung.com/solid-principles):

> Awesome. Not only have we developed a class that relieves the Book of its printing duties, but we can also leverage our BookPrinter class to send our text to other media.  
> Whether it’s email, logging, or anything else, we have a separate class dedicated to this one concern.

```java
public class BookPrinter {

    // methods for outputting text
    void printTextToConsole(String text){
        // our code for formatting and printing the text
    }

    void printTextToAnotherMedium(String text){
        // code for writing to any other location...
    }
}
```

O trecho de código apresentado junto com a citação não menciona um ponto importante: **quais serão os atores dessas funcionalidades?**. 
Nós separamos o código, o que certamente vai ajudar na manutenção mas não fica claro quais são os atores das respectivas funcionalidades, o que, ao meu ver, não ajuda o leitor a entender o verdadeiro significado do princípio S de SOLID. 

Novamente, não está errado, mas o conteúdo é totalmente diferente do livro.

Um último exemplo [SOLID Design Principles Explained: Building Better Software Architecture](https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design):

> For example, consider an application that takes a collection of shapes—circles and squares—and calculates the sum of the area of all the shapes in the collection.

A citação acompanha um exemplo de código que você pode conferir no artigo original, mas basta dizer aqui que no exemplo os **atores** em nenhum momento são mencionados no exemplo. 

**Estamos separando funções de código e não funcionalidades de negócio.**

### Conclusão

Em algumas comunidades que frequento online sempre vejo a pergunta: **vale a pena aprender programação através de livros?**. 

Acredito que esse _post_ talvez seja a minha pequena contribuição para aqueles que possuem essa dúvida. 

Livros são fontes de informação e tal como qualquer outra fonte de informação devemos saber escolher boas fontes; fontes com boa aceitação da comunidade, autores com bom histórico etc. mas se minha opinião vale de alguma coisa deixo ela aqui: **leiam livros**! 

O conteúdo de alguns _posts_ encontrados na internet quase sempre será inferior às 300 páginas escritas por um profissional experiente. 


### Aviso

- **Aviso 1**: Importante deixar claro aqui que não sou contrário a ler _posts_ para se informar e acredito que eles contribuem bastante para o nosso aprendizado. O que trago aqui em formato de opinião é que, livros são fontes ricas de um conhecimento aprofundado e trará novas formas de pensar e novas ferramentas para trabalhar que muitas vezes não encontramos em postagens rápidas com apenas 10 parágrafos
- **Aviso 2**: Se você precisa de uma informação pontual é claro que uma postagem de fácil leitura é preferível a um livro de 300 páginas, apenas tome cuidado com a fonte da sua informação. 
- **Aviso 3**: Você não precisa se apegar a apenas uma fonte de estudo. De fato no livro [The Pragmatic Programmer: From Journeyman to Master](https://www.amazon.com.br/Pragmatic-Programmer-Journeyman-Master/dp/020161622X) o autor deixa uma dica importante: **diversifique**.
 Leia livros, leia artigos/posts, faça cursos, leia documentação, não se apegue a apenas um meio de aprendizado. 
- **Aviso 4**: Não desprezem os livros.