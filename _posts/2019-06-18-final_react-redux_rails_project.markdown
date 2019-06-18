---
layout: post
title:      "Final React-Redux + Rails Project "
date:       2019-06-18 14:06:48 +0000
permalink:  final_react-redux_rails_project
---

### Learning About Redux Flow

Today I completed my final project for Flatiron called Tennis Journal, an app that allows the user to keep track of matches and tournaments. This app uses a Rails backend API to persist data to, and a React frontend for the user interface. While working on this project, I learned a ton about the process of using Redux to manage the state of a React app. In this post I will go over the process of how I add a newly created match object to the Redux store

The basic flow of Redux goes:  Action -> Reducer -> New State. So in order to add a new match, we need a form that collects data about the new match through inputs, an action that accepts that data as an argument, and a reducer that will receive that action and update the store with the new data. First let's start with the form:

```
import React, { Component } from 'react'

class MatchForm extends Component {
  constructor(){
    super()

    this.state = {
      date: "",
      time: "",
      round: "",
      result: "",
      score: "",
      notes: ""
    }
  }

  handleOnChange = event => {
    const {name, value} = event.target
    this.setState({
      [name]: value
    })
  }

  handleOnSubmit = event => {
    event.preventDefault()

    const match = this.state
    this.props.addMatchToDatabase(match)
    this.setState({
      date: "",
      time: "",
      round: "",
      result: "",
      score: "",
      notes: ""
    })
  }

  render() {
    return (
      <div>
        <form onSubmit={this.handleOnSubmit}>
          <h2>Add Match</h2>
          <p>
            <label htmlFor="match-date">Date: </label>
            <input type="date" name="date" value={this.state.date} onChange={this.handleOnChange} />
          </p>
          <p>
            <label htmlFor="match-time">Time: </label>
            <input type="time" name="time" value={this.state.time} onChange={this.handleOnChange} />
          </p>
          <p>
					  <label htmlFor="match-round">Round: </label>
            <input type="text" name="round" value={this.state.round} onChange={this.handleOnChange} 
          </p>
          <p>
					  <label htmlFor="match-result">Result: </label>
            <input type="text" name="result" value={this.state.result} onChange={this.handleOnChange} 
          </p>
          <p>
					  <label htmlFor="match-score">Score: </label>
            <input type="text" name="score" value={this.state.score} onChange={this.handleOnChange} 
          </p>
          <p>
					  <label htmlFor="match-notes">Notes: </label>
            <textarea name="notes" value={this.state.notes} onChange={this.handleOnChange} 
          </p>
          <button>Add Match</button>
        </form>
      </div>
    )
  }
}

export default MatchForm

```

I set up the form for a new match as a React class component. This allowed me to give the component its own state to keep track of the inputs from the user. Each input field has an `onChange` property, which executes the `handleOnChange` function. So every time a user types something into an input, that input gets added to the component's state. When the user submits the completed form, the `handleOnSubmit` function gets called. This sets the component's state, which now contains all of the match data, equal to a variable called `match` and passes it in to the `addMatchToDatabase` method. `addMatchToDatabase` is an action creator which will use the match data to create an action to be sent to the reducer. This function was imported into the parent component of `MatchForm` and passed down as a prop

Next let's look at the Action Creators:

```
//  'actions/matches.js'

export const addMatchToStore = match => {
  return {
    type: 'ADD_MATCH',
    match
  }
}

export const addMatchToDatabase = match => {
  const request = {
    method: 'POST',
    body: JSON.stringify({
      match: match
    }),
    headers: {
      'Content-Type': 'application/json'
    }
  }
	
	
  return dispatch => {
    return fetch('/matches', request)
    .then(response => response.json())
    .then(match => dispatch(addMatchToStore(match)))
  }
}
```

Here you can see the `addMatchToDatabase` that was called in the `handleOnSubmit` for the match form. This function accepts the match data and sends a `'POST' fetch` request to the database. The databse then uses that information to create a new match, and returns a JSON object containing the new match data back to the frontend. That JSON object is then passed into `addMatchToStore` and dispatched to the reducer. 

`addMatchToStore` is an action creator that returns an action object with `type: 'ADD_MATCH'` and the match data inside it. This action actions is dispatched to the reducer using the `dispatch()` function, which is available here because of `redux-thunk`

Now that the action has been dispatched, we need a reducer to handle it:

```
// 'reducers/matches.js'

export default (state = [], action) => {
  switch(action.type) {
    case 'SET_MATCHES':
      return action.matches
    case 'ADD_MATCH':
      return [...state, action.match]
    case 'DELETE_MATCH':
      return state.filter(match => match.id !== action.matchId)
    default:
      return state
  }
}
```

The reducer sets an initial state equal to an empty array. This state is the matches portion of the Redux store and can be accessed in other connected components as state.matches. The reducers also contains a switch statement which evaluates `action.type`, which is the type property of the action that was dispatched. If `action.type` is 'ADD_MATCH', it will return `[...state, action.match]`, which is evertything in the current state plus the new match. Because the reducer is connected to the store in the inital Redux setup, whatever it returns will be added to the store.

And thats it! The newly created match is now available in the Redux store and can be accessed by any component in the application 
