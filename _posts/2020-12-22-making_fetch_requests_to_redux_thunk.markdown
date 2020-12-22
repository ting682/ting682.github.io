---
layout: post
title:      "Making fetch requests to Redux thunk"
date:       2020-12-22 16:33:10 +0000
permalink:  making_fetch_requests_to_redux_thunk
---


React is such a powerful Javascript library! In my time using it, the advantages of using it over plain vanilla Javascript are amazing. Being able to actively render, store and manipulate state, and pass down props is so critical in making an interactive experience on the front-end. With that said, there are so many tools that enhance the web development experience but if you want to design a website that gives you centralized state for a large web application, you need Redux. Dan Abramov goes into detail in this youtube video below.

[Redux justification](https://www.youtube.com/watch?v=xsSnOQynTHs&feature=emb_logo)

I love being able to time-travel in your state with the Redux web development toolkit. It really helps the developer fully understand where anything broke down, what are the actions passed, what was the payload. But with an application that uses Redux, we really need to use the middleware thunk to specifically help with AJAX fetch requests. Typically, when we are performing our fetch requests, our result needs to be fed into our Redux store. Let's dive into some code, shall we?

```
export function fetchLogin(data) {
    fetch('http://localhost:3000/api/v1/login', {

            credentials: "include",
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify({
                user: {
                    email_address: data.email,
                    password: data.password

                }
            })
        })
       return {
            type: 'GET_CURRENT_USER', payload: {
                user: {
                    userId,
                    bio,
                    name
                }
            }

        }
}
```
In this code above, I am performing a fetch to login a user via email and password. What I want afterwards is the user info. Now the problem with doing this is that fetch has Promises. If your promise is returned later than when you call "return", you will have no returned data. The function has no way to dispatch to Redux the desired action to update the Redux store. Let's This is where Redux-Thunk steps in. Here is what the top level index.js would look like. This is very basic. We are only calling on "applyMiddleware" on thunk. Make sure to perform 'npm install redux-thunk'.

```
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers';

const store = createStore(rootReducer, applyMiddleware(thunk));

ReactDOM.render(
  <Provider store={store} >
    <App />
  </Provider>, document.getElementById('root')
)
```

What we need is dispatch to be passed in so that we can use it. In my LoginContainer component, I need to pass in dispatch to props. Here is what the whole component looks like:

```
import React, { Component } from 'react'
import { Form, Row, Col, Button } from 'react-bootstrap'
import { connect } from 'react-redux'
import { fetchLogin } from '../actions/fetchLogin'


class LoginContainer extends Component {
    constructor (props) {
        super(props)
        this.state = {
            email: "",
            password: ""
        }

        this.handleChange = this.handleChange.bind(this)
        this.handleSubmit = this.handleSubmit.bind(this)
    }

    handleChange = (event) => {
        this.setState(previousState => {
            return {
                ...previousState,
                [event.target.name]: event.target.value
            }
        })

    }

    handleSubmit = (event) => {
        event.preventDefault()

        this.props.fetchLogin({
            email: this.state.email,
            password: this.state.password
        },this.props.history)

    }

    render () {
        return (
            <React.Fragment>
                <Form onSubmit={this.handleSubmit} >
                    <Form.Group as={Row} controlId="formHorizontalEmail">
                        <Form.Label column sm={2}>
                        Email
                        </Form.Label>
                        <Col sm={10}>
                        <Form.Control as="input" name="email" type="email" placeholder="Email" onChange={this.handleChange} />
                        </Col>
                    </Form.Group>

                    <Form.Group as={Row} controlId="formHorizontalPassword">
                        <Form.Label column sm={2}>
                        Password
                        </Form.Label>
                        <Col sm={10}>
                        <Form.Control as="input" name="password" type="password" placeholder="Password" onChange={this.handleChange} />
                        </Col>
                    </Form.Group>
                        <Form.Group as={Row}>
                            <Col sm={{ span: 10, offset: 2 }}>
                            <Button type="submit">Sign in</Button>
                            </Col>
                    </Form.Group>
                </Form>
            </React.Fragment>
        )
    }
}

function mapDispatchToProps(dispatch){
    return { fetchLogin: (data, history) => dispatch(fetchLogin(data, history)) }
  }

  function mapStateToProps(state){
    return {
        loggedIn: !!state.currentUser,
        router: state.router,
        entries: state.entries.entries,
        currentUser: state.currentUser
    }
  }

export default connect(mapStateToProps, mapDispatchToProps)(LoginContainer)
```

The main takeaway is my mapDispatchToProps function that feeds in dispatch to fetchLogin. Then fetchLogin will invoke dispatch to perform Redux actions.

Here is what our fetchLogin looks like now that we've implemented Redux-Thunk:

```
export function fetchLogin(data, history) {
    return (dispatch) => {
        dispatch({type: 'LOGIN_REQUEST_STARTED'})

        return fetch('http://localhost:3000/api/v1/login', {

            credentials: "include",
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify({
                user: {
                    email_address: data.email,
                    password: data.password

                }
            })
        })
        .then(resp => resp.json())
        .then(userData => {

            dispatch({type: 'SET_CURRENT_USER', userData})
            history.push("/entries")

        })
    }
}
```

What we're doing here is part of the Redux-Thunk middleware magic. We are able to return a function that passes in 'dispatch' because we need to use it later to make updates to our Redux store. After getting the userData, we can invoke 'dispatch' to set the current user. After that, I redirect the user to my '/entries' route. A full justification to use Redux-Thunk can be found here:

[Redux Thunk Github](https://github.com/reduxjs/redux-thunk)

