# Unit 4: A mini Reddit


# Criteria C: Development (no more than 1500 words)

## Technique Used
1. Flask Library/Routes
2. Python inside HTML
3. CSS Styling
4. For loops for showing posts
5. if statements
6. Password Hashing
7. Token-based authentication
8. Arrays and Lists
9. HTTP server
10. POST, GET Request
11. SQLite databases


## Sources

https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-iv-database 


## User profile picture


```.py
app.config['UPLOAD_FOLDER'] = 'static/profile_pics'
app.config['ALLOWED_EXTENSIONS'] = {'png', 'jpg', 'jpeg', 'gif'}

def save_profile_pic(file):
    if file and allowed_file(file.filename):
        filename = secure_filename(file.filename)
        file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
        return filename
    return None

```

The function ```save_profile_pic``` is designed to handle the upload and saving of user profile pictures securely. This function starts by checking if the file exists and if its filename is valid using the allowed_file function. The ```allowed_file``` function verifies that the file has one of the permitted extensions like 'png', 'jpg', 'jpeg', or 'gif', which are defined in the application's configuration ```app.config['ALLOWED_EXTENSIONS']```. This validation is performed by checking if there is a period in the filename and if the part after the last period matches one of the allowed extensions. The reason why I choose to user ```app.config``` is that it allows for centralized configuration management within the Flask application. By storing the allowed file extensions and the upload folder path in app.config, it becomes easier to manage and modify these settings in one place, ensuring consistency throughout the application. If the file passes this validation, the filename is sanitized using ```secure_filename``` from the ```werkzeug.utils``` module. This step is essential to prevent security issues that could arise from malicious filenames, such as those containing special characters or directory traversal sequences. The sanitized filename is then used to save the file to the server in the directory specified by ```app.config['UPLOAD_FOLDER']```. The path to the file is constructed using the ```os.path.join``` function from the ```os``` module, ensuring compatibility across different operating systems. The ```save``` method of the file object is then called to save the file at this location. If the file is successfully saved, the function returns the filename. If the file is not valid or the saving process fails, the function returns None. This mechanism ensures that only valid image files are uploaded and saved securely, enhancing user experience by allowing them to personalize their profiles.


## User profile information

These SQL Queries are used to fetch various information about a user from the database ```follows```, ```reviews```, and ```likes```, which will be displayed on the user's profile page. I will explain in detail. 

```.py
user = db.search(f"SELECT * FROM user WHERE id={user_id}", multiple=False)

following = db.search(f"SELECT * FROM follows WHERE follower_id={current_user_id} AND followed_id={user_id}", multiple=False) is not None
review_count = db.search(f"SELECT COUNT(*) FROM reviews WHERE user_id={user_id}", multiple=False)[0]

like_count = db.search(f"""SELECT COUNT(*) FROM likes l JOIN reviews r ON l.review_id = r.id WHERE r.user_id={user_id}""", multiple=False)[0]

follower_count = db.search(f"SELECT COUNT(*) FROM follows WHERE followed_id={user_id}", multiple=False)[0]
following_count = db.search(f"SELECT COUNT(*) FROM follows WHERE follower_id={user_id}", multiple=False)[0]
```

The first query retrieves all information about the user from the ```user``` table based on their ```user_id```, including details like the username, email, and profile picture. 

The second query checks if the current user (the one logged in) is following the user whose profile is being viewed. This is done by looking for a record in the ```follows``` table where the ```follower_id``` is the current user's ID and the ```followed_id``` is the profile user's ID. If such a record exists, it means the current user is following the profile user.

The third query counts the number of reviews posted by the user by selecting the count of rows in the ```reviews``` table where the ```user_id``` matches the profile user's ID. This is achieved using the ```COUNT(*)``` function, an built-in function in SQL that returns the number of rows that match the specified criteria. In this case, it returns the number of reviews that the user has posted. 

The fourth query counts the number of likes received on the user's reviews by joining the ```likes``` and ```reviews``` tables. It selects the count of rows in the likes table where the ```review_id``` matches a review ID from the ```reviews``` table, which in turn matches the profile user's ID. The ```JOIN``` method is used to combine rows from two or more tables based on a related column between them.

The fifth query counts how many users are following the profile user by selecting the count of rows in the ```follows``` table where the ```followed_id``` matches the profile user's ID. Similarly, the final query counts how many users the profile user is following by selecting the count of rows in the ```follows``` table where the ```follower_id``` matches the profile user's ID. 

These queries together provide a detailed profile page with all relevant user information, enhancing the user experience by showing their activities, interactions, and social metrics on the platform.


## SQL Query for Review likes

```.py
revs = db.search(query=f"""
    SELECT r.id, r.date, r.stars, r.comment, r.movie_id, COUNT(l.id) as likes, r.user_id, u.uname,
           (SELECT 1 FROM likes WHERE review_id = r.id AND user_id = {current_user_id}) as liked
    FROM reviews r
    JOIN user u ON r.user_id = u.id
    LEFT JOIN likes l ON r.id = l.review_id
    WHERE r.movie_id = {movie_id}
    GROUP BY r.id
""", multiple=True)
```

The SQL Query used to retrieve reviews for a specific movie, corresponding with the number of likes each review has received and whether the current user liked each review, is designed to display detailed information about both movie and book reviews and user interactions. The query selects columns from the ```reviews``` table, including review details such as the review ID, date, stars, comment, and movie ID, as well as user details like the user ID and username. The ```COUNT()``` function is used to count the number of likes each review has received. The ```JOIN``` clause combines rows from the ```reviews``` table with the ```user``` table to get the username of the reviewer and with the ```likes``` table to count the number of likes for each review.

The query includes a subquery to check if the current user has liked each review. This subquery returns ```1``` if the current user liked the review; otherwise, it returns ```NULL```. This check is performed by selecting from the ```likes``` table where the ```review_id``` matches the review ID and the ```user_id``` matches the current user's ID. The query filters reviews by the specified movie ID using the ```WHERE``` clause and groups the results by the review ID using the ```GROUP BY``` clause to ensure correct aggregation of likes. This comprehensive view of movie reviews, including user engagement metrics, enhances the user experience by showing interactive and relevant content.


# Criteria D: Functionlaity

## Demonstration video


# Criteria E: Evaluation

## Functionality test


## Evaluation by Peers
My peer is very satisfied with the website, as detailed in Appendix 1 and Appendix 2, confirming that it meets all the success criteria. However, they suggested an area for improvement: adding functionality on the 'Profile' page that allows users to see lists of their followers and whom they are following. Implementing this feature could allow user engagement by making the social connections within the platform more visible and accessible.

## Evaluation of beta testing



# Appendix

<img width="max" alt="Screenshot 2024-05-28 at 1 55 53 PM" src="https://github.com/hasmhib/unit4-2024/assets/142870448/be290c05-58db-47e2-b748-f28108ca5ae1">

##### _Appendix.1 Contact between developer and client for evaluation of website_

<img width="max" alt="Screenshot 2024-05-28 at 1 56 27 PM" src="https://github.com/hasmhib/unit4-2024/assets/142870448/5c23a7f1-cdcb-4d60-a2f6-1fd4e1a00813">

##### _Appendix.2 Contact between developer and client for evaluation of website_

<img width="max" alt="Screenshot 2024-05-28 at 1 56 13 PM" src="https://github.com/hasmhib/unit4-2024/assets/142870448/d5995505-fa6a-4ff4-8bdd-d5c01fabcaf3">

##### _Appendix.3 Contact between developer and client regarding beta testing and feedback_


[^1]: https://www.geeksforgeeks.org/how-to-use-flask-session-in-python-flask/

