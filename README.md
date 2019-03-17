# React Single File Components

Some thoughts on bringing Single File Components to React.

> ⚠️ Note that this isn't a formal proposal for the React community, this is just a half baked idea completely for fun that I am not yet serious about.

## Why? I don't need this!

That's right, you don't -need- it. SFCs are always sugar, just like JSX. You don't need it, but when it is enough of a community standard it makes things nicer for almost everyone. SFC's aren't a required part of Vue, but they are a welcome community norm.

The goal isn't to evaluate this idea based on need. In my mind this will live or die based on how well it accomplishes two goals:

- For beginners, provide a blessed structure in a chaotic world of anything-goes.
- For experts, provide a nicer DX by encoding extremely common boilerplatey patterns in syntax.

Any new file format starts with a handicap of not working with existing tooling e.g. syntax highlighting. So a successful React SFC effort will also need to have a plan for critical tooling.

## Prior art

- vue-loader:
  - https://vue-loader.vuejs.org/guide/#vue-cli
  - vue [parseComponent](https://github.com/vuejs/vue/blob/dev/src/sfc/parser.js)
  - vue compiler utils [parse](https://github.com/vuejs/component-compiler-utils/blob/master/lib/parse.ts)
  - [vue html parsing](https://github.com/vuejs/vue/blob/dev/src/compiler/parser/html-parser.js)
- https://github.com/digitalie/one-loader ([HN comments](https://news.ycombinator.com/item?id=15408140))
- https://github.com/windyGex/react-template-loader (very old, uses `<template>` for some reason)
- https://github.com/LukasBombach/single-file-components
- reddit discusssions:
  - https://www.reddit.com/r/reactjs/comments/6kzqm0/is_there_a_react_equivalent_to_vues_single_file/
  - https://www.reddit.com/r/reactjs/comments/9495ft/single_file_components_no_one_misses_it/
  - every few months it pops up: https://www.reddit.com/r/reactjs/search?q=single%20file%20components&restrict_sr=1

## Basic Proposal

All top level tags within SFCs would be extensible, but come with some good defaults.

```vue
<style>
div {
  // scoped by default
  background-color: papayawhip;
  .Button {
    border-color: cadetblue;
  }
}
</style>
<jsx>
  export default function() {
    return (
    <div>
      Some Text
      <button className="Button">Call to Action</button>
    </div>
    )
  }
</jsx>
```

The styles can be compiled to CSS modules or Linaria or some similar no-runtime alternative.

The component name would be taken from the filename. Named exports would also be externally accessible.

## Scoped syntax

Being able to specify options for a particular top level tag can make it very low friction to switch syntaxes based on need. This will probably be most useful for switching from no-runtime styling to CSS-in-JS:

```vue
<style lang="styled-components">
div {
  background-color: ${({theme}) => theme.bgColor}
  ${props.isActive && `
    .Button {
      border-color: cadetblue;
    }
  `
  }
}
</style>
```

_Note: there are smaller details to sweat here with regards to passing down className, but we're staying high level for now_

While we're at it, lets locally opt into [JSX 2.0](https://github.com/facebook/jsx/issues/65), _without affecting the rest of the project_:

```vue
<jsx v2>
  import UserProfile from 'comopnents/UserProfile'
  import UserContext from 'contexts/User'
  export default function() {
    const userData = useContext(UserContext) // this is an object
    return <UserProfile userData />
  }
</jsx>
```

## Not just styles and JSX

What about data requirements?

```vue
<gql>
query Blogposts {
  posts {
    _id
    title
    date
  }
}
</gql>
<jsx v2>
  import BlogCard from 'comopnents/BlogCard'
  export default function({data, loading}) {
    if (loading) return "loading..."
    return (
    <div>
      {data.posts.map(post => <BlogCard post />
    </div>
    )
  }
</jsx>
```

What else colocates? Tests? sure.

<details>
  <summary>Aside: taking the gql tag to the extreme </summary>
Since loading is boilerplatey and distracts from the actual component JSX, what opportunities are there for rendering multiple UIs for component state? *Note: this is reaaaally sugary, but just a fun thought experiment*.

```vue
<gql>
query Blogposts {
  posts {
    _id
    title
    date
  }
}
</gql>
<jsx when={props => props.data.loading}>
  export default function() {
    return 'loading...'
  }
</jsx>
<jsx when={props => props.data.error} errorBoundary>
  // errorBoundary can be a flag for when the tree has run into a rendering error
  export default function() {
    return 'error!'
  }
</jsx>
<jsx v2>
  // here we can just focus on our component happy path
  import BlogCard from 'comopnents/BlogCard'
  export default function({data}) {
    return (<div>
              {data.posts.map(post => <BlogCard post />
            </div>)
  }
</jsx>
```

</details>

<details>
  <summary>
    Aside: crazy idea for serverless functions
  </summary>
How about compiling Next.js style `getInitialProps` for a static site generator, so that it runs in the static build, but the function also builds to a serverless function where you can pull the data clientside?

```vue
<initialprop name="joke">
import fetch from 'node-fetch'; // this is serverless node code!
export async function handler(event, context) {
  const response = await fetch('https://api.chucknorris.io/jokes/random');
  const data = await response.json();
  return {
    statusCode: 200,
    body: JSON.stringify({ msg: data.value })
  };
}
</initialprop>
<jsx v2>
  export default function({joke}) {
    return <div>{joke}</div>
  }
</jsx>
```

</details>

What about documentation?

```vue
<stories>
import { storiesOf } from '@storybook/react';
import { action } from '@storybook/addon-actions';

storiesOf('Button', module)
  .add('with text', () => <Button onClick={action('clicked')}>Hello Button</Button>)
  .add('with some emoji', () => (
    <Button onClick={action('clicked')}>
      <span role="img" aria-label="so cool">
        😀 😎 👍 💯
      </span>
    </Button>
  ));
</stories>
<jsx v2>
  export default function Button() { // Naming component needed?
    return (<div>
      Some Text
      <button className="Button">Call to Action</button>
    </div>)
  }
</jsx>
```

<details>
  <summary> Extra sugar version of this idea
  </summary>
  
  Breaking out individiual stories:
  
```vue
<story title="with text">
  export default () => <Button>Hello Button</Button>
</story>
<story title="with some emoji">
  import { action } from '@storybook/addon-actions';
  export default () => (
    <Button onClick={action('clicked')}>
      <span role="img" aria-label="so cool">
        😀 😎 👍 💯
      </span>
    </Button>
  )
</story>
```
</details>

Or instead of storybook we can use MDX and [the Docz model](http://docz.site). Note we can pass args to the top level tags as well here:

```mdx
## <mdx docz>

## name: Button

# Look at my Button

<Playground>
  <Button />
</Playground>

<Playground>{() => <Button />}</Playground>

<Playground>
  {() => {
    const someVar = 'value'
    return <Button example={someVar} />
  }}
</Playground>
</mdx>
```

## MDX-SFC alternative proposal

Having all these tags seem like wasted opportunity in top level whitespace. Why not make SFC's a subset of MDX and have documentation tooling parse it accordingly?

```mdx
---
name: MyComponent
---

## Source

<jsx v2>
  export default function Button() {
    return (<div>
      Some Text
      <button className="Button">Call to Action</button>
    </div>)
  }
</jsx>

## Data Requirements

<gql>
query Blogposts {
  posts {
    _id
    title
    date
  }
}
</gql>

## Play with my Button

<Playground>
  <Button />
</Playground>

<Playground>{() => <Button />}</Playground>

<Playground>
  {() => {
    const someVar = 'value'
    return <Button example={someVar} />
  }}
</Playground>
```

This could be documentation.

## General principle: Loaders vs SFCs

Stepping back from concrete examples to discuss how this might affect DX. In a sense, SFCs simply centralize what we already do with loaders. Instead of

```
Component.jsx
Component.scss
Component.graphql
Component.types.ts
Component.stories.js
Component.test.js
```

we have

```
<tsx> // etc...
<style> // etc...
<gql> // etc...
<stories> // etc...
<test> // etc...
```

in a file. Why would we exchange file separation for a super long file? Although there are ways to mitigate this, it is not very appealing on its own.

However, to the extent that the React SFC loader is a single entry point to webpack for all these different filetypes, we have the opportunity to simplify config, skip small amounts of boilerplate, and enforce some consistency with the single file format. Having fewer files causes less pollution of IDE file namespace, and makes it easier to set up these peripheral concerns around jsx (styling, data, tests, documentation, etc) incrementally without messing with creating/deleting files.

## Am I missing some obvious idea or some critical flaw?

File an issue or PR or [tweet at me](https://twitter.com/swyx), lets chat.
