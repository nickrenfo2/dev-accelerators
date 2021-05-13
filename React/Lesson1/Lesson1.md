# Lesson 1 - Getting Started in React

## What

### What is React?

React.js is a library for building user interfaces. It focuses on componentization, reusability, and readability.

Its use of JSX allows you to keep the logic for components inside the components they belong to, so it's all in one
place. No more switching between `template.hbs`, `model.js`, and `client.js`. Even styling can live within the
component.

The introduction of Hooks in React allows the reuse of stateful logic across components, as well as theming, context,
and other reusable logic.

- In particular, this could be useful for our Recirculation components


## Why

Why use React? Why not stick with Vue.js or Handlebars? Why not use Angular instead?

There are several key reasons why React is a good choice:

### Support

React is currently maintained by Facebook, inc. and used in their production environments. Facebook is highly interested
in creating performant, reusable, and easy-to-read and easy-to-write library for development. There are constant updates
improving security, performance, and design patterns. Due to its widespread adoption, even if Facebook decided to stop
supporting it, there are many developers out there with an interest in its operation. [As big as jQuery was, React is
almost, and soon to be, bigger](https://insights.stackoverflow.com/survey/2020#technology-web-frameworks). It is 
difficult to overstate the widespread adoption of React.js

Beyond just codebase maintenance, the there is a very high availability of user support and knowledgebase within the
official React docs, but also third-party sites like StackOverflow. For virtually every problem or question you might
have, someone else has had that, too. And if not, then the odds are that someone out there knows how to solve your
problem or question.

### JSX

[JSX](https://reactjs.org/docs/introducing-jsx.html) (JavaScript-in-XML) is one of the main reasons to choose React.
GUIs are inherently coupled with logic and styling, so why should we store these things separately? JSX takes all the
best parts of JavaScript and XML and unites them.

Handlebars attempted to do this, but it's simplicity was ultimately its doom. Handlebars allows you to perform simple
operations like displaying a variable's contents, or format a date, but what about reaching out to the server to get a
list of articles to display? This complex logic must be separated from the component itself into a separate file we
call `model.js`.

And what about styling? Why should we be stuck creating a complicated classname for a component in one place, then
defining the styles for that class in a completely unrelated part of the codebase? What happens if I change a classname?
What happens if I accidentally reuse a classname? Why should my CSS selectors be so specific that it's virtually
impossible to override them, just so that I don't accidentally select something outside the scope of what I'm working
on?

With JSX, these are all united into a single place.

Component logic is built into the component itself, rather than defined somewhere else and applied to it. Components are
truly designed to do one thing. Rather than having a component that must collect data, format it, display it, and change
the way it displays based on that data (e.g. 'airing now'), and handling user input, components can be separated into
those that "do" and those that "display". A parent component can focus solely on collecting and formatting data, and
pass down relevant data to child components who are focused solely on displaying it, or children components that handle
collecting user input.

Generic themes and palette information like `primaryColor.main` or
`secondaryColor.dark` can be defined at the global level, and then an individual component can define its own styles in
the place where it lives. No more jumping around the codebase in order to define what a component is, does, and looks
like.

### One-Way Data flow

In React, data flows down and events bubble up. This forces a stricter standard of data management, and means you always
know what data comes from where. It also means that you control the flow of events.

### Hooks

A recent addition to React is Hooks. Until the introduction of hooks, react use Classes to define state, and confusing
method names and rebinding of the lexical `this` had to be constantly and carefully considered in order for a component
to have a changing state. Now, all components can be defined as a function returning a JSX element, and stateful logic
can be applied to it, and reused across components.

Hooks also introduced Context, which allows components to hook into data and methods that are used throughout the
codebase, rather than having to redefine them each time they are used. While this breaks the "one-way data flow" to some
degree, when used sparingly, context can clean up and simplify your codebase significantly.

## How

So how do we use React?

####Composing a component

Components are functions that return a JSX element. Consider the following Paragraph component:

```jsx
// Component names are always CamelCase
function Paragraph() {
	return (
		<p>Your Text Here</p>
	)
}

```

The entire file for your component may look like this:

`Paragraph.jsx`
```jsx
import React from 'react';

function Paragraph(){
	return (
		<p>Your Text Here</p>
    )
}

module.exports = Paragraph;
```

When we want to display this data, we'll add it to another component as an XML tag, like so:


`App.jsx`
```jsx
import React from 'react';
import Paragraph from './paragraph';

function App() {
	return (
		<div>
			Here's My App!
			<Paragraph/>
		</div>
	)
}
```

As you can see, components can be nested within each other. All React components are valid children to other React 
components.


####Adding dynamic data (props)

A component that only renders one thing isn't all that useful. Lets add in some data that we can pass to it.

Data is passed to components via the `props` object - the first argument to a React component. Let's see how we can 
pass some data



`App.jsx`
```jsx
import React from 'react';
import Paragraph from './paragraph';

function App() {
	return (
		<div>
			Here's My App!
			<Paragraph text="Text from App.jsx"/>
		</div>
	)
}
```

Now the `props` object for the `Paragraph` component will contain a `text` property, with a value of "Text from App.jsx"

Let's see:

`Paragraph.jsx`
```jsx
import React from 'react';

function Paragraph(props) {
	const {text} = props;
	return (
		<p>
			{text}
		</p>
	)
}

module.exports = Paragraph;
```

Similar to handlebars, we can interpolate variables using curly braces (`{}`), though we only use one set, rather 
than two. This paragraph will now display the text "Text from App.jsx" rather than "Your Text Here".


####Adding state

Even still, we can pass any arbitrary content to our paragraph component, but we need to accept input from the user. 
Let's discover how to useState.

First, we'll need to add an input field:

`App.jsx`
```jsx
import React from 'react';
import Paragraph from './paragraph';

function App() {
	return (
		<div>
			<input type="text"/>
			Here's My App!
			<Paragraph text="Text from App.jsx"/>
		</div>
	)
}
```

And now, let's collect the value from input (as it changes), and store it in the state. To create and use a piece of 
state, we use the useState() hook, provided by react.


`App.jsx`
```jsx
import React, { useState } from 'react';
import Paragraph from './paragraph';

function App() {
	// The first parameter returned by useState is the current state.
	// The second paramter returned by useState is a setter function, to update the state
	// The parameter we pass to useState is the initial state
	const [text, setText] = useState('');
	const handleChange = (event) => {
		const newText = event.target.value;// The current value of the input field, each time it changes
		setText(setText(newText));
	};

	return (
		<div>
			<input type="text"/>
			Here's My App!
			{/*We can pass the value to the paragraph by using curly braces */}
			<Paragraph text={text}/>
		</div>
	)
}
```


### Product Demo - TODO App

Todo apps are a great "hello world" style project to get introduced to React. They demonstrate what components are, how
they nest and interact with each other, props, state, and one-way data binding. They're simple enough to be a basic
example, but can be improved and extended to become incredibly complex applications (like JIRA).

Check out [Ron's Todo App](https://github.com/ronaldporch/todo-app)


##Get started on your own


### Installation

The easiest way to create a new React application is to use the `create-react-app` package. 

You can create your first
project by running `npx create-react-app my-app` (where `my-app` is the name of the project you are creating). 

The`npx` command will download the `create-react-app` package (similar to `npm install --global create-react-app`) and then
immediately execute it, passing along any arguments you gave it (in this case, `my-app`).

`create-react-app` will create the directory for your project (in this case, `./my-app/`), initialize a new `npm`
project, install several packages, initialize a directory structure and a few files for a barebones homepage, and set up
a package bundler (Webpack), a transpiler (Babel), and set up all the configurations for you.

It should be enough to get you started without any effort, and give you enough of an example to continue to work off of.


###"Homework"

Create a Todo app that has the following functionality

* Ability to add many items
* Ability to remove items
* Ability to mark items complete or incomplete

Bonus points for:

* Remembering todo list between refreshes
* Remembering which items were completed between refreshes
* Ability to change a todo item that's been created already


##Related Links

* [JSX](https://reactjs.org/docs/introducing-jsx.html)
* [Thinking in React](https://reactjs.org/docs/thinking-in-react.html)
* [Material UI](https://material-ui.com/getting-started/installation/)
* [Ron's Todo App](https://github.com/ronaldporch/todo-app)
* [Web Framework Stats](https://insights.stackoverflow.com/survey/2020#technology-web-frameworks)
