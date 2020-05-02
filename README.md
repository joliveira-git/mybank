#Micro-App Em Angular Usando Web Components

Projeto

Tecnologias utilizadas: 
- Angular
- Web Components


Danilo Rodrigues - Zup
28/04 - 18:00

#Era Monolítica

Base do desenvolvimento front-end atual
Como se entrega front-end: equipe responsável por toda a aplicação de um produto

E-Comerce
- Vitrine
- Checkout
- Dashbord

A cada alteração tem que gerar tudo de novo. Uma peça grande entregável


#O antes e o depois:
 
# Monolítico
  - Deploy de uma grande aplicação
  - Geralmente desolvidade por um só time
  - Atualizações ou correções de bugs afetam toda a aplicação

# Micro Frontends
  Arquitetura que afeta não só o código, mas tbm a estrutura organizacional da empresa.
  Flexibilidade com times autônomos

  - Maior escalabilidade organizacional com time autônomos
  - Maior habilidade de atualizações e correção de bugs sem afetar o todo
  - Agilidade de deploy de partes

  Entrega de valor mais rápido

  Micro Frotend: estilo arquitetural onde aplicações front-end são entregáveis independentemente e que formam um todo maior.


# Web Components

  1- Registra um novo elemento customizado
  Ou seja, o elemento customizado vem estendido de um elemento HTML já existente.

  2 - O aplicativo adiciona o custom element no no DOM
  o Browser guarda a tag e quando vier a classe web component, automaticamente instancia

  3 - Browser instancia a component-based class 

  4 - Provê a instância com data binding e change detection.

Exemplo:

 ```
  class WordCount extends HTMLParagraphElement {
      constructor(){
          // Always call super first in constructor
          super();

          // Element functionality written in here

      }
  }

  customElements.define('word-count', WordCount, { extends: 'p' });
```

# Angular + Web Components
Como funciona Web Components no Angular:

Angular tem um módulo AngularElement

Angular App:
1 - Constroi o componente no Angular e nele deve-se construir um custom element class
```
MyComponent:

 CreateCustomElement()
           |
           V
      MyElementClass
           |
           V
 customElements.define("my-tag", MyElementClass)
```
  2 - Registra com browser

  Browser:
      CustomElementRegistry:
          ...
          myTag
          MyElementClass
          ...
              |
              V
      myElementInstance

      --------------------------
              |
              v

      DOM:
      <my-tag my-input='x'> </my-tag>
```

#Arquitetura da Aplicação:

    - APP MASTER: Shell ou Wrapper da aplicação -----
        - HEADER-------------------------------------
        -                                           -
        ---------------------------------------------

        Extrato | Pagamento
        ---------------------------------------------
        -                                           -
        -         <micro-app><micro-app>            -
        -                                           -
        -                                           -
        ---------------------------------------------

# Bora pro código:

# 1 - Criar a aplicação principal
```
ng new app appmaster
```

# 2 - Criar as aplicações filhas
Quando cria um app dentro de outro, o Angular coloca o novo projeto na pasta projects, podendo ter vários apps trabalhando dentro do mesmo repositório.
Criar sem a configuração de rotas.

Criar a aplicação extrato:
```
ng g app extrato
```
Criar a aplicação pagamento:
```
ng g app pagamento
```
Os apps Extrato e pagamento serão instanciados através do web component, ou seja serão apps dentro de outro.
Não precisa instanciar como um app normal, por isso é necessário fazer as seguintes mudanças:
Em app.module.ts:
- Retirar o metadado bootstrap, pq não dar o boostrap sozinho
- Instalar o pacote Angular Elements em todos os projetos:
Quando se trabalha com mais de um projeto dentro de outro, é necessário indicar o projeto onde se quer fazer a instalação. Quando não passa nada, o módulo é instalado no app master.

```
    ng add @angular/elements 
    ng add @angular/elements --project extrato
    ng add @angular/elements --project pagamento
```
Adiciona document-register-element nas dependências e inclui também no polyfills para suportar Browser antigos.
- Adicionar um construtor para construir o web component que será instanciado no webmaster. A mágica é feita pela função createCustomElement que recebe o componente Angular e retorna uma classe webcomponent. Em seguida, usar a função customElements passando o nome da tag e o elemento. Por fim, deve-se rodar o ngDoBootstrap para fazer o bootstrap já que o mesmo foi retirado dos metadados do @NgModule.

```
    import { BrowserModule } from '@angular/platform-browser';
    import { NgModule, Injector } from '@angular/core';
    import { createCustomElement } from '@angular/elements';
    import { AppComponent } from './app/component';

    export class AppModule{
        constructor(private injector: Injector){
            // O código abaixo tbm pode ficar dentro do método ngDoBootstrap
            const elem = createCustomElement(AppComponet, {injector: this.injector});
            customElements.define('micro-app-extrato', elem);
        }

        ngDoBootstrap(){

        }
    }

```

# 3 - Fazer a chamada dos apps filhos no app master
Em appmaster:

app.component.html:
Incluir as tags que serão usadas para injetar os web components no app master.
```
    ...
    <main>
        <micro-app-extrato></micro-app-extrato>
        <micro-app-pagamento></micro-app-pagamento>
    </main>        
