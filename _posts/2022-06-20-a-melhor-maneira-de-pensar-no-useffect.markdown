---
layout: post
title:  "A melhor maneira que encontrei de pensar no useEffect"
date:   2022-06-20 17:33 -0300
categories: [javascript, reactjs]
---

### Introdução

Quando comecei a estudar React, eu costumava pensar no <code>useEffect</code> apenas como um <i>hook</i> relacionado ao ciclo de vida dos componentes e isso gerava alguns problemas.

Constantemente me deparava com comportamentos inesperados e era comum uma certa confusão com o <code>useState</code>.

Neste post, pretendo contribuir com uma forma mais fácil de olhar para este recurso.

### Pra começar, vamos lembrar o que é o useEffect

O <i>useEffect</i> é um <i>hook</i>. Hooks são recursos que foram adicionados na versão 16.8 do React que permitem utilizar o <i>state</i> e outras funcionalidades da biblioteca sem o uso de classes.

Mais especificamente, <i>hooks</i> são funções. Em versões anteriores à 16.8, sempre que havia necessidade de adicionar estados a um componente, você obrigatoriamente precisava transformar esse componente em um <i>class component</i>. Para isso, um método construtor era chamado antes do componente ser montado, como mostra esse trecho de código retirado da documentação do React:

```javascript
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
    	count: 0
    };
}
```

O motivo para a adoção de <i>hooks</i> é explicado na <a href="https://pt-br.reactjs.org/docs/hooks-intro.html#classes-confuse-both-people-and-machines" target=_blank> na própria documentação</a>:
>
Classes Confundem tanto Pessoas quanto Máquinas
>
Além de deixar o reuso de código e a organização de código mais difícil, nós percebemos que classes podem ser uma grande barreira no aprendizado de React. Você tem que entender como o this funciona em JavaScript, o que pode ser diferente de como funciona na maioria das linguagens. Você tem que lembrar de fazer bind de event handlers. Sem propostas de sintaxe instáveis, o código pode ficar muito verboso. As pessoas podem entender props, state e fluxo de dados de cima para baixo perfeitamente bem, mas ainda tem dificuldade com classes. A distinção entre componentes de classe e de função em React e quando utilizar cada um deles acabam levando a desentendimentos, mesmo entre desenvolvedores experientes de React.

### ComponentDidMount, ComponentDidUpdate e ComponentWillUnmount

A maneira mais intuitiva que a maioria das pessoas encontram para entender o <code>useEffect</code>, é pensá-lo como um gerenciador do ciclo de vida dos componentes.

Antes do <code>useEffect</code>, o React usava três métodos de ciclo de vida para lidar com efeitos colaterais: <code>ComponentDidMount</code>, <code>ComponentDidUpdate</code> e <code>ComponentWillUnmount</code>.

![printscrap]({{"/img/post-2/print_lifecycle.png" | absolute_url }})

Por agora, iremos apenas pensar no <code>useEffect</code> como um <i>hook</i> que combina todos esses métodos em apenas um método. Mais à frente, abordarei algumas problemáticas que acompanham essa interpretação.

#### 1. ComponentDidMount

<code>ComponentDidMount</code> era invocado sempre que um componente precisava ser montado.

```javascript
componentDidMount() {
  setTimeout(() => {
    this.setState({favoritecolor: "yellow"})
  }, 1000)
}
````

Para obtermos um efeito semelhante ao <code>ComponentDidMount</code> com o <i>hook</i> <code>useEffect</code>, passamos no segundo parâmetro da função, um <i>array</i> de dependências vazio <code>[]</code> como argumento:

```javascript
useEffect(()=>{
   setTimeout(() => {
      setFavoriteColor("yellow");
   }, 1000)
   //array de dependências vazio
}, [])
````

O <i>array</i> de dependências é o responsável por informar ao <code>useEffect</code> que ele deverá ser executado.

Um <i>array</i> vazio significa que o <code>useEffect</code> deverá ser executado quando o componente for renderizado pela primeira vez. 

