# cancelbl
Yet another cancelable Promise wrapper,
inspired by [this](https://facebook.github.io/react/blog/2015/12/16/ismounted-antipattern.html) article from React blog.

## Installation

```shell
# npm
npm install cancelbl

# yarn
yarn add cancelbl
```

Or you can copy-paste it from [here](https://github.com/sergeysolovev/cancelbl/blob/master/src/cancelbl.js) if ES6 sounds good to you.

## Overview
For React components, that use [fetch](https://developer.mozilla.org/en/docs/Web/API/Fetch_API) to update the state, unmounting can lead to the following issue:
```
setState(…): Can only update a mounted or mounting component.
This usually means you called setState() on an unmounted component. This is a no-op
```
The correct way to fix this issue, according to [the article](https://facebook.github.io/react/blog/2015/12/16/ismounted-antipattern.html), is to cancel any callbacks in `componentWillUnmount`, prior to unmounting.
Suggested `makeCancelable` wraps target promise and returns `cancel()` function, which can be called in `componentWillUnmount`.

The advantage of this implementation is that it wraps and handles the promise in a single statement. Check out Example section below to see how simple it is.

For complete specification, see [cancelbl.test.js](https://github.com/sergeysolovev/make-cancelable/blob/master/src/cancelbl.test.js).

## Usage: React component
A typical example of two subsequent fetches without taking care of the method `setState`:

```javascript
componentDidMount() {
  fetchResource(this.props.url)
    .then(item => {
      this.setState({item})
      fetchRelatedResources(item)
        .then(relatedItems => {
          this.setState({relatedItems});
        })
        .catch(error => { /* handle */ })
    })
    .catch(error => { /* handle */ });
}
```

Let's import cancelbl:

```javascript
import cancelbl from 'cancelbl';
```

And update the component:

```javascript
cancel = cancelbl.default;

componentDidMount() {
  this.cancel = cancelbl.make(
    fetchResource(this.props.url),
    item => {
      this.setState({item});
      this.cancel.with(
        fetchRelatedResources(item),
        relatedItems => this.setState({relatedItems}),
        error => { /* handle */ }
      );
    },
    error => { /* handle */ }
  );
}

componentWillUnmount() {
  this.cancel.do();
}
```

Here we go.

## Running the tests

```shell
# with npm
npm test

# with yarn
yarn test
```

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

[SemVer](http://semver.org/) is used for versioning. For the versions available, see the [tags on this repository](https://github.com/sergeysolovev/cancelbl/tags).

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Credits
- [@istarkov](https://github.com/istarkov) for original makeCancelable implementation [here](https://github.com/facebook/react/issues/5465#issuecomment-157888325);
