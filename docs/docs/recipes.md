---
title: Recipes
---

<!-- Basic template for a Gatsby recipe:

## Task to accomplish.
1-2 sentences about it. The more concise and focused, the better!

### Prerequisites
- System/version requirements
- Everything necessary to set up the task
- Including setting up accounts at other sites, like Netlify
- See [docs templates](/docs/docs-templates/) for formatting tips

### Directions
Step-by-step directions. Each step should be repeatable and to-the-point. Anything not critical to the task should be omitted.

#### Live example (optional)
A live example may not be possible depending on the nature of the recipe, in which case it is fine to omit.

### Additional resources
- Tutorials
- Docs pages
- Plugin READMEs
- etc.

See [docs templates](/docs/docs-templates/) in the contributing docs for more help.
-->

Craving a happy medium between [full-length tutorials](/tutorial/) and crawling the [docs](/docs/)? Here's a cookbook of guiding recipes on how to build things, Gatsby style.

## 1. Pages and Layouts

### Project structure

Inside a Gatsby project, you may see some or all of the following folders and files:

```
|-- /.cache
|-- /plugins
|-- /public
|-- /src
    |-- /pages
    |-- /templates
    |-- html.js
|-- /static
|-- gatsby-config.js
|-- gatsby-node.js
|-- gatsby-ssr.js
|-- gatsby-browser.js
```

Some notable files and their definitions:

- `gatsby-config.js` — configure options for a Gatsby site, with metadata for project title, description, plugins, etc.
- `gatsby-node.js` — implement Gatsby’s Node.js APIs to customize and extend default settings affecting the build process
- `gatsby-browser.js` — customize and extend default settings affecting the browser, using Gatsby’s browser APIs
- `gatsby-ssr.js` — use Gatsby’s server-side rendering APIs to customize default settings affecting server-side rendering

#### Additional resources