#### 2. ComponentDidUpdate

<code>ComponentDidUpdate</code> era invocado sempre que um componente fosse atualizado.

```javascript
componentDidUpdate() {
   document.title = `Você clicou ${this.state.count} vezes`; 
}
````

Atualizações são causadas quando o <i>state</i> e as propriedades (<i>props</i>) são alterados.

Para obtermos o efeito de <code>ComponentDidUpdate</code> usando <code>useEffect</code>, precisamos especificar as <i>props</i> e o <i>state</i> dentro do array de dependências.


```javascript
useEffect(() => {
   document.title = `Você clicou ${count} vezes`;
}, [count]);
````
Sempre que o valor das dependências mudarem, o <code>useEffect</code> será executado.

#### 3. ComponentWillUnmount

<code>ComponentWillUnmount</code> era chamado quando um componente precisava ser desmontado (removido do DOM).

Esse <i>hook</i> estava atrelado à limpeza dos efeitos colaterais do componente. 

#### Limpeza com useEffect

Depois que o <code>useEffect</code> executa uma ação, ele não necessariamente realiza a limpeza dos efeitos colaterais dessa ação, a não ser que a necessidade dessa limpeza esteja explícita. 

Deve-se limpar o componente sempre que um efeito colateral não precisar mais ser renderizado. Você pode pensar nisso como um "lixo" que é deixado após a execução das ações.

```javascript
const Component => () => {

   useEffect(() => {

     return () => {
       //aqui a limpeza do componente é executada
       }
   }, [])
}
```

#### O estado <i>default</i> do useEffect

