# Closing the confusion between Redux and MobX

I used Redux excessively the last year, but spent the recent time with [MobX](https://mobxjs.github.io/mobx/) as state management alternative. It seems that an upcoming alternative to Redux evolves naturally into confusion in the community. People are uncertain which solution to pick. The issue isn't necessarily Redux or MobX. Whenever there exists an alternative, people are curious what's the best way to solve their problem. I am writing these lines now to clear the confusion around both state management solutions Redux and MobX.

In the beginning of 2016 I wrote a [fairly big application in React + Redux](https://github.com/rwieruch/favesound-redux). After I discovered MobX as alternative some time ago, I took the time to refactor the [application from Redux to MobX](https://github.com/rwieruch/favesound-mobx/pull/1). Now I am pretty comfortable in using both and hopefully in explaining both approaches.

## What problem do we solve?

Everyone wants to have state management in an application. But what problem does it solve for us? Most people start with a small application and already introduce a state management library. That's way too fast introducing it, because you will never experience which problem you solve by using it.

It doesn't matter if you use a library/framework like React or Angular. Nowadays the status quo is to build a frontend application with components. Components have internal state, yet you may have external state in services as well. In a growing application the state management can escalate quickly.

* components share state
* state should be accessible in services
* components need to mutate the state of another component
* ...

It gets harder and harder to reason about the application state.

The solution therefore is to introduce a state management library like MobX or Redux. It gives you tools to save your state somewhere, to update your state and to receive notifications from your state once it updated. You have one place to find your state, one place to change it and one place to get updates from.

## Functional Programming vs Object-Oriented (Reactive) Programming

Redux is influenced by functional programming principles. It can be done in JavaScript, but a lot of people come from an object-oriented background and have difficulties to adopt functional programming principles in the first place. In Redux your state is immutable. Instead of mutating your state you always return a new state. You don't depend on mutations of object references.

Additionally Redux embraces the usage of pure functions. A function gets an input, returns an output and does not have other dependencies but pure functions. You can be sure that you will always get the same output when the input stays the same. Both principles, immutability and pure functions, enable you to avoid side effects.

MobX is influenced by object-oriented programming, but also by reactive programming. One can declare data as observable and it embraces to mutate the data directly. The data can have plain setters and getters, but the observable makes it possible to retrieve updates once the data changed.

In Redux it would need the following lines of code to add a new user to the global state. You can see how we make use of the ES6 object spread operator to return a new state object.

```javascript
// global state
{
  users: [
    {
      name: 'Dan'
    },
    {
      name: 'Michel'
    }
  ]
}

// action to add a user
{ type: 'USER_ADD', user: { id: 3, name: Andre } }

// reducer to apply the change in the global state
function users(state, action) {
  switch (action.type) {
  case 'USER_ADD':
    return { ...state, users: [ ...state.users, action.user ] };
  default:
    return state;
  }
}
```

In MobX a store would only manage a substate, but you are able to mutate the state directly. An annotation like @observable makes it possible to observe state changes.

```javascript
class UserStore {
  @observable users = [
    {
      name: 'Dan'
    },
    {
      name: 'Michel'
    }
  ];

  @action addUser = (user) => {
    this.users.push(user);
  }
}
```

## Learning Curve in React State Management

Both Redux and MobX are mostly used in React applications. But they are standalone libraries for state management, which could be used everywhere without React. Only their interoperability libraries leverage the connection to React components. It is [react-redux](https://github.com/reactjs/react-redux) for Redux + React and [mobx-react](https://github.com/mobxjs/mobx-react) for MobX + React. Later I will explain how to use both in a component tree.

In recent discussions it happened that people argued about the learning curve in Redux. It was often in the context of React: people began to learn React and already wanted to leverage state management with Redux. Most people would say that React and Redux itself have a good learning curve, but both together can be overwhelming. An alternative therefore would be MobX, because it is more suitable for beginners.

I would suggest a different approach for newcomers to learn state management in React ecosystem. Start to learn React with its own component state management functionality: setState. In a React application you will first learn the lifecycle methods and you will deal with component state management by using setState. Eventually you will realize that the (internal) state management of components is getting difficult once you want to share state across components.

Now we are at the point: What problem does MobX or Redux solve for us. Both libraries give a way of managing application state external to the components. Components can access the state, manipulate it (explicit, implicit) and update with the new state.
Now you have to make the decision to choose a state management library. You know why you need to solve the problem in the first place. Moreover after having already a larger application in place, you should feel comfortable with React by now.

## Redux or MobX for Newcomers?

Once you are familiar with React components and the internal state management, you can choose a state management library to solve your problem. After I used both libraries, I would say MobX can be very suitable for beginners.

It is suitable, because it can be used for [internal component state in exchange for React setState](https://medium.com/@mweststrate/3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e) as well. I would say you should keep setState over MobX for internal component state management, but the knowledge that MobX can leverage it quite easy might help to get started with MobX.

In MobX one doesn't need to be familiar with functional programming. Terms like immutability might be still foreign. Functional programming is a rising paradigm, but novel for most people in JavaScript. There is a clear trend towards it, but since not everyone has a functional programming background, it might be easier for people with an object-oriented background to adopt the principles of MobX.

## A Growing Application

In MobX you are mutating annotated objects and your components will render an update. MobX comes with more internal implementation magic than Redux, which makes it easier to use in the beginning without much boilerplate code. Coming from an Angular background it felt very much like using two-way data binding. You hold some state in a service, watch the state and update the component once the state was mutated.

MobX allows it to mutate the state directly from the component tree.

```javascript
// component
<button onClick={() => store.users.push(user)} />
```

A better way of doing it would be to have a MobX @action in the store.

```javascript
// component
<button onClick={() => store.addUser(user)} />

// store
@action addUser = (user) => {
  this.users.push(user);
}
```

It would make the state mutating more explicit with actions. After that one doesn't have the feeling anymore of two-way data binding. Moreover there exists a little functionality to enforce state mutations on via explicit actions like you have seen above.

```javascript
// root file
import { useStrict } from 'mobx';

useStrict(true);
```

Mutating the state directly in the store like we did in the first example wouldn't work anymore. Coming from the first to the latter example shows how to embrace best practices in MobX. Moreover once you are doing explicit actions only, you are not far away from using Redux constraints.

I would recommend to use MobX to kickstart projects. Once the application grows in size and contributors, it makes sense to apply best practices like using explicit actions. They are embracing the Redux constraints, which say you can only change the state by using actions.

## Transition to Redux

Once your application gets bigger and has multiple developers working on it, you should consider to use Redux. It enforces by nature to use explicit actions to change the state. The action has a type and payload, which a reducer can use to change the state. In a team of developers it is very easy to reason about state changes that way.

```javascript
// reducer
(state, action) => newState
```

MobX is less opinionated, but by using useStrict you can enforce clearer constraints like in Redux. That's why I wouldn't say one cannot use MobX in scaling applications. There is a clear way of doing things in Redux, but not in MobX. The documentation in MobX even says: "[MobX] does not tell you how to structure your code, where to store state or how to process events." The development team would have to establish a state management architecture in the first place.

State management can be learned. When we recap the recommendations, a newcomer in React would first learn to use setState properly. After a while the newcomer would realize the problems of using only setState to maintain state in a React application. When looking for a solution, the newcomer stumbles upon state management libraries like MobX or Redux. But which one to choose? Since MobX is less opinionated, has less boilerplate and can be used similar to setState I would recommend in smaller projects to give MobX a shot. Once the application grows in size and contributors, one should consider to enforce more restrictions in MobX or give Redux a shot. I enjoyed to use both libraries. Even if you don't use one of them after all, it makes sense to have seen an alternative way of doing state management. Both libraries deal in a different way with state management.

## How to refactor?

Consider to compare both real world [MobX](https://github.com/rwieruch/favesound-mobx) and [Redux](https://github.com/rwieruch/favesound-redux) applications. I made one big [Pull Request](https://github.com/rwieruch/favesound-mobx/pull/1) to show all changes at one place. In the case of the PR, it is a refactoring from Redux to MobX. But you could apply it vice versa.

Basically one has to exchange Redux Actions, Action Creator, Action Types, Reducer, Global Store with MobX Stores. It is very much decoupled from the components.

Additionally the interface to connect React components changes from [react-redux](https://github.com/reactjs/react-redux) to [mobx-react](https://github.com/mobxjs/mobx-react). The [presenter + container pattern](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) can still be applied. You would have to refactor only the container components. In MobX one could use inject to get a store dependency. After that the store can pass a substate and actions to a component. MobX observer makes sure that the component updates (render) after an observable property in the store changed.

```javascript
const UserProfileContainer = inject(
  'userStore'
)(observer(({
  id,
  userStore,
}) => {
  return (
    <UserProfile
      user={userStore.getUser(id)}
      onUpdateUser={userStore.updateUser}
    />
  );
}));
```

In Redux one would use mapStateToProps and mapDispatchToProps to pass a substate and actions to a component.

```javascript
function mapStateToProps(state, props) {
  const { id } = props;
  const user = state.users[id];

  return {
    user,
  };
}

function mapDispatchToProps(dispatch) {
  return {
    onUpdateUser: bindActionCreators(actions.updateUser, dispatch),
  };
}

const UserProfileContainer = connect(mapStateToProps, mapDispatchToProps)(UserProfile);
```

I wrote a little tutorial on [how to refactor from Redux to MobX](http://www.robinwieruch.de/mobx-react/). But one could also apply the refactoring vice versa.

## Boilerplate vs Magic

Whenever I read the comments in a Redux vs MobX discussion, there is always this one comment: "Redux has too much boilerplate, you should use MobX instead. I was able to remove XXX lines of code." The comment might be true, but no one considers the trade off. Redux comes with more boilerplate as MobX, hence it has advantages over MobX.

Redux library is pretty small. Most of the time you are dealing only with plain JavaScript objects and arrays. It is closer to vanilla JavaScript than MobX. In MobX one wraps the objects and arrays into observable objects which hide most of the boilerplate. There the magic happens, but it is harder to understand the underlying mechanisms. In Redux it is easier to reason about it with plain JavaScript.

Additionally one has again to consider where we came from in single page applications. All frameworks/libraries had the same problems of state management, which eventually got solved by the overarching Flux pattern. Redux is the successor of the pattern. In MobX it goes the opposite direction again. Again we start to mutate objects without embracing the advantages of functional programming. Some people already argue, that it feels again closer to two-way data binding. Maybe people run into the same problems again before they used a state management library. MobX is not opinionated, but it could embrace best practices of how to organize a good to reason about state management architecture. Otherwise people tend to mutate state directly in components.

Both libraries are great. While Redux is already well established, MobX becomes an valid alternative for state management.

## Key Takeaways

* learn React and setState first
* learning curve: setState -> MobX -> MobX more restricted (e.g. useStrict) -> Redux
* use MobX in a smaller size & few developers project
* use Redux in a bigger size & several developers / teams project
* MobX: simple to use (magic), easy to start, less opinionated
* Redux: clear constraints in a scaling application
* container + presenter components is a valid pattern for both
* react-redux & mobx-react are exchangeable interfaces to React container components
* useStrict of MobX makes state changes more obvious in a scaling application

I would like to hear your feedback. Maybe I missed or could improve something, please let me know.
