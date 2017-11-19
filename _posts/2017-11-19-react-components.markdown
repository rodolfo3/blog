# Component classifications

[TOC]

## by state usage
> Stateful
Keep state internally and changes it

Example:
```
class Button extends Component {
	state = {
		count: 0
	}
	inc = () => {
		this.setState((s) => { count: s.count + 1 })
	}
	
	render() {
		<button onClick={this.inc}>
			{ this.state.count }
		</button>
	}
}
```
> Stateless
Only based on props (values and callbacks)

Usually pure functons

Example:
```
const Button = ({children, onClick}) => (
	<button onClick={onClick}>
		{children}
	</button>
);
```
Stateless components are usually to reuse and test.
But stateful components are powerful.

## By goal
> Presentional
Generate the VirtualDom to show the content by the received props.

Are free of dependencies from the rest of the app (only depends on props and it's own state)

> Container
Responsible to reach data that lives outside react scope (like redux store)

	"A container does data fetching and then renders it's corresponding sub-component. That's it."
	Jason Bonta
Usually have a `render` method just calling a presentional component.

## How do we make sure components are reusable?
- by ensuring out UI components are pure presentional components (dumb)
What does that means?
 -- no data fetch into component
 -- if we have `renderBla()` functions in the component, it's better move it to separate components
 
- Single Responsibility Principle
-- Components/Containers should deal with only one chunk of the UI feature/functionality

> "if in doubt, create another component"

# Some useful patterns

## HOC (high-order-components)
A function that takes a component as argument and returns a new component.
Allows apply logic to components without coding it as a dependency
Real-world examples:

 - connect (from redux)
 - withRouter (from ReactRouter)

The code looks like:
```
export const withActions = acts => OriginalComponent => {
	class WithActioncComponent extends Component {
		render() {
			return (
				<OriginalComponent
					extra={someData}
					{...this.props}
				/>
			);
		}
	}
	return WithActioncComponent;
```
Them the usage is like:
```
const NewComponent = withActions(acts)(OriginalComponent);
```

## render callbacks (function as a child components)
"Children is literally just a JavaScript function."
Separate state manipulation (state and callbacks) from the "render".

The parent component is like
```
const Component = ({ children }) => {
	return (
		<strong>
			{ children({ count, callback }) }
		</strong>
	);
};

Component.propTypes = {
  children: React.PropTypes.func.isRequired,
};
```
And, the usage, is like:
```
<Component>
	{ (count, callback) => 
		<button onClick={callback}>
			{ count + 1 }
		</button>
	}
</Component>
```

## render callbacks x HOC

Function as Child Component is a better alternative for abstracting your UI concerns with the exception that your child component is truly coupled to the Higher Order Component it is composed with.

https://medium.com/merrickchristensen/function-as-child-components-5f3920a9ace9

## Children pass-through
Do not include extra markup to component, just render the children:
```
class MyComponent extends Component {
	... 
	render() {
		return React.Children.only(this.props.children);
	}
}
```
Useful to add context (`getChildContext`) and/or for HOC.

## Style component
Create a specialized component just to style it (using an special property or even `className`):
```
const Btn = ({ children, primary = false }) => (
	<button
		className={primary ? 'primary' : 'secondary'}>
		{ children }
	</button>
); 
```
And, then, use a style component, instead of an extra property:
```
const PrimaryBtn = props => <Btn {...props} primary />
```

## Layout component
A component that never updates - it exists only to layout things (like a fixed sidebar or a footer).
It is a standard component where `shouldComponentUpdate` always returns `false`.

## Components for formatting text
Instead of use helpers to format text (like from `30` to `R$ 30,00`), use a component:
```
<Money>30</Money>
```
With the code:
```
const Money = ({ children }) => (
	<span>R$ {children.toLocaleString('en')}</span>
);
```

## Context Wrapper
Context allows to send data/objects to inner components without send it as props.
But is a good practice use HOC to inject it.
ref.: `getChildContext` & `childContextTypes`

Use-case: our player (send play, pause, etc methods)
Others use-case: redux `dispatch` and react-router `route`

Into the parent component:
```
getChildContext() {
	return {
		addToShelf: this.addToShelf,
		getShelf: this.getShelf,
	}
}
```
Into the child component:
```
Book.contextTypes = {
	addToShelf: PropTypes.func,
	getShelf: PropTypes.func,
}
```


## Class Properties
Configuring `class-properties` into Babel allows initialize `state` outside the `constructor`:

- From
```
class MyNiceComponent extends Component {
  constructor() {
    this.state = {
      initialState: 'ok',
    };
  }
}
```

- To
```
class MyNiceComponent extends Component {
  state = {
     initialState: 'ok',
  }
}
```

This also avoids the usage of `bind` in methods to allow use the `this`
variable:

- From
```
class MyNiceComponent extends Component {
  constructor() {
    this.magic = this.magic.bind(this);
  }

  magic() {
    // I can use "this"!
  }
}
```

- To
```
class MyNiceComponent extends Component {
  magic = () => {
    // I can use "this"!
  }
}
```

## Naming convention (for event handlers)
Pattern:
```
handle{eventName}
```

Example:
```
handleClick = (e) => {
	// code
}
```

## Send render as a prop
If we separate container and presentional components, it will be hard to put inner container inside presentional components. (Like `CourseCard`, that includes the `FavoriteButton`).

To solve that, the `CourseCard` could receive a property with the component:
```
const CourseCard = ({ name, etc, favoriteButton }) => (
	<div>
	   ... code ...
	   <div className="favorite-container">
		   { favoriteButton }
	   </div>
	</div>
); 
```
Then, `favoriteButton` could be any react component.
Alternatively, it could be a function (like `children` when it is a function):
```
```
const CourseCard = ({ name, id, favoriteButton }) => (
	<div>
	   ... code ...
	   <div className="favorite-container">
		   { favoriteButton({ courseId: id }) }
	   </div>
	</div>
); 
``` 
Now, `favoriteButton` should be a render-like `function` instead of a component.

It could also be a component class (or function):
```
	   <div className="favorite-container">
		   <FavoriteButton courseId={id} />
	   </div>
```

# More props x Objects as props
React allows to pass objects as props to components. It also provides `propTypes` to document (and validate) component usage.
So, we could send props in 2 ways: full object or spread props

## full object
Looks like a component that receive the object as props:
```
<BookCover book={book} />
```
Then, it can validates using `shape` :
```
BookCover.propTypes = {
	book: PropTypes.shape({
	    title: PropTypes.string,
	    coverSrc: PropTypes.string
	})
}
```

## spread props
The component receives each attribute as separated prop:
```
<BookCover title={book.title} coverSrc={book.coverSrc} />
```
Then, the props validation will be like:
```
BookCover.propTypes = {
	title: PropTypes.string,
	coverSrc: PropTypes.string,
}
```
This can use the spread operator:
```
<BookCover {...book} />
```
## so what

> Sending full objects allows pass-through objects to inner components.

But

> Sending separated props are easier to read, test and describes the component with the code

Opinion: send objects to containers (extracted from `store`), but send props to presentional components.

## setState is async
Call `setState`  do not modify `this.state` synchronously: it enqueue the change to react apply at the end. Then:
```
this.state = { count: 0 }
``` 
```
this.setState({ count: this.state.count + 1 });
this.setState({ count: this.state.count + 1 });
this.setState({ count: this.state.count + 1 });
```
After this, into the `render` method, `this.state.count` is... `1`, not `3`.
## use setState callback
`setState` has a second parameter: a callback that runs after the state changes.
```
this.setState({ count: 2 }, () => {
	console.log('state updated!');
})
``` 
## use a function to change the state
More useful to solve that initial use case (change state 3 times):
```
this.setState((state) => ({
	count: state.count + 1
});
this.setState((state) => ({
	count: state.count + 1
});
this.setState((state) => ({
	count: state.count + 1
});
```
This will handle and the new `this.state.count` will be `3`, as expected.

# Performande
Don't fret about performance optimizations until you have problems.

## shouldComponentUpdate
If the render method is expensive, is possible to implement the `shouldComponentUpdate` method. It receive the new `props` and it only runs `render` if it returns `true`.

## PureComponent
Extends `PureComponent` will avoid component re-render if the `props` are equal. It just improves the performance if the component do not change.
It is implemented by react using `shouldComponentUpdate`
Use it into components that change could have a performance penalty: it will compare things twice: the `props` and also the new virtualDOM (returned by `render`).