```javascript
const [count, setCount] = useState(0);

useEffect(() => {
  setCount((count) => count + 1); 	
});

console.log(count)
````

O código acima será executado em um <i>loop</i> infinito.

Se você já tentou realizar alguma requisição à uma API sem declarar um <i>array</i> de dependências vazio de modo similar ao trecho de código acima, deve ter se deparado com o mesmo problema.

Isso acontece porque se um <i>array</i> de dependências não for declarado, o <code>useEffect</code> entenderá que deverá ser executado à cada renderização.

Um outro resultado semelhante pode ocorrer se você declarar dentro do <i>array</i> de dependências um estado que se altera constantemente.

Efeitos acontecem como um produto da renderização. Quando você atribui um estado, esse estado muda e desencadeia um efeito que desencadeia uma outra renderização. É isso que causa o <i>loop</i> infinito no estado padrão do <code>useEffect</code>.

### O problema de pensar no useEffect apenas como um <i>hook</i> ligado ao ciclo de vida dos componentes

É um fato que a <a href="https://pt-br.reactjs.org/docs/hooks-effect.html" target=_blank>própria documentação oficial do React</a> recomenda pensar no <code>useEffect</code> como uma forma de combinar <code>ComponentDidMount</code>, <code>ComponentDidUpdate</code> e <code>ComponentWillUnmount</code>. 

Temos usado essa abordagem até então e tem funcionado muito bem até aqui. Porém,  eventualmente essa interpretação poderá ser uma fonte de problemas, pois embora pensar neste <i>hook</i> do ponto de vista de um método de ciclo de vida seja mais intuitivo, pois estes ciclos têm uma natureza descritiva (componente é montado, atualizado e desmontado), essa nem sempre é a melhor abordagem. Olhar somente sob essa perspectiva pode nos levar a um uso descontrolado desse componente.

É importante entender que as diferenças entre classes e funções podem até não ser tão grandes, mas são suficientes para levar a algumas confusões se não forem entendidas completamente.

Neste <a href="https://overreacted.io/how-are-function-components-different-from-classes/" target=_blank>artigo</a>, Dan Abramov usa este <a href="https://codesandbox.io/s/pjqnl16lm7" target=_blank>exemplo</a> para explorar um comportamento "inadequado" que encontraríamos no código ao confundir o uso de <i>class components</i> com o uso de <i>classless components</i>.

Se você tentar a sequência:

1. Clicar em um dos botões "Follow"
2. No seletor, mudar de "Dan" para "Sophie" antes que se passe 3 segundos
3. Ler o alerta de texto

Você perceberá que ao clicar no segundo botão "Follow" (construído com <i>class component</i>) e depois navegar até o segundo perfil, será exibido o alerta "Followed Dan". O mesmo não acontece ao clicar no primeiro botão "Follow" (construído sem o uso de classes), que é o comportamento esperado.

Ele explica:

>
Esse método de classe faz a leitura de <code>this.props.user</code>. Props são imutáveis em React. Entretanto, o <code>this</code> muda e sempre mudou.
>
De fato, esse é o propósito do <code>this</code> numa classe. O próprio React muda com o tempo para que se obtenha a versão mais nova no <code>render</code> e nos métodos de ciclo de vida.
>
Então se o nosso componente re-renderizar enquanto a requisição está em percurso, <code>this.props</code> vai mudar. O método <code>showMessage</code> lê o <code>user</code> das <code>props</code> "mais recentes".
>
Isso expõe uma observação interessante sobre a natureza das interfaces de usuário. Se nós dissermos que o UI é conceitualmente  uma função da aplicação atual, os event handlers são parte do resultado da renderização - assim como o resultado visual. Nossos event handlers "pertencem" à uma renderização particular com props e states particulares.
>
Entretanto, programar um timeout cujo callback lê de <code>this.props</code> quebra essa associação. Nosso callback de <code>showMessage</code> não é ligado a nenhuma renderização em particular, então ele "perde" as props corretas. Ler a partir do <code>this</code> corta essa conexão.

Foi procurando entender a diferença entre <i>hooks</i> e classes um pouco mais a fundo que me deparei com um <a href="https://beta.reactjs.org/learn/synchronizing-with-effects" target=_blank>outro artigo</a> na documentação beta do React que me levou a ter uma nova visão sobre este <i>hook</i>.

>
Você pode não precisar de um Effect
>
Não se apresse para adicionar Effects aos seus componentes. Mantenha em mente que Effects são tipicamente usados para "sair" do seu código React e sincronizar com algum sistema externo. Isso inclui APIs, widgets de terceiros, redes e assim por diante. Se o seu effect apenas ajusta algum state baseado em outro state, você pode não precisar de um Effect.

Aqui eu entendi a fonte de todo o meu desentendimento com o <code>useEffect</code>.

Olhar para o <code>useEffect</code> do ponto de vista de montagem e atualização está nos impossibilitando de enxergar do ponto de vista das dependências (<i>states e props</i>) pois o verdadeiro propósito do <code>useEffect</code> é <strong>sincronizar</strong> essas dependências com os efeitos colaterais!

Quando pensamos em ciclos de vida, estamos pensando em termos de tempo de renderização, enquanto <i>hooks</i> foram desenvolvidos para resolver o problema dos estados e a sincronização com o DOM. Ou seja, a pergunta "Quando devemos usar o useEffect?" não é a pergunta certa.

A pergunta certa é "Quando esse estado mudar, quais efeitos colaterais eu preciso renderizar?".

Ryan Florence resume muito bem em um único <a href="https://twitter.com/ryanflorence/status/1125041041063665666" target=_blank><i>tweet</i></a>:

>
The question is not "when does this effect run" the question is "with which state does this effect synchronize with"
>
useEffect(fn) // all state
>
useEffect(fn, []) // no state
>
useEffect(fn, [these, states])

### Resumo (TL;DR):

Então diferentemente de componentes de classe, componentes funcionais capturam <i>state</i> e <i>props</i>. 

Resumidamente, entender o <code>useEffect</code> se torna mais fácil se ele for pensado:

- Do ponto de vista das dependências
- Como um controlador de fluxo de efeitos
- Como um sincronizador de um sistema externo (ex: consumir APIs, conectar ao servidor)

E não como:

- Um <i>state setter</i>

E muito raramente como:

- Apenas um <i>hook</i> ligado ao ciclo de vida dos componentes