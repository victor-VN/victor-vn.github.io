---
title: Como publicar um site gratuitamente no Github usando Jekyll.
categories: [Jekyll, Github]
lang: pt-PT
---
<br>

## Importante: 

{: .notice--warning}
Este tutorial foi escrito no Windows 10 (Versão	10.0.19044 Compilação 19044) e os comandos podem ser diferentes se você usar Linux, MacOS ou outra versão do Windows <br>
Para seguir este tutorial é importante que você possua conhecimentos básicos em terminal e **git bash**


## Ferramentas: 

- jekyll 4.3.2
- MS-DOS (ou prompt de comando)
- Github
- Git Bash
- Editor da sua preferência (Utilizamos o VSCode)


## Instalando Ruby e Jekyll

Você pode utilizar o [tutorial oficial](https://jekyllrb.com/docs/installation/windows/#installing-ruby-and-jekyll) do Jekyll para fazer a instalação na sua máquina Windows se preferir.

1 - Baixe e execute o instalador do **Ruby+DevKit** - usaremos a versão 3.1.3-1 (x64) mas versões acima desta devem funcionar bem - [download](https://github.com/oneclick/rubyinstaller2/releases/download/RubyInstaller-3.1.3-1/rubyinstaller-devkit-3.1.3-1-x64.exe)
<br><br>
2 - Click em _next_ e depois _next_ novamente sem alterar nenhuma das opções
<br><br>
3 - Na última tela clique em _finish_ com a opção "Run 'ridk install' to setup MSYS2 and..." selecionada.
<br><br>
4 - Um novo terminal como o da imagem abaixo se abrirá, aperte ENTER e aguarde a finalização <br><br>
![ruby installer command prompt](/assets/images/posts/2023-02-04/img-installer-cmd.png)
<br><br>
5 - Quando a instalação terminar a mensagem 
"Install MSYS2 and MINGW development toolchain <span style="color: green">succeeded</span>" deve aparecer. 
<br><br>
6 - Abra um novo terminal e execute o comando
```
C:\Users\myuser> jekyll -v
```
Você deve receber uma mensagem parecida com a mensagem abaixo confirmadno que o **Jekyll** foi instalado corretamente
```
C:\Users\myuser> jekyll 4.3.2
```

## Criando um novo repositório no Github

1 - Crie um novo repositório no github com o nome seguindo o padrão **nome-do-usuario.github.io** tudo em minúsculo e com a visibilidade **Public**
<br><br>

Exemplo: 

![ruby installer command prompt](/assets/images/posts/2023-02-04/github-create-repo.png)

2 - Clone o repositório para sua máquina usando o git bash
```console
myuser@DESKTOP-637R5MB MINGW64 ~ git clone https://github.com/nome-do-usuario/nome-do-usuario.github.io.git
```

## Instalando o tema Minimal Mistakes

Se você acompanhou até aqui deve ter percebido que instalamos a ferramenta **Jekyll** anteriormente mas ainda não fizemos nada com ela de fato.<br>
Isso porque o Jekyll é uma ferramenta que gera sites estáticos a partir de arquivos de _markdown_, mas nós ainda não geramos nenhum conteúdo para o nosso site. 


Para começar nosso site vamo utilizar um [tema jekyll](https://jekyllthemes.io/), que nada mais é que um template com estrutura básica do nosso site (página inicial, header, footer, navbar, etc). O tema que vamos utilizar será o [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) mas você pode explorar outros temas, basta ter em mente que os temas são diferentes em sua estrutura e pode exigir configurações e plugins adicionais que não estarão no escopo desse post. Para informações consulte a documentação do tema que você escolher.

{:.notice--info}
 O motivo de ter escolhido o minimal mistakes foi a sua aparência mais "sóbria" que facilita quando você for estilizar para deixar o site com a sua cara. Além disso o Minimal Mistakes possui um visual que facilita a leitura de postagens como essa. 

Agora para começar a dar uma aparência para o nosso site:

1 - Baixe o repositório do tema direto do [Github](https://github.com/mmistakes/minimal-mistakes) 
<br><br>
![ruby installer command prompt](/assets/images/posts/2023-02-04/github-downloa-minimal.png)

2 - Extraia os arquivos do .zip <br><br>
3 - Copie todos os arquivos do diretório gerado para o diretório do seu repositório github (aquele que clonamos) na etapa anterior.<br><br>
4 - Remova os diretórios **/docs** e **/test**<br><br>
5 - Inclua o conteúdo abaixo no arquivo **Gemfile**

```yaml
source "https://rubygems.org"


gem "jekyll"
gem "minimal-mistakes-jekyll"


group :jekyll_plugins do
    gem "jekyll-feed"
    gem "jekyll-seo-tag"
    gem "jekyll-sitemap"
    gem "jekyll-paginate"
    gem "jekyll-include-cache"
    gem "jekyll-algolia"
end
``` 

6 - No arquivo **_config.tyml** descomente a linha 

```yaml
# remote_theme           : "mmistakes/minimal-mistakes"
```

Deverá ficar assim

```yaml
 remote_theme           : "mmistakes/minimal-mistakes"
```

7 - No terminal abra o diretório do seu site 

```
C:\Users\myuser> cd caminho\para\diretorio
```

e execute o comando 
```
C:\Users\myuser\caminho\para\diretorio> bundle
```

Você deverá ver algo parecido com a seguinte mensagem
```
Bundle complete! 8 Gemfile dependencies, 52 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

8 - Execute o comando abaixo para rodar o seu site localmente
```
C:\Users\myuser\caminho\para\diretorio> jekyll serve
```

Ao acessar o endereço [http://localhost:4000/](http://localhost:4000/) você deverá ver o seu site. <br><br>
![ruby installer command prompt](/assets/images/posts/2023-02-04/site-localhos-4000.png) 


## Publicando o site no Github

Chegamos até aqui e nosso site já está rodando na nossa máquina. O que falta agora? Publicar definitivamente o site no github.

1 - Abra seu diretório no gitbash e crie o primeiro _commit_. Primeiro adicione os arquivos ao commit que vai criar
```console
myuser@DESKTOP-637R5MB MINGW64 ~ git add .
```
crie o commit com uma mensagem
```console
myuser@DESKTOP-637R5MB MINGW64 ~ git commit -m "primeiro commit"
```

2 - Publique sua branch no github 
```console
myuser@DESKTOP-637R5MB MINGW64 ~ git push origin main
```

Ao publicar sua branch, o [Github Actions](https://github.com/features/actions) será disparado e se tudo estiver funcionando corretamente, ao abrir a aba "Actions" do seu repositório você deverá ver um ícone verde indicando o sucesso como na imagem abaixo <br><br>
![ruby installer command prompt](/assets/images/posts/2023-02-04/github-actions.png) 

3 - Com seu site finalmente publicado basta acessar o endereço https://**nome-do-seu-usuario**.github.io/ e você deverá ver seu site com a mesma aparência de quando rodamos localmente. <br><br>
![ruby installer command prompt](/assets/images/posts/2023-02-04/site-localhos-4000.png)

## Como configurar o site para mudar sua aparência

Na [documentação oficial](https://mmistakes.github.io/minimal-mistakes/docs/configuration/) você vai encontrar diversos parâmetros diferentes que permitem você mudar todo o esquema de cores do seu site, o título, subtítulo, adicionar aba de comentários para as postagens, entre outras coisas. <br>
Um ótimo material de referência também pode ser encontrado [aqui neste site](https://www.fabriziomusacchio.com/blog/2021-08-11-Minimal_Mistakes_Cheat_Sheet/). <br><br>

Explorar todas as configurações possíveis fugiria do escopo desse post e pra ser sincero deixaria a postagem muito grande, por isso recomendo que você leia a documentação e explore suas necessidades. <br><br>

{: .notice--success}
Caso tenha encontrado algum problema no processo ou tenha algo a acrescentar, por favor deixe um comentário. Até a próxima.
