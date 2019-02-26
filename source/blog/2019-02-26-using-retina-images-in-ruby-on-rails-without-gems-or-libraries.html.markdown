---

title: Using Retina images in Ruby on Rails without gems or libraries
date: 2019-02-26 13:54 EST
tags: Ruby On Rails, Retina

---

A long time ago I set up [retina.js](https://github.com/strues/retinajs) on one of my sites to provide Retina quality image assets when needed to the relevant devices. Like many external JavaScript plugins, it didn't necessarily play nicely with Turbolinks and I later noticed erratic issues where certain assets were not being swapped out as expected.

Thankfully [browser support](https://caniuse.com/#feat=srcset) for the `srcset` image attribute has considerably improved and there's no real need for such a library or an additional gem to handle multiple variants of the same image asset and let the browser do the rest. The markup you're aiming for should look like this:

```html
<img
  alt="Nostromo"
  title="Nostromo"
  src="/system/releases/images/000/000/920/thumb/nostromo.jpg"
  srcset="/system/releases/images/000/000/920/thumb_2x/nostromo.jpg 2x"
  class="th" />
```

Rails 5.2.1 now provides [support](https://github.com/rails/rails/commit/43efae22a7a59d5fe0be599bd074c5e7d881266a#diff-d36c3bf0d0a61b0b5dbae953a4852ad2) for directly setting the srcset attribute through `image_tag`. The example below uses Paperclip, but the premise remains the same for Carrierwave:

```erb
<%= image_tag(
      item.image.url(:thumb),
      alt: "#{item.title}",
      title: "#{item.title}",
      class: "th",
      srcset: { item.image.url(:thumb_2x) => '2x' } ) %>
```

If you're on an older version of Rails, this will work:

```erb
<%= image_tag(
      item.image.url(:thumb),
      alt: "#{item.title}",
      title: "#{item.title}",
      class: "th",
      srcset: "#{item.image.url(:thumb_2x)} 2x" ) %>
```

Here's an example of the same image being provided with a 2x variant that shows how much of a difference this makes. You'll obviously need to be on a device that supports Retina display to see it.

<div style='text-align: center;'>
      <img src="/images/nostromo_1x.jpg" alt="Nostromo 1x" width='400px'>
      <img src="/images/nostromo_1x.jpg" srcset="/images/nostromo_2x.jpg" alt="Nostromo 2x" width='400px' />
</div>
