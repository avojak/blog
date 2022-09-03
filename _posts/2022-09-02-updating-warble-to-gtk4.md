---
title:  "Updating Warble to GTK4"
description: "How I updated Warble to GTK4, and a few of the issues I encountered"
author: avojak
image: https://i.imgur.com/2e2fEwZ.png
tags:
  - software
  - app
  - elementary-os
  - evergreen
---

Warble is officially at version 2.0.0, which means it's now using GTK4!

{% include twitter-card.html
  name="Andrew Vojak"
  account="avojak"
  avatar="/images/avojak.jpeg"
  id="1565773075336966144"
  timestamp="2022-09-02"
  contents="Warble v2.0.0 is here! 🥳<br>
<br>
✨Updated to GTK4<br>
✨Cleaner UI<br>
✨New (barely modified) icon<br>
✨Same fun gameplay!<br>
<br>
Get the update soon on 
@elementary
 #AppCenter, or on #Flathub!<br>
<br>
#vala #gtk #gtk4"
%}

It was a fun challenge to update Warble from GTK3 to GTK4, although a bit frustrating at times. I hope that by walking through some of the specific changes I made, you might not struggle quite as much as I did! Wherever possible I will include links to the diff in GitHub so you can see the exact change that I made.

You can view the full diff of changes here: [avojak/warble/pull/39](https://github.com/avojak/warble/pull/39/files)

## Updating the Dependencies

The obvious first step is to, well, pull in GTK4 as a dependency!

    dependency('gtk4', version: '>= 4.6.3')

And if you're using an IDE, you'll immediately be met with a giant slap in the face. Compile errors out the wazoo! If you're like me, you had no idea that Libhandy was going away (and I was just getting used to it...). Turns out, [Libadwaita is the successor](https://gnome.pages.gitlab.gnome.org/libadwaita/doc/main/migrating-libhandy-1-4-to-libadwaita.html). Not to worry, this is an easy substitution!

    dependency('libadwaita-1', version: '>= 1.1.0'),
    # I'm using Granite as well, so you need to grab the new version of it as well!
    dependency('granite-7', version: '>= 7.0.0')

Now all you have to do is quickly rush through your code, take a quick guess at how to fix the compile errors, rebuild, run, and...

{% include twitter-card.html
  name="Andrew Vojak"
  account="avojak"
  avatar="/images/avojak.jpeg"
  id="1555739691760197632"
  timestamp="2022-08-05"
  contents="Me: *Updates Warble to GTK4 and spends a couple hours resolving all the compile and runtime errors*<br><br>Warble: 🙃"
  image="https://pbs.twimg.com/media/FZcZAQ5XkAEKDo6?format=jpg&name=small"
%}

Welp. This was when I realized that it wasn't going to be a quick update!

## Helpful Resources

I'm going to pause briefly here to list some resources that I found helpful along the way:

- [GTK - Migrating 3 to 4](https://docs.gtk.org/gtk4/migrating-3to4.html) - This guide was very helpful at identifying specific API changes, and the replacements. When in doubt, `Ctrl+f` on this page first!
- [GTK Inspector](https://wiki.gnome.org/Projects/GTK/Inspector) - The GTK inspector application is incredibly useful in general, but specifically when you're making UI changes. If you're using Flatpak, simply add `--env=GTK_DEBUG=interactive` to your `flatpak run` command and the Inspector will open alongside your app!
- [Valadoc.org](https://valadoc.org/) - Another probably obvious resource, but nothing beats reading the docs.

## Input Handling

There are two main input methods in Warble: using the physical keyboard attached to the computer, and the on-screen keyboard.

### Keyboard Input

With GTK3 and Libhandy, I used the `key_press_event` signal on the `Hdy.Window`, but GTK4 introduced a new [event controller system](https://docs.gtk.org/gtk4/migrating-3to4.html#stop-using-gtkwidget-event-signals).

```vala
var key_event_controller = new Gtk.EventControllerKey ();
key_event_controller.key_pressed.connect (on_key_pressed_event);
((Gtk.Widget) this).add_controller (key_event_controller);
```

A relatively small difference in this case, but worth noting.

### Mouse Input

Mouse input is more significantly different. Instead of needing to use widgets such as `EventBox`, and using `Gdk.EventMask` for the various event types you care about, we now use `Gtk.GestureClick` (or other similar controllers depending on the desired event type):

```vala
var gesture_event_controller = new Gtk.GestureClick ();
gesture_event_controller.pressed.connect ((n_press, x, y) => {
    if (n_press != 1) {
        return;
    }
    get_style_context ().add_class ("key-pressed");
    clicked (letter);
});
gesture_event_controller.released.connect ((n_press, x, y) => {
    if (n_press != 1) {
        return;
    }
    get_style_context ().remove_class ("key-pressed");
});
this.add_controller (gesture_event_controller);
```

([Diff](https://github.com/avojak/warble/pull/39/files#diff-e9ec1e63eb9175d660c1b054f23cc7a3a59ebaace77d5190a379ab490cebd79dR35-R38))

## Final Classes

One error you will most certainly encounter is `field 'parent_instance' has incomplete type`. This is because many Gtk classes are now final and [cannot be subclassed](https://docs.gtk.org/gtk4/migrating-3to4.html#subclassing). I ran into this because I had subclassed `Gtk.HeaderBar`. In my case I simply created the header bar directly in a different class, but in other cases I had to use a different container widget.

## Drawing with Cairo

Under GTK3, the method for doing custom drawing on a widget was to override the `draw(Cairo.Context)` method:

```vala
protected override bool draw (Cairo.Context ctx) {
    base.draw (ctx); // Draw the widget as normal
    ctx.save ();
    custom_drawing (ctx); // Do some custom drawing after the base widget is drawn
    ctx.restore ();
    return false;
}
```

For GTK4, there's a different approach that I can use. To draw the squares on the game board with letters, the squares are now `Gtk.DrawingArea` widgets. This class exposes a `set_draw_func(Gtk.DrawingArea da, Cairo.Context ctx, int width, int height)` method with some new convenience parameters for width and height.

```vala
private void draw_func (Gtk.DrawingArea drawing_area, Cairo.Context ctx, int width, int height) {
    var color = Gdk.RGBA ();
    color.parse (Warble.ColorPalette.get_text_color (state));
    ctx.set_source_rgb (color.red, color.green, color.blue);

    ctx.select_font_face ("Inter", Cairo.FontSlant.NORMAL, Cairo.FontWeight.BOLD);
    ctx.set_font_size (FONT_SIZE);

    Cairo.TextExtents extents;
    ctx.text_extents (letter.to_string (), out extents);
    double x = (width / 2) - (extents.width / 2 + extents.x_bearing);
    double y = (height / 2) - (extents.height / 2 + extents.y_bearing);
    ctx.move_to (x, y);
    ctx.show_text (letter.to_string ());
}
```


([Diff](https://github.com/avojak/warble/pull/39/files#diff-e9ec1e63eb9175d660c1b054f23cc7a3a59ebaace77d5190a379ab490cebd79d))

## Window Positioning

Gone are the days of saving the position of the application window to the GSettings - now the window manager handles that for you!

([Diff](https://github.com/avojak/warble/pull/39/files#diff-9af23ba992f6eda45848cc8442543801a8c1bae99e960f6ac02b44887695f3df))

## CSS Blur

This isn't a change, but rather a new feature in GTK4 that I love. You can now use a `blur()` filter. In Warble, I combine blur with transparency on certain overlay views as a nice replacement for using a dialog:

    .faded {
        transition-property: opacity, filter;
        transition-duration: 0.5s;
        filter: blur(5px);
        opacity: 0.3;
    }

I'm a big fan of how this turned out:

<figure class="third" markdown=1>
![]() <!-- placeholder to force the real picture to be in the middle and zoom properly -->
![Warble screenshot with blur](https://raw.githubusercontent.com/avojak/warble/2.0.0/data/assets/screenshots/warble-screenshot-03.png)
![]() <!-- placeholder to force the real picture to be in the middle and zoom properly -->
</figure>

## Granite `ModeButton` No More

Not necessarily GTK4 related, but the `ModeButton` widget was [removed from Granite](https://github.com/elementary/granite/issues/619). This is really a "shame on me" moment for using this widget incorrectly, but it might pose a "gotcha" to anyone else using it in a similar way.

The replacement is to use grouped `Gtk.CheckButton` widgets. For Warble, I created a simple wrapper widget ([`CheckButtonGroup`](https://github.com/avojak/warble/pull/39/files#diff-4e8d6a9aecab109cdceab4ad2f0d06868a628c24747f8bf1fecfe58f6c4a7cbb)) to handle the grouping.

I think the GTK method of grouping widgets in this case is rather clumsy. One common theme in GTK4 is the use of proper container widgets, yet here we create a "group" by assigning a sibling widget to the `group` property. Huh?? Now you end up with a widget that's pulling double-duty as both an individual button, but also the group. In my opinion, a better design would be to create a specific group container widget that you add buttons to.

## CSS Changes

While not directly related to GTK4, I took this opportunity to leverage CSS for styling the game area and on-screen keyboard instead of using custom icons for each possible color scenario. 

The icons for the squares were replaced with two lines of code to apply some CSS classes:

```vala
get_style_context ().add_class (Granite.STYLE_CLASS_CARD);
get_style_context ().add_class (Granite.STYLE_CLASS_ROUNDED);
```

And the various states for the squares and keyboard keys were also represented with CSS classes:

```css
.guess-correct {
    background-color: @LIME_300;
    border: 1px solid @LIME_700;
}

.guess-incorrect {
    background-color: @SILVER_500;
    border: 1px solid @SILVER_900;
}

.guess-close {
    background-color: @BANANA_300;
    border: 1px solid @BANANA_700;
}

.key-pressed {
    transform: translateY(2px);
}

.tile-active {
    border: 1px solid @BLACK_700;
}
```

When switching to high-contrast mode, all I had to do was swap out the stylesheet - no more changing ALL the icons!

```vala
private void load_theme_stylesheet () {
    var gtk_settings = Gtk.Settings.get_default ();
    bool dark_mode = gtk_settings.gtk_application_prefer_dark_theme;
    bool high_contrast_mode = settings.get_boolean ("high-contrast-mode");

    if (dark_mode && high_contrast_mode) {
        theme_provider.load_from_resource (Constants.APP_ID.replace (".", "/") + "/warble-dark-hicontrast.css");
    } else if (dark_mode && !high_contrast_mode) {
        theme_provider.load_from_resource (Constants.APP_ID.replace (".", "/") + "/warble-dark.css");
    } else if (!dark_mode && high_contrast_mode) {
        theme_provider.load_from_resource (Constants.APP_ID.replace (".", "/") + "/warble-light-hicontrast.css");
    } else {
        theme_provider.load_from_resource (Constants.APP_ID.replace (".", "/") + "/warble-light.css");
    }
}
```

As an added bonus, I was able to very easily add a subtle outline for the square on the game board that's currently waiting for input. Previously this would've required a whole new icon.

## End Result

Check out the end result over on GitHub!

{% include github-card.html
  user="avojak"
  repository="warble"
%}