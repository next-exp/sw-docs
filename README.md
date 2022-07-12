# sw-docs

This repository englobes all the information related to the **NEXT Software Documentation** that you can find on

https://next-exp-sw.readthedocs.io/

# How to contribute

This repository, as the others in next-exp, follow a specific git workflow. More information about it, can be found here: https://next-exp-sw.readthedocs.io/en/latest/workflow.html.

When contribution to documentation, you can check your read-the-docs status in your local computer before pushing the changes to the local branch. In order to do so, you need to have `sphinx` installed, together with an extension (`sphinx-autodoc-typehints`) and the sphinx theme (`sphinx_rtd_theme`). The three of them can be instaled by `pip install`. Afterwards, you can run 

$ make html

in the `sw-docs/docs/` directory. You can check your documentation in your browser typing:

`file:///path_to_repository/sw-docs/docs/build/html/index.html`

**NOTE: It is recommended to check the html using incognito mode in your browser. Otherwise, cookies need to be erased everytime a new change is added.**

Whenever a new Pull Request (PR) is added in this repository, a read-the-docs link should created to simplify the review. Read-the-docs links are created signing in on the https://readthedocs.org/ webpage with your GitHub account. There, the user should import a new project from the forked repository (`user_name/sw-docs`) using the PR branch and the following name

`prXXX-sw-docs.readthedocs.io`

where `XXX` corresponds to the number of the PR. 
**IMPORTANT: path should be erased once the PR is approved**.

If you have any comment, suggestions, or you would like to contribute, don't hesitate to reach me ( @halmamol )!