- For a tour of all the common folders and files, read the docs on [Gatsby's Project Structure](/docs/gatsby-project-structure/)
- For common commands, check out the [Gatsby CLI docs](/docs/gatsby-cli)
- Check out the [Gatsby Cheat Sheet](/docs/cheat-sheet/) for downloadable info at a glance

### Creating pages automatically

Gatsby core automatically turns React components in `src/pages` into pages with URLs.
For example, components at `src/pages/index.js` and `src/pages/about.js` would automatically create pages from those filenames for the site's index page (`/`) and `/about`.

#### Prerequisites

- A [Gatsby site](/docs/quick-start)
- The [Gatsby CLI](/docs/gatsby-cli) installed

#### Directions

1. Create a directory for `src/pages` if your site doesn't already have one.
2. Add a component file to the pages directory:

```jsx:title=src/pages/about.js
import React from "react"

const AboutPage = () => (
  <main>
    <h1>About the Author</h1>
    <p>Welcome to my Gatsby site.</p>
  </main>
)

export default AboutPage
```

3. Run `gatsby develop` to start the development server.
4. Visit your new page in the browser: `http://localhost:8000/about`

#### Additional resources

- [Creating and modifying pages](/docs/creating-and-modifying-pages/)

### Linking between pages

Routing in Gatsby relies on the `<Link />` component.

#### Prerequisites

- A Gatsby site with two page components: `index.js` and `contact.js`
- The Gatsby `<Link />` component
- The [Gatsby CLI](/docs/gatsby-cli/) to run `gatsby develop`

#### Directions

1. Open the index page component (`src/pages/index.js`), import the `<Link />` component from Gatsby, add a `<Link />` component above the header, and give it a `to` property with the value of `"/contact/"` for the pathname:

```jsx:title=src/pages/index.js
import React from "react"
import { Link } from "gatsby"

export default () => (
  <div style={{ color: `purple` }}>
    <Link to="/contact/">Contact</Link>
    <p>What a world.</p>
  </div>
)
```

2. Run `gatsby develop` and navigate to the index page. You should have a link that takes you to the contact page when clicked!

> **Note**: Gatsby's `<Link />` component is a wrapper around [`@reach/router`'s Link component](https://reach.tech/router/api/Link). For more information about Gatsby's `<Link />` component, consult the [API reference for `<Link />`](/docs/gatsby-link/).

### Creating pages with `createPage`

Using Gatsby's [`createPages` API](/docs/actions/#createPage), you can create pages dynamically from a variety of data sources, including Markdown or Wordpress content.

This recipe shows how to create pages from Markdown files on your local filesystem using Gatsby's GraphQL data layer.

#### Prerequisites

- A [Gatsby site](/docs/quick-start) with a `gatsby-config.js` file
- The [Gatsby CLI](/docs/gatsby-cli) installed
- The [gatsby-source-filesystem plugin](/packages/gatsby-source-filesystem) installed
- The [gatsby-transformer-remark plugin](/packages/gatsby-transformer-remark) installed
- A `gatsby-node.js` file

#### Directions

1. In `gatsby-config.js`, configure `gatsby-transformer-remark` along with `gatsby-source-filesystem` to pull in Markdown files from a source folder. This would be in addition to any previous `gatsby-source-filesystem` entries, such as for images:

```js:title=gatsby-config.js
module.exports = {
  plugins: [
    `gatsby-transformer-remark`,
    {
      resolve: `gatsby-source-filesystem`,
      options: {
        name: `content`,
        path: `${__dirname}/src/content`,
      },
    },
  ]
```

2. Add a Markdown post to `src/content`, including frontmatter for the title, date, and path, with some initial content for the body of the post:

```markdown:title=src/content/my-first-post.md
---
title: My First Post
date: 2019-07-10
path: /my-first-post
---

This is my first Gatsby post written in Markdown!
```

3. Add the JavaScript code to generate pages from Markdown posts at build time with a GraphQL query in `gatsby-node.js`:

```js:title=gatsby-node.js
const path = require(`path`)

exports.createPages = async ({ actions, graphql }) => {
  const { createPage } = actions

  const blogPostTemplate = path.resolve(`src/templates/post.js`)

  const result = await graphql(`
    {
      allMarkdownRemark(
        sort: { order: DESC, fields: [frontmatter___date] }
        limit: 1000
      ) {
        edges {
          node {
            frontmatter {
              path
            }
          }
        }
      }
    }
  `)
  if (result.errors) {
    console.log(result.errors)
    throw new Error("Things broke, see console output above")
  }

  result.data.allMarkdownRemark.edges.forEach(({ node }) => {
    createPage({
      path: node.frontmatter.path,
      component: blogPostTemplate,
      context: {}, // additional data can be passed via context
    })
  })
}
```

4. Add a post template in `src/templates`, including a GraphQL query for generating pages dynamically from Markdown content at build time:

```jsx:title=src/templates/post.js
import React from "react"
import { graphql } from "gatsby"

export default function Template({ data }) {
  const { markdownRemark } = data // data.markdownRemark holds your post data
  const { frontmatter, html } = markdownRemark
  return (
    <div className="blog-post">
      <h1>{frontmatter.title}</h1>
      <h2>{frontmatter.date}</h2>
      <div
        className="blog-post-content"
        dangerouslySetInnerHTML={{ __html: html }}
      />
    </div>
  )
}

export const pageQuery = graphql`
  query($path: String!) {
    markdownRemark(frontmatter: { path: { eq: $path } }) {
      html
      frontmatter {
        date(formatString: "MMMM DD, YYYY")
        path
        title
      }
    }
  }
`
```

5. Run `gatsby develop` to start the development server. View your post in the browser: `http://localhost:8000/my-first-post`

#### Additional resources

- [Inspect Gatsby's data layer in GraphiQL](/docs/introducing-graphiql/)
- [Tutorial: Programmatically create pages from data](/tutorial/part-seven/)
- [Creating and modifying pages](/docs/creating-and-modifying-pages/)
- [Adding Markdown pages](/docs/adding-markdown-pages/)
- [Adding a list of Markdown blog posts](/docs/adding-a-list-of-markdown-blog-posts/)
- [Guide to creating pages from data programmatically](/docs/programmatically-create-pages-from-data/)

### Creating pages from data without GraphQL

You can use the node `createPages` API to pull unstructured data directly into Gatsby sites rather than through GraphQL and source plugins. In this recipe, you'll create dynamic pages from data fetched from the [PokéAPI’s REST endpoints](https://www.pokeapi.co/). The [full example](https://github.com/jlengstorf/gatsby-with-unstructured-data/) can be found on GitHub.

#### Prerequisites

- A Gatsby Site with a `gatsby-node.js` file
- The [Gatsby CLI](/docs/gatsby-cli) installed
- The [axios](https://www.npmjs.com/package/axios) package installed through npm

#### Directions

1. In `gatsby-node.js`, add the JavaScript code to fetch data from the PokéAPI and programmatically create an index page:

```js:title=gatsby-node.js
const axios = require("axios")

const get = endpoint => axios.get(`https://pokeapi.co/api/v2${endpoint}`)

const getPokemonData = names =>
  Promise.all(
    names.map(async name => {
      const { data: pokemon } = await get(`/pokemon/${name}`)
      return { ...pokemon }
    })
  )
exports.createPages = async ({ actions: { createPage } }) => {
  const allPokemon = await getPokemonData(["pikachu", "charizard", "squirtle"])

  // Create a page that lists Pokémon.
  createPage({
    path: `/`,
    component: require.resolve("./src/templates/all-pokemon.js"),
    context: { allPokemon },
  })
}
```

2. Create a template to display Pokémon on the homepage:

```js:title=src/templates/all-pokemon.js
import React from "react"

export default ({ pageContext: { allPokemon } }) => (
  <div>
    <h1>Behold, the Pokémon!</h1>
    <ul>
      {allPokemon.map(pokemon => (
        <li key={pokemon.id}>
          <img src={pokemon.sprites.front_default} alt={pokemon.name} />
          <p>{pokemon.name}</p>
        </li>
      ))}
    </ul>
  </div>
)
```

3. Run `gatsby develop` to fetch the data, build pages, and start the development server.
4. View your homepage in a browser: `http://localhost:8000`

#### Additional resources

- [Full Pokemon data repo](https://github.com/jlengstorf/gatsby-with-unstructured-data/)
- More on using unstructured data in [Using Gatsby without GraphQL](/docs/using-gatsby-without-graphql/)
- When and how to [Query data with GraphQL](/docs/querying-with-graphql/) for more complex Gatsby sites

### Creating a layout component

It's common to wrap pages with a React layout component, which makes it possible to share markup, styles, and functionality across multiple pages.

#### Prerequisites

- A Gatsby Site

#### Directions

1. Create a layout component in `src/components`, where child components will be passed in as props:

```jsx:title=src/components/layout.js
import React from "react"

export default ({ children }) => (
  <div style={{ margin: `0 auto`, maxWidth: 650, padding: `0 1rem` }}>
    {children}
  </div>
)
```

2. Import and use the layout component in a page:

```jsx:title=src/pages/index.js
import React from "react"
import Layout from "../components/layout"

export default () => (
  <Layout>
    <Link to="/contact/">Contact</Link>
    <p>What a world.</p>
  </Layout>
)
```

#### Additional resources

- Create a layout component in [tutorial part three](/tutorial/part-three/#your-first-layout-component)
- Styling with [Layout Components](/docs/layout-components/)

## 2. Styling with CSS

There are so many ways to add styles to your website; Gatsby supports almost every possible option, through official and community plugins.

### Using Styled Components

#### Prerequisites

- A [Gatsby site](/docs/quick-start/) with an index page component
- [gatsby-plugin-styled-components, styled-components, and babel-plugin-styled-components](/packages/gatsby-plugin-styled-components/) installed in `package.json`

#### Directions

1. Inside your `gatsby-config.js` file add `gatsby-plugin-styled-components`

```javascript:title=gatsby-config.js
module.exports = {
  plugins: [`gatsby-plugin-styled-components`],
}
```

2. Open the index page component (`src/pages/index.js`) and import the `styled-components` package

3. Style components by creating style blocks for each element type

4. Apply to the page by including styled components in the JSX

```jsx:title=src/pages/index.js
import React from "react"
import styled from "styled-components" //highlight-line

const Container = styled.div`
  margin: 3rem auto;
  max-width: 600px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
`

const Avatar = styled.img`
  flex: 0 0 96px;
  width: 96px;
  height: 96px;
  margin: 0;
`

const Username = styled.h2`
  margin: 0 0 12px 0;
  padding: 0;
`

const User = props => (
  <UserWrapper>
    <Avatar src={props.avatar} alt={props.username} />
    <Username>{props.username}</Username>
  </UserWrapper>
)

export default () => (
  <Container>
    <h1>About Styled Components</h1>
    <p>Styled Components is cool</p>
    <User
      username="Jane Doe"
      avatar="https://s3.amazonaws.com/uifaces/faces/twitter/adellecharles/128.jpg"
    />
    <User
      username="Bob Smith"
      avatar="https://s3.amazonaws.com/uifaces/faces/twitter/vladarbatov/128.jpg"
    />
  </Container>
)
```

4. Run `gatsby develop` to see the changes

#### Related Links

- [More on Using Styled Components](/docs/styled-components/)
- [Egghead lesson](https://egghead.io/lessons/gatsby-style-gatsby-sites-with-styled-components)

### Adding a Local Font

#### Prerequisites

- A [Gatsby site](/docs/quick-start/)
- A font file: `.woff2`, `.ttf`, etc.

#### Directions

1. Copy a font file into your Gatsby project, such as `src/fonts/fontname.woff2`.

```
src/fonts/fontname.woff2
```

2. Import the font asset into a CSS file to bundle it into your Gatsby site:

```css:title=src/css/typography.css
@font-face {
  font-family: "Font Name";
  src: url("../fonts/fontname.woff2");
}
```

**Note:** Make sure the font name is referenced from the relevant CSS, e.g.:

```css:title=src/components/layout.css
body {
  font-family: "Font Name", sans-serif;
}
```

By targeting the HTML `body` element, your font will apply to most text on the page. Additional CSS can target other elements, such as `button` or `textarea`.

#### Additional resources

- More on [importing assets into files](/docs/importing-assets-into-files/)

### Using Google Fonts

Hosting your own [Google fonts](https://fonts.google.com/) locally within a project means they won't have to be fetched over the network when your site loads, increasing your site's speed index by up to ~300 milliseconds on desktop and 1+ seconds on 3G. It's also recommended to limit custom font usage to only the essential for performance.

#### Prerequisites

- A [Gatsby site](/docs/quick-start)
- The [Gatsby CLI](/docs/gatsby-cli/) installed
- Choosing a font package from [the typefaces project](https://github.com/KyleAMathews/typefaces)

#### Directions

1. Run `npm install --save yourchosenfont`, replacing `yourchosenfont` with the name of the font you want to install from [the typefaces project](https://github.com/KyleAMathews/typefaces).

2. Add `import "yourchosenfont"` to a layout template, page component, or `gatsby-browser.js`.

```jsx:title=src/components/layout.js
import "yourchosenfont"
```

3. Once it's imported, you can reference the font name in a CSS stylesheet, CSS module, or CSS-in-JS.

```css:title=src/components/layout.css
body {
  font-family: yourchosenfont;
}
```

#### Additional Resources

- [Typography.js](/docs/typography-js/) - Another option for using Google fonts on a Gatsby site
- [The Typefaces Project Docs](https://github.com/KyleAMathews/typefaces/blob/master/README.md)
- [Live example on Kyle Mathews' blog](https://www.bricolage.io/typefaces-easiest-way-to-self-host-fonts/)

## 3. Working with starters

[Starters](/docs/starters/) are boilerplate Gatsby sites maintained officially, or by the community.

### Using a starter

#### Prerequisites

- The [Gatsby CLI](/docs/gatsby-cli) installed

#### Directions

1. Find the starter you'd like to use. (_The [Starter Library](/starters/?v=2) is a good place to look!_)

2. Generate a new site based on the starter. In the terminal, run:

```shell
gatsby new {your-project-name} {link-to-starter}
```

> _Don't run the above command as-is -- remember to replace {your-project-name} and {link-to-starter}!_

3. Run your new site:

```shell
cd {your-project-name}
gatsby develop
```

#### Additional resources

- Follow a [more detailed guide](/docs/starters/) on using Gatsby starters.
- Learn how to use the [Gatsby CLI](/docs/gatsby-cli) tool to use starters in [tutorial part one](/tutorial/part-one/#using-gatsby-starters)
- Browse the [Starter Library](/starters/?v=2)
- Check out Gatsby's [official default starter](https://github.com/gatsbyjs/gatsby-starter-default)

## 4. Working with themes

A Gatsby theme abstracts Gatsby configuration (shared functionality, data sourcing, design) into an installable package. This means that the configuration and functionality isn’t directly written into your project, but rather versioned, centrally managed, and installed as a dependency. You can seamlessly update a theme, compose themes together, and even swap out one compatible theme for another.

- Read more on [What is a Gatsby Theme?](/docs/themes/what-are-gatsby-themes)

### Creating a new site using a theme starter

Creating a site based on a starter that configures a theme follows the same process as creating a site based on a starter that **doesn't** configure a theme. In this example we'll use the [starter for creating a new site that uses the official Gatsby blog theme](https://github.com/gatsbyjs/gatsby-starter-blog-theme).

#### Prerequisites

- The [Gatsby CLI](/docs/gatsby-cli) installed

#### Directions

1. Generate a new site based on the blog theme starter:

```shell
gatsby new {your-project-name} https://github.com/gatsbyjs/gatsby-starter-blog-theme
```

2. Run your new site:

```shell
cd {your-project-name}
gatsby develop
```

#### Additional resources

- Learn how to use an existing Gatsby theme in the [shorter conceptual guide](/docs/themes/using-a-gatsby-theme) or the more detailed [step-by-step tutorial](/tutorial/using-a-theme).

### Building a new theme

<EggheadEmbed
  lessonLink="https://egghead.io/lessons/gatsby-use-the-gatsby-theme-workspace-starter-to-begin-building-a-new-theme"
  lessonTitle="Use the Gatsby Theme Workspace Starter to Begin Building a New Theme"
/>

#### Prerequisites

- The [Gatsby CLI](/docs/gatsby-cli) installed

* [Yarn](https://yarnpkg.com/lang/en/docs/install/#mac-stable) installed

#### Directions

1. Generate a new theme workspace using the [Gatsby theme workspace starter](https://github.com/gatsbyjs/gatsby-starter-theme-workspace):

```shell
gatsby new {your-project-name} https://github.com/gatsbyjs/gatsby-starter-theme-workspace
```

2. Run the example site in the workspace:

```shell
yarn workspace example develop
```

#### Additional resources

- Follow a [more detailed guide](/docs/themes/building-themes/) on using the Gatsby theme workspace starter.
- Learn how to build your own theme in the [Gatsby Theme Authoring video course on Egghead](https://egghead.io/courses/gatsby-theme-authoring), or in the [video course's complementary written tutorial companion](/tutorial/building-a-theme).

## 5. Sourcing data

Data sourcing in Gatsby is plugin-driven; Source plugins fetch data from their source (e.g. the `gatsby-source-filesystem` plugin fetches data from the file system, the `gatsby-source-wordpress` plugin fetches data from the WordPress API, etc). You can also source the data yourself.

### Creating source nodes

#### Directions

1. In `gatsby-node.js` use `sourceNodes()` and `actions.createNode()` to create and export nodes to be able to query the data.

```javascript:title=gatsby-node.js
exports.sourceNodes = ({ actions, createNodeId, createContentDigest }) => {
  const pokemons = [
    { name: "Pikachu", type: "electric" },
    { name: "Squirtle", type: "water" },
  ]

  pokemons.forEach(pokemon => {
    const node = {
      name: pokemon.name,
      type: pokemon.type,
      id: createNodeId(`Pokemon-${pokemon.name}`),
      internal: {
        type: "Pokemon",
        contentDigest: createContentDigest(pokemon),
      },
    }
    actions.createNode(node)
  })
}
```

2. Run `gatsby develop`.

   > _Note: After making changes in `gatsby-node.js` you need to re-run `gatsby develop` for the changes to take effect._

3. Query the data (in GraphiQL or in your components).

```graphql
query MyPokemonQuery {
  allPokemon {
    nodes {
      name
      type
      id
    }
  }
}
```

#### Additional resources

- Walk through an example using the `gatsby-source-filesystem` plugin in [tutorial part five](/tutorial/part-five/#source-plugins)
- Search available source plugins in the [Gatsby library](/plugins/?=source)
- Understand source plugins by building one in the [Pixabay source plugin tutorial](/docs/pixabay-source-plugin-tutorial/)
- The createNode function [documentation](/docs/actions/#createNode)

## 6. Querying data

### Using a Page Query

You can use the `graphql` tag to query data in the pages of your Gatsby site. This gives you access to anything included in Gatsby's data layer, such as site metadata, source plugins, images, and more.

### Deploying to Netlify

Use [`netlify-cli`](https://www.netlify.com/docs/cli/) to deploy your Gatsby application without leaving the command line interface.

#### Prerequisites

- A [Gatsby site](/docs/quick-start) with a single component `index.js`
- The [netlify-cli](https://www.npmjs.com/package/netlify-cli) package installed
- The [Gatsby CLI](/docs/gatsby-cli) installed

#### Directions

1. Build your gatsby application using `gatsby build`

2. Login into netlify using `netlify login`

3. Run the command `netlify build`. Select the "Create & configure a new site" option.

4. Choose a custom website name if you want or press enter to receive a random one.

5. Choose your [Team](/docs/teams/).

6. Change the deploy path to `public/`

7. Make sure that everything looks fine before deploying to production using `netlify deploy --prod`

#### Additional resources

- [Hosting on Netlify](/docs/hosting-on-netlify)
- [gatsby-plugin-netlify](/packages/gatsby-plugin-netlify)

## Querying data

#### Directions

1. Import `graphql` from `gatsby`.

2. Export a constant named `query` and set its value to be a `graphql` template with the query between two backticks.

3. Pass in `data` as a prop to the component.

4. The `data` variable holds the queried data and can be referenced in JSX to output HTML.

```jsx:title=src/pages/index.js
import React from "react"
// highlight-next-line
import { graphql } from "gatsby"

import Layout from "../components/layout"

// highlight-start
export const query = graphql`
  query HomePageQuery {
    site {
      siteMetadata {
        title
      }
    }
  }
`
// highlight-end

// highlight-next-line
const IndexPage = ({ data }) => (
  <Layout>
    // highlight-next-line
    <h1>{data.site.siteMetadata.title}</h1>
  </Layout>
)

export default IndexPage
```

#### Additional resources

- [GraphQL and Gatsby](/docs/graphql/): understanding the expected shape of your data
- [More on querying data in pages with GraphQL](/docs/page-query/)
- [MDN on Tagged Template Literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) like the ones used in GraphQL

### The StaticQuery Component

`StaticQuery` is a component for retrieving data from Gatsby's data layer in [non-page components](/docs/static-query/), such as a header, navigation, or any other child component.

#### Directions

1. The `StaticQuery` Component requires two render props: `query` and `render`.

```jsx:title=src/components/NonPageComponent.js
import React from "react"
import { StaticQuery, graphql } from "gatsby"

const NonPageComponent = () => (
  <StaticQuery
    query={graphql` // highlight-line
      query NonPageQuery {
        site {
          siteMetadata {
            title
          }
        }
      }
    `}
    render={(
      data // highlight-line
    ) => (
      <h1>
        Querying title from NonPageComponent with StaticQuery:
        {data.site.siteMetadata.title}
      </h1>
    )}
  />
)

export default NonPageComponent
```

2. You can now use this component as you would [any other component](/docs/building-with-components#non-page-components) by importing it into a larger page of JSX components and HTML markup.

### Querying data with the useStaticQuery hook

Since Gatsby v2.1.0, you can use the `useStaticQuery` hook to query data with a JavaScript function instead of a component. The syntax removes the need for a `<StaticQuery>` component to wrap everything, which some people find simpler to write.

The `useStaticQuery` hook takes a GraphQL query and returns the requested data. It can be stored in a variable and used later in your JSX templates.

#### Prerequisites

- You'll need React and ReactDOM 16.8.0 or later (keeping Gatsby updated handles this)
- Recommended reading: the [Rules of React Hooks](https://reactjs.org/docs/hooks-rules.html)

#### Directions

1. Import `useStaticQuery` and `graphql` from `gatsby` in order to use the hook query the data.

2. In the start of a stateless functional component, assign a variable to the value of `useStaticQuery` with your `graphql` query passed as an argument.

3. In the JSX code returned from your component, you can reference that variable to handle the returned data.

```jsx:title=src/components/NonPageComponent.js
import React from "react"
import { useStaticQuery, graphql } from "gatsby" //highlight-line

const NonPageComponent = () => {
  // highlight-start
  const data = useStaticQuery(graphql`
    query NonPageQuery {
      site {
        siteMetadata {
          title
        }
      }
    }
  `)
  // highlight-end
  return (
    <h1>
      Querying title from NonPageComponent: {data.site.siteMetadata.title}{" "}
      //highlight-line
    </h1>
  )
}

export default NonPageComponent
```

#### Additional resources

- [More on Static Query for querying data in components](/docs/static-query/)
- [The difference between a static query and a page query](/docs/static-query/#how-staticquery-differs-from-page-query)
- [More on the useStaticQuery hook](/docs/use-static-query/)
- [Visualize your data with GraphiQL](/docs/introducing-graphiql/)

### Limiting with GraphQL

When querying for data with GraphQL, you can limit the number of results returned with a specific number. This is helpful if you only need a few pieces of data or need to [paginate data](/docs/adding-pagination/).

To limit data, you'll need a Gatsby site with some nodes in the GraphQL data layer. All sites have some nodes like `allSitePage` and `sitePage` created automatically: more can be added by installing source plugins like `gatsby-source-filesystem` in `gatsby-config.js`.

#### Prerequisites

- A [Gatsby site](/docs/quick-start/)

#### Directions

1. Run `gatsby develop` to start the development server.
2. Open a tab in your browser at: `http://localhost:8000/___graphql`.
3. Add a query in the editor with the following fields on `allSitePage` to start off:

```graphql
{
  allSitePage {
    edges {
      node {
        id
        path
      }
    }
  }
}
```

4. Add a `limit` argument to the `allSitePage` field and give it an integer value `3`.

```graphql
{
  allSitePage(limit: 3) { // highlight-line
    edges {
      node {
        id
        path
      }
    }
  }
}
```

5. Click the play button in the GraphiQL page and the data in the `edges` field will be limited to the number specified.

#### Additional resources

- Learn about [nodes in Gatsby's GraphQL data API](/docs/node-interface/)
- [Gatsby GraphQL reference for limiting](/docs/graphql-reference/#limit)
- Live example:

<iframe
  title="Limiting returned data"
  src="https://711808k40x.sse.codesandbox.io/___graphql?query=%7B%0A%20%20allSitePage(limit%3A%203)%20%7B%0A%20%20%20%20edges%20%7B%0A%20%20%20%20%20%20node%20%7B%0A%20%20%20%20%20%20%20%20id%0A%20%20%20%20%20%20%20%20path%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A&explorerIsOpen=false"
  width="600"
  height="300"
/>

### Sorting with GraphQL

The ordering of your results can be specified with the GraphQL `sort` argument. You can specify which fields to sort by and the order to sort in.

For this recipe, you'll need a Gatsby site with a collection of nodes to sort in the GraphQL data layer. All sites have some nodes like `allSitePage` created automatically: more can be added by installing source plugins.

#### Prerequisites

- A [Gatsby site](/docs/quick-start)
- Queryable fields prefixed with `all`, e.g. `allSitePage`

#### Directions

1. Run `gatsby develop` to start the development server.
2. Open the GraphiQL explorer in a browser tab at: `http://localhost:8000/___graphql`
3. Add a query in the editor with the following fields on `allSitePage` to start off:

```graphql
{
  allSitePage {
    edges {
      node {
        id
        path
      }
    }
  }
}
```

4. Add a `sort` argument to the `allSitePage` field and give it an object with the `fields` and `order` attributes. The value for `fields` can be a field or an array of fields to sort by (this example uses the `path` field), the `order` can be either `ASC` or `DESC` for ascending and descending respectively.

```graphql
{
  allSitePage(sort: {fields: path, order: ASC}) { // highlight-line
    edges {
      node {
        id
        path
      }
    }
  }
}

```

5. Click the play button in the GraphiQL page and the data returned will be sorted ascending by the `path` field.

#### Additional resources

- [Gatsby GraphQL reference for sorting](/docs/graphql-reference/#sort)
- Learn about [nodes in Gatsby's GraphQL data API](/docs/node-interface/)
- Live example:

<iframe
  title="Sorting data"
  src="https://711808k40x.sse.codesandbox.io/___graphql?query=%7B%0A%20%20allSitePage(sort%3A%20%7Bfields%3A%20path%2C%20order%3A%20ASC%7D)%20%7B%0A%20%20%20%20edges%20%7B%0A%20%20%20%20%20%20node%20%7B%0A%20%20%20%20%20%20%20%20id%0A%20%20%20%20%20%20%20%20path%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A&explorerIsOpen=false"
  width="600"
  height="300"
/>

### Filtering with GraphQL

Queried results can be filtered down with operators like `eq` (equals), `ne` (not equals), `in`, and `regex` on specified fields.

For this recipe, you'll need a Gatsby site with a collection of nodes to filter in the GraphQL data layer. All sites have some nodes like `allSitePage` created automatically: more can be added by installing source and transformer plugins like `gatsby-source-filesystem` and `gatsby-transformer-remark` in `gatsby-config.js` to produce `allMarkdownRemark`.

#### Prerequisites

- A [Gatsby site](/docs/quick-start)
- Queryable fields prefixed with `all`, e.g. `allSitePage` or `allMarkdownRemark`

#### Directions

1. Run `gatsby develop` to start the development server.
2. Open the GraphiQL explorer in a browser tab at: `http://localhost:8000/___graphql`
3. Add a query in the editor using a field prefixed by 'all', like `allMarkdownRemark` (meaning that it will return a list of nodes)

```graphql
{
  allMarkdownRemark {
    edges {
      node {
        frontmatter {
          title
          categories
        }
      }
    }
  }
}
```

4. Add a `filter` argument to the `allMarkdownRemark` field and give it an object with the fields you'd like to filter by. In this example, Markdown content is filtered by the `categories` attribute in frontmatter metadata. The next value is the operator: in this case `eq`, or equals, with a value of 'magical creatures'.

```graphql
{
  allMarkdownRemark(filter: {frontmatter: {categories: {eq: "magical creatures"}}}) { // highlight-line
    edges {
      node {
        frontmatter {
          title
          categories
        }
      }
    }
  }
}
```

5. Click the play button in the GraphiQL page. The data that matches the filter parameters should be returned, in this case only sourced Markdown files tagged with a category of 'magical creatures'.

#### Additional resources

- [Gatsby GraphQL reference for filtering](/docs/graphql-reference/#filter)
- [Complete list of possible operators](/docs/graphql-reference/#complete-list-of-possible-operators)
- Learn about [nodes in Gatsby's GraphQL data API](/docs/node-interface/)
- Live example:

<iframe
  title="Filtering data"
  src="https://711808k40x.sse.codesandbox.io/___graphql?query=%7B%0A%20%20allMarkdownRemark(filter%3A%20%7Bfrontmatter%3A%20%7Bcategories%3A%20%7Beq%3A%20%22magical%20creatures%22%7D%7D%7D)%20%7B%0A%20%20%20%20edges%20%7B%0A%20%20%20%20%20%20node%20%7B%0A%20%20%20%20%20%20%20%20frontmatter%20%7B%0A%20%20%20%20%20%20%20%20%20%20title%0A%20%20%20%20%20%20%20%20%20%20categories%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A&explorerIsOpen=false"
  width="600"
  height="300"
/>

### GraphQL Query Aliases

You can rename any field in a GraphQL query with an alias.

If you would like to run two queries on the same datasource, you can use an alias to avoid a naming collision with two queries of the same name.

#### Directions

1. Run `gatsby develop` to start the development server.
2. Open the GraphiQL explorer in a browser tab at: `http://localhost:8000/___graphql`
3. Add a query in the editor using two fields of the same name like `allFile`

```graphql
{
  allFile {
    totalCount
  }
  allFile {
    pageInfo {
      currentPage
    }
  }
}
```

4. Add the name you would like to use for any field before the name of the field in your GraphQL schema, separated by a colon. (E.g. `[alias-name]: [original-name]`)

```graphql
{
  fileCount: allFile { // highlight-line
    totalCount
  }
  filePageInfo: allFile { // highlight-line
    pageInfo {
      currentPage
    }
  }
}
```

5. Click the play button in the GraphiQL page and 2 objects with alias names you provided should be output.

#### Additional resources

- [Gatsby GraphQL reference for aliasing](/docs/graphql-reference/#aliasing)
- Live example:

<iframe
  title="Using aliases"
  src="https://711808k40x.sse.codesandbox.io/___graphql?query=%7B%0A%20%20fileCount%3A%20allFile%20%7B%20%0A%20%20%20%20totalCount%0A%20%20%7D%0A%20%20filePageInfo%3A%20allFile%20%7B%0A%20%20%20%20pageInfo%20%7B%0A%20%20%20%20%20%20currentPage%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A&explorerIsOpen=false"
  width="600"
  height="300"
/>

### GraphQL Query Fragments

GraphQL fragments are shareable chunks of a query that can be reused.

You might want to use them to share multiple fields between queries or to colocate a component with the data it uses.

#### Directions

1. Declare a `graphql` template string with a Fragment in it. The fragment should be made up of the keyword `fragment`, a name, the GraphQL type it is associated with (in this case of type `Site`, as demonstrated by `on Site`), and the fields that make up the fragment:

```jsx
export const query = graphql`
  // highlight-start
  fragment SiteInformation on Site {
    title
    description
  }
  // highlight-end
`
```

2. Now, include the fragment in a query for a field of the type specified by the fragment. This includes those fields without having to declare them all independently:

```diff
export const pageQuery = graphql`
  query SiteQuery {
    site {
-     title
-     description
+   ...SiteInformation
    }
  }
`
```

**Note**: Fragments don't need to be imported in Gatsby. Exporting a query with a Fragment makes that Fragment available in _all_ queries in your project.

Fragments can be nested inside other fragments, and multiple fragments can be used in the same query.

#### Additional Resources

- [Simple example repo using fragments](https://github.com/gatsbyjs/gatsby/tree/master/examples/using-fragments)
- [Gatsby GraphQL reference for fragments](/docs/graphql-reference/#fragments)
- [Gatsby image fragments](/docs/gatsby-image/#image-query-fragments)
- [Example repo with co-located data](https://github.com/gatsbyjs/gatsby/tree/master/examples/gatsbygram)

## 7. Working with images

### Import an image into a component with webpack

Images can be imported right into a JavaScript module with webpack. This process automatically minifies and copies the image to your site's `public` folder, providing a dynamic image URL for you to pass to an HTML `<img>` element like a regular file path.

#### Prerequisites

- A [Gatsby Site](/docs/quick-start) with a `.js` file exporting a React component
- an image (`.jpg`, `.png`, `.gif`, `.svg`, etc.) in the `src` folder

#### Directions

1. Import your file from its path in the `src` folder:

```jsx:title=src/pages/index.js
import React from "react"
// Tell webpack this JS file uses this image
import FiestaImg from "../assets/fiesta.jpg" // highlight-line
```

2. In `index.js`, add an `<img>` tag with the `src` as the name of the import you used from webpack (in this case `FiestaImg`), and add an `alt` attribute [describing the image](https://webaim.org/techniques/alttext/):

```jsx:title=src/pages/index.js
import React from "react"
import FiestaImg from "../assets/fiesta.jpg"

export default () => (
  // The import result is the URL of your image
  <img src={FiestaImg} alt="A dog smiling in a party hat" /> // highlight-line
)
```

3. Run `gatsby develop` to start the development server.
4. View your image in the browser: `http://localhost:8000/`

#### Additional resources

- [Example repo importing an image with webpack](https://github.com/gatsbyjs/gatsby/tree/master/examples/recipe-webpack-image)
- [More on all image techniques in Gatsby](/docs/images-and-files/)

### Reference an image from the `static` folder

As an alternative to importing assets with webpack, the `static` folder allows access to content that gets automatically copied into the `public` folder when built.

This is an **escape route** for [specific use cases](/docs/static-folder/#when-to-use-the-static-folder), and other methods like [importing with webpack](#import-an-image-into-a-component-with-webpack) are recommended to leverage optimizations made by Gatsby.

#### Prerequisites

- A [Gatsby Site](/docs/quick-start) with a `.js` file exporting a React component
- An image (`.jpg`, `.png`, `.gif`, `.svg`, etc.) in the `static` folder

#### Directions

1. Ensure that the image is in your `static` folder at the root of the project. Your project structure might look something like this:

```
├── gatsby-config.js
├── src
│   └── pages
│       └── index.js
├── static
│       └── fiesta.jpg
```

2. In `index.js`, add an `<img>` tag with the `src` as the relative path of the file from the `static` folder, and add an `alt` attribute [describing the image](https://webaim.org/techniques/alttext/):

```jsx:title=src/pages/index.js
import React from "react"

export default () => (
  <img src={`fiesta.jpg`} alt="A dog smiling in a party hat" /> // highlight-line
)
```

3. Run `gatsby develop` to start the development server.
4. View your image in the browser: `http://localhost:8000/`

#### Additional resources

- [Example repo referencing an image from the static folder](https://github.com/gatsbyjs/gatsby/tree/master/examples/recipe-static-image)
- [Using the Static Folder](/docs/static-folder/)
- [More on all image techniques in Gatsby](/docs/images-and-files/)

## 8. Transforming data

Transforming data in Gatsby is plugin-driven. Transformer plugins take data fetched using source plugins, and process it into something more usable (e.g. JSON into JavaScript objects, and more). `gatsby-transformer-plugin` can transform Markdown files to HTML.

### Prerequisites

- A Gatsby site with `gatsby-config.js` and an `index.js` page
- A Markdown file saved in your Gatsby site `src` directory
- A source plugin installed, such as `gatsby-source-filesystem`
- The `gatsby-transformer-remark` plugin installed

### Directions

1. Add the transformer plugin in your `gatsby-config.js`:

```js:title=gatsby-config.js
plugins: [
  // not shown: gatsby-source-filesystem for creating nodes to transform
  `gatsby-transformer-remark`
],
```

2. Add a GraphQL query to the `index.js` file of your Gatsby site to fetch `MarkdownRemark` nodes:

```jsx:title=src/pages/index.js
export const query = graphql`
  query {
    allMarkdownRemark {
      totalCount
      edges {
        node {
          id
          frontmatter {
            title
            date(formatString: "DD MMMM, YYYY")
          }
          excerpt
        }
      }
    }
  }
`
```

3. Restart the development server and open GraphiQL at `http://localhost:8000/___graphql`. Explore the fields available on the `MarkdownRemark` node.

### Additional resources

- [Tutorial on transforming Markdown to HTML](/tutorial/part-six/#transformer-plugins) using `gatsby-transformer-remark`
- Browse available transformer plugins in the [Gatsby plugin library](/plugins/?=transformer)

## 9. Deploying your site

Showtime. Once you are happy with your site, you are ready to go live with it!

### Preparing for deployment

#### Prerequisites

- A [Gatsby site](/docs/quick-start)
- The [Gatsby CLI](/docs/gatsby-cli) installed

#### Directions

1. Stop your development server if it is running (`Ctrl + C` on your command line in most cases)

2. For the standard site path at the root directory (`/`), run `gatsby build` using the Gatsby CLI on the command line. The built files will now be in the `public` folder.

```shell
gatsby build
```

3. To include a site path other than `/` (such as `/site-name/`), set a path prefix by adding the following to your `gatsby-config.js` and replacing `yourpathprefix` with your desired path prefix:

```js:title=gatsby-config.js
module.exports = {
  pathPrefix: `/yourpathprefix`,
}
```

There are a few reasons to do this--for instance, hosting a blog built with Gatsby on a domain with another site not built on Gatsby. The main site would direct to `example.com`, and the Gatsby site with a path prefix could live at `example.com/blog`.

4. With a path prefix set in `gatsby-config.js`, run `gatsby build` with the `--prefix-paths` flag to automatically add the prefix to the beginning of all Gatsby site URLs and `<Link>` tags.

```shell
gatsby build --prefix-paths
```

5. Make sure that your site looks the same when running `gatsby build` as with `gatsby develop`. By running `gatsby serve` when you build your site, you can test out (and debug if necessary) the finished product before deploying it live.

```shell
gatsby build && gatsby serve
```

#### Additional Resources

- Walk through building and deploying an example site in [tutorial part one](/tutorial/part-one/#deploying-a-gatsby-site)
- Learn about [performance optimization](/docs/performance/)
- Read about [other deployment related topics](/docs/preparing-for-deployment/)
- Check out the [deployment docs](/docs/deploying-and-hosting/) for specific hosting platforms and how to deploy to them
