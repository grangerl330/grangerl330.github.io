---
layout: post
title:      "JavaScript Final Project"
date:       2019-05-20 14:34:54 +0000
permalink:  javascript_final_project
---

### And Learning About Event Delegation With JQuery


Today I completed my Javascript section final project. For this project I used my already existing Rails app 'Tournament Tracker' that I developed for the Rails section final project and added JavaScript to change how information was displayed on certain pages. 

The main goal behind adding the JavaScript was to be able to click a link, load information from the server, and display that information on the page without having to refresh the page or navigate to a new page. This is done using a set of techniques called "Asynchronous JavaScript and XML" or AJAX for short.

When the user clicks a link to get information, for example a link to "Show All Matches", we use JavaScript to make an ajax `GET`  request to the  server database that then returns the information we need in JSON (JavaScript Object Notation)  format. We can then parse that information and add it to the HTML of the page. To do this we use JQuery to select specifc elements on the page by their CSS selectors and either add or remove HTML from them based on how we want the page display to look. This is where event delegation comes into play.

At first when I was attempting to make the above process work, I was able to perform the proper ajax request and display the information about matches to the page as a list of all matches, with each match being a link that the user can click on to load the indivdual information about that match. However I ran into issues after adding a new match to the page. When a new match was added via an ajax `POST` request and the information about that match added to the display via JQuery, the link the the new match would not work. This is because the JavaScript file that contains the event listener functions is loaded when the page initially loads, and the event listeners are only able to be attached to elements that already exist on the page when the page is initially loaded. So when JavaScript is used to create a new match and a new match link is added to the page, the event listeners that are set to monitor match links are not bound to the newly created match. 

To solve this problem event delegation can be used. Event delegation is the process of binding the event listener to a parent element such as a`<div>` and providing it selectors for children within that `<div>` to be applied to. So if we bind an `on click`  listener to a parent `<div id="match-links">` and pass the listener a selector of `a`, the listener will be applied to any `a` tag within the `<div id="match-links>` even if those `a` tags are added by JavaScript after the initial page load. This allows for all links to function properly even if they are added by JavaScript
