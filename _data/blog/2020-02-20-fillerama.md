---
template: BlogPost
path: /best-practices-storybook
date: 2020-01-09T14:59:36.571Z
title: Best Practices Storybook
titulo: Mejores prácticas de Storybook
metaDescription: >-
  Storybook is a user interface development environment and playground for UI
  components. The tool enables developers to create components independently and
  showcase components interactively in an isolated development environment.
  Storybook runs outside of the main app so users can develop UI components in
  isolation without worrying about app-specific dependencies and requirements.
thumbnail: /assets/image-5.jpg
---
Storybook is a user interface development environment and playground for UI components. The tool enables developers to create components independently and showcase components interactively in an isolated development environment. Storybook runs outside of the main app so users can develop UI components in isolation without worrying about app-specific dependencies and requirements.

<video autoplay="" muted="" loop="" playsinline="" alt="Storybook video" style="width:100%">
<source src="https://storybook.js.org/videos/storybook-hero-video-optimized.mp4" type="video/mp4">
</video>

Since Storybook 5.2, Storybook’s Component Story Format (CSF) is the recommended way to [write stories](https://storybook.js.org/docs/basics/writing-stories/). In CSF, stories and component metadata are defined as ES6 modules. Every Component story file consists of a required default export and one or more named exports.

### Default export

The default export defines metadata about your component, including the component itself, its title (where it will show up in the [navigation UI story hierarchy](https://storybook.js.org/docs/basics/writing-stories/#story-hierarchy)), [decorators](https://storybook.js.org/docs/basics/writing-stories/#decorators), and [parameters](https://storybook.js.org/docs/basics/writing-stories/#parameters). title should be unique, i.e. not re-used across files.

```javascript
import { withKnobs, select } from "@storybook/addon-knobs";
import Button from "@material-ui/core/Button";
import NavigationIcon from "@material-ui/icons/Navigation";
import React from "react";

export default {
  title: "atoms|Button",
  decorators: [withKnobs]
};

export const containedButton = () => (
  <Button
    variant="contained"
    color={select("color", ["primary", "secondary"], "primary")}
    size={select("size", ["small", "medium", "large"], "large")}
  >
    Default
  </Button>
);

export const outlinedButton = () => (
  <Button
    variant="outlined"
    color={select("color", ["primary", "secondary"], "primary")}
    size={select("size", ["small", "medium", "large"], "large")}
  >
    <NavigationIcon />
    Default
  </Button>
);
```

### Stories for Components that use Redux

By using brackets {} we are importing the RAW component before it has tied to the HOC and therefore should only be expecting props in order to render correctly.

```javascript
import React from "react";
import { withKnobs, select } from "@storybook/addon-knobs";

import { MyComponent } from "../MyComponent";

export default {
  title: "Atom|MyComponent",
  decorators: [withKnobs]
};

export const Basic = () => <MyComponent />;
export const WithProp = () => (
  <MyComponent
    variant="contained"
    color={select("color", ["primary", "secondary"], "primary")}
    size={select("size", ["small", "medium", "large"], "large")}
  />
);
```
