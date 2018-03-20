# An example MkDocs custom theme

The is a sample [MkDocs] project with a [custom_dir] defined to add a
lightbox feature to the default `mkdocs` theme for displaying images in a
lightbox using the [Lightbox for Boostrap Plugin][lb].

[MkDocs]: http://www.mkdocs.org/
[custom_dir]: http://www.mkdocs.org/user-guide/custom-themes/#creating-a-custom-theme
[lb]: http://ashleydw.github.io/lightbox/

To test this example, clone it and run `mkdocs serve` from the root directory.
Then click on the image on the homepage in your browser.

Note that this example requires at least **MkDocs version 0.17**. Earlier
versions do not support most of the mechanisms used here. See
[revision 915e1ac] of this guide if you're working with MkDocs 0.16.

[revision 915e1ac]: (https://github.com/waylan/mkdocs-theme_dir-example/tree/915e1acf4fc912f9371bcabfc9e095fec9bf00ec)

It should also be noted that this example is using the Lightbox for Bootstrap
Plugin as the `mkdocs` theme is built on Bootswatch, a Bootstrap wrapper. If you
are not using a Bootswatch or Bootstrap theme, then you will need to use a
different library for implementing lightboxes.

## The config file

First, MkDocs needs to be pointed to the custom theme directory. In this
example, the directory is unimaginatively named `custom_theme`. Therefore,
the `mkdocs.yml` config file defines its theme like this:

```yaml
theme:
  name: mkdocs
  custom_dir: custom_theme
```

Note that we need to explicitly define the theme which we are customizing,
(via `theme: mkdocs`) otherwise MkDocs assumes that the `theme_dir` is replacing
the entire theme.

# Adding media files

The CSS and JavaScript files have been copied from [version 4.0.2 of Lightbox
for Bootstrap](https://github.com/ashleydw/lightbox/tree/v4.0.2/dist). It's 
important to use version 4.0.2 as more recent versions require Bootstrap 4
(MkDocs uses Bootstrap version 3).

```
custom_theme/
    css/
        ekko-lightbox.min.css
    js/
        ekko-ligthbox.min.js
```

While those files technically do not *need* to go in the `css/` and `js/`
subdirectories, that is how files are organized in the parent `mkdocs` theme, so
we've done the same to keep things neat and tidy.

# Customizing the template

Finally, to tell MkDocs to use our media files we have the file
`custom_theme/main.html`. This file will replace/override the file of the same
name in the parent theme. Therefore, at the very least it needs to inherit from
the base template:

```django
{% extends "base.html" %}
```

Then, via [template inheritance], we define blocks to override the default
blocks defined in the parent theme. For example, we need the CSS file added
above to be listed in the `styles` block along with the other CSS files.
However, we don't want to remove the other stylesheet links defined in that
block. Nevertheless, the default behavior is for a block to replace the parent 
block of the same name. Therefore we have two options:

1. Redefine every link in the parent `styles` block.
2. Use a [super block] to have the parent `styles` included in our new block.

[template inheritance]: http://jinja.pocoo.org/docs/dev/templates/#template-inheritance
[super block]: http://jinja.pocoo.org/docs/dev/templates/#super-blocks

Obviously, the first option would require us to edit our custom block every time
the parent theme was updated. Therefore, we've used super blocks. In this case,
we want our new CSS styles to be defined after the existing styles, so we've
made the `super()` call first:

```django
{% block styles %}
    {{ super() }}
    <link rel="stylesheet" href="{{ base_url }}/css/ekko-lightbox.min.css">
{% endblock styles %}
```

And to ensure the URL is accurate for any page nested at any level in our site,
we have used the `base_url` template variable to ensure the stylesheet is always
properly linked to.

The JavaScript library we added above is included in the `lib` block in the same
manner.

```django
{% block libs %}
    {{ super() }}
    <script src="{{ base_url }}/js/ekko-lightbox.min.js"></script>
{% endblock libs %}
```


Finally, we need the JavaScript which runs on page load and defines a click
event for each image. This would go in the `scripts` block. Rather than defining
another JavaScript file and requiring the client to make another request
(perhaps using the `extra_javascript` config setting), we can just define it as
an inline script in the template.

```django
{% block scripts %}
    {{ super() }}
    <script>
        $(document).on('click', '[data-toggle="lightbox"]', function(event) {
            event.preventDefault();
            $(this).ekkoLightbox();
        });
    </script>
{% endblock scripts %}
```

Again, we've made sure to use `super` to avoid disabling any other scripts which
may be defined through the `extra_javascript` config setting.

# Wrapping it all up

The final piece of the puzzle is to have some image links which conform to the
Lightbox for Bootstrap format. As Markdown simply passes raw HTML through
unaltered, we've simple added an image link as raw HTML in the `docs/index.html`
file:

```html
<a href="https://unsplash.it/1200/768.jpg?image=250" data-toggle="lightbox" data-title="A random title" data-footer="A custom footer text">
    <img src="https://unsplash.it/300.jpg?image=250" class="img-fluid">
</a>
```

Perhaps a Markdown Extension could be created which could then be defined in the
config file (via the [markdown_extensions] config option) and some simple
Markdown image links would work as well. However, that is beyond the scope of
this example.

**GOTCHA** The lightbox plugin allows navigating through an image gallery by
left/right arrow keys. As this feature is turned on by default (see 
[plugin_options]) it's in conflict with MkDocs's own arrow key navigation. The
result is that if you hit the left or right arrow key when the gallery lightbox
is shown you will effectively navigate to the next MkDocs page!

[markdown_extensions]: http://www.mkdocs.org/user-guide/configuration/#markdown_extensions
[plugin_options]: https://ashleydw.github.io/lightbox/
