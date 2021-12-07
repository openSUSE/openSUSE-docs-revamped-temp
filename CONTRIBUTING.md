## Get in touch?
We often hang out on [this Telegram chat](https://t.me/opensuse_docs), which is bridged to the IRC `#docs:opensuse.org` and to `#docs` on [this Discord channel](https://discord.gg/opensuse).

## Review process
```
new document ------> review on structure and contents #1 ------> review on language, style and punctuation #1
                                                                                        |
                                                                                        |
marked "ready for release" <--- review on language #2 <--- review on contents #2  <------
```

## Getting started
### Understand the repo structure
```
... 
Pipfile           <<< project dependencies in pipenv format. That is what 'pipenv install' operates on.
requirements.txt  <<< project dependencies in pip format. Added for compatibility.
pyproject.toml    <<< project dependencies in poetry format. That is what 'poetry install' operates on.
project/    <<< 'cd' in here to run mkdocs commands (pipenv commands are to be run from the root directory)
    docs/   <<< WHERE YOU CONTRIBUTE. You will be adding or editing files there.
    mkdocs.yml <<< build config, but also used as Table of Contents. If you add a new file, reference it there.
    ...
...
```
`.gitignore` should not let you commit any `build` directory. Please make sure that is the case.

### Set up your work environment
You can either install mkdocs from pip or from a virtual environment.

It's highly recommended to use a virtual environment manager and not `pip`, so that the dependencies of this project won't mess with your system-wide python packages / modules. You still need to run pip once to install your favorite virtual environment manager.

## Pipenv installation

We like using pipenv, which you install on openSUSE distributions with: `pip3 install --user pipenv`. Then you'll need to add `~/.local/bin` to your `PATH`. The best method for that depends on your shell:
* for `bash` add `PATH=$PATH:/home/your-user-name/.local/bin` to `~/.bashrc`; reload to apply changes with `source ~/.bashrc`
* for `fish` run the following command once from a fish shell: `set -Ua fish_user_paths /home/your-user-name/.local/bin`; start a fresh shell with `exec fish`
* for `zsh` add `export PATH=$PATH/home/your-user-name/.local/bin` to `~/.zshrc`; reload to apply changes with `source ~/.zshrc`

## Poetry installation (requires Python 3.5+)

Install via poetry installer : `curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python -`

After the installation command completes, you will find your Poetry install under `$HOME/.poetry/bin`
This directory will be automatically added to your $PATH environment variable, by appending a statement to your $HOME/.profile configuration (or equivalent files). If you do not feel comfortable with this, pass the `--no-modify-path` flag to the installer and manually add the Poetry's bin directory to your path.

Another ways to install poetry are via openSUSE repository `zypper install python38-poetry` for Python 3.8 and `zypper install python36-poetry` for Python 3.6.

### Enable tab completion for `poetry`
```
# Bash
poetry completions bash > /etc/bash_completion.d/poetry.bash-completion

# Bash (Homebrew)
poetry completions bash > $(brew --prefix)/etc/bash_completion.d/poetry.bash-completion

# Fish
poetry completions fish > ~/.config/fish/completions/poetry.fish

# Fish (Homebrew)
poetry completions fish > (brew --prefix)/share/fish/vendor_completions.d/poetry.fish

# Zsh
poetry completions zsh > ~/.zfunc/_poetry

# Oh-My-Zsh
mkdir $ZSH_CUSTOM/plugins/poetry
poetry completions zsh > $ZSH_CUSTOM/plugins/poetry/_poetry

# prezto
poetry completions zsh > ~/.zprezto/modules/completion/external/src/_poetry
```

## Clone, edit, test
1. Clone this repo to where you want in your home folder by running `git clone https://github.com/openSUSE/openSUSE-docs-revamped-temp.git`.
2. `cd` to it and run `pipenv install` or `poetry install` to install the dependencies, and then `pipenv shell` or `poetry shell` to run the environment. 
3. finally `cd` to `project` and run the `mkdocs` commands from there.
4. To generate the website from the source files and serve it, run `mkdocs serve`.
5. View the generated documents in your browser on http://127.0.0.1:8000/.

You can now work on the docs. Whenever you save your changes, the site will be regenerated and your browser will update the current page.
To shut down the server, press `Ctrl + C`.

### Get ready to open a Pull Request
If you are new to `git` and GitHub, read https://jarv.is/notes/how-to-pull-request-fork-github/. 

When you are done working, go through the checklist before opening a Pull Request against our repository.

__Checklist__:
- [x] When doing `git checkout -b <meaningful name> <...>`, did you make sure that `meaningful name` satisfied the schema described under __Branches__ below? If not, you can still rename it using `git branch -m <new name that satisfies the schema> `.
- [x] Have you followed the style guidelines below under __Style__?
- [x] If you have added a new article:
  - a) did it land in `/project/docs`? If not, move it there.
  - b) have you added it to [table of contents](https://github.com/openSUSE/openSUSE-docs-revamped-temp/blob/dev/ToC.md)? Just follow the examples already there. The urls look like `https://github.com/openSUSE/openSUSE-docs-revamped-temp/blob/dev/<some file.md>`
- [x] Have you tested your work? If not consider the __Clone, edit, test__ section above.
- [x] Are you going to make your PR editable for us? If you don't know how, you will have to check the `Allow edits from maintainers` checkbox on the Pull Request screen, in GitHub. Otherwise we won't be able to work with you on your PR.

## Formal constraints
### Branches
* The default branch -- the working branch -- is not `main` or `master` but `dev`. I will merge from one milestone to the other.
* 4 types of branch names, named after the type of commits you want to contribute. PRs should whenever possible concern just one type of commit.
  * `structure` (how the textual and multimedia contents breaks down into different parts)
  * `design` (web and non-web visuals)
  * `web-functions` (functionalities invoked from the web release of the docs)
  * `contents` (textual and multimedia contents)

### Style
_Structure_. Each document should start with an intro stating the end goal, the important presuppositions (typically about pre-requirements) that the document is making, and an outline of the main steps on the path to the goal.

_Technical jargon_. Important and unavoidable jargon should be defined (typically inlined, with info boxes). Overall the document should be understandable by a teenager (think secondary school textbook).

_Format_:
* reference points and path items in italics, ex: "_Settings_ > _Energy Saving_"
* action in bold, ex. "click __Yes__"
* code instructions part of a stepwise recipe between line breaks in code, ex. 
```
$ sudo zypper dup
```
* short snippets of code or not part of a stepwise recipe can be inlined as in "... run `sudo zypper dup` before anything else".
* first occurrence(s) of technical terms: italics
