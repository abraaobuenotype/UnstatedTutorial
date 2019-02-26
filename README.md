## Intorução

Usando o **Redux** para esse estudo de caso precisaríamos criar um arquivo **reducer**, que armazenaria o state e comumente teria um _switch_ para alterar o _state_ dependendo do _action.type_, precisaríamos de um arquivo para armazenar as **actions** com funções que dispararia os _eventos_ para alterar o _state_. Além disso mais um arquivo que seria a configuração da **Store**. Não vou entrar em detalhes quanto a utilização do Redux, pois apenas nessa introdução já observamos uma certa complexidade. Mostrarei a seguir um estudo de caso utilizando a Context API do React através da biblioteca Unstated.

## Unstated

O Unstated possiu dois componentes e uma classe, que são:

**Provider**

Componente responsável pela distribuição do estados, geralmente usado uma única vez sobre o App

**Subscribe**

Componente responsável por _ouvir_ a mudança no state e fornecer acesso às actions

**Container**

Classe extendida pelo arquivo de estado. É aqui que definimos nossos states e actions

## User Case

Precisamos de um state para armazenar a configuração de idioma que o usuário está utilizando e preciso de actions que permita modificar esse state caso o usuário deseje alterar o idioma do aplicativo.

**passo 1** - criando o state:

**Redux:**

Arquivo ui_reducer.js
```
const initialState = {
    language: null
}

const uiReducer = (state = initialState, action) => {
    switch (action.type) {
        case 'SET_LANGUAGE':
            state = {
                ...state,
                language: action.payload
            }
            break;
    }

    return state
}

export default uiReducer
```
Arquivo ui_action.js
```
export function getLanguage () {
    let lang = getBrowserLanguage()
    if (typeof lang === 'string') {
        lang = lang.substr(0, 2).toLowerCase() || 'en'
    }

    return window.localStorage.getItem('ui.language') || lang || 'en'
}

export function setLanguage (language) {
    return {
        type: 'SET_LANGUAGE',
        payload: language
    }
}
```

***
**Unstated:**


Criaremos um arquivo chamado UiState.js na pasta states (sugestão)
```
import { Container } from 'unstated'

class UiState extends Container {
    // aqui definimos nosso state
    state = {
        language: null
    }

   //action para obter o idioma atual de um arquivo externo (note que essa action não modifica o state)
   getLanguage = () => {
        let lang = getBrowserLanguage()
        if (typeof lang === 'string') {
            lang = lang.substr(0, 2).toLowerCase() || 'en'
        }

        return window.localStorage.getItem('ui.language') || lang || 'en'
    }


   //action para setar o idioma (esse método altera a action)
    setLanguage = (language) => {
        this.setState({ language })
    }
}

export default UiState
```

**passo 2** - configurando o App:
No nosso aquivo Main utilizamos o Provider para ter acesso ao contexto nos outros componentes

**Redux:**
```
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import Store from './Store'
import App from './App'

render(
    <Provider store={Store}>
        <App />
    </Provider>,
    document.querySelector('#react-app')
)
```

***
**Unstated:**

```
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'unstated'
import App from './App'

render(
    <Provider>
        <App />
    </Provider>,
    document.querySelector('#react-app')
)
```

**passo 3** - utilizando o state
Agora utilizaremos o state que criamos para dar acesso ao usuário modificar o idioma através de botões na interface.

**Redux:**

```
import React, { Component, Fragment } from 'react'
import { connect } from 'react-redux'
import { setLanguage } from './ui_action'

class App extends Component {
    toggleLanguage = (event) => {
        let { ui, setLanguage } = this.props
        let { language } = ui

        setLanguage(language === 'pt' ? 'en' : 'pt')
    }

    render () {
        let { ui } = this.props
        let { language } = ui

        return (
            <Fragment>
                <button
                    onClick={this.toggleLanguage}
                >
                    linguagem: {language === 'pt' ? 'en' : 'pt'}
                </button>
            </Fragment>
        )
    }
}

const mapStateToProps = state => {
    return {
        ui: state.ui
    }
}

const mapDispatchToProps = dispatch => {
    return {
        setLanguage: language => dispatch(setLanguage(language))
    }
}

export default connect(mapStateToProps, mapDispatchToProps)(App)
```

***
**Unstated:**

```
import React, { Component, Fragment } from 'react'
import { Subscribe } from 'unstated'

import UiState from './states/UiState'

class App extends Component {
    toggleLanguage = (language, setLanguage) => (event) => {
        //aqui alteramos o state diretamente, utilizando o método setLanguage
        setLanguage(language === 'pt' ? 'en' : 'pt')
    }

    render () {
        return (
            <Fragment>
                {/* aqui passamos noss UiState importado para o componente Subscribe /*}
                <Subscribe to={[UiState]}>
                    {
                        //aqui recebemos o arquivo UiState como argumento, então desconstruímos
                        //para obter o state e o método setLanguage
                        ({ state, setLanguage }) => {
                            let { language } = state

                            return (
                                <button
                                    onClick={this.toggleLanguage(language, setLanguage)}
                                >
                                    linguagem: {language === 'pt' ? 'en' : 'pt'}
                                </button>
                            )
                        }
                    }
                </Subscribe>
            </Fragment>
        )
    }
}

export default App
```

## Outros Exemplos

Você pode utilizar quantos states forem necessário em um mesmo arquivo:

```
<Subscribe to={[UiState, UserState]}>
    {
        ({ state: uistate, setLanguage }, { state: userstate }) => {
           let { language } = uistate
           let { name, email } = userstate

           return (
               <div>
                   {//...}
               </div>
           )
         }
     }
</Subscribe>
```

Se precisar executar uma action antes de iniciar um componente pode fazer da forma a seguir:

```
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'unstated'

import App from './App'
import UiState from './states/UiState'

//criamo uma nova instancia da classe UiState
let uiState = new UiState()

//agora temos acesso aos métodos da classe
uiState.setLanguage(uiState.getLanguage())

render(
    //injetamos no provider nossa classe inicializada
    //fazendo isso os outros componentes ja terão acesso ao state atualizado
    <Provider inject={[uiState]}>
        <App />
    </Provider>,
    document.querySelector('#react-app')
)
```
