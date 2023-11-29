#Microservice Code
#To call the data run the file and use one of the following links to return data(replace ISBN with 13 digit isbn)
#http://127.0.0.1:5000/library/books/ISBN
#http://127.0.0.1:5000/library/books/ISBN/details
#http://127.0.0.1:5000/library/books/ISBN/cover
#http://127.0.0.1:5000/library/books/ISBN/cover?size=L
#You can also use a get request using postman/your service to the same links

from flask import Flask, jsonify, request, redirect, Blueprint
from flask_cors import CORS
import requests

app = Flask(__name__)
CORS(app)

#Create a blueprint with 'CS361 Library' as the URL prefix
library_bp = Blueprint('CS361 Library', __name__, url_prefix='/library')

@library_bp.route('/books/<isbn>', methods=['GET'])
def get_book_by_isbn(isbn):
    response = requests.get(f'https://openlibrary.org/isbn/{isbn}.json')
    if response.status_code == 200:
        return jsonify(response.json())
    else:
        return jsonify({"error": "Book not found"}), 404

@library_bp.route('/books/<isbn>/cover', methods=['GET'])
def get_book_cover(isbn):
    size = request.args.get('size', 'M')  # Default to medium size
    cover_url = f'http://covers.openlibrary.org/b/isbn/{isbn}-{size}.jpg?default=false'

    cover_response = requests.get(cover_url)
    if cover_response.status_code == 200:
        return (cover_url)
    else:
        return jsonify({"error": "Cover not found"}), 404

@library_bp.route('/books/<isbn>/details', methods=['GET'])
def get_book_details(isbn):
    response = requests.get(f'https://openlibrary.org/isbn/{isbn}.json')
    if response.status_code == 200:
        book_data = response.json()
        title = book_data.get('title', 'No title available')
        description = book_data.get('description', 'No description available')
        if isinstance(description, dict):
            description = description.get('value', description)

        # Fetching author names
        authors = book_data.get('authors', [])
        author_names = []
        for author in authors:
            author_key = author.get('key')
            if author_key:
                author_response = requests.get(f'https://openlibrary.org{author_key}.json')
                if author_response.status_code == 200:
                    author_data = author_response.json()
                    author_names.append(author_data.get('name', 'Unknown Author'))

        return jsonify({"title": title, "description": description, "authors": author_names})
    else:
        return jsonify({"error": "Book not found"}), 404

#Register the blueprint with the Flask application
app.register_blueprint(library_bp)

if __name__ == '__main__':
    app.run(debug=True)


