# `sphinx-book-theme`-based Template for Scientific Text

You can write your content in Markdown or reStructuredtext. I recommend [writing in Markdown](https://commonmark.org/help).

# Getting started

## Install `sphinx-book-theme`

Create a Python virtual environment to not influence your standard Python development environment:

```sh
virtualenv venv
. ./venv/bin/activate
pip install -r python-requirements.txt
```
Sphinx uses reStructuredtext as default. Using `myst_parser` we can also use Markdown.

## Cloning the template

First fork this project using the fork button on the top right. Then clone your project on your computer. Before modifying the template to your needs, make sure that the continuous integration script can successfully run. You need to setup authentication for this. Refer to the section Deployment on World Wide Web (WWW) below.
# Creating html

```sh
cd sphinx-book-template
make html
```

You can view the output using your browser, e.g.:

```sh
xdg-open _build/html
```

# Creating PDF

Using PDF we can compile the whole report content in a single file. PDF does not provide the interactivity and the beautiful format that the HTML output provides though. Still PDF is useful for archiving purposes.

For PDF output Latex is required. Instead of generating the PDF locally, I recommend leaving this job to continuous integration (CI) documented below. If you insist, you can run Docker on your own computer and include the commands from the CI. Run the following on the root of the template directory:

```sh
docker run --rm -v .:/docs sphinxdoc/sphinx-latexpdf sh -c 'apt update; apt install -y enchant-2; pip install -U -r python-requirements.txt; make latexpdf'
```

Breakdown of the command:

- `--rm` *removes* the container after running
- `-v HOST_DIR:CONTAINER_DIR` creates a *volume* inside the container for bidirectional data exchange. In the example above we create a link between the current folder `.` and the folder `/docs` in the container. So every file that the container generates in `/docs` will be stored in your current folder before the container is removed. The container's [default working folder is `/docs`](https://hub.docker.com/r/sphinxdoc/sphinx-latexpdf).
- `sphinxdoc/sphinx-latexpdf` is the name of the container image. Docker will automatically fetch it if the image does not exist on your computer.
- `sh -c '...;...'` runs a sequence of commands.

At the end you should see a PDF file in the build folder:

```sh
$ find -iname '*pdf'
./_build/latex/sphinx-booktemplate.pdf
```

# Spellcheck

```sh
make spelling
```

Refer to the spellcheck section on the rendered template for more info.

# Deployment on World Wide Web (WWW)

The deploy step copies the folder `html` to a shared directory in your home folder called `public_html`. We assume that `$HOME/public_html` is served to the internet by a web server.

## Permissions on your home folder

You have to make sure that the web server has access to this folder. In the following example we are using a server called `joan.th-deg.de`:

```sh
ssh joan.th-deg.de
mkdir public_html  # This folder will be shared
```

We have to give other users (your group & others) the permission to public_html by giving listing access to your home folder. However other users could now read the files by guessing their filename, so  we have to remove the read permission (`r`) and execute/enter-directory permission (`x`) for other users for every file/directory but `public_html`. To give others access to `public_html` the `x` permission must exist for the whole chain until `public_html`, this means that not only `public_html`, but also your home folder must have the `x` permission for others.

```sh
chmod go-rx --recursive ~  # Remove rx from other users from all files/folders, `~` means our home folder
chmod go+x ~  # Add directory enter permission (listing the files is forbidden, because `r` is missing)
 
# with the exception of public_html, which may be read
chmod go+rx public_html
```

If you already have other files and folders in `public_html`, then make them readable by other users:

```sh
# Give others the permission to read all the files
find ~/public_html -type f -exec chmod go+r {} +
# Give others the permission to enter and list the directories
find ~/public_html -type d -exec chmod go+rx {} +
```

## CI/CD settings

This template includes a continuous integration and deployment (CI/CD) configuration (`.gitlab-ci.yml`) for Gitlab which copies the html files to a server whenever you push a new commit. To use it, you need to have an SSH access to a web server.

Three steps:

1. Setting up the values for the variables in `.gitlab-ci.yml` (explained below)
1. Modifying `.gitlab-ci.yml` according to your username and the deployment folder on the web server
1. Activating a runner in `Settings - CI/CD - Runners`. A shared runner may be available. 
1. Check the status of the runner on the main page of your repository. After about a few minutes the runner should pick up your job and execute your pipeline. If successful, you should see the result on, e.g., https://joan.th-deg.de/~gaydos/sphinx-book-template


<details><summary>For the explanation of the first step click here</summary>

### Setting up the values for the variables in `.gitlab-ci.yml`

First store your SSH private key (`$SSH_PRIVATE_KEY`) and the SSH public key of the SSH server (`$SSH_KNOWN_HOSTS`) in the CI/CD variables. You find a tutorial here: [Add a CI/CD variable to a project](https://docs.gitlab.com/ee/ci/variables/#add-a-cicd-variable-to-a-project). You do not have to activate `Protected` nor `Masked`. However make sure that [`Expand variable`](https://docs.gitlab.com/ee/ci/variables/#prevent-cicd-variable-expansion) is activated, otherwise the variable used in `.gitlab-ci.yml` won't be substituted with its value (this is called expansion).

#### What do I store in `SSH_PRIVATE_KEY`?

Create a new public-private key pair using `ssh-keygen`:

```
ssh-keygen -P '' -f $HOME/.ssh/gitlab-deploy-key
```

The key pair is now placed in `$HOME/.ssh`. A public key has the extension `*.pub`. A private key is extensionless, e.g., `$HOME/.ssh/id_rsa`. The `-----BEGIN` header and the footer belong to the key. Example:

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAlLAHB1jr+muOFMbAtJWcPQJZOJtiAdtSNauyYU9i5/geckOuKHP9
...
K4oavZecZ5XW8AAAASYXJjaEBheWRvcy1yb3MtbWwyAQ==
-----END OPENSSH PRIVATE KEY-----
```

You (or a software agent) keeps a private key for yourself and provide the public key to others so that others can identify you. In our case the pipeline runner will keep the private key for itself and will use it to identify itself to joan.th-deg.de and to copy the web pages to your home folder here. So in a later step we have to authorize the gitlab runner to access our home folder by using the public key that belongs to the private key.

#### What do I store in `SSH_KNOWN_HOSTS`?

This is the key that you accept when you first connect to an SSH server. You can find the public keys of the SSH servers that you have connected to under `$HOME/.ssh/known_hosts`. Alternatively use `ssh-keyscan SERVER_ADDRESS`.

For example a SSH public key of `joan.th-deg.de` is:

```
joan.th-deg.de ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPYxtnUvjyLdIDkzs4GEzv6KnSN88uPQCC3H/IcuEToe
```

#### Giving access to an SSH key

Finally we have to give an SSH key access to our account by using the public key. To give access append the public key to `$HOME/.ssh/authorized_keys` on the web server. For example:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOBL7HM8eE9jmBm5Yz/sJeStc3mGAJp5R8EvVJ4zb9T9 gaydos@joan
```
- If `.ssh` does not exist, `mkdir .ssh` first. 
- If you create `authorized_keys` for the first time, then you have to give 

</details>

# Further docs

- [Sphinx-book theme – Get Started](https://sphinx-book-theme.readthedocs.io)
- What is this template capable of?
  - [Sphinx-book theme – Paragraph level markup](https://sphinx-book-theme.readthedocs.io/en/stable/reference/kitchen-sink/paragraph-markup.html)
  - [Gallery of books based on this library](https://executablebooks.org/en/latest/gallery.html)

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.
