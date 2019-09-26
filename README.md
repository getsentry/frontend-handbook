# frontend-handbook

Frontend at Sentry

> This guide covers how we write frontend code at Sentry, and is specifically focussed on the [Sentry](https://github.com/getsentry/sentry) and [Getsentry](https://github.com/getsentry/getsentry) codebases. It assumes you are using the eslint rules outlined by [eslint-config-sentry](https://github.com/getsentry/eslint-config-sentry); hence code style enforced by these linting rules will not be discussed here.

## Quick links:

- [Directory structure](#directory-structure)
- [React](#react)
- [CSS and Emotion](#css-and-emotion)
- [State management](#state-management)
- [Testing](#testing)
- [Babel Syntax Plugins](#babel-plugins)
- [Contributing](#contributing)

## Directory structure

The frontend codebase is currently located under `src/sentry/static/sentry/app` in sentry and `static/getsentry` in getsentry. (We intend to align to `static/sentry` in future.)

## React

### Defining React components

New components use the class syntax, and the class field+arrow function method definition when they need to access this.

```javascript
class Note extends React.Component {
  static propTypes = {
    author: PropTypes.object.isRequired,
    onEdit: PropTypes.func.isRequired,
  };

  // Note that method is defined using an arrow function class field (to bind "this")
  handleChange = value => {
    let user = ConfigStore.get('user');

    if (user.isSuperuser) {
      this.props.onEdit(value);
    }
  };

  render() {
    let {content} = this.props; // use destructuring assignment for props

    return <div onChange={this.handleChange}>{content}</div>;
  }
}

export default Note;
```

Some older components use `createReactClass` and mixins, but this is deprecated.

### Components vs views

Both the `app/components/` and `app/views` folders contain React components.

- Use a view for UI that will typically not be reused in other parts of the codebase
- Use a component for UI that is designed to be highly reusable.

Components should have an associated `.stories.js` file that documents how it should be used.

Run Storybook locally with `yarn storybook` or view the hosted version at `https://storybook.getsentry.net/`

### PropTypes

Use them, be explicit, use the shared custom proptypes when possible.

Prefer `Proptypes.arrayOf` to `PropTypes.array` and `PropTypes.shape` to `PropTypes.object`

If you’re passing Objects with an important, well defined set of keys (that your component relies on) then define them explicitly with `PropTypes.shape`:

```javascript
PropTypes.shape({
  username: PropTypes.string.isRequired,
  email: PropTypes.string
})
```

If you’re re-using a custom prop-type or passing around a common shared shape like an organization, project, or user, then be sure to import a proptype from our useful collection of custom ones! [https://github.com/getsentry/sentry/blob/master/src/sentry/static/sentry/app/sentryTypes.jsx](https://github.com/getsentry/sentry/blob/master/src/sentry/static/sentry/app/sentryTypes.jsx)

### Event handlers

We use different prefixes to better distinguish event handlers from event callback props.

Use the `handle` prefix for event handlers, e.g:

```javascript
<Button onClick={this.handleDelete}/>
```

For event callback props passed to the component use the `on` prefix, e.g:

```javascript
<Button onClick={this.props.onDelete}>
```

## CSS and Emotion

Use Emotion, use the `theme` object.

The best styles are ones you don’t write - whenever possible use existing components.

New code should use the css-in-js tool [e m o t i o n](https://emotion.sh/) - it lets you bind styles to elements without the indirection of global selectors. You don’t even need to open another file!

Take constants (z-indexes, paddings, colors) from [p.theme](https://github.com/getsentry/sentry/blob/master/src/sentry/static/sentry/app/utils/theme.jsx)

```javascript
import styled from 'react-emotion';

const SomeComponent = styled('div')`
  border-radius: 1.45em;
  font-weight: bold;
  z-index: ${p => p.theme.zIndex.modal};
  padding: ${p => p.theme.grid}px ${p => p.theme.grid * 2}px;
  border: 1px solid ${p => p.theme.borderLight};
  color: ${p => p.theme.purple};
  box-shadow: ${p => p.theme.dropShadowHeavy};
`;

export default SomeComponent;
```

Note that `emotion-grid` (e.g. `Flex` and `Box`) is being deprecated, avoid using in new code.

## State management

We currently use [Reflux](https://github.com/reflux/refluxjs) for managing global state.

Reflux implements the unidirectional data flow pattern outlined by [Flux](https://facebook.github.io/flux/docs/overview.html). Stores are registered under `app/stores` and are used to store various pieces of data used by the application. Actions need to be registered under `app/actions`. We use action creator functions (under `app/actionCreators`) to dispatch actions. Reflux stores listen to actions and update themselves accordingly.

We are currently exploring alternatives to the `Reflux` library for future use.

## Testing

Note: Your filename needs to be .spec.jsx or jest won’t run it!

We have useful fixtures defined in [setup.js](https://github.com/getsentry/sentry/blob/master/tests/js/setup.js) Use these! If you are defining mock data in a repetitive way, it’s probably worth adding this this file. routerContext is a particularly useful one for providing the context object that most view are written to rely on.

Client.addMockResponse is the best way to mock API requests. it’s [our code](https://github.com/getsentry/sentry/blob/master/src/sentry/static/sentry/app/__mocks__/api.jsx) so if it’s confusing you, just put `console.log()` statements into its logic!

An important gotcha in our testing environment is the way that enzyme modifies many aspects of the react lifecycle to evaluate synchronously (even when they’re usually async). This can lull you into a false sense of security when you trigger some logic and don’t find it reflected immediately afterwards in your assert logic.

Marking your test method `async` and using the `await tick();` utility can let the event loop flush run events and fix this:

```javascript
wrapper.find('ExpandButton').simulate('click');
await tick();
expect(wrapper.find('CommitRow')).toHaveLength(2);
```

Selectors:
If you are writing jest tests, you can use a Component (and Styled Component) names as a selector. Additionally, if you need to use a DOM query selector, use `data-test-id` instead of a class name. We currently don’t, but it is something we can use babel to strip out during the build process.

## Babel Syntax Plugins
We have decided to only use ECMAScript proposals that are in stage 3 (or later) (See [TC39 Proposals](https://github.com/tc39/proposals)). Additionally, because we are migrating to typescript, we will align with what their compiler supports.
The only exception to this are decorators.


## Contributing

The [issue tracker](https://github.com/getsentry/frontend-handbook/issues/) is the prefered channel to propose and disucss a change. Or create a [pull request](https://github.com/getsentry/frontend-handbook/pulls)!
