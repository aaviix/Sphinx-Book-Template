# Sphinx-Book-Template

This project provides a template for creating scientific texts using Sphinx and the Sphinx-book-theme. The content can be written in Markdown or reStructuredText. Below are the steps to get started with this template.

## Getting Started

### Prerequisites
- Python 3.x
- pip
- virtualenv

### Installation

1. **Clone the Repository**

    Fork the repository and clone it to your local machine:

    ```sh
    git clone https://github.com/yourusername/Sphinx-Book-Template
    cd Sphinx-Book-Template
    ```

2. **Set Up a Virtual Environment**

    Create and activate a virtual environment to manage dependencies:

    ```sh
    virtualenv venv
    source venv/bin/activate
    ```

3. **Install Dependencies**

    Install the required Python packages:

    ```sh
    pip install -r python-requirements.txt
    ```

## Building the Book

### Generate HTML

To create HTML documentation:

```sh
make html
```

Open the output in your browser:

```sh
xdg-open _build/html/index.html
```

### Generate PDF

To create a PDF version, ensure LaTeX is installed and use Docker:
```sh
docker run --rm -v $(pwd):/docs sphinxdoc/sphinx-latexpdf sh -c 'apt update; apt install -y enchant-2; pip install -U -r python-requirements.txt; make latexpdf'
```

Find the generated PDF in the _build/latex directory.

### Spellcheck

To check spelling:

```sh
make spelling
```

### Deployment

#### Setting Up Permissions

Ensure your web server has access to the public_html directory:

```sh
ssh yourserver.com
mkdir public_html
chmod go-rx --recursive ~
chmod go+x ~
chmod go+rx public_html
```

### Continuous Integration

Configure ci.yml for Github CI to automate deployment. Set up SSH keys and variables as described in the documentation.

## License

This project is licensed under the MIT License. See the LICENSE file for details.