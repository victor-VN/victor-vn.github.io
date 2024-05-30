---
title: "Como enviar documentos para assinatura com Docusign e Java, usando eSignature API SDK"
categories:
  - Java
  - Docusign
  - eSignature
tags:
  - java
  - docusign
  - esignature
toc: true
toc_label: "Conteúdo"
toc_icon: "file"
---

Neste artigo mostro como fazer uma integração básica com o Docusign enviando documentos por email para serem assinados digitalmente

**Objetivo:** configurar um projeto **Spring** capaz de enviar por email **documentos** para serem assinados via Docusign.
{: .notice--info}

**Dica 1:** Como um artigo com mais de 3000 palavras não é aconselhável ler em um celular. Salve o link e leia mais tarde na tela de um computador.
{: .notice--warning}

**Dica 2:** Apesar de parecer muito extenso boa parte do tamanho desse artigo é devido aos prints usados para ilustrar as configurações do Docusign. 
{: .notice--warning}

**Dica 3:** Se você já sabe o que é o [Docusign](https://www.docusign.com/pt-br/produtos/comofunciona), talvez você queira ir direto para a [próxima sessão](#configurar-projeto) e [configurar o projeto](#configurar-projeto)
{: .notice--warning}

## O que é o Docusign?

Docusign é um [_SaaS_](https://www.cloudflare.com/pt-br/learning/cloud/what-is-saas/) baseado no envio e assinatura digital de documentos.

**Cenário**: A empresa onde você trabalha agora possui contratos que precisam ser assinados primeiro pelos clientes, depois pela própria empresa e por fim após as duas partes assinarem o arquivo um e-mail deve ser enviado para ambos com uma cópia do contrato assinado. 

Você decidiu que vai então construir uma aplicação do zero que faz tudo o que foi solicitado. Nesse caso você precisa de: 

- uma forma de armazenar os documentos (nem pensar em usar blob nos bancos de dados!)
- uma forma de informar aonde o documento deve ser assinado e uma UI que permita que o usuário assine o documento
- uma forma de enviar emails
- uma forma de armazenar o status de cada um dos documentos que foi enviado e saber qual das partes já assinou o documento

a lista continua, mas já vimos que o trabalho aqui não será simples. 

Para nossa sorte não precisamos fazer tudo isso, o **Docusign** fará por nós. 

O processo de assinatura mais básico do Docusign (que é explicado [aqui](https://www.docusign.com/pt-br/produtos/comofunciona) e que é possível fazer através do [painel](https://apps-d.docusign.com/) que eles disponibilizam) possui 3 etapas: 

- Fazer upload de um documento (geralmente um PDF ou uma imagem)
- Indicar o email de quem deve assinar o documento
- Indicar aonde o documento deve ser assinado e depois enviar o documento

Após realizar as etapas acima dentro do mesmo painel você poderá acompanhar o progresso das assinaturas até que tudo seja concluído e no final você tenha um documento assinado pelas duas partes (no nosso cenário as partes são o cliente e a empresa). 

Nesse processo o Docusign: 

- Armazenou nosso arquivo (documento PDF)
- Enviou os emails com o arquivo para as partes assinarem já com a marcação de onde a assinatura deveria ser aplicada
- Após a assinatura enviou uma cópia em ambos os emails do contrato já assinado

Dessa forma nós não tivemos que implementar do zero nenhum dos serviços que nos foi solicitado, nem tivemos que nos preocupar com o _status_ desses serviços, os possíveis _bugs_ de implementação, etc. 

**Mas agora temos um outro dilema**: e se a empresa tiver que enviar documentos para 5000 clientes diferentes assinar? 

Fazer um por um pelo painel não é uma opção já que pode levar muito tempo e trabalhar com esse volume de dados manualmente provavelmente levará a erros. 

Pra isso o Docusign providencia sua API - [eSignature](https://developers.docusign.com/docs/esign-rest-api/) - e que vamos explorar adiante.



## Configurando o projeto
{: #configurar-projeto}

Para fazer nossa integração com o Docusign vamos precisar: 
- Java 17 ou versão mais recente instalada
- criar uma **conta _developer_** (caso ainda não possua uma crie [aqui](https://developers.docusign.com/)) e acessar o painel de controle dos seus Envelopes [aqui](https://apps-d.docusign.com/send/home)
- criar um projeto **Spring Boot** (você pode criar usando o [initializr](https://start.spring.io/)) com **apenas Spring Web** como dependência
- Ter o [**Postman**](https://www.postman.com/) ou uma ferramenta de seu gosto instalado para enviar as requisições
- 3 contas de e-mail diferentes para testar o envio dos e-mails

### Configurando o projeto Spring
{: #configurar-conta}
Após configurar e baixar seu projeto **Spring** o conteúdo do seu arquivo **pom.xml** deve parecer com o arquivo abaixo

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.4</version>
    <relativePath/>
    <!--  lookup parent from repository  -->
  </parent>
  <groupId>com.example</groupId>
  <artifactId>demo</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>demo</name>
  <description>Demo project for Spring Boot</description>
  <properties>
    <java.version>17</java.version>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

Dentro da tag **dependencies** vamos adicionas as seguintes dependências

```xml
  <dependency>
    <groupId>com.docusign</groupId>
    <artifactId>docusign-esign-java</artifactId>
    <version>4.6.0-RC1</version>
  </dependency>

  <dependency>
    <groupId>com.docusign</groupId>
    <artifactId>docusign-admin-java</artifactId>
    <version>1.3.0</version>
  </dependency>
  <!-- Docusign Dependencies-->
  <dependency>
    <groupId>jakarta.ws.rs</groupId>
    <artifactId>jakarta.ws.rs-api</artifactId>
    <version>3.1.0</version>
  </dependency>

  <dependency>
    <groupId>org.glassfish.jersey.core</groupId>
    <artifactId>jersey-client</artifactId>
  </dependency>

  <dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-multipart</artifactId>
  </dependency>

  <dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-joda</artifactId>
  </dependency>

  <dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
  </dependency>

  <dependency>
    <groupId>com.fasterxml.jackson.jakarta.rs</groupId>
    <artifactId>jackson-jakarta-rs-base</artifactId>
  </dependency>

  <dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-jackson</artifactId>
  </dependency>

  <dependency>
    <groupId>org.apache.oltu.oauth2</groupId>
    <artifactId>org.apache.oltu.oauth2.client</artifactId>
    <version>1.0.2</version>
  </dependency>

  <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.24</version>
      <scope>provided</scope>
  </dependency>


  <!-- JWT -->
  <dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
  </dependency>

  <dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.2.0</version>
  </dependency>

  <!-- https://mvnrepository.com/artifact/commons-codec/commons-codec-->
  <dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.15</version>
  </dependency>

  <dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.0</version>
  </dependency>
```
Explicar cada depência que adicionamos foge um pouco do escopo do artigo então por enquanto vamos nos contentar em saber que todas as dependências abaixo do comentário ``` <!-- JWT --> ``` são necessárias para gerar o token JWT para autenticação no **Docusign** e as demais dependências são necessárias para o funcionamento da dependência **docusign-esign-java**, que é a lib que usaremos para fazer as chamadas HTTP para o Docusign. 

Crie um arquivo **application.properties** (caso não exista) e cole o conteúdo abaixo nele

```properties
spring.application.name=docusign-playground

docusign.account.id=##YOUR_ACCOUNT_ID_HERE
docusign.private.key.file.name=##YOUR_PRIVATE_KEY_FILE_NAME_HERE

docusign.client.id=##YOUR_INTEGRATION_KEY_HERE
docusign.user.id=##YOUR_USER_ID_HERE
docusign.user.scopes=signature,impersonation
```

Por fim basta rodar o projeto, você agora deve ter um projeto Spring funcionando e com todas dependências necessárias instaladas. 

### Configurando um aplicativo no Docusign

Após configurar o nosso projeto Spring vamos acessar o [painel da Docusign](https://account-d.docusign.com/logout) e configurar nossa primeira aplicação. É nesta etapa que vamos conseguir todas informações necessárias para preencher o arquivo **aplication.properties**. 

Acesse a o painel e clique em **Settings**

![home do painel do docusign](/assets/images/2024-04-14-img/1-painel.png){: .align-center}

Do lado esquerdo inferior clique em **Apps and Keys**

![settings do painel do docusign](/assets/images/2024-04-14-img/2-painel-apps-keys.png){: .align-center}

Já nessa tela podemos ver algumas informações que vamos precisar para preencher nosso arquivo. Copie o **User ID** e o **API Account ID** e preencha no arquivo de **properties**. 

![apps and keys do painel do docusign](/assets/images/2024-04-14-img/3-painel-first-keys.png){: .align-center}

Clique em **Add App and Integration Key**

![adicionando app e chave de integracao no docusign](/assets/images/2024-04-14-img/4-painel-add-app.png){: .align-center}

Crie um novo App (aqui vamos chamar de **playground-app**)

![adicionando nome do app no docusign](/assets/images/2024-04-14-img/5-painel-create-app.png){: .align-center}

Copie a **Integration Key** e cole no arquivo **properties**

![copiando a integration key do app no docusign](/assets/images/2024-04-14-img/6-painel-integration-key.png){: .align-center}

Clique em **Generate RSA** 

![gerando a chave RSA do app no docusign](/assets/images/2024-04-14-img/7-painel-generate-key.png){: .align-center}

Salve o conteúdo da **Public Key** e da **Private Key** em arquivos nomeados **public-key** e **private-key** respectivamente e mova os arquivos para a **src/main/resource** dentro da sua aplicação Spring

![salvando o conteudo da chave em arquivos](/assets/images/2024-04-14-img/8-painel-generate-key-save.png){: .align-center}

Em **Redirect URLs** adicione ```http://localhost:8080/ds/callback```

![adicionando uma redirect url](/assets/images/2024-04-14-img/9-painel-redirect-url.png){: .align-center}

Mais para baixo preencha os métodos ```GET, POST, PUT, DELETE``` e clique em **Salvar**

![permitindo todos metodos http para o app](/assets/images/2024-04-14-img/10-painel-http-methods.png){: .align-center}

Ao final seu arquivo **application.properties** deve estar preenchido com as informações da aplicação e no diretório **resources** deve conter os arquivos **public-key.txt** e **private-key.txt**

```properties
spring.application.name=docusign-playground

docusign.account.id=66ab2a90-9f83-XXXX-XXXX-XXXXXXXXXXXX
docusign.private.key.file.name=private-key.txt

docusign.client.id=cfedf726-5354-XXXX-XXXX-XXXXXXXXXXXX
docusign.user.id=e1a2d709-6d6d-XXXX-XXXX-XXXXXXXXXXXX
docusign.user.scopes=signature,impersonation
```

### Configurando um Template
{: #configurar-projeto-template}

Para resumir e simplificar podemos entender o **template** como o documento que será enviado para os _recipients_ assinarem. 

Aqui nosso documento será um arquivo **.docx** com o seguinte conteúdo

![documento .docx que sera o arquivo de template](/assets/images/2024-04-14-img/21-mydoc-view.png){: .align-center}

Depois vamos no [painel Docusign](https://account-d.docusign.com/logout) criar o template clicando em **Templates** e depois **New** > **Create Template**

![criando um novo template no painel docusign](/assets/images/2024-04-14-img/22-create-template.png){: .align-center}

Preencha o nome do template, adicione uma descrição e faça upload do arquivo **.docx**. 

![preenchendo nome do template e fazendo upload do arquivo](/assets/images/2024-04-14-img/23-create-template-filling.png){: .align-center}

Ainda na mesma página mais abaixo clique em **Add Recipient**

![adicionando recipients para o template](/assets/images/2024-04-14-img/24-create-template-recipient.png){: .align-center}

Depois marque o _checkbox_ **Set signing order** e preencha os dados de **Role** para os dois _signers_ com **owner_signer** e **stranger_signer** respectivamente. Preencha para o owner_signer um **Name** e em **Email** preencha com um e-mail que você terá acesso. É nele que o Docusign vai enviar o seu novo envelope. 

![preenchendo dados do recipient](/assets/images/2024-04-14-img/25-create-template-recipient-orders.png){: .align-center}

Clique em **Next**

![clicando em next no recipient](/assets/images/2024-04-14-img/26-create-template-recipient-next.png){: .align-center}

Do lado esquerdo clique em **Signature** e posicione o novo campo aonde deseja que o usuário que receberá o e-mail faça a assinatura. 

![adicionando campo de assinatura no template no docusign](/assets/images/2024-04-14-img/27-create-template-recipient-first-sign.png){: .align-center}

Repita o processo para adicionar a assinatura no segundo campo do documento mas desta vez do lado direito clique em **Recipient** e selecione a opção **stranger_signer**

![adicionando segundo campo de assinatura no template no docusign](/assets/images/2024-04-14-img/28-create-template-recipient-second-sign.png){: .align-center}

Por fim clique em **Save and Close**.

Ao finalizar veremos que nosso novo template já aparece na lista de templates criados. Clique no seu template criado

![visualizando template criado no docusign](/assets/images/2024-04-14-img/29-all-templates.png){: .align-center}

Clique em **Template ID** e copie o valor mostrado na tela. É ele que vamos usar para criar nossos novos envelopes. 

![copiando o id do template no docusign](/assets/images/2024-04-14-img/30-template-id.png){: .align-center}

Agora que finalizamos a criação do nosso template podemos seguir adiante. 


### Autenticando no Docusign

O Docusign providencia 3 tipos de autenticação; _Authorization Code Grant_, _Implicit Grant_ e _JWT Grant_. 

Baseado no [diagrama](https://developers.docusign.com/platform/auth/) que a Docusign disponibiliza, usaremos a autenticação via **JWT** 

![diagrama de autenticacao docusign](/assets/images/2024-04-14-img/11-public-docauth-diagram.png){: .align-center}

Para fazer a autenticação preencha a url abaixo trocando **INTEGRATION_KEY** pelo valor da sua _Integration Key_ (a mesma usada para preencher a property **docusign.client.id**) e cole no navegador (você pode entender melhor o processo de autenticação via JWT [aqui](https://developers.docusign.com/platform/auth/jwt/jwt-get-token/))

```
  https://account-d.docusign.com/oauth/auth?response_type=code&scope=signature%20impersonation&client_id=<INTEGRATION_KEY>f&redirect_uri=http://localhost:8080/ds/callback
```

Você será direcionado para uma tela de login novamente, preencha e faça login 

![fazendo login de autenticacao no docusign](/assets/images/2024-04-14-img/12-login-jwt-first.png){: .align-center}

Clique em **Allow Access**

![permitindo acesso do app no docusign](/assets/images/2024-04-14-img/13-login-jwt-allow-access.png){: .align-center}

Como a nossa URL de callback aponta para localhost (_http://localhost:8080/ds/callback_) uma tela preta de erro deve aparecer, ainda assim isso não indica erro necessariamente. <span style="color: green">Nosso aplicativo está autenticado!!</span>

![mensagem de erro após autenticcao no docusign](/assets/images/2024-04-14-img/14-login-jwt-callback.png){: .align-center}

## Criando Controllers, Models e Services

Finalmente, vamos por as mãos no código. Mas antes vale lembrar que esse artigo não busca substituir a [documentação do Docusign](https://developers.docusign.com/docs/esign-rest-api/sdks/) que é muito mais extensa. Não deixe de conferir a documentação oficial. 

Todo o código feito abaixo está no github: [docusign-playground](https://github.com/victor-VN/docusign-playground)

Prosseguindo, nossa aplicação terá 1 endpoint ```/docusign``` com dois métodos: ```POST``` e ```PUT```. Vamos começar criando 3 novos _packages_ na nossa aplicação; ```controller```, ```model``` e ```service```. 

![visualizando pacotes java da aplicacao no intellij](/assets/images/2024-04-14-img/15-app-packages.png){: .align-center}

### Criando os models
No pacote ```model``` vamos criar 3 classes para nossas requisições; ```CreateEnvelopeRequestBody```, ```SignerRequest``` e ```UpdateEnvelopeRequestBody```

```java
import lombok.Data;
import java.util.List;

@Data
public class CreateEnvelopeRequestBody {
    private String templateId;
    private String status;
    private List<SignerRequest> signers;
}
```

```java
import lombok.Data;

@Data
public class SignerRequest{
    private String name;
    private String email;
    private String roleName;
}
```

```java
import lombok.Data;

@Data
public class UpdateEnvelopeRequestBody {
    private String envelopeId;
    private String email;
}
```
### Criando os controllers
Depois disso vamos criar nosso controller ```EnvelopeController``` retornando apenas duas mensagens sem importância por enquanto, apenas para garantir que tudo está funcionando. 

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/docusign")
public class EnvelopeController {

    @PostMapping
    public ResponseEntity<Object> sendEnvelope(){
        return ResponseEntity.ok().body("Hello Post!");
    }

    @PutMapping
    public ResponseEntity<Object> updateEmail(){
        return ResponseEntity.ok().body("Hello Put!");
    }
}
```
Após testar nossos métodos no **Postman** vemos que tudo está funcionando normalmente

Testando o ```POST```
![testando a requisicao post inicial no postman](/assets/images/2024-04-14-img/16-test-1.png){: .align-center}

Testando o ```PUT```
![testando a requisicao put inicial no postman](/assets/images/2024-04-14-img/17-test-2.png){: .align-center}

Vamos agora à parte mais legal, vamos criar nossos services para nos comunicar com o **Docusign**.

### Criando o service de autenticação
Crie dentro do _package_ ```service``` um novo _package_ chamado ```auth```

![visualizando pacotes java no intellij](/assets/images/2024-04-14-img/18-app-package-auth.png){: .align-center}

Dentro do _package_ ```auth``` vamos criar a interface ```DocusignAuthService```

```java
import com.docusign.esign.client.ApiClient;

public interface DocusignAuthService { 
    public ApiClient getDocusignClient();
}
```
E logo em seguida vamos implementar a _interface_ criada com a classe ```DocusignAuthServiceImpl```

```java
import com.docusign.esign.client.ApiClient;
import com.docusign.esign.client.auth.OAuth;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.ApplicationContext;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Service;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.Arrays;
import java.util.List;

@Service
public class DocusignAuthServiceImpl implements DocusignAuthService {

    @Autowired
    private ApplicationContext ctx;
    @Value("${docusign.private.key.file.name}")
    private String privateKeyFileName;
    @Value("${docusign.client.id}")
    private String clientId;
    @Value("${docusign.user.id}")
    private String userId;
    @Value("${docusign.user.scopes}")
    private String scope;

    private static final long TOKEN_EXPIRATION_IN_SECONDS = 3600;

    @Override
    public ApiClient getDocusignClient(){

        // 1
        InputStream privateKey = readFileFromResources(privateKeyFileName);

        // 2
        List<String> scopes = Arrays.stream(scope.split(",")).toList();

        try {
            // 3
            ApiClient apiClient = new ApiClient();

            // 4
            byte[] privateKeyBytes = privateKey.readAllBytes();

            //5
            OAuth.OAuthToken oAuthToken = apiClient.requestJWTUserToken(
                    clientId,
                    userId,
                    scopes,
                    privateKeyBytes,
                    TOKEN_EXPIRATION_IN_SECONDS
            );

            //6
            String accessToken = oAuthToken.getAccessToken();

            //7
            apiClient.addDefaultHeader("Authorization", "Bearer " + accessToken);

            return apiClient;
        } catch (Exception e) {
            return null;
        }
    }

    //8
    private InputStream readFileFromResources(String fileName){
        Resource resource = ctx.getResource("classpath:" + fileName);
        File file = null;
        InputStream inputStream = null;
        try {
            file = resource.getFile();
            inputStream = new FileInputStream(file);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

        return inputStream;
    }
}
```

Tem bastante coisa acontecendo aqui então vamos explicar por partes:

1 - Primeiro recuperamos o conteúdo arquivo **private-key.txt**, necessário para fazer a autenticação.

2 - Depois recuperamos da _property_ **docusign.user.scopes** os _scopes_ da nossa autenticação - ou seja, as permissões que aquele token terá para fazer alterações no Docusign - e transformarmos em uma lista de _Strings_. 

3 - Criamos uma instância do objeto **[ApiClient](https://javadoc.io/doc/com.docusign/docusign-esign-java/latest/com/docusign/esign/client/ApiClient.html)**. Esse objeto é o que será retornado e é ele que vamos usar pra fazer todas nossas chamadas HTTP.

4 - Transformamos o conteúdo do nosso arquivo **private-key.txt** em _bytes_.

5 - É gerado um **token JWT** com base nos parâmetros do arquivo **application.properties**, esse é o token necessário para nos comunicarmos com o **Docusign** posteriormente.

6 - Pegamos somente o valor do token gerado

7 - Adicionamos no nosso objeto **ApiClient** um _header_ HTTP chamado **Authorization** e passamos como valor o token gerado concatenado com a palavra **Bearer**. 

8 - Esse é o método que recupera o arquivo **private-key.txt** com base no nome do arquivo. 

Ufa! Vimos bastante coisa de uma só vez mas agora já estamos quase lá. Já é possível agora fazer autenticação no Docusign ao chamar o método 

```java
authService.getDocusignClient()
```

mas antes de seguirmos em frente, vale ressaltar alguns pontos sobre o código acima: 

- **NÃO USE EM PRODUÇÃO**. Este código pode te ajudar e te guiar a como fazer uma implementação inicial para consumir os recursos do Docusign porém esse código deve apenas ser usado como material de estudo e não diretamente em produção. 

- Esse código tem alguns problemas como: um bloco _catch_ que apenas retorna _null_ e não faz mais nada, um método pra recuperar um arquivo local com uma chave privada que deveria estar guardada em algum gerenciador de segredos, o objeto ApiClient poderia ser uma classe singleton, entre outros. Como nosso objetivo aqui é fazer testes apenas locais, para não adicionar mais complexidade ao código decidi ignorar algumas questões de qualidade. Eu te encorajo a fazer uma implementação bem melhor do que estamos fazendo aqui. 

### Criando o service de comunicação com o Docusign
Seguindo adiante vamos criar agora nossa _interface_ ```DocusignService``` dentro do pacote ```service```

```java
import com.docusign.esign.client.ApiException;
import com.docusign.esign.model.EnvelopeSummary;
import com.docusign.esign.model.RecipientsUpdateSummary;
import io.github.victorvn.docusignplayground.model.CreateEnvelopeRequestBody;

public interface DocusignService {
    public EnvelopeSummary sendNewEnvelope(CreateEnvelopeRequestBody requestBody) throws ApiException;
    public RecipientsUpdateSummary updateEnvelopeEmail(String email, String envelopeId) throws ApiException;
}
```
Em seguida vamos criar a classe ```DocusignServiceImpl``` que implementa a _interface_ criada logo acima

```java
import com.docusign.esign.api.EnvelopesApi;
import com.docusign.esign.client.ApiClient;
import com.docusign.esign.client.ApiException;
import com.docusign.esign.model.*;
import io.github.victorvn.docusignplayground.model.CreateEnvelopeRequestBody;
import io.github.victorvn.docusignplayground.service.auth.DocusignAuthService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class DocusignServiceImpl implements DocusignService {

    @Autowired
    private DocusignAuthService authService;

    @Value("${docusign.account.id}")
    private String accountId;

    @Override
    public EnvelopeSummary sendNewEnvelope(CreateEnvelopeRequestBody requestBody) throws ApiException {
        //1
        ApiClient apiClient = authService.getDocusignClient();

        //2
        EnvelopesApi envelopesApi = new EnvelopesApi(apiClient);

        //3
        Envelope envelope = new Envelope();

        //4
        List<TemplateRole> roleList = requestBody.getSigners().stream().map(
                 item -> new TemplateRole()
                         .roleName(item.getRoleName())
                         .email(item.getEmail())
                         .name(item.getName())
        ).collect(Collectors.toList());

        //5
        EnvelopeDefinition definition = new EnvelopeDefinition()
                .envelopeId(envelope.getEnvelopeId())
                .status(requestBody.getStatus())
                .templateId(requestBody.getTemplateId())
                .templateRoles(roleList);

        //6
        return envelopesApi.createEnvelope(accountId, definition);
    }

    @Override
    public RecipientsUpdateSummary updateEnvelopeEmail(String email, String envelopeId) throws ApiException {
        ApiClient apiClient = authService.getDocusignClient();
        EnvelopesApi envelopesApi = new EnvelopesApi(apiClient);

        Envelope envelope = new Envelope();

        //7
        Signer signer = new Signer()
                .email(email)
                .recipientId("2")
                .roleName("stranger_signer");

        //8
        Recipients recipients = new Recipients()
                .addSignersItem(signer);

        //9
        envelope.setRecipients(recipients);

        //10
        EnvelopesApi.UpdateRecipientsOptions updateOptions = envelopesApi.new UpdateRecipientsOptions();
        updateOptions.setResendEnvelope("true");

        //11
        return  envelopesApi.updateRecipients(accountId, envelopeId, recipients, updateOptions);
    }
}
```

Novamente temos muita coisa acontecendo aqui então vamos por partes. Primeiro vamos analisar o método ```sendNewEnvelope```

1 - Dentro do método recuperamos uma nova instância do objeto **ApiClient** que contém a autenticação do **Docusign**.

2 - Depois criamos uma instância do objeto **EnvelopesApi** passando como parâmetro o **apiClient**. Esse objeto possui os métodos para buscar informações de um _envelope_ existente, criar novos envelopes, deletar documentos dentro de um _envelope_ entre outras (mais informações [aqui](https://javadoc.io/doc/com.docusign/docusign-esign-java/latest/com/docusign/esign/api/EnvelopesApi.html))

3 - Criamos uma nova instância do objeto **Envelope**.

4 - Criamos uma lista de objetos do tipo **TemplateRole**. Cada objeto nessa lista representa um _signer_ dentro de um _template_. Seguindo o exemplo do print abaixo cada objeto seria preenchido com um _roleName_, _name_ e _email_.

![visualizando recipients do template no docusign](/assets/images/2024-04-14-img/19-templates-signers.png){: .align-center}

5 - Criamos uma instância do objeto **EnvelopeDefinition**. Nele preenchemos um ID de um _envelope_, o _status_ em que queremos criar esse _envelope_ (caso queira que um e-mail seja enviado já na criação do envelope use o status **sent**), o ID do template que criamos no momento da [configuração da conta](#configurar-conta-template). 

![visualizando informacoes basicas do templa no docusign](/assets/images/2024-04-14-img/20-template-id-view.png){: .align-center}

6 - Enviamos a requisição para o Docusign para a criação do novo _envelope_. Neste momento o primeiro _signer_ da ordem já deverá receber um e-mail e já devemos ver um novo _envelope_ criado no Docusign. 

No segundo método, ```updateEnvelopeEmail``` vamos pular as partes repetidas do primeiro método e vamos direto ao que interessa

7 - Criamos uma nova instância do objeto **Signer** e indicamos qual o _signer_ que vamos querer atualizar o e-mail. Deixamos _hardcoded_ o ID 2 e o _roleName_ **stranger_signer** indicando que somente podemos atualizar o segundo _signer_ do nosso _envelope_. 

8 - Criamos uma instância de um objeto **Recipients** e dentro dela colocamos nosso objeto **signer**

9 - Colocamos o objeto **recipients** dentro do nosso _envelope_ já com as informações atualizadas. 

10 - Criamos um objeto do tipo **UpdateRecipientsOptions** para indicar quais informações adicionais de configuração queremos adicionar ao nosso _recipient_. Nesse caso estamos informando que ao atualizar nosso _envelope_ queremos que um e-mail seja reenviado para o _recipient_ que foi atualizado. 

11 - Enviamos a requisição para o Docusign para a atualização do _envelope_. 

### Finalizando a criação dos controllers
Agora só falta mais uma etapa e nossa aplicação estará pronta. Vamos corrigir nosso ```controller``` para chamar nosso novo ```service``` criado. 

```java
import com.docusign.esign.client.ApiException;
import io.github.victorvn.docusignplayground.model.CreateEnvelopeRequestBody;
import io.github.victorvn.docusignplayground.model.UpdateEnvelopeRequestBody;
import io.github.victorvn.docusignplayground.service.DocusignService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/docusign")
public class EnvelopeController {

    //1
    @Autowired
    private DocusignService docusignService;


    //2
    @PostMapping
    public ResponseEntity<Object> sendEnvelope(@RequestBody CreateEnvelopeRequestBody requestBody){
        try {
            //3
            return ResponseEntity.ok().body(docusignService.sendNewEnvelope(requestBody));
        } catch (ApiException e) {
            return ResponseEntity.internalServerError().body(e.getMessage());
        }
    }

    //4
    @PutMapping
    public ResponseEntity<Object> updateEmail(@RequestBody UpdateEnvelopeRequestBody requestBody){
        try {
            //5
            return ResponseEntity.ok().body(
              docusignService.updateEnvelopeEmail(requestBody.getEmail(), requestBody.getEnvelopeId())
            );
        } catch (ApiException e) {
            return ResponseEntity.internalServerError().body(e.getMessage());
        }
    }
}
``` 

Mais uma vez vamos por partes: 

1 - Primeiro injetamos nosso ```service``` **DocusignService**;

2 - No nosso método **sendEnvelope** adicionamos o _request body_ através do trecho de código

```java
@RequestBody CreateEnvelopeRequestBody requestBody
```

3 - Chamamos o método para criar um novo _envelope_ e retornamos o resultado da chamada feita. 

4 - Aqui também adicionamos o _request body_ para atualizar o _envelope_

```java
@RequestBody UpdateEnvelopeRequestBody requestBody
```

5 - Chamamos o método para atualizar nosso _envelope_ e retornamos o resultado da chamada feita.

## Testando a criação e atualização no Docusign

Primeiro vamos criar um novo envelope. 

### Criando um novo envelope

Para isso vamos enviar uma requisição como no exemplo abaixo:

![enviando requisicao para criar envelope para o docusign via postman](/assets/images/2024-04-14-img/31-testing-envelope.png){: .align-center}

Corpo da requisição

```json
{
    "templateId": "f9d701e5-05d2-XXXX-XXXX-XXXXXXXXXX",
    "status": "sent",
    "signers": [
        {
            "name": "Victor teste",
            "email": "victor.XXXXXXXXX.com",
            "roleName": "owner_signer"
        },
        {
            "name": "Stranger",
            "email": "v1ctorXXXXXXXXX.com",
            "roleName": "stranger_signer"
        }
    ]
}
```

Os campos razurados devem ser substituído pelo **Envelope ID** que salvamos quando [criamos o template](#configurar-conta-template) e pelo e-mail da primeira e segunda pessoa que devem assinar o documento respectivamente. 

Após enviar a requisição você deve receber um retorno com status HTTP ```200``` se tudo der certo como no print

![visualizando retorno com status 200 - sucesso apos requisicao ao docusign](/assets/images/2024-04-14-img/32-testing-envelope-created.png){: .align-center}

Ao clicar em **Manage** no Docusign já devemos ver nosso novo _envelope_ criado.

![visualizando envelopes criados no docusign](/assets/images/2024-04-14-img/33-testing-envelope-created-docusign.png){: .align-center}

Também já podemos visualizar nosso documento no email cadastrado

![visualisando envelope recebido via email](/assets/images/2024-04-14-img/34-testing-envelope-created-email.png){: .align-center}

Vamos assinar nosso documento: clique no link recebido no email, adicione sua assinatura clicando no ícone amarelo e depois clique em **Finish**

![assinando envelope no link recebido via email](/assets/images/2024-04-14-img/35-testing-envelope-signed-first.png){: .align-center}

Ao voltar para o painel do Docusign vemos que o status do nosso envelope foi alterado e que agora só falta 1 dos 2 integrantes do documento assinar.

![visualizando status do envelope no docusign](/assets/images/2024-04-14-img/36-testing-envelope-signed-first-docusign.png){: .align-center}

Vemos também que o email foi enviado para a segunda pessoa que deve assinar o documento.

![visualizando segundo email enviado via docusign](/assets/images/2024-04-14-img/37-testing-envelope-signed-second.png){: .align-center}

### Atualizando um envelope

Agora vamos imaginar que ocorreu um erro e que na verdade a segunda pessoa que recebeu o e-mail para assinar não possui mais acesso aquele e-mail e portanto deseja receber o documento em um novo e-mail. 

Para isso vamos usar o segundo método que criamos. 

Quando fizemos a criação do envelope recebemos como resposta um status ```200``` com um corpo de response, nesse corpo é possível ver um campo chamado **envelopeId**. Copie o valor retornado nesse campo para usar na próxima requisição

![enviando requisicao para atualizar envelope no docusign pelo postman](/assets/images/2024-04-14-img/32-testing-envelope-created.png){: .align-center}

Nossa nova requisição será feita com o ID do envelope que desejamos atualizar e o **novo email** para onde desejamos enviar nosso documento. 

![visualizando body da requisicao](/assets/images/2024-04-14-img/38-testing-envelope-update.png){: .align-center}

Após enviar a requisição recebemos um HTTP ```200``` como resposta caso tudo dê certo. 

![visualizando retorno 200 - sucesso apos atualizar envelope](/assets/images/2024-04-14-img/39-testing-envelope-update-ok.png){: .align-center}

Ao analisar a caixa de entrada do novo e-mail cadastrado podemos ver que já recebemos o documento para assinar

![visualizando novo email para assinatura do envelope](/assets/images/2024-04-14-img/40-testing-envelope-resend-ok.png){: .align-center}

Ao abrir o documento, assinar e clicar em concluir podemos finalizar o processo de assinatura do nosso documento. 

![assinando novo email no docusign](/assets/images/2024-04-14-img/41-testing-envelope-sign.png){: .align-center}

Documento finalizado no Docusign. 

![visualizando status concluido do envelope no docusign](/assets/images/2024-04-14-img/42-testing-envelope-sign-finished.png){: .align-center}

## Próximos passos e Considerações finais

Chegamos ao final da longa jornada!! Ao longo do caminho nós:

- Configuramos uma conta Docusign
- Configuramos um template do zero
- Criamos um mecanismo de autenticação no Docusign
- Criamos uma aplicação capaz de enviar documentos por e-mail com assinatura embutida onde o usuário pode com apenas 3 cliques assinar um documento

O que vem agora?

- Caso queira se aprofundar ainda mais no mundo do Docusign a [Docusign University](https://dsucustomers.docusign.com/page/all-courses) disponibiliza cursos gratuitos
- Antes de ir para produção consulte os preços dos [planos da Docusign](https://ecom.docusign.com/pt-BR/plans-and-pricing/esignature)
- A API da Docusign possui [rate limit](https://blog.hubspot.com/website/api-rate-limit#:~:text=API%20rate%20limiting%20is%20a,what%20actions%20can%20be%20taken.) o que significa que cada chamada de API para criar ou atualizar um _envelope_ deve ser muito bem pensada. Pra isso o Docusign disponibiliza [Webhooks](https://developers.docusign.com/platform/webhooks/) para que sua aplicação não dependa tanto do _rate limit_, vale a pena conferir. 

Bons estudos!

Repositório no github: [docusign-playground](https://github.com/victor-VN/docusign-playground)

Fontes: 

- [Docusign University - Manage Envelopes with the eSignature API](https://dsudevelopers.docusign.com/manage-envelopes-with-the-esignature-api?next=%2Fmanage-envelopes-with-the-esignature-api%2F745977)
- [Stackoverflow - How to get the list of envelopes from a DocuSign account?](https://stackoverflow.com/questions/78009488/how-to-get-the-list-of-envelopes-from-a-docusign-account)
- [Docusign blog - Building best practices webhook listeners, part 4: Behind the firewall with AWS](https://www.docusign.com/blog/dsdev-webhook-listeners-part-4)
- [eSignature Documentation - How to list envelope documents](https://developers.docusign.com/docs/esign-rest-api/how-to/list-envelope-documents/)
- [Javadoc - docusign-esign-java 2.6.2 API](https://javadoc.io/static/com.docusign/docusign-esign-java/2.6.2/overview-summary.html)
- [eSignature Documentation - SDKs](https://developers.docusign.com/docs/esign-rest-api/sdks/)
- [Stackoverflow - Resend DocuSign Emails](https://stackoverflow.com/questions/21565765/resend-docusign-emails)
- [eSignature Documentation - How to send an envelope via your app](https://developers.docusign.com/docs/esign-rest-api/how-to/embedded-sending/)
- [eSignature Documentation - How-to guides overview
](https://developers.docusign.com/docs/esign-rest-api/how-to/)
- [Docusign University - Send Your First Envelope](https://dsudevelopers.docusign.com/send-your-first-envelope)
- [Docusign University - All courses](https://dsudevelopers.docusign.com/page/all-courses#-c,content-types_self-paced,languages_english)
- [Docusign blog - From the Trenches: Common authentication issues post Go-Live](https://www.docusign.com/blog/dsdev-from-the-trenches-may-2020)
- [eSignature Documentation - Templates](https://developers.docusign.com/docs/esign-rest-api/esign101/concepts/templates/)
- [eSignature Documentation - How to create a template](https://developers.docusign.com/docs/esign-rest-api/how-to/create-template)
- [eSignature Documentation - Java SDK authentication
](https://developers.docusign.com/docs/esign-rest-api/sdks/java/auth/)
