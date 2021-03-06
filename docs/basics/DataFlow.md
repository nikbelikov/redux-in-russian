# Поток данных (Data Flow)

Архитектура Redux вращается вокруг **строго однонаправленного потока данных**.

Это значит, что все данные в приложении следуют одному паттерну жизненного цикла, делая логику Вашего приложения более предсказуемой и легкой для понимания. Также это способствует большей упорядоченности данных (data normalization), так что в конечном итоге у Вас не будет нескольких изолированных копий одних и тех же данных, которые ничего не знают друг о друге.

Если Вы до сих пор не убеждены, прочтите [Мотивацию (Motivation)](../introduction/Motivation.md) и [The Case for Flux](https://medium.com/@dan_abramov/the-case-for-flux-379b7d1982c6) для ознакомления с убедительными аргументами в пользу однонаправленного потока данных. Хотя [Redux - это не совсем Flux](../introduction/PriorArt.md), он дает такие же основные преимущества.

Жизненный цикл данных в любом  Redux приложении включает в себя 4 шага:

1. **Вы вызываете** [`store.dispatch(action)`](../api/Store.md#dispatch).

  Действие - это простой javascript объект, который описывает *что случилось*. Например:

    ```js
    { type: 'LIKE_ARTICLE', articleId: 42 };
    { type: 'FETCH_USER_SUCCESS', response: { id: 3, name: 'Megan' } };
    { type: 'ADD_TODO', text: 'Read the Redux docs.'};
    ```

  Думайте о действии, как об очень коротком фрагменте новостей. "Мэри залайкала статью 42" или "'Прочитать документацию Redux' было добавлено в ToDo список".

  Вы можете вызвать [`store.dispatch(action)`](../api/Store.md#dispatch) из любого места Вашего приложения, включая компоненты и XHR-колбеки или даже с запланированными интервалами.

2. **Хранилище Redux вызывает функцию-редьюсер, который Вы ему передали.**

  Хранилище передаст два аргумента при вызове редьюсера: текущее дерево состояния (current state tree) и действие (action). Например, в ToDo приложении главный редьюсер может принимать что-то такое:
   
    ```js
    // Текущее состояние приложения (список дел и выбранный фильтр)
    let previousState = {
      visibleTodoFilter: 'SHOW_ALL',
      todos: [{
        text: 'Read the docs.',
        complete: false
      }]
    };

    // Выполнение действия (добавление дела)
    let action = {
      type: 'ADD_TODO',
      text: 'Understand the flow.'
    };

    // Ваш редьюсер возвращает следующее состояние приложения
    let nextState = todoApp(previousState, action);
    ```

    Обратите внимание на то, что редьюсер - это чистая функция. Он только *вычисляет* следующее состояние. Он должен быть совершенно предсказуемым: тип возвращаемых данных не должен меняться, если на вход подаются данные одного типа. Он не должен совершать никаких сайд-эффектов, таких как обращение к API или маршрутизация по приложению. Все это должно происходить только после того, как действие будет совершено.

3. **Главный редьюсер может комбинировать результат работы несольких редьюсеров в единственное дерево состояния приложения.**

  Каким образом вы будете структурировать главный редьюсер зависит только от Вас. Redux поставляется с [`combineReducers()`](../api/combineReducers.md) хелпером, полезным для "разделения" главного редьюсера на отдельные функции, которые управляют отдельными ветвями дерева состояния.

  [`combineReducers()`](../api/combineReducers.md) работает следующим образом. Допустим у Вас есть два редьюсера: один для списка ToDo дел, второй - для выбранного сейчас режима отображения этого списка:

    ```js
    function todos(state = [], action) {
      // как то вычисляет nextState...
      return nextState;
    }

    function visibleTodoFilter(state = 'SHOW_ALL', action) {
      // как то вычисляет nextState...
      return nextState;
    }

    let todoApp = combineReducers({
      todos,
      visibleTodoFilter
    });
    ```

  Когда вы инициируете действие, `todoApp`, которое вернул `combineReducers` вызовет оба редьюсера:

    ```js
    let nextTodos = todos(state.todos, action);
    let nextVisibleTodoFilter = visibleTodoFilter(state.visibleTodoFilter, action);
    ```

  Затем оба набора состояний будут снова собраны в единое состояние:

    ```js
    return {
      todos: nextTodos,
      visibleTodoFilter: nextVisibleTodoFilter
    };
    ```

  Так как [`combineReducers()`](../api/combineReducers.md) - это просто удобная утилита, Вы совершено не обязаны ее использовать. Вы можете написать главный редьюсер самостоятельно!

4. **Хранилище Redux сохраняет полное дерево состояния, которое возвращает главный редьюсер.**

  Это новое дерево является следующим состоянием Вашего приложения! Каждый слушатель, зарегистрированный с помощью [`store.subscribe(listener)`](../api/Store.md#subscribe), будет вызван. Слушатели могут вызывать [`store.getState()`](../api/Store.md#getState) для получения текущего состояния приложения.

  Теперь UI может быть обновлен для отражения нового состояния приложения. Если Вы используете такие биндинги (bindings), как [React Redux](https://github.com/gaearon/react-redux), то это та точка, в которой стоит вызвать `component.setState(newState)`

## Следующие шаги

Теперь, когда Вы знаете как работает Redux, давайте [свяжем его с React приложением](UsageWithReact.md).

>##### Заметка для опытных пользователей

> Если Вы уже знакомы с основными концепциями и уже освоили это обучающее руководство, то не забудьте посетить [асинхронный поток (async flow)](../advanced/AsyncFlow.md) в [руководстве для опытных](../advanced/README.md) для изучения того, как именнно middleware изменяют [асинхронные действия](../advanced/AsyncActions.md) 