```

app.module.ts: 
Acrescentar CUSTOM_ELEMENTS_SCHEMA nos schemas no módulo para reconhecer as tag customizadas dos aplicativos externos, ou seja as tags utilizadas não fazem parte de nenhum componente do app master. Indica que as tags são web component.
```
@NgModule({
    declarations: [
        AppComponent
    ],
    imports:[
        BrowserModule,
        NgbModule
    ],
    providers: [],
    schemas:[CUSTOM_ELEMENTS_SCHEMA],
    bootstrap: [AppComponent]
})
export class AppModule{ }
```

# 4 - Chamar no app master os outros apps

appmaster:

app.component.ts:
```
    @Component({
        selector: 'app-root',
        templateUrl: './app.component.html',
        styleUrls: ['./app.component.css']
    })
    export class AppComponent{
        ngOnInit(){
            const scriptExtrato = document.createElement('script');
            scriptExtrato.src = 'http://localhost:8080/extrato/main-es2015.js';
            document.body.appendChild(scriptExtrato);
            
            const scriptPagamento = document.createElement('script');
            scriptPagamento.scr = 'http://localhost:8080/pagamento/main-es2015.js';
            document.body.appendChild(scriptPagamento);

        }
    }

```
* Vantagem: Equipes remotas deployando as partes de forma independente. Pode-se fazer o deploy de cada app forma independente e o app master não precisar ser deployado quando muda alguma coisa em um de seus apps.

# 5 - Deploy

- Instalar a dependência ngx-build-plus no app master.
Essa dependência faz um wrapper em todos os scripts de um app Angular e coloca em apenas um Bundle.
Então vamos usar ngx-build ao invés do ng-build padrão do Angular.
```
   npm i --save ngx-build-plus

```

- Adicionar os esquema do ngx-build nos apps.
Vai escrever no angula.json quem é que vai fazer o build do projeto (quando rodar o ng-build).
```
    ng add ngx-build-plus
    ng add ngx-build-plus --project extrato
    ng add ngx-build-plus --project pagamento

```

em angular.json trocou de:
          "builder": "@angular-devkit/build-angular:karma",
para: 
          "builder": "ngx-build-plus:browser",

- Deployar a aplicação
Os artefatos serão gerados na pasta dist

Adicionar comando para fazer build dos apps no package.json:

```
    "build:extrato": "ng build --prod --project extrato --single-bundle --output-hashing none",
    "build:pagamento": "ng build --prod --project pagamento --single-bundle --output-hashing none",
```
para rodar o script: btn dir em cima do nome do script / run script.

singule-bundle: não gera o hash no final do nome do arquivo para ficar mais fácil deployar pq senão teremos que ficar trocando no script.

Quando terminar de rodar o bundle será gerado na pasta dist
dist/extrato/main-es5.js
dist/pagamento/main-es5.js

```
ng serve
```

ERRO! A aplicação não abre e ao verificar o console do browser é possível ver a seguinte mensagem: 

```
src sync:2 Uncaught Error: Cannot find module 'tslib'
```

O problema ocorre pq o autoimport do VSCode pega CUSTOM_ELEMENTS_SCHEMA de '@angular/compiler/src/core', enquanto o correto seria pegar de '@angular/core'.
```
Para resolver o problema troque:
import { CUSTOM_ELEMENTS_SCHEMA } from '@angular/compiler/src/core'; (ERRADO)
Por:
import { CUSTOM_ELEMENTS_SCHEMA } from '@angular/core'; 
```

NOTA: Lembrar sempre de conferir os autoimports do VSCode. ;-)

# Criar o componente extrato-list dentro do componente extrato.

```
ng g c extrato-list --project extrato

```
no componente filho (extrato-list):
extrato-list.component.html:

extrato-list.component.ts:


no componente Raiz do app extrato:
Definir o local onde será renderizada a lista:
app.component.html:
```
<router-outlet><router-outlet>
```

Definir a rota em app.module.ts:
ATENÇÃO: Tratar os módulos como aplicativos independentes.
Construir a rota raiz da cada aplicação usando RouterModule.forRoot e não confundir com os módulos lazy load que são carregados com forChild.
```
@NgModule({
  declarations: [
    AppComponent,
    ExtratoListComponent
  ],
  imports: [
    BrowserModule,
    RouterModule.forRoot([
      {
        path: 'extrato',
        component: ExtratoListComponent
      }
    ])
  ],
  providers: []
})
export class AppModule { 
  constructor(private injector: Injector){
    const elem = createCustomElement(AppComponent, {injector: this.injector});
    customElements.define('mirco-app-extrato', elem);
  }

  ngDoBootstrap(){

  }

}
```

Se rodar o código, não vai instanciar o componente.
Como o app extrato não está fazendo o bootstrap, é necessário adicionar um construtor em app.component.ts do componente extrato.
Precisa instanciar a nagevação de rotas para ela identificar as rotas dentro do Angular. Da forma como está se rodarmos a aplicação vai instanciar apenas o webcomponent e não o Angular.

app.component.ts:
```
import { Component } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'extrato';
  constructor(private router:Router){
    this.router.initialNavigation();
  }
}
```

fazer o deploy novamente do app extrato.

- Instalar live-server
```
    npx live-server ./dist

```
precisa instanciar o localhost na porta 8080

------------------------------------------------
Para identificar erros específicos de um determinado módulo é interessante subir o módulo separado do app master.
ng serve --project extrato --port 4002


ERRO:

SOLUÇÃO: Instalar ng-bootstrap:
Esse comando funciona:
npm install -save @ng-bootstrap/ng-bootstrap

Esse comando não funciona:
ng add @ng-bootstrap/ng-bootstrap
ng add @ng-bootstrap/ng-bootstrap --project myproject



Dica:
conversor de código CSS para SASS
https://css2sass.herokuapp.com/


Problemas de abordagem: Aplicativo fica grande

Cada app mantém o seu próprio estado. Variáveis de estado podem ser compartilhadas entre apps utilizando input e output property do Angular.

O estilo de Arquitetura microapps tem os seus trade-offs: A decisão depende da definição de onde se quer ganhar. Verificar se vale a pena ou não arcar com os custos do trade-off.
Para qualquer tipo de arquitetura há vantagens e desvantagens.


Artigo sobre micro front-end com exemplo em react.
