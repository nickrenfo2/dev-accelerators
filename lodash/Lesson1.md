# Lodash

Lodash is an excellent helper library. They provide all sorts of useful utility functions, so you don't have to write
them yourself. It really takes the hassle out of working with objects, arrays, and even numbers and strings. Take a 
look at the [lodash docs](https://lodash.com/docs/4.17.15) to see what sorts of tools they have.

Oftentimes, Lodash provides utility functions before they are included in Node.js (e.g. `.includes`, `.map`, etc).

It is worth looking over their list of methods once or twice a week to keep a fresh list in your mind of what
functions they actually have, and the sorts of functions they have generally. I often find myself writing some code and
thinking "I wonder if Lodash has a function for this," and more often than not, they do.

## General Stuff

The lodash docs displays a list of all their functions, sorted alphabetically, and grouped by the type of data they 
operate on. For the most part, the most interesting/useful functions belong to the "Array," "Object," and 
"Collection" categories. Arrays and Objects you know well, and the "Collection" methods operate on Arrays _or_ 
Objects, or any other Iterable Symbol. There are also some useful methods that operate on functions, and there are 
some interesting methods in the "Util" section. I've picked a few of my favorite methods to mention, as well as a 
few others.

As a general rule, Lodash accepts the data to operate on as the first parameter, and relevant parameters come second.
This makes the function calls look similar to calling a dot-method on the data (e.g. `myArr.map()` vs `_.map(myArr)`).
Lodash also generally defaults all of its parameters, allowing you to only pass the ones you care about. In 
particular, they make frequent use of the `_.identity` method.


Additionally, lodash generally avoids mutation. That is to say, it will return an updated copy of the data you pass 
it, rather than mutate that data directly. However, for certain methods, they do provide siblings that mutate 
rather than copy data. For example, the `_.without` method and the `_.pull` method are 'cousins'. While both functions 
perform the same fundamental task (remove elements with a given value from an array), the `_.pull` method will 
mutate the passed array, and the `_.without` method will return a copy. When viewing the documentation for methods 
like this, it will describe whether this method mutates the passed data, and provide a link to its cousin, so you 
don't have to search for it. If the documentation does not specify whether this method mutates the data or not, feel 
free to assume that it does not.

One more thing - unless explicitly stated otherwise, generally Lodash will use 
[SameValueZero](https://262.ecma-international.org/7.0/#sec-samevaluezero) for equality comparison. Basically, `===`.


### Util

These functions are general utility functions, offering simple logic like `_.identity` (return first param) or `_.
noop` (return `undefined`), or slightly more complex logic like `_.range` (get a range of numbers), or even `_.
matches`(create a matcher function to compare objects).


#### _.identity

This function accepts one parameter, and immediately returns it. While it's unlikely you'll find yourself explicitly 
using this function, it's a very useful default, used in functions like `_.map()`, or `_.find()`.

#### _.noop

Short for "no operation." A function that returns `undefined`. Useful if you need to stub a function and want it to do 
nothing.

#### _.matches

This method accepts an object (source), and returns a function. The return function accepts another object (target), 
returning `true` if the target object has equivalent property values as the source, otherwise `false`.
This is what's used under the hood in the `_.find` method, for example, allowing us to do something 
like the following:

```
//                               vvv passed to _.matches vvv
_.find(data.stationSyndication, {stationSlug:'my-station-slug'})
```
 

Now, onto the "Meat and Potatoes" 

### Array

These methods are used to compare, manipulate, search, and generally manage arrays.

Methods you may be interested in include the following:

#### _.chunk

Takes an array of elements, and splits it into chunks of a given size.

```js
_.chunk(['a', 'b', 'c', 'd'], 2);
// => [['a', 'b'], ['c', 'd']]
_.chunk(['a', 'b', 'c', 'd'], 3);
// => [['a', 'b', 'c'], ['d']]
```

#### _.concat

Just like the `Array.prototype.concat` method, except this does not mutate your data.

```js
const array = [1];
const other = _.concat(array, 2, [3], [[4]]);
console.log(other);
// => [1, 2, 3, [4]]
console.log(array);
// => [1]
```

#### _.flatten and _.flattenDeep

These methods will take nested arrays, and flatten them so they are not nested. the `_.flatten` method will flatten 
one level of nesting, and `_.flattenDeep` will recursively flatten, until the result is totally flat.

```js
const arr = [1, [2, [3, [4]], 5]]; 
_.flatten(arr)// => [1, 2, [3, [4]], 5]
_.flattenDeep(arr)// => [1, 2, 3, 4, 5]
```

#### _.take

Returns the first `n` elements of the given array.

```js
_.take([1, 2, 3], 2);
// => [1, 2]
```

We could use this for things like auto-backfilling content to
components.

e.g.

```js
const 
	maxItems = 10,
	curated = data.items || [],
	backfilled = getBackfillItems(),
	displayItems = _.take(_.concat(curated, backfilled), maxItems); // add curated + backfilled 
																																	// and then give us only the number of items we need  
```

#### _.union, _.unionBy, _.unionWith

`_.union` adds the given arrays together, ignoring duplicate values. Its cousin `_.unionBy` is the same thing, but 
additionally accepts a predicate used to determine whether a value is 'unique'. Its other cousin `_.unionWith` 
performs a very similar job, accepting a comparator function used to compare elements of arrays to determine if they 
are 'the same'. In other words, where the predicate of `_.unionBy` is passed a single array element and returns a 
value, the comparator of `_.unionWith` is passed two elements, and returns whether those two elements are identical.

These methods preserve order - they prefer the first element, and subsequent duplicates are ignored.

```js
_.union([2], [1, 2]);

// => [2, 1]
_.unionBy([2.1], [1.2, 2.3], Math.floor);
// => [2.1, 1.2]
// The `_.property` iteratee shorthand.
_.unionBy([{ 'x': 1 }], [{ 'x': 2 }, { 'x': 1 }], 'x');
// => [{ 'x': 1 }, { 'x': 2 }]

const objects = [{ 'x': 1, 'y': 2 }, { 'x': 2, 'y': 1 }];
const others = [{ 'x': 1, 'y': 1 }, { 'x': 1, 'y': 2 }];
_.unionWith(objects, others, _.isEqual);
// => [{ 'x': 1, 'y': 2 }, { 'x': 2, 'y': 1 }, { 'x': 1, 'y': 1 }]
```

So again with the example of auto-backfilling items, we would actually want to do something like this:

```js
const 
	maxItems = 10,
	curated = data.items || [],
	backfilled = getBackfillItems(),
	displayItems = _.take(
	  _.unionBy(curated, backfilled, 'slug'), // adds curated + backfilled without adding two items with the same slug
		maxItems
	);
```

#### _.uniq, _.uniqBy, _.uniqWith

These functions are essentially the same as above, except rather than concatenating multiple arrays together, these 
methods accept a single array and return a new array with only the unique elements. Original order is preserved.


### Collection

These methods will operate on Objects, Arrays, or any other iterable Symbol, like Strings.

#### _.filter

Just like the JavaScript `.filter` method, this will reduce an array down to the elements that the given predicate 
returns a truthy value for. However, this method is more useful than the native JS method, for three main reasons:
It defaults the predicate to the `_.identity` function, it is smart enough to use other lodash matchers if you pass 
it a value instead of a function, and most importantly, it can be used to filter objects, not just arrays. For example:

```js
const arr = [
  {
    val: 'foo'
  },
	undefined,
	{
    val: 'bar'
  }, 
	null,
	{
    val: 'baz'
  }
];

arr.filter() // => TypeError (no predicate function)
_.filter(arr)// => [{val: "foo"},{val: "bar"},{val: "baz"}]; (defaults to _.identity)

arr.filter((itm)=>itm.val==='foo')// => TypeError itm is undefined (can't handle data that doesn't fit the scheme...
// lets fix it up real quick...

const truthyArr = _.filter(arr)
truthyArr.filter((itm) => itm.val === 'foo'); // [{val: 'foo}]
_.filter(arr, { val: 'foo' })// handles 'bad' data gracefully. Also looks much cleaner.

// and the kicker:

const data = {
  a: 1,
  b: 2,
  c: 3,
	d: 4,
};
// lodash can iterate over objects
_.filter(data, (val, key) => val%2===0); // [2, 4]
// or even strings
_.filter('HelloWorld',(char)=> char === char.toUpperCase())// => ['H','W']
```


#### _.find

Just like `_.filter`, `_.find` is the lodash implementation of the JS `.find` method. For the same reasons as above, 
the lodash method is superior, so there is no need to reiterate here.

#### _.includes

Again, Lodash has implemented their own version of `.includes` - see above for why you may want to use the Lodash 
implementation over the native JS implementation. 

#### _.map

Again - Lodash has implemented a superior `.map` method. You can even do fun things with this function, like so:

```js
const syndicatedStations = _.map(data.stationSyndication, 'stationSlug') // => ['kool', 'kmox', '94wip', '1010wins',...]
```

##### _.map vs _.forEach

`_.map` and `_.forEach` are like cousins. The only difference is that `_.map` will return the results from running 
the given function, and `_.forEach` returns the passed data. When deciding between which to use, ask yourself the 
following question: Am I trying to perform side-effects (mutate something, fire db requests, etc.) or am I trying to 
collect some results based on input? If you want to perform side effects, use `_.forEach`. If you want to see the 
results of your function, use `_.map`. Consider this - if you can ignore the results of the function call, you're 
performing side effects and should use `_.forEach`. Otherwise, use `_.map`

e.g.
```js
const addToDb = (item) => {/*...*/};
const myData = [
  {
    val: 'foo'
  },
  {
    val: 'bar'
  },
  {
    val: 'baz'
  },
];

// DO
_.forEach(myData, addToDb);
// DO NOT (you're ignoring the results of _.map, use _.forEach instead)
_.map(myData, addToDb);

// DO
const vals = _.map(myData, 'val');
// DO NOT (you're trying to collect results, use _.map instead
const vals = [];
_.forEach(myData, (item) => vals.push(item.val));

// DO
const updatedData = _.map(myData, (item) => ({ ...item, newProp: 'newVal' }));
// DO NOT (rather than mutating the data in-place, it is better to return an updated copy of the data. Use _.map
_.forEach(myData, (item) => {item.newProp = 'newVal'});
```

#### _.reduce

Same as native JS `.reduce`, but with the same advantages as listed above


#### _.reject

The opposite of `_filter` - reduces a list down to only the elements for which predicate does **not** return truthy for.

#### _.shuffle

Another function you're unlikely to use, but which demonstrates the kinds of utility functions which Lodash provides.
This method will return a shuffled version of the passed array, using a version of the highly performant 
[Fisher-Yates shuffle](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle)

It is worth noting that while technically the Fisher-Yates shuffle will shuffle an array in-place, this method does 
not mutate the passed array, and will return a new array with the shuffled values.


### Object

The object methods provide utilities specific to managing/working with objects, that don't really apply to Arrays or 
other iterables.

#### _.defaults, _.defaultsDeep

Accepts a target object, and other 'source' objects. This method mutates the target object, and will assign 
properties and values to the target based on the source objects. It will only assign a property once, ignoring 
subsequent definitions of that property. It is particularly useful for defaulting `options` objects.

e.g.
```js
const myFunc = (data, opts = {}) => {
  const options = _.defaults(
    {}, // create a new, empty object to be the target (since this object will be mutated)
    opts, // assign whatever options were passed to the function
    { // default the rest of the options
      shouldDedupeContent: true,
	    logAmphoraRenderTimes: false,
    });
  //...
};

// now we can call myFunc and pass it some options, and the rest will be defaulted
myFunc(myData, { shouldDedupeContent: false });
```

the `_.defaultsDeep` method does the same thing, but will recursively assign properties, rather than just shallow.

```js
_.defaultsDeep({ 'a': { 'b': 2 } }, { 'a': { 'b': 1, 'c': 3 } });
// => { 'a': { 'b': 2, 'c': 3 } }
```

#### _.get

Many of you are probably familiar with this one, but this method allows you to specify a path of a given object to 
retrieve. The value will fail gracefully if the path doesn't exist, and you can even supply it with a default value 
if the given property is undefined.

e.g.
```js
const stationSlug = _.get(locals,'station.site_slug')
// the same as:
const stationSlug = _.get(locals, ['station', 'site_slug']);
// and can be defaulted
const stationSlug = _.get(locals, 'station.site_slug', DEFAULT_STATION.site_slug);
```

See also: `_.set`

#### _.has

Similar to _.get, but returns whether the given property exists on the object, rather than the value of it.



#### _.invert

Another weird one. For a given source object, returns a new object where the keys and values are swapped.

```js
var object = { 'a': 1, 'b': 2, 'c': 1 };
 
_.invert(object);
// => { '1': 'c', '2': 'b' }
```

#### _.mapKeys, _.mapValues

In addition to the `_.map` method that operates on Collections, there are object-specific mappers.

`_.mapValues` will pass each key/value pair to the mapper, and update the values of each property based on the results.
The inverse of that, `_.mapKeys`, will update the keys of each property based on the results.


```js
const users = {
  'fred':    { 'user': 'fred',    'age': 40 },
  'pebbles': { 'user': 'pebbles', 'age': 1 }
};
 
_.mapValues(users, function(o) { return o.age; });
// => { 'fred': 40, 'pebbles': 1 } (iteration order is not guaranteed)
// The `_.property` iteratee shorthand.
_.mapValues(users, 'age');
// => { 'fred': 40, 'pebbles': 1 } (iteration order is not guaranteed)

_.mapKeys({ 'a': 1, 'b': 2 }, function(value, key) {
  return key + value;
});
// => { 'a1': 1, 'b2': 2 }
```

#### _.omit, _.pick

These methods will return the passed object, either with only the given properties (`_.pick`) or with everything but 
the given properties (`_.omit`)

```js
var object = { 'a': 1, 'b': '2', 'c': 3 };
 
_.omit(object, ['a', 'c']);
// => { 'b': '2' }
_.pick(object, ['a', 'c']);
// => { a: 1, c: 3 }
```

Note: For performance reasons, you should prefer `_.pick`


#### _.result

Another example of the types of utility Lodash can provide.

This method is like `_.get` except that if the resolved value is a function it's invoked (passing `this` as a parameter), 
and its result is returned.


#### _.update

Just like `_.set`, except this method is passed an updater function rather than a value in order to determine the 
new value.


#### _.values

Returns an array consisting of the values of each property of the given object.


### Sequence

Before straying too far from the iterable methods, I'd like to talk about the `_.chain` method. It's very useful 
when you want to apply a series of lodash functions to some data, and don't want to nest a bunch of calls within 
each other. This `_.chain` method allows you to use dot-chain syntax to manipulate your data. Furthermore, the 
`_. chain` method is aliased to `_` itself. In other words - `_(value)` is identical to `_.chain(value)`.

The result of one function is piped to the input of the next function.

The `_.chain` method is lazy-loaded, and the value is not unwrapped (nor any of the functions in the chain called)
until you call the `.value()` method.

Taking the example from backfilling above, we could do something like the following:

```js
// example from before: 
const 
	maxItems = 10,
	curated = data.items || [],
	backfilled = await getBackfillItems(),
	displayItems = _.take(_.unionBy(curated, backfilled, 'slug'), maxItems);

// after:
const
  maxItems = 10,
  displayItems = _(data.items || []) // we're operating on (a copy of) data.items
    .unionBy(await getBackfillItems(), 'slug') // adding in the backfilled items, ignoring duplicates (by slug) 
    .take(maxItems) // collecting only the number of items we need
    .value();
```

The latter form is a little cleaner, and it's clearer that you're chaining data.items through a series of 
transformations. It's a little nicer looking to see the series of functions you're running as a dot-chain, rather 
that wrapping them up and nesting them within each other.

Note: Not all methods work with `_.chain`. Certain methods are not available. These can be found 
[listed in the docs](https://lodash.com/docs/4.17.15#lodash)



### Function

There are some useful methods that operate on functions, some of which we already use throughout our codebase. In 
particular, we use the debounce method:

#### _.debounce

Accepts a function, and returns a version of that function that will only be run once over some given period of time.
Particularly useful when you want to do something each time the user resizes the window, for example. Since the `resize`
event is triggered each time a frame is rendered when the user is resizing, an event listener will trigger many times.
Usually, we want to wait until the user is "finished" resizing, and then call our event listener:

e.g.

```js
function resizeHandler(e){
  // Do some stuff,..
};

const debouncedResizeHandler = _debounce(
      resizeHandler, 350, { leading:false, trailing:true }
    );
window.on('resize', debouncedResizeHandler);
// "resizeHandler" will be called a maximum of once every 350ms
// More specifically, with these options, it will listen for a 'resize' event,
// wait until the 'resize' event has not been called again for 350ms
// and then run the resizeHandler
// In this way, we can wait for the user to "finish resizing" before calling our handler
```




### Conclusion

Basically, Lodash is awesome, and you should take the time once or twice each week to peruse all their offerings. 
Most of the functions are pretty obvious what they do from the name, but if you don't know immediately what a 
function does based on its name, then click on it and take a couple minutes to se what it does. Come back to the 
docs every now and then to refresh your memory and remind yourself of what they have. You'll find yourself asking 
yourself "does Lodash have a function for this?" And odds are, yes, they do. The functions are organized sensibly 
based on the kind of data they operate on, they have sensible defaults, they fail gracefully, and they intelligently 
use other helper functions when appropriate.

Don't re-invent the wheel, especially when the wheel that exists is slick and chrome.
