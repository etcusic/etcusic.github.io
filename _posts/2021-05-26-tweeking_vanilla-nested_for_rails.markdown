---
layout: post
title:      "Tweeking Vanilla-Nested for Rails"
date:       2021-05-26 15:13:56 -0400
permalink:  tweeking_vanilla-nested_for_rails
---


The project I'm working on requires a nested form where a Deck has many Cards, and I wanted the user to be able to add and/or remove as many cards as they want. I was doing some research on how to accomplish this, and it seemed like finding a gem was the best way to go. I tried Cocoon, but I couldn't get it to work for some reason. I found a different gem: vanilla_nested, which seemed like to be more streamlined, so I gave it a shot instead. I ran into a little bug with it, but I was able to get into the code, tweak it, and then it worked great for me, so I'm going to go through how I got it to work.

(GitHub link for[ vanilla_nested gem](https://github.com/arielj/vanilla-nested))

First of course I added `gem vanilla_nested` to my gemfile, and then since I'm using webpacker, in the CLI I input `yarn add arielj/vanilla-nested`. Last thing to do for the setup was to add `import 'vanilla-nested'` in my app/javascript/packs/application.js file. Then, it was a matter of setting up my nested form, which I made as a partial to be used in both the new and edit files in the deck views. Here is what my form looks like:
```
# app/views/decks/_deck_form.html.erb

<%= form_with model: [current_user, @deck], local: true do |f| %>
    <%= f.label :name, "Name: " %>
    <%= f.text_field :name %>

    <%= f.label :level, "Level: " %>
    <%= f.number_field :level, in: 1..8, step: 1 %>

    <% if current_user.admin %>
        <%= f.label :admin_approved, "Aprroved: " %>
        <%= f.check_box :admin_approved %>
    <% end %>

    <br><br>

    <h3><strong>CARDS:</strong></h3>
    <table>
        <thead>
            <tr>
                <td>English:</td>
                <td>Spanish:</td>
            </tr>
        </thead>

        <tbody id="nested-cards-table" class="fields">
            <%= f.fields_for :cards do |ff| %>
                <%= render 'card_fields', ff: ff %>
            <% end %>
        </tbody>
    </table>
    
    <br>

    <button>
        <%= link_to_add_nested(f, :cards, '#nested-cards-table', partial_form_variable: :ff) %>
    </button>

    <br><br>
    
    <%= f.submit "Submit Flash Cards" %>
    
<% end %>
```
And then here is what my cards fields partial looks like:
```
# app/views/decks/_card_fields.html.erb

<tr class="card-row">
    <td>
        <%= ff.text_field :english %>
    </td>
    <td>
        <%= ff.text_field :spanish %>
    </td>
    <td>
        <button>
            <%= link_to_remove_nested(ff, fields_wrapper_selector: 'card-row', link_text: "remove") %>
        </button>
    </td>
</tr>
```
For details on the syntax needed to set up the `link_to_add_nested` and `link_to_remove_nested`, I recommend visiting the GitHub link I included above, which gives a great breakdown of how to use the gem. This blog, however, is going to focus on a bug that took a bit of troubleshooting to get through. The issue was that since my `link_to_remove_nested` tag was not the direct child of its `<tr class="card-row">` wrapper, which is the target that needs to be removed if clicked, and so when I clicked the link it would only remove the button. Face palm.

The `fields_wrapper_selector: 'card-row'` was supposed to take of this, but instead, I got this error in the console:
```
Uncaught TypeError: Cannot read property 'style' of null
    at hideWrapper (vanilla_nested.js:72)
    at HTMLAnchorElement../app/javascript/packs/vanilla_nested.js.window.removeVanillaNestedFields (vanilla_nested.js:62)
```
So, after returning to the vanilla_nested repo to figure out what to do, I ultimately decided to put the javascript file directly into my application. I created a vanilla_nested.js file with the code, and then added `import './vanilla_nested.js'` to my application.js file. Then, I started trouble shooting to see where the bug was. Ultimately, I found the issue in the block of code in lines 46-52. Here are those lines, with some console.logs mixed in:
```
let element = event.target;
if (!element.classList.contains('vanilla-nested-remove'))
  element = element.closest('.vanilla-nested-remove')
console.log(element)  // => <a> tag created by vanilla_nested gem
const data = element.dataset;
console.log(data)  // => DOMStringMap {fieldsWrapperSelector: "card-row", undoText: "Undo", undoLinkClasses: ""}
let wrapper = element.parentElement;
console.log(wrapper)  // => <button> element that the link is encased in
if (sel = data.fieldsWrapperSelector) wrapper = element.closest(sel);   // -- previous line => didn't account for class
console.log(wrapper)  // => null
```
As you can probably see, something happens on that last line where the wrapper should be altered to the `<tr class="card-row">` element, but instead produces `null`. After a bit of research and playing around with it, I realized that the `element.closest( )` method needs a period preceding the class name (data.fildsWrapperSelector => "card-row"). So all it took was altering that line to this:
```
if (sel = data.fieldsWrapperSelector) wrapper = element.closest(`.${sel}`);
```
And there you have it. Just interpolate the `sel` variable behind the dot and then the `element.closest( )` method works just fine.
