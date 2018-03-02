---
layout: post
title:      "The One with the Final Project"
date:       2018-03-02 09:01:13 -0500
permalink:  the_one_with_the_final_project
---

## [Final]ly... JobSearch Task Tracker

Its hard to believe that I made it to this final project in just about the time I intended to! React/Redux was definitely challenging to wrap my head around at the beginning, but with the help of some outside resources (shout out to Steven Grider and [this Udemy course](https://www.udemy.com/react-redux/learn/v4/overview), I feel fairly confident in my skills and I feel like I would definitely like to pursue a job where I would be utilizing React! Ahead of my assesessment, I thought it would be a good idea to go into a detailed explanation of what happens when a user clicks something or does something that has an action tied to it. 

My application is something that I plan to use in my next step; the job search! As a part of the Flatiron program, during the job search, students are required to keep track of their job related activities, so I thought it would be useful to build a task tracker. One of the main features of my app is a form to fill out a completed or schedule activity. The action that I would like to discuss for purposes of this blog post is inside of the lifecycle method componentDidMount( ). 
```
componentDidMount() {
    this.props.fetchTypes();
  }
```
Lifecycle methods are used to run code at very specific times related to the mounting (creation and insertion) process of an instance of a component. componentDidMount( ) is called right after the component mounts. I chose this lifecycle method to call the action fetchTypes( ) because I needed my form to have a list of activity types in a select dropdown bar. componentDidMount( ) is the perfect place to call this function because as soon as the form is available to the user, that complete list of tasks will be available as well. If I waited to call fetchTypes( ) in the render function, the data would not be available in the dropdown when the user started filling out the form.

```
const mapDispatchToProps = (dispatch) => {
  return bindActionCreators({ fetchTypes, createTask }, dispatch);
}
```

Another important piece of the action flow is what happens in mapDispatchToProps. Later on, we are going to want fetchTypes() to dispatch an action because dispatching an action is the only way to change our App's state. In order for this to happen, we need to utilize bindActionCreators( ), which essentially binds 'dispatch' to fetchTypes so that dispatch can be used later on in the action. I like to think about this like dispatch is a 'prop' that fetchTypes is going to pass on to some other function later, so we need to 'pass the dispatch prop' to fetchTypes to be passed down again.

SOO..fetchTypes is being imported from '../actions.js' at the top of my form component file, so when it is called, we are going to head over to the actions file to see what happens next.

```
export function fetchTypes() {
  return (dispatch) => {
    dispatch({type: FETCH_TYPES});
    fetch(`${ROOT_URL}/types`)
      .then(response => response.json())
      .then(json => {
        dispatch({ type: RECEIVED_TYPES, payload: json })
      })
  };
}
```

When fetchTypes() is called right after the instance of the form component is mounted, fetchTypes dispatches the first action `dispatch({type: FETCH_TYPES});` As I mentioned before, dispatching an action is the only way to manipulate the app's state. App's state is manipulated through the use of reducers. So, when an action is dispatched, it is dispatched to the reducer, which then decides, based on the action type, how to manipulate the state. The first action that is dispatched in the async function fetchTypes() is FETCH_TYPES. If we look at my reducerTypes file....

```
import { FETCH_TYPES, RECEIVED_TYPES } from '../actions';

export default function(state = [], action) {
  switch (action.type) {
    case FETCH_TYPES:
      return [...state]
    case RECEIVED_TYPES:
      return [...state, action.payload]
    default:
      return state;
  }
}
```
FETCH_TYPES doesnt do anything to manipulate the state. This action is there to say to Redux, 'Hold on. I am fetcing some data. Return the state as is for now.' This would be a good place to have a spinning wheel display so that the user knows that the data is being fetched.

Okay, so back to the action.

The next thing that happens is `fetch(`${ROOT_URL}/types`)` . Fetch will go out and grab the data from our API at the route specified. In this case, we want fetch the data from the types index page because this will give us all of the types. Once we fetch the data, fetch then returns a response object made up of meta data. In order to get the body of this  response object (where the types are), we translate the body into json code as shown here: 
`.then(response => response.json())`

Then, we dispatch another action ( `dispatch({ type: RECEIVED_TYPES, payload: json })`) with the newly translated json code. We head back into the reducer and find RECEIVED_TYPES. This is where we are manipulating our applications state! We are returning a new state object with the payload of the action (json code for all types) merged into an array with the current state. When this step occurs, the global application state (specifically the types piece of state in combineReducers function)  is modified to contain the newly fetched data in the specified format. 

```
const RootReducer = combineReducers({
  types: TypesReducer,
  tasks: TasksReducer
});

```

When the global application state changes, it then updates and injects the modified state into any container component that is utilizing that specific piece of the state, in our example, state.types. Back in our form, we have mapStateToProps that utilizes the piece of global state, types:
```
const mapStateToProps = (state) => {
  return {
    types: state.types
  }
}
```
Now, the form will have access to the newly modified types piece of state as a prop of this instance of a form component. 
Finally, we utilize this.props.types in our render function to create select options for the select dropdown bar, with each type being a different selection.

```
    const { types } = this.props;
    let typesForSelect;
    if (types.length !== 0){
      typesForSelect = types[0].map((type, index) => {
        return <option key={index} value={type.id}>{type.name}</option>
      });
    }
		<select
				required
				type="select"
				name="type_id"
				className="form-select"
				onChange={this.handleOnChange}>
				<option defaultValue>Select Type...</option>
				{typesForSelect}
   </select>
	 ```
	 

Thanks for reading! YAY Redux!
								
		

