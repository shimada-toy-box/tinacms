# react-tinacms-editor

This package provides a WYSIWYM Editor for HTML and Markdown. This editor can be used as a [Field Plugin](https://tinacms.org/docs/plugins/fields) or as an [Inline Field](https://tinacms.org/docs/ui/inline-editing) in websites built with [TinaCMS](https://tinacms.org).

> Note: The `react-tinacms-editor` package is quite large. Whether you're using the [Field Plugin](https://tinacms.org/docs/plugins/fields) or the [Inline Field](#example-dynamic-imports) it is recommended that you use dynamic imports to reduce your JS bundle size.

## Install

```
yarn add react-tinacms-editor
```

## Field Plugins

This package provides two field plugins for TinaCMS: `MarkdownFieldPlugin` and `HtmlFieldPlugin`.

### Registering the Plugins
This is the simplest approach to registering plugins:

```js
import { MarkdownFieldPlugin, HtmlFieldPlugin } from 'react-tinacms-editor'

cms.plugins.add(MarkdownFieldPlugin)
cms.plugins.add(HtmlFieldplugin)
```

The `react-tinacms-editor` is a large package so it is recommended that [load the plugins dynamicallyj](https://tinacms.org/docs/plugins/fields):

```js
import("react-tinacms-editor").then(
  ({ MarkdownFieldPlugin, HtmlFieldPlugin }) => {
    cms.plugins.add(MarkdownFieldPlugin)
    cms.plugins.add(HtmlFieldplugin)
  }
)
```



### Field Plugin Options

```typescript
interface Config {
  component: 'markdown' | 'html'
  name: string
  label?: string
  description?: string
}
```

| Option        | Description                                                                                     |
| ------------- | ----------------------------------------------------------------------------------------------- |
| `component`   | The name of the plugin component. Either `'markdown'` or `'html'`.                                          |
| `name`        | The path to some value in the data being edited.                                                |
| `label`       | A human readable label for the field. Defaults to the `name`. _(Optional)_                      |
| `description` | Description that expands on the purpose of the field or prompts a specific action. _(Optional)_ |

### Example: Using Field Plugins in Forms

Once registered you will be able to use the plugins in your [Forms](https://tinacms.org/docs/forms):

```js
const formConfig = {
  fields: [
    {
      name: "description",
      label: "Description",
      component: "html",
    },
    {
      name: "body",
      label: "Blog Body",
      component: "markdown",
    }
  ]
}
```

These will both show up in your sidebar looking roughly like this:

![tinacms-markdown-field](https://tinacms.org/img/fields/markdown.png)


## InlineWysiwyg

The `InlineWysiwyg` is a React [inline editing component](https://tinacms.org/docs/ui/inline-editing) for Markdown and HTML.


### InlineWysiwyg Interface

```typescript
interface InlineWysiwygConfig {
  name: string
  children: any
  sticky?: boolean
  format?: 'markdown' | 'html'
  imageProps?: WysiwysImageProps
}

interface WysiwygImageProps {
  upload?: (files: File[]) => Promise<string[]>
  previewUrl?: (url: string) => string | Promise<string>
}
```

| Key           | Description                                                                                  |
| ------------- | -------------------------------------------------------------------------------------------- |
| `name`        | The path to some value in the data being edited.                                             |
| `children`    | Child components to render.                                                                  |
| `sticky?`     | A boolean determining whether the Wysiwyg Toolbar 'sticks' to the top of the page on scroll. |
| `format?`     | This value denotes whether Markdown or HTML will be rendered.                                |
| `imageProps?` | Configures how images in the Wysiwyg are uploaded and rendered.

### Example: Basic Usage

Below is an example of how an `InlineWysiwyg` field could be defined in an [Inline Form](/docs/ui/inline-editing).

```jsx
import ReactMarkdown from 'react-markdown'
import { useForm, usePlugin } from 'tinacms'
import { InlineForm } from 'react-tinacms-inline'
import { InlineWysiwyg } from 'react-tinacms-editor'

// Example 'Page' Component
export function Page(props) {
  const [data, form] = useForm(props.data)
  usePlugin(form)
  return (
    <InlineForm form={form}>
      <InlineWysiwyg name="markdownBody" format="markdown">
        <ReactMarkdown source={data.markdownBody} />
      </InlineWysiwyg>
    </InlineForm>
  )
}
```

### Example: Dynamic Imports

The `react-tinacms-editor` is a large package so it is recommended that you make sure it's only being loaded when necessary. The example below will make sure that the editor is only loaded _if_ the CMS is actually enabled, saving the visistors to your website from the extra load time.

**your-app/src/components/InlineWysiwyg.js**
```jsx
import React from 'react'
import { useCMS } from 'tinacms'

export function InlineWysiwyg(props) {
  const cms = useCMS()
  const [{ InlineWysiwyg }, setEditor] = React.useState({})

  React.useEffect(() => {
    if (!InlineWysiwyg && cms.enabled) {
      import('react-tinacms-editor').then(setEditor)
    }
  }, [cms.enabled])

  if (InlineWysiwyg) {
    return (
      <InlineWysiwyg {...props}/>
    )
  }

  return props.children
}
```

> #### Why do I have to load the editor dynamically myself?
> Code splitting and dynamic imports are handled by the website's JavaScript bundlers (e.g. rollup, webpack, etc.). Since the package does not load itself into the application, it is unfortunately not possible to provide this behaviour in the package itself.