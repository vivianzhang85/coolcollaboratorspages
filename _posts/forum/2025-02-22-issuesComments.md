---
permalink: /studentComments/
title: Issues Post
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
        .container {
            width: 80%;
            margin: 40px auto;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .box {
            width: 100%;
            background-color: #222;
            padding: 20px;
            border: 1px solid white;
            text-align: left;
        }
        .issue-entry {
            background-color: #333;
            padding: 10px;
            margin-top: 10px;
            border-left: 5px solid gray;
            cursor: pointer;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .issue-entry:hover {
            background-color: #444;
        }
        .comment-section {
            margin-top: 20px;
        }
        .comment {
            background-color: #444;
            padding: 10px;
            margin-top: 5px;
            border-left: 3px solid #777;
        }
        .input-field {
            width: 100%;
            padding: 10px;
            margin-top: 10px;
            font-size: 16px;
        }
        .submit-button {
            background-color: black;
            color: white;
            border: 1px solid white;
            padding: 10px 20px;
            cursor: pointer;
            margin-top: 10px;
        }
        .submit-button:hover {
            background-color: gray;
        }
        .comment-header {
            font-size: 14px;
            color: gray;
            margin-bottom: 5px;
        }
        .edit-button, .delete-button {
            background-color: #444;
            color: white;
            border: 1px solid #777;
            padding: 5px 10px;
            cursor: pointer;
            margin-left: 10px;
        }
        .edit-button:hover, .delete-button:hover {
            background-color: #555;
        }
        .issue-details {
            display: none;
            width: 80%;
            margin: 20px auto;
            background-color: #222;
            padding: 20px;
            border: 1px solid white;
            text-align: left;
        }
        .pagination {
            margin-top: 20px;
            display: flex;
            justify-content: center;
            gap: 10px;
        }
        .page-button {
            background-color: black;
            color: white;
            border: 1px solid white;
            padding: 5px 10px;
            cursor: pointer;
        }
        .page-button:hover {
            background-color: gray;
        }
        .search-bar {
            width: 100%;
            margin-bottom: 10px;
            padding: 8px;
            font-size: 16px;
        }
        
    </style>
</head>
<body>
  <div class="header-container">
        <h1>Issue Home Page</h1>
        <a href="http://localhost:4100/pages/studentIssue/post" class="post-button">Create New Issue</a>
    </div>

    <div class="container">
        <input type="text" id="search-input" class="search-bar" placeholder="Search issues..." onkeyup="searchIssues()">
        <div class="box" id="latest-issues">
            <h2>Latest Issues</h2>
            <div id="issues-container"></div>
            <div class="pagination" id="pagination"></div>
        </div>
    </div>

    <div id="issue-details" class="box" style="display: none;">
        <h2 id="issue-title"></h2>
        <p id="issue-author"></p>
        <p id="issue-date"></p>
        <p id="issue-context"></p>
        <form id="comment-section">
            <h3>Comments</h3>
            <div id="comments-container"></div>
            <textarea id="comment-input" class="input-field" placeholder="Write a comment..."></textarea>
            <button type="submit" class="submit-button" onclick="postComment(event)">Post Comment</button>
        </form>
        <button class="submit-button" onclick="showIssueList()">Back</button>
    </div>

    <script>
        let allIssues = [];
        let currentPage = 1;
        const issuesPerPage = 5;
        let selectedIssueId = null;
        let loggedInUsername = localStorage.getItem('ghid') || "Anonymous"; // Get logged-in user's ID

        // Fetching latest issues from the API
        async function fetchLatestIssues() {
            try {
                const response = await fetch('http://localhost:8585/forum/get');
                if (!response.ok) throw new Error('Network error');
                
                allIssues = await response.json();
                displayIssues();
            } catch (error) {
                console.error('Fetch error:', error);
            }
        }

        // Displaying fetched issues
        function displayIssues() {
           const container = document.getElementById('issues-container');
            const paginationContainer = document.getElementById('pagination');
            container.innerHTML = '';
            paginationContainer.innerHTML = '';

            const start = (currentPage - 1) * issuesPerPage;
            const paginatedIssues = allIssues.slice(start, start + issuesPerPage);

            paginatedIssues.forEach(issue => {
                const issueElement = document.createElement('div');
                issueElement.className = 'issue-entry';
                issueElement.addEventListener("click", function() {
                    showIssueDetails(issue);
                });
                issueElement.innerHTML = `
                    <div class="issue-title">${issue.title}</div>
                    <div class="issue-author">By: ${issue.author}</div>
                `;
                container.appendChild(issueElement);
            });

            const totalPages = Math.ceil(allIssues.length / issuesPerPage);
            for (let i = 1; i <= totalPages; i++) {
                const pageButton = document.createElement('button');
                pageButton.className = 'page-button';
                pageButton.textContent = i;
                pageButton.onclick = function () {
                    currentPage = i;
                    displayIssues();
                };
                paginationContainer.appendChild(pageButton);
            }
        }

        // Showing the issue details
        async function showIssueDetails(issue) {
            selectedIssueId = issue.id;
            document.getElementById('issue-title').textContent = issue.title;
            document.getElementById('issue-author').textContent = "By: " + issue.author;
            document.getElementById('issue-date').textContent = "Date: " + issue.date;
            document.getElementById('issue-context').textContent = issue.context || "No description provided.";
            
            document.querySelector('.container').style.display = 'none';
            document.getElementById('issue-details').style.display = 'block';
            fetchComments(issue.id);
        }

          function searchIssues() {
            const query = document.getElementById('search-input').value.toLowerCase();
            allIssues = allIssues.filter(issue => issue.title.toLowerCase().includes(query));
            displayIssues();
        }

        // Showing the list of issues
        function showIssueList() {
            document.querySelector('.container').style.display = 'flex';
            document.getElementById('issue-details').style.display = 'none';
        }

        // Fetching comments for a selected issue
        async function fetchComments(forumId) {
            try {
                const response = await fetch(`http://localhost:8585/comments/get/${forumId}`);
                if (!response.ok) throw new Error('Network error');
                
                const comments = await response.json();
                console.log(comments);
                displayComments(comments);
            } catch (error) {
                console.error('Fetch error:', error);
            }
        }

        // Displaying the comments for the selected issue
        function displayComments(comments) {
            const container = document.getElementById('comments-container');
            container.innerHTML = '';

            comments.forEach(comment => {
                const commentElement = document.createElement('div');
                commentElement.className = 'comment';
                commentElement.innerHTML = `
                    <div class="comment-header">${comment.author} commented on ${comment.timestamp}</div>
                    <div class="comment-body">${comment.comment}</div>
                    ${comment.author === loggedInUsername ? `
                        <button class="edit-button" onclick="editComment(${comment.id})">Edit</button>
                        <button class="delete-button" onclick="deleteComment(${comment.id})">Delete</button>
                    ` : ''}
                `;
                container.appendChild(commentElement);
            });
        }

        // Posting a new comment
        async function postComment(event) {
            alert("hello");
            event.preventDefault();
            
            const token = document.cookie.split('; ').find(row => row.startsWith('jwt_java_spring='));
            if (!token) {
                alert('You are not logged in. Comment will not be saved.');
                return;
            } 

            const commentText = document.getElementById('comment-input').value;
            const author = loggedInUsername; // Use logged-in username
            const commentData = {
                forumId: selectedIssueId,
                comment: commentText,
                author: author
            };

            try {
                const response = await fetch('http://localhost:8585/comments/post', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': `Bearer ${token.split('=')[1]}`
                    },
                    body: JSON.stringify(commentData)
                });

                if (response.ok) {
                    alert('Comment posted successfully!');
                    fetchComments(selectedIssueId); // Refresh comments
                } else {
                    throw new Error('Failed to post comment');
                }
            } catch (error) {
                console.error('Error posting comment:', error);
                alert('Error posting comment');
            }
        }

        // Edit a comment
        async function editComment(commentId) {
            const newCommentText = prompt("Edit your comment:");

            if (newCommentText === null || newCommentText === "") {
                return; // Cancel editing if no input
            }

            const token = document.cookie.split('; ').find(row => row.startsWith('jwt_java_spring='));
            if (!token) {
                alert('You are not logged in. Edit will not be saved.');
                return;
            }

            try {
                const response = await fetch(`http://localhost:8585/comments/update/${commentId}?username=${loggedInUsername}`, {
                    method: 'PUT',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': `Bearer ${token.split('=')[1]}`
                    },
                    body: JSON.stringify({ comment: newCommentText })
                });

                if (response.ok) {
                    alert('Comment edited successfully!');
                    fetchComments(selectedIssueId); // Refresh comments
                } else {
                    throw new Error('Failed to edit comment');
                }
            } catch (error) {
                console.error('Error editing comment:', error);
                alert('Error editing comment');
            }
        }

        // Delete a comment
        async function deleteComment(commentId) {
            const confirmDelete = confirm("Are you sure you want to delete this comment?");
            if (!confirmDelete) return;

            const token = document.cookie.split('; ').find(row => row.startsWith('jwt_java_spring='));
            if (!token) {
                alert('You are not logged in. Deletion will not be saved.');
                return;
            }

            try {
                const response = await fetch(`http://localhost:8585/comments/delete/${commentId}?username=${loggedInUsername}`, {
                    method: 'DELETE',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': `Bearer ${token.split('=')[1]}`
                    }
                });

                if (response.ok) {
                    alert('Comment deleted successfully!');
                    fetchComments(selectedIssueId); // Refresh comments
                } else {
                    throw new Error('Failed to delete comment');
                }
            } catch (error) {
                console.error('Error deleting comment:', error);
                alert('Error deleting comment');
            }
        }

        // Initialize page by fetching issues
        window.onload = fetchLatestIssues;
    </script>
</body>
</html>
