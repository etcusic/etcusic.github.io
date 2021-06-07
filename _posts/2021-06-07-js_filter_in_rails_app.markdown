---
layout: post
title:      "JS Filter in Rails App"
date:       2021-06-07 16:23:04 +0000
permalink:  js_filter_in_rails_app
---


**Context:**
In my last blog post I created a search filter for my Rails application by creating a class and controller to handle querries, and then rendering to the page. In this blog, I'm going to explore doing this with JavaScript instead. It's perhaps not as elegant, but it doesn't require going back and forth through the MVC structure. So, let's get straight into it:

**The View:**
The view is pretty straight forward - it requires select tags with options and values, and then I set up a table that renders the data for the decks. A couple things to note: 1) I altered the "admin-approved" value to 'Yes' and 'No' rather than `true` and `false` for the sake of making it more friendly to the user. 2) I used `data-user-id` for the user info to compare against the select tag. Here is what my view looks like:
```
<label>Admin Approved: </label>
<select id="admin-select">
    <option value="All">All</option>
    <option value="Yes">Approved</option>
    <option value="No">Not Approved</option>
</select>
<br>
<label>Level: </label>
<select id="level-select">
    <option value="All">All</option>
    <option value="1">1</option>
    <option value="2">2</option>
    <option value="3">3</option>
    <option value="4">4</option>
    <option value="5">5</option>
    <option value="6">6</option>
    <option value="7">7</option>
</select>
<br>
<label>User: </label>
<select id="user-select">
    <option value="All">All</option>
    <% User.all.map do |user| %>
        <option value=<%= user.id %>><%= user.name %></option>
    <% end %>
</select>
<br><br>
<button id="search-button">Submit Search</button>
<br><br><br>
<table>
    <thead>
        <tr>
            <td>Deck Title: </td>
            <td>Approved: </td>
            <td>Level: </td>
            <td>Creator: </td>
        </tr>
    </thead>

    <tbody>
        <% @deck_search.decks.map do |deck| %>
            <tr class="deck-row" id=<%= "deck-#{deck.id}" %>>
                <td class="deck-name"><%= link_to deck.name, deck_path(deck) %></td>
                <td class="deck-admin"><%= deck.admin_approved ? "Yes" : "No" %></td>
                <td class="deck-level"><%= deck.level %></td>
                <td class="deck-user" data-user-id=<%= deck.user.id %>><%= deck.user.name %></td>
            </tr>
        <% end %>
    </tbody>
</table>
```
**Implementing JS:**
First, let's go to the directory `/app/javascript/packs` and create a file named `deck_search.js` and then in our file `application.js` in the same directory we'll import that file with `import './deck_search.js'`. Now we're ready to start writing out our JavaScript function. I begin by attaching an event listener on the search button, and then I create variables representing the select tags, which provide the values needed for the search. Then I use a query selector to get all of the table rows, which represent the decks that we'll need to check to see if they pass the search criteria. Here is what that start looks like:
```
//=>  /app/javascript/packs/deck_search.js

document.getElementById("search-button").addEventListener('click', () => {
    const admin = document.getElementById("admin-select").value
    const level = document.getElementById("level-select").value
    const user = document.getElementById("user-select").value
    const deckRows = document.querySelectorAll(".deck-row")

})
```
**Filter function:**
Now that we've got our variables to work with, let's figure out how to filter out what we want. For starters, I'll attach a `forEach()` method on the deckRows variable to iterate through each `<tr>` tag. Within that method, we need to set up a variable for each relevant field - admin approved, level, and user. With those variables, we can now match them against our select tag variables. What I chose to do was to set up an array with the select tag variable and the td tag variable, and pass it into a function. The function will then check to see if the slecect tag `=== "All"` or if the select value and td value equal each other. If all of those in the `<tr>` tag come back true, then we render the row, and if just one comes back false, then `tag.hidden = true`. This way, the tag remains there to be iterated over again if there is another search, but is only rendered if it matches the search criteria. Here is what our JS code looks like now:
```
document.getElementById("search-button").addEventListener('click', () => {
    const admin = document.getElementById("admin-select").value
    const level = document.getElementById("level-select").value
    const user = document.getElementById("user-select").value
    const deckRows = document.querySelectorAll(".deck-row")

    if (![admin, level, user].every(value => value === "All")){
        // filter through rows
        // check if value does not equal select value and 
        deckRows.forEach(deckRow => {
            let tdAdmin = deckRow.querySelector(".deck-admin").innerHTML
            let tdLevel = deckRow.querySelector(".deck-level").innerHTML
            let tdUser = deckRow.querySelector(".deck-user").dataset.userId
            let nestedArray = [[admin, tdAdmin], [level, tdLevel], [user, tdUser]]
            nestedArray.every(arr => checkValues(arr)) ? deckRow.hidden = false : deckRow.hidden = true
        })
    }
})

function checkValues(arr){
    if (arr[0] === "All" || arr[0] === arr[1]){
        return true
    } else {
        return false
    }
}
```
And there you have it. Lots of iterating, but the code itself is rather concise.
