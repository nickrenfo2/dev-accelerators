#Lesson 2 - Thinking in React

This lesson is largely based on the content from the ["Thinking in React"](https://reactjs.org/docs/thinking-in-react.html) 
page on the React docs website. We will learn how to break down an application, or a component, into appropriately 
sized and purposed components and sub-components.


##5 Steps to Success

Whether building a complex application or a small component, following these 5 steps may help you build more clearly 
and efficiently.

###1. Start with a mock

For the most part, we tend to receive mocks from the Design team, which means we really don't have to build them 
ourselves. In any case, take a good look at the mock, and identify how you might break it up into more manageable 
pieces, and identify where your state/props will live. Which leads us to our next step:


For our demonstration, we'll be building a Filterable Inventory List:

![Filterable Inventory List](https://reactjs.org/static/1071fbcc9eed01fddc115b41e193ec11/d4770/thinking-in-react-mock.png)

Our inventory API returns the following data:

```json
[
  {"category": "Sporting Goods", "price": "$49.99", "stocked": true, "name": "Football"},
  {"category": "Sporting Goods", "price": "$9.99", "stocked": true, "name": "Baseball"},
  {"category": "Sporting Goods", "price": "$29.99", "stocked": false, "name": "Basketball"},
  {"category": "Electronics", "price": "$99.99", "stocked": true, "name": "iPod Touch"},
  {"category": "Electronics", "price": "$399.99", "stocked": false, "name": "iPhone 5"},
  {"category": "Electronics", "price": "$199.99", "stocked": true, "name": "Nexus 7"}
];
```

###2. Break the UI into a component Hierarchy

Start by drawing boxes around each component and subcomponent, literally. How you do you determine what should be 
its own component? The same way you write a function - use the Single Responsibility Principle, and your best judgement.
Just like a function, a component should do only one thing.

![Component Hierarchy Diagram](https://reactjs.org/static/eb8bda25806a89ebdc838813bdfa3601/6b2ea/thinking-in-react-components.png "Component Hierarchy Diagram")


You’ll see here that we have five components in our app. We’ve italicized the data each component represents.

1. FilterableProductTable (orange): contains the entirety of the example - this is the parent component, the "container"
2. FilterInput (blue): receives all user input
3. ProductTable (green): displays and filters the data collection based on user input
4. ProductCategoryRow (turquoise): displays a heading for each category
5. ProductRow (red): displays a row for each product

