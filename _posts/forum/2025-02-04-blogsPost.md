---
permalink: /studentBlogs/post
title: Blogs Post
---

<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            background-color: black;
            color: white;
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 0;
            padding: 0;
        }
        h1, h2 {
            margin-top: 10px;
        }
        .nav-buttons {
            margin-top: 20px;
            display: flex;
            justify-content: center;
        }
        .nav-buttons button {
            background-color: black;
            color: white;
            border: 1px solid white;
            padding: 10px 20px;
            margin: 0 10px;
            cursor: pointer;
            font-size: 16px;
        }
        .nav-buttons button:hover {
            background-color: gray;
        }
        .signout {
            text-align: right;
            padding: 10px;
            margin-right: 20px;
        }
        .container {
            margin-top: 40px;
            width: 80%;
            margin-left: auto;
            margin-right: auto;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        #ask-question {
            margin-top: 30px;
        }
        textarea {
            width: 50%;
            padding: 10px;
            margin: 10px 0;
            box-sizing: border-box;
            background-color: #333;
            border: 1px solid #555;
            color: white;
            resize: none;
            overflow: hidden;
        }
        button {
            padding: 10px 20px;
            margin: 10px;
            background-color: black;
            border: 1px solid white;
            color: white;
            cursor: pointer;
        }
        button:hover {
            background-color: gray;
        }
        .question {
            background-color: #222;
            padding: 10px;
            margin-top: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .reply-box {
            display: none;
            background-color: #333;
            padding: 10px;
            margin-top: 10px;
        }
        .arrow {
            cursor: pointer;
            font-size: 24px;
            padding: 0 10px;
        }
        .section-title {
            font-size: 36px;
            margin-bottom: 30px;
        }
        .output {
            margin-top: 20px;
        }
        .form-box {
            width: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .form-box label {
            width: 50%;
            text-align: left;
            margin-top: 10px;
        }
        select {
            width: 50%;
            padding: 10px;
            margin: 10px 0;
            box-sizing: border-box;
            background-color: #333;
            border: 1px solid #555;
            color: white;
        }
        .body {
            width: 400px;
            height: 300px;
        }
    </style>
</head>
<body>
    <h1>Create a Blog Post</h1>
    <form id="issueForm">
        <label for="title">Title:</label><br>
        <input type="text" id="title" name="title" required><br><br>
        <label for="body">Body:</label><br>
        <textarea id="body" name="body" required class="body"></textarea><br><br>
        <button type="submit">Submit</button>
    </form>
    <div id="response"></div>
    <script type="module">
        import {javaURI, fetchOptions} from '{{site.baseurl}}/assets/js/api/config.js';
        function isLoggedIn() {
            //get the cookie jwt_java_spring
            const token = localStorage.getItem('ghid');
            return token !== undefined;
        }
        document.getElementById('issueForm').addEventListener('submit', function(event) {
            event.preventDefault(); // Prevents the form from refreshing the page
            // print out button pressed in console
            console.log('Button pressed');
            if (!isLoggedIn()) {
                if (!confirm('You are not logged in. Submitting will not save your post. Do you want to continue?')) {
                    return;
                }
                else
                {
                    const author = null;
                    const title = document.getElementById('title').value;
                    const body = document.getElementById('body').value;
                    fetch(`${javaURI}/blogs/blog/post`, {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                        },
                        body: JSON.stringify({ title, body, author }),
                    })
                    .then(response => response.text())
                    .then(data => {
                        document.getElementById('response').innerText = 'Response: ' + data;
                    })
                    .catch(error => {
                        document.getElementById('response').innerText = 'Error: ' + error;
                    });
                }
            }
            if (isLoggedIn()) {
                const title = document.getElementById('title').value;
                const body = document.getElementById('body').value;
                const author = localStorage.getItem('ghid');
                fetch(`${javaURI}/blogs/blog/post`, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({ title, body, author }),
                })
                .then(response => response.text())
                .then(data => {
                    document.getElementById('response').innerText = 'Response: ' + data;
                })
                .catch(error => {
                    document.getElementById('response').innerText = 'Error: ' + error;
                });
            };
            //wait 3 seconds then send them back
            setTimeout(function() {
                window.location.href = "/pages/studentBlogs/";
            }, 3000);
        });
    </script>
</body>
</html>
