---
layout: post 
title: Portfolio Home 
hide: true
show_reading_time: false
---

<style>
    /* Clean Grid Setup */
    .btn-grid { 
        display: grid; 
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); 
        gap: 15px; 
        margin-top: 25px; 
    }
    
    .rubric-btn {
        display: block;
        padding: 18px;
        text-decoration: none;
        border-radius: 8px;
        font-weight: 800; /* Maximum thickness for maximum clarity */
        text-align: center;
        transition: all 0.3s ease;
        backdrop-filter: blur(4px); /* Subtle background blur to keep it clean */
    }
    
    .rubric-btn:hover {
        opacity: 0.95;
        box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
        transform: scale(1.02);
    }
</style>

# Anika Seksaria | CS111 Portfolio

### 🏰 Course Projects
> Current development progress for CS111.

<div class="btn-grid">
    <!-- Semi-transparent dark sage with solid white text -->
    <a href="{{site.baseurl}}/games/red-riding/overview" class="rubric-btn" style="background-color: rgba(45, 76, 58, 0.25); border: 2px solid #2d4c3a; color: #ffffff;">🐺 Red Riding Hood</a>
    <a href="{{site.baseurl}}/cs111/2026/03/18/index.html" class="rubric-btn" style="background-color: rgba(61, 96, 73, 0.25); border: 2px solid #3d6049; color: #ffffff;">📝 Homework Lessons</a>
</div>

<br>

### 🎓 Required Rubric Evidence
> Click to explore the technical evidence required for CS111 college credit.

<div class="btn-grid">
    <!-- Points directly to the generated stable permalink output -->
    <a href="{{ '/projects/rubric/oop/' | relative_url }}" class="rubric-btn" style="background-color: rgba(30, 51, 42, 0.2); border: 2px solid #1e332a; color: #ffffff;">Object-Oriented Programming</a>
    <a href="{{ '/cs111/control/' | relative_url }}" class="rubric-btn" style="background-color: rgba(41, 66, 55, 0.2); border: 2px solid #294237; color: #ffffff;">Control Structures</a>
    <a href="{{ '/cs111/data/' | relative_url }}" class="rubric-btn" style="background-color: rgba(52, 82, 68, 0.2); border: 2px solid #345244; color: #ffffff;">Data Types</a>
    <a href="{{ '/cs111/operators/' | relative_url }}" class="rubric-btn" style="background-color: rgba(64, 99, 82, 0.2); border: 2px solid #406352; color: #ffffff;">Operators</a>
    <a href="{{ '/cs111/io/' | relative_url }}" class="rubric-btn" style="background-color: rgba(77, 117, 97, 0.2); border: 2px solid #4d7561; color: #ffffff;">Input/Output</a>
    
    <!-- Using ultra-clear dark text on the light tinted components -->
    <a href="{{ '/cs111/docs/' | relative_url }}" class="rubric-btn" style="background-color: rgba(163, 201, 177, 0.25); border: 2px solid #a3c9b1; color: #111827;">Documentation</a>
    <a href="{{ '/cs111/debug/' | relative_url }}" class="rubric-btn" style="background-color: rgba(188, 224, 202, 0.25); border: 2px solid #bce0ca; color: #111827;">Debugging</a>
    <a href="{{ '/cs111/testing/' | relative_url }}" class="rubric-btn" style="background-color: rgba(213, 242, 223, 0.25); border: 2px solid #d5f2df; color: #111827;">Testing & Verification</a>
</div>