---
template: BlogPost
path: /best-practices-component
date: 2019-12-03T00:12:25.364Z
title: Best Practices Components
metaDescription: >-
  Components should be able to be reusable in different applications and
  therefore should not be bound to a particular store. A previous way to
  accomplish this was to use the container/component design pattern and create
  separate files for the HOC (High Order Components).
thumbnail: /assets/puzzle-1713170_1280.jpg
---
Components should be able to be reusable in different applications and therefore should not be bound to a particular store. A previous way to accomplish this was to use the container/component design pattern and create separate files for the HOC (High Order Components). A similar separation can be accomplished by exporting the component when not connected to Redux. Directly export your unconnected component alongside default export of the connected component. E.g.:

```javascript
  // raw, unconnected component for testing
  export function HeaderLinks(props) {
      ...
      return (
      <Grid container item className={classes.nav}>
          <HeaderMenu renderMenuLinks={() => menuLinks} />
      </Grid>
          )
  }
  // connected (or any other sort of HOC component, etc) for use in App
  export default connect(mapStateToProps)(compose(withStyles(styles), withWidth())(HeaderLinks));
```

## Styles

In order to create more self-contained code and to reduce the overhead of using extra HoC components. Each component should have their own styles in the same file unless that object becomes too large and it makes it harder to read. makeStyles is preferred for adding custom classes.

```javascript
import React from "react";
import { makeStyles } from "@material-ui/core/styles";

const useStyles = makeStyles({
  root: {
    backgroundColor: "red",
    color: props => props.color
  }
});

export default function MyComponent(props) {
  const classes = useStyles(props);
  return <div className={classes.root} />;
}
```

## PropTypes

PropTypes should be used on every component. The following are the ways that they should be implemented on each type of component.

### Class Components and PureComponent

```javascript
import React from "react";
import PropTypes from "prop-types";

export class MyComponent extends React.Component {
  static propTypes = {
    prop1: PropTypes.string.isRequired,
    prop2: PropTypes.bool,
    prop3: PropTypes.func
  };

  defaultProps = {
    prop2: false,
    prop3: () => {}
  };

  constructor() {}
  render() {}
}

export default MyComponent;
```

### Functional Component

```javascript
import React from "react";
import PropTypes from "prop-types";

export const MyComponent = props => {};

MyComponent.propTypes = {
  prop1: PropTypes.string.isRequired,
  prop2: PropTypes.bool,
  prop3: PropTypes.func
};
MyComponent.defaultProps = {
  prop2: false,
  prop3: () => {}
};

export default MyComponent;
```
