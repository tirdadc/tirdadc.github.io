---
title: Migrating from react-rails to Webpacker
date: 2017-03-07 08:39 EST
tags: Ruby On Rails, React.js, react-rails, Webpacker
---

As an early React adopter, [react-rails](https://github.com/reactjs/react-rails) was the obvious choice for integrating it into Ruby on Rails, and despite countless Medium posts over the years about the need to switch to [Webpack](https://webpack.github.io/) right this minute using some [Rube Goldberg machine](https://www.google.com/search?q=rube+goldberg+machine&tbm=isch) of a setup, the additional overhead and effort rarely felt warranted on existing projects. I also had little interest in fully discarding Sprockets, because it simply works incredibly well for most things.

This did present some challenges every now and then when trying to integrate external React components, as they tend to be written with the larger general JavaScript ecosystem in mind (Webpack, Browserify, etc...) and there wasn't always a way to include them with **react-rails** without some rewrites.

With [Rails 5.1 around the corner](http://weblog.rubyonrails.org/2017/2/23/Rails-5-1-beta1/) and the [announcement that it will have native support for both Webpack and Yarn](https://twitter.com/dhh/status/808348184481124352?lang=en) thanks to [Webpacker](https://github.com/rails/webpacker), it finally makes sense to move things over and to also start using Webpack to manage JS dependencies without losing the benefits of Sprockets for any existing stuff.

Here's a recap of what the steps are for this process:

Add the following gems to your `Gemfile` and run `bundle install`.

```diff
+gem 'webpacker', github: 'rails/webpacker'
+gem 'webpacker-react', "0.1.0"
```

Run this Webpacker command to easily add React:

```
rails webpacker:install:react
```

In development mode, you will now need to have the Webpack watcher running in a separate process to recompile your JS/JSX on the fly.

```
./bin/webpack-watcher
```

Configure the `.babelrc` file if needed (I added stage 0 support):

```json
{
  "presets": ["env", "react", "stage-0"]
}
```

Remove your **react-rails** related `require` statements from `app/assets/javascripts/application.js` and delete `app/assets/javascripts/components.js`. While you're at it, clean up your `config/environments` files to remove the config settings for **react-rails**.

Move your React components to `app/javascript/components/`.

I didn't feel like renaming the `.js.jsx` extension so I made sure that Webpack would still parse them by editing `config/webpack/shared.js`:

```js
...

resolve: {
  extensions: ['.js', '.jsx', '.js.jsx', '.coffee'],
  modules: [
    path.resolve('app/javascript'),
    path.resolve('node_modules')
  ]
}

...
```

Rewrite your components to include proper `import` and `export` declarations:

```diff
+import React from 'react'
class TextField extends React.Component {
  // your component code
}
+export default TextField;
```

Keep in mind that you will have to `import` child components that you're using in any given component.

If you're using the [immutability helpers](https://facebook.github.io/react/docs/update.html) anywhere, be sure to add:

```diff
+import update from 'react-addons-update'
```

I would recommend reading [Webpacker's README](https://github.com/rails/webpacker) at this point to familiarize yourself with the notion of packs and how to include them in Rails. You will have to add a line to include the appropriate root component in your layout view:

```diff
<%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
<%= javascript_include_tag 'vendor', 'data-turbolinks-track': 'reload' %>
<%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
+<%= javascript_pack_tag 'my_pack' %>
```

This is what my pack file looks like:

```js
import RootComponent from 'components/test/root_component';
import WebpackerReact from 'webpacker-react';

WebpackerReact.register(RootComponent);
```

Using [Webpacker::React](https://github.com/renchap/webpacker-react) allows me to still have a `react_component` view helper, although there's no support for pre-rendering yet. At some point I may just move to using the regular DOM mounting and skip this helper altogether.

If you do plan to use it, you will need to edit `config/webpack/production.js` to not mangle your registered components [until version 0.2 ships](https://github.com/renchap/webpacker-react/issues/23) during the compilation step:

```js
new webpack.optimize.UglifyJsPlugin({
  mangle: {
    except: ['RootComponent']
  }
}),
```

### Production

You will need to make sure [Yarn](https://yarnpkg.com/en/) is installed on your production environment:

```
curl https://dl.yarnpkg.com/rpm/yarn.repo -o /etc/yum.repos.d/yarn.repo
yum install -y yarn
```

I also had to add this at the top of my `config/webpack/shared.js`:

```js
'use strict';
```

You'll also need to install your packages:

```
yarn install --production
```

Don’t forget to add a final compilation step to your production deployment script:

```
RAILS_ENV=production rails webpacker:compile
```
