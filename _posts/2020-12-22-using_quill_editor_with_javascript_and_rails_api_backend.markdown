---
layout: post
title:      "Using Quill editor with Javascript and Rails API backend"
date:       2020-12-22 16:29:20 +0000
permalink:  using_quill_editor_with_javascript_and_rails_api_backend
---


Recently, I have been putting together code for my Javascript/Rails portfolio project. During my coding, I realized I wanted to have a rich text editor for my user's experience. Rails has ActionText built-in, but after some reading, I realized that there was no good way to implement this if I was to use Rails as an API backend.
This is where the Quill rich text editor comes in. I don't know if I'm the only one that came across issues in implementing this, but I would like to share my journey if in case someone else ran into this issue as well. So after following the quick start documentation provided by Quill, I had to build a form, so my code looked like this:

```
 static renderNewTopicForm() {
        document.getElementsByClassName('modal-card-body')[0].innerHTML = 
            `
                <label><strong>Title</strong></label><br>
                <input type=\"text\" name=\"title\" id=\"newtitle\"></input><br><br>
                <div id=\"editor\" style=\"height: 375px\">
                </div><br>
                <button class=\"button is-primary\" id=\"addpassage\">Add passage</button><br><br>
                <div id=\"topicpassages\"></div>
                <button class=\"button is-primary\" id=\"submittopic\">Submit topic</button>
            `
        
            Topic.quill = new Quill('#editor', {
                modules: {
                  toolbar: [
                    [{ 'font': [] }, { 'size': [] }],
                    [ 'bold', 'italic', 'underline', 'strike' ],
                    [{ 'color': [] }, { 'background': [] }],
                    [{ 'script': 'super' }, { 'script': 'sub' }],
                    [{ 'header': '1' }, { 'header': '2' }, 'blockquote', 'code-block' ],
                    [{ 'list': 'ordered' }, { 'list': 'bullet'}, { 'indent': '-1' }, { 'indent': '+1' }],
                    [ 'direction', { 'align': [] }],
                    [ 'link', 'image', 'video', 'formula' ],
                    [ 'clean' ]]
                },
                placeholder: '',
                theme: 'snow'  // or 'bubble'
              });

            Passage.addPassageForm()
            Topic.postFetchNewTopic()
```

After building the form, I had to store the contents into the database. There wasn't as much documentation on the website that showed how this was to be done. I started off by finding some articles like this one that explains that you should store it by taking the quill instance and saving it to the database as shown below

```
var form = document.querySelector('form');
form.onsubmit = function() {
  // Populate hidden form on submit
  var about = document.querySelector('input[name=about]');
  about.value = JSON.stringify(quill.getContents());
  
  console.log("Submitted", $(form).serialize(),
```

Quill's contents are stored in a format called Delta. This means that when storing then retrieving this from the database, you need to then convert the stringified Delta back into an object.
At least for me, the problem with this method was the part where I used JSON.stringify to store the contents into my PostgreSQL database. Once I retrieved it from the database, I needed to use JSON.parse to convert this back to an object like below:

```
quill.setContents(JSON.parse("{"ops":[{"insert":"example text\n"}]}"))
```

The problem was that JSON.parse would not convert it back to an object. If you read the Delta documentation it tells you that you can stringify and then parse the data. That did not happen for me. This became very troublesome. For my project, I not only wanted to submit and retrieve the contents from the database, I also needed to edit the form. After some digging, I realized I could simply store this line of code stringified into the database:

```
Topic.quill.root.innerHTML
```

I would store this into my topic.content JSON object like so:

```
let configObj = {
                method: "POST",
                headers: {
                    'Content-Type': 'application/json'
                },
                
                body: JSON.stringify({
                    
                    topic: {
                        title: document.getElementById('newtitle').value,
                        content: Topic.quill.root.innerHTML,
                        passages_attributes: newPassagesForTopic
                    }   
                    
                })
            }
```

By doing this, I was finally able to store the raw HTML into the database and retrieve it! Also, I found out for the edit form, all I needed to do was pre-populate the editor form after retrieving it from the database

```
document.getElementsByClassName('ql-editor')[0].innerHTML = topic.content
```

This solved a lot of headaches for me and I hope this was helpful. Happy coding!

