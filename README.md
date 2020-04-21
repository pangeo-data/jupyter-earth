# jupytearth-meta


## Build the documentation

The documentation for this repository is built using the beta version of
[Jupyter Book](https://beta.jupyterbook.org). It will be automatically updated
any time changes are made to the `docs/` folder and pushed to master.

To build the documentation locally, run these steps:

* **Install the Jupyter Book 2.0 CLI**:

  ```
  pip install -r requirements.txt
  ```
* **Build the `docs/` folder**:

  ```
  jupyter-book build docs/
  ```

This will create a folder in `docs/_build/html` where you can preview your
site. For example, with `chrome docs/_build/html/index.html`.
