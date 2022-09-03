## Creating-Web-APIs-with-Python-and-Flask

## Table of Contents
* [General Info](#general-information)
* [Technologies Used](#technologies-used)
* [Screenshots](#screenshots)
* [Setup](#setup)
* [Usage](#usage)
* [Project Status](#project-status)
* [Acknowledgements](#acknowledgements)
* [Contact](#contact)



## General Information
- This project aims to create an API that allows users to not only access a database but find specific resources within the database. The test catalog organizes information by ID, matches the books that have the provided ID and appends the list that will be returned to the user. 
- Finally we connect our API to our database and store the results from the API into a SQLite3 database. When our user requests an entry, our API provides that information from the database by creating and performing an SQL query. 


## Technologies Used
- Python Flask
- Terminal

## Screenshots
file:///C:/Users/John/Desktop/readme.PNG


## Setup

- In macOS, you can directly create a an api folder inside a projects folder in your home directory with this terminal command:

mkdir -p ~/projects/api

- On Windows, you can create the api folder with these commands in your cmd command line environment:

md projects
cd projects
md api

- In order to install the project, you must first navigate to your api folder(cd projects/api)
- Once you are in the project directory, run the flask application with the command python api.py 

## Usage

# Creating the Flask application

Next, open a text editor (such as Notepad++ or BBEdit) and enter the following code:

```
import flask

app = flask.Flask(__name__)
app.config["DEBUG"] = True


@app.route('/', methods=['GET'])
def home():
    return "<'h1'>Distant Reading Archive<'/h1'><'p'>This site is a prototype API for distant reading of science fiction novels.<'/p'>"

app.run()
```

- Save this code as api.py in the api folder you created for this tutorial.
- Navigate to cd projects/api
- Run the Flask application with the command: python api.py
- You should see the output: Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
- Follow the link above, http://127.0.0.1:5000/, using your web browser to see the running application

# Creating the API

Replace our previous code in api.py with the code below:
```
import flask
from flask import request, jsonify

app = flask.Flask(__name__)
app.config["DEBUG"] = True

    #Create some test data for our catalog in the form of a list of dictionaries.
books = [
    {'id': 0,
     'title': 'A Fire Upon the Deep',
     'author': 'Vernor Vinge',
     'first_sentence': 'The coldsleep itself was dreamless.',
     'year_published': '1992'},
    {'id': 1,
     'title': 'The Ones Who Walk Away From Omelas',
     'author': 'Ursula K. Le Guin',
     'first_sentence': 'With a clamor of bells that set the swallows soaring, the Festival of Summer came to the city Omelas, bright-towered by the sea.',
     'published': '1973'},
    {'id': 2,
     'title': 'Dhalgren',
     'author': 'Samuel R. Delany',
     'first_sentence': 'to wound the autumnal city.',
     'published': '1975'}
]


@app.route('/', methods=['GET'])
def home():
    return <'h1'>Distant Reading Archive<'/h1'>
<'p'>A prototype API for distant reading of science fiction novels.<'/p'>


    # A route to return all of the available entries in our catalog.
@app.route('/api/v1/resources/books/all', methods=['GET'])
def api_all():
    return jsonify(books)

app.run()
```

Run the code (navigate to your api folder in the command line and enter python api.py). Once the server is running, visit our route URL to view the data in the catalog:

http://127.0.0.1:5000/api/v1/resources/books/all

# Finding Specific Resources

- In this section we will add a function that allows users to filter their returned results using a more specific request. 
```
import flask
from flask import request, jsonify

app = flask.Flask(__name__)
app.config["DEBUG"] = True

    # Create some test data for our catalog in the form of a list of dictionaries.
books = [
    {'id': 0,
     'title': 'A Fire Upon the Deep',
     'author': 'Vernor Vinge',
     'first_sentence': 'The coldsleep itself was dreamless.',
     'year_published': '1992'},
    {'id': 1,
     'title': 'The Ones Who Walk Away From Omelas',
     'author': 'Ursula K. Le Guin',
     'first_sentence': 'With a clamor of bells that set the swallows soaring, the Festival of Summer came to the city Omelas, bright-towered by the sea.',
     'published': '1973'},
    {'id': 2,
     'title': 'Dhalgren',
     'author': 'Samuel R. Delany',
     'first_sentence': 'to wound the autumnal city.',
     'published': '1975'}
]


@app.route('/', methods=['GET'])
def home():
    return '''<'h1'>Distant Reading Archive<'/h1'>
<'p'>A prototype API for distant reading of science fiction novels.<'/p'>'''


@app.route('/api/v1/resources/books/all', methods=['GET'])
def api_all():
    return jsonify(books)


@app.route('/api/v1/resources/books', methods=['GET'])
def api_id():
    # Check if an ID was provided as part of the URL.
    # If ID is provided, assign it to a variable.
    # If no ID is provided, display an error in the browser.
    if 'id' in request.args:
        id = int(request.args['id'])
    else:
        return "Error: No id field provided. Please specify an id."

  
   results = []

   
   for book in books:
        if book['id'] == id:
            results.append(book)

   
   return jsonify(results)

app.run()
```
Once you’ve updated your API with the api_id function, run your code as before (python api.py from your api directory) and visit the below URLs to test the new filtering capability:

127.0.0.1:5000/api/v1/resources/books?id=0 127.0.0.1:5000/api/v1/resources/books?id=1 127.0.0.1:5000/api/v1/resources/books?id=2 127.0.0.1:5000/api/v1/resources/books?id=3

Each of these should return a different entry, except for the last, which should return an empty list: [], since there is no book for which the id value is 3. (Counting in programming typically starts from 0, so id=3 would be a request for a nonexistent fourth item.) In the next section, we’ll explore our updated API in more detail.

# Connecting Our API to a Database

- This last example of our Distant Reading Archive API pulls data from a database, implements error handling, and can filter books by publication date. 

- Before we modify our code, first download the example database(books.db) and copy the file to your api folder using your graphical user interface. The final version of our API will query this database when returning results to users.

- Copy the below code into your text editor:
```
import flask
from flask import request, jsonify
import sqlite3

app = flask.Flask(__name__)
app.config["DEBUG"] = True

def dict_factory(cursor, row):
    d = {}
    for idx, col in enumerate(cursor.description):
        d[col[0]] = row[idx]
    return d


@app.route('/', methods=['GET'])
def home():
    return '''<'h1'>Distant Reading Archive<'/h1'>
<'p'>A prototype API for distant reading of science fiction novels.<'/p'>'''


@app.route('/api/v1/resources/books/all', methods=['GET'])
def api_all():
    conn = sqlite3.connect('books.db')
    conn.row_factory = dict_factory
    cur = conn.cursor()
    all_books = cur.execute('SELECT * FROM books;').fetchall()

return jsonify(all_books)

    

@app.errorhandler(404)
def page_not_found(e):
    return "<'h1'>404<'/h1'><p>The resource could not be found.<'/p'>", 404

@app.route('/api/v1/resources/books', methods=['GET'])
 def api_filter():
        query_parameters = request.args

        id = query_parameters.get('id')
        published = query_parameters.get('published')
        author = query_parameters.get('author')

        query = "SELECT * FROM books WHERE"
        to_filter = []

        if id:
            query += ' id=? AND'
            to_filter.append(id)
        if published:
            query += ' published=? AND'
            to_filter.append(published)
        if author:
            query += ' author=? AND'
            to_filter.append(author)
        if not (id or published or author):
            return page_not_found(404)

        query = query[:-4] + ';'

        conn = sqlite3.connect('books.db')
        conn.row_factory = dict_factory
        cur = conn.cursor()

        results = cur.execute(query, to_filter).fetchall()

        return jsonify(results)

    app.run()
```
- Save the code as api_final.py in your api folder and run it by navigating to your project folder in the terminal and entering the command:

python api_final2.py

- Note that if a previous version of the code is still running, you will first need to end that process by pressing Control-C before executing the new code. Once this example is running, try out the filtering functionality with these HTTP requests:

http://127.0.0.1:5000/api/v1/resources/books/all http://127.0.0.1:5000/api/v1/resources/books?author=Connie+Willis http://127.0.0.1:5000/api/v1/resources/books?author=Connie+Willis&published=1999 http://127.0.0.1:5000/api/v1/resources/books?published=2010

## Project Status
Project is: complete


## Acknowledgements
- This project was inspired by The Programming Historian 
- This project was based on https://programminghistorian.org/en/lessons/creating-apis-with-python-and-flask#connecting-our-api-to-a-database



## Contact
Created by [@johngeorge142](https://github.com/johngeorge142/Creating-Web-APIs-with-Python-and-Flask/) 

