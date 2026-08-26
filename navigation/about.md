---
layout: post
title: About
permalink: /about/
comments: true
---

# As a Conversation Starter

Here are a few places I call home or have traveled to!

<comment>
# As a Conversation Starter
</comment>

<style>
    /* Style looks pretty compact, 
       - grid-container and grid-item are referenced the code 
    */
    .grid-container {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(150px, 1fr)); /* Dynamic columns */
        gap: 10px;
    }
    .grid-item {
        text-align: center;
    }
    .grid-item img {
        width: 100%;
        height: 100px; /* Fixed height for uniformity */
        object-fit: contain; /* Ensure the image fits within the fixed height */
    }
    .grid-item p {
        margin: 5px 0; /* Add some margin for spacing */
    }

    .image-gallery {
        display: flex;
        flex-wrap: nowrap;
        overflow-x: auto;
        gap: 10px;
        }

    .image-gallery img {
        max-height: 150px;
        object-fit: cover;
        border-radius: 5px;
    }
</style>

<!-- This grid_container class is used by CSS styling and the id is used by JavaScript connection -->
<div class="grid-container" id="grid_container">
    <!-- content will be added here by JavaScript -->
</div>

<script>
    // 1. Make a connection to the HTML container defined in the HTML div
    var container = document.getElementById("grid_container"); // This container connects to the HTML div

    // 2. Define a JavaScript object for our http source and our data rows for the Living in the World grid
    var http_source = "https://upload.wikimedia.org/wikipedia/commons/";
    var living_in_the_world = [
        {"flag": "0/01/Flag_of_California.svg", "greeting": "San Diego", "description": "California (Home)"},
        {"flag": "6/65/Flag_of_Belgium.svg", "greeting": "Leuven", "description": "Belgium"},
        {"flag": "0/03/Flag_of_Italy.svg", "greeting": "Cinque Terre", "description": "Italy"},
        {"flag": "5/5c/Flag_of_Portugal.svg", "greeting": "Albufeira", "description": "Portugal"},
        {"flag": "f/f3/Flag_of_Switzerland.svg", "greeting": "Zurich", "description": "Switzerland"},
        {"flag": "b/ba/Flag_of_Germany.svg", "greeting": "Berlin", "description": "Germany"},
        {"flag": "c/c3/Flag_of_France.svg", "greeting": "Paris", "description": "France"},
        {"flag": "d/da/Flag_of_Luxembourg.svg", "greeting": "Luxemborg City", "description": "Luxembourg"},
        {"flag": "2/20/Flag_of_the_Netherlands.svg", "greeting": "Amsterdam", "description": "Netherlands"},
        {"flag": "4/41/Flag_of_India.svg", "greeting": "Hyderabad", "description": "India"}
    ];

    // 3a. Consider how to update style count for size of container
    // The grid-template-columns has been defined as dynamic with auto-fill and minmax

    // 3b. Build grid items inside of our container for each row of data
    for (const location of living_in_the_world) {
        // Create a "div" with "class grid-item" for each row
        var gridItem = document.createElement("div");
        gridItem.className = "grid-item";  // This class name connects the gridItem to the CSS style elements
        // Add "img" HTML tag for the flag
        var img = document.createElement("img");
        img.src = http_source + location.flag; // concatenate the source and flag
        img.alt = location.flag + " Flag"; // add alt text for accessibility

        // Add "p" HTML tag for the description
        var description = document.createElement("p");
        description.textContent = location.description; // extract the description

        // Add "p" HTML tag for the greeting
        var greeting = document.createElement("p");
        greeting.textContent = location.greeting;  // extract the greeting

        // Append img and p HTML tags to the grid item DIV
        gridItem.appendChild(img);
        gridItem.appendChild(description);
        gridItem.appendChild(greeting);

        // Append the grid item DIV to the container DIV
        container.appendChild(gridItem);
    }
</script>

### Journey through Life

Here is a look at what I do and where I study:

-  **Middle School:** Oak Valley Middle School 
-  **High School:** Del Norte High School 
-  **Activities:**  Martial Arts training and guitar performing
### Hobbies & Interests

- Playing guitar.
- Listening to music.
- Reading novels and literature.

<comment>
Gallery of Pics, scroll to the right for more ...
</comment>
<div class="image-gallery">
  <img src="{{site.baseurl}}/images/about/1.jpeg" alt="Image 1">
  <img src="{{site.baseurl}}/images/about/2.jpeg" alt="Image 2">
  <img src="{{site.baseurl}}/images/about/3.jpeg" alt="Image 3">
  <img src="{{site.baseurl}}/images/about/4.jpeg" alt="Image 4">
  <img src="{{site.baseurl}}/images/about/5.jpeg" alt="Image 5">
  <img src="{{site.baseurl}}/images/about/6.jpeg" alt="Image 6">
  <img src="{{site.baseurl}}/images/about/7.jpeg" alt="Image 7">
  <img src="{{site.baseurl}}/images/about/8.jpeg" alt="Image 8">
  <img src="{{site.baseurl}}/images/about/9.jpeg" alt="Image 9">
  <img src="{{site.baseurl}}/images/about/10.jpeg" alt="Image 10">
  <img src="{{site.baseurl}}/images/about/11.jpeg" alt="Image 11">
  <img src="{{site.baseurl}}/images/about/12.jpeg" alt="Image 12">
</div>
