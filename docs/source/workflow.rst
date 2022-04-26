Workflow
============

NEXT software is in the Github repository `next-exp <https://github.com/next-exp/>`_. This page englobes the information for users and developers related to the git workflow that can be applied to any of the main NEXT repositories (:doc:`NEXUS` and :doc:`IC`).
It is based on what was presented in the `IC-crash-course <https://github.com/mmkekic/IC-crash-course/blob/master/presentations/git.pdf>`_, where more detailed information about git basics can be found.

Info for users
------------------------
:guilabel:`Fork` the NEXT repository you are going to use, and clone it in your local computer,

 .. code-block:: text

   git clone git@github.com:user/repository.git

It is recommended to use ssh key to clone repositories. A complete guide about how to generate ssh keys can be found
`here <https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent>`_.
Based on this, you can define:

 * ``upstream`` as the remote of the original **next-exp/repository**.
 * ``origin`` your forked version **user/repository**.
 * and ``local`` your local version, from cloning your fork.

These three should be always at the same level. Keep your code updated!

 .. note::
   **next-exp/IC** repository use lfs (large file storage) for the files that are not related to the code. Github offers an `open source extension <https://git-lfs.github.com/>`_.
   However, in NEXT we are using our own `lfs server at IFIC <https://next.ific.uv.es:8888/users/sign_in>`_ to storage this files. Note that you would need your own account
   even manipulating with our branch. In case of question, contact us.

 .. image:: images/workflow.png
   :width: 850


Info for developers
------------------------------------
In case you want to become a NEXT developers, you would need to follow the next procedure whenever you want to add a change in the main repository:

 * Any code change should be made in a development branch (``dev_branch``). First created locally, then :guilabel:`Push` to ``origin/dev_branch``, and then requested its merge to ``upstream`` via a :guilabel:`Pull Request` (**PR**, GitHub feature).
 * Your **PR** will be reviewed by other software developers.
 * When/before **PR** approved, the ``dev_branch`` should be rebased onto ``upstream/master``.
 * Once it is approved, it will be merged with ``upstream/master`` -> merging is only done by designated persons:

    * **IC**: Carmen R, Miryam M-V, Helena A.
    * **NEXUS**: Paola F.

 * You can delete your ``dev_branch`` locally and remotely.
