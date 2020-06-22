# redux-loop-immutable

ImmutableJS helpers for use with [Redux Loop](https://redux-loop.js.org/)

## Installation

The latest Redux Loop Immutable requires the latest redux-loop version. If you need to support an older version of redux-loop, use an older version of redux-loop-immutable.

```
yarn add redux-loop-immutable
```

## API

### `combineReducers(reducersMap)`

This is simply an ImmutableJS-optimized version of the `combineReducers` provided by core redux-loop.
Instead of the default state being a plain object, it is an Immutable.Map instance.

#### Arguments
* `reducersMap: Object<string, ReducerFunction>` &ndash; a plain object map of keys to nested
  reducers, just like the `combineReducers` you would find in Redux itself.

#### Examples
```js
import { combineReducers } from 'redux-loop-immutable';
import reducerWithSideEffects from './reducer-with-side-effects';
import plainReducer from './plain-reducer';

export default combineReducers({
  withEffects: reducerWithSideEffects,
  plain: plainReducer
});
```

### `mergeChildReducers(parentResult, action, childMap)`

NOTE: `mergeChildReducers` is deprecated as of version 2.0.0. It will be removed in 3.0.0. `combineReducers` and `reduceReducers` (which needs no immutable helper) should cover the cases where `mergeChildReducers` is needed. If you need to use it temporarily without a console warning, use `DEPRECATED_mergeChildReducers(parentResult, action, childMap)`. This will also be removed in 3.0.0 though.

This is an ImmutableJS-optimized version of the `mergeChildRedcuers` provided by core redux-loop.
Like that version of `mergeChildReducers`, it is a more generalized version of `combineReducers` that allows
you to nest reducers underneath a common parent that has functionality of its own (rather than restricting the parent to simply passing actions to its children like `combineReducers` does)

* `parentResult: Immutable.Map | loop(Immutable.Map, Cmd)` &ndash; The result from the parent reducer before any child results have been applied.
* `action: Action` &ndash; a redux action
* `childMap: Object<string, ReducerFunction>` &ndash; a plain object map of keys to nested
  reducers, similar to the map in combineReducers. However, a key can be given a value of null to have it removed from the state.

#### Examples
```js
import { mergeChildReducers } from 'redux-loop-immutable';
import {getModel, isLoop} from 'redux-loop';
import { fromJS } from 'immutable'
import pageReducerMap from './page-reducers';

// a simple reducer that keeps track of your current location and nests the correct
//child reducer for that location at state.data

const initialState = fromJS({
   location: 'index'
   //data will be filled in with the result of the child reducer
});

function parentReducer(state = initialState, action){
  if(action.type !== 'LOCATION_CHANGE')
     return state;
     
  return state.set('location', action.newLocation);
}

export default function reducer(state, action){
  const parentResult = parentReducer(state, action);
  
  let location;
  if(isLoop(parentResult))
    location = getModel(parentResult).get('location');
  else
    location = parentResult.get('location');
   
  const childMap = {data: pageReducerMap[location]};
  
  return mergeChildReducers(parentResult, action, childMap);
}

```

## Support

Potential bugs, generally discussion, and proposals or RFCs should be submitted
as issues to this repo, we'll do our best to address them quickly. We use this
library as well and want it to be the best it can! For questions about using the
library, [submit questions on StackOverflow](http://stackoverflow.com/questions/ask)
with the [`redux-loop` tag](http://stackoverflow.com/questions/tagged/redux-loop).

## Contributing

Please note that this project is released with a [Contributor Code of Conduct](CODE_OF_CONDUCT.md). By participating in this project you agree to abide by its terms. Multiple language translations are available at [contributor-covenant.org](https://www.contributor-covenant.org/translations.html)
