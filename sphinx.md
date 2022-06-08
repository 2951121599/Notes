## sphinx

```shell
sphinx-build --version
sphinx-quickstart docs
sphinx-build -b html docs/source/ docs/build/html
```

```shell
D:\Sphinx>sphinx-build --version
sphinx-build 5.0.0

D:\Sphinx>sphinx-quickstart docs
Welcome to the Sphinx 5.0.0 quickstart utility.

Please enter values for the following settings (just press Enter to
accept a default value, if one is given in brackets).

Selected root path: docs

You have two options for placing the build directory for Sphinx output.
Either, you use a directory "_build" within the root path, or you separate
"source" and "build" directories within the root path.
> Separate source and build directories (y/n) [n]: y

The project name will occur in several places in the built documentation.
> Project name: Note
> Author name(s): 2951121599
> Project release []: 0.1

If the documents are to be written in a language other than English,
you can select a language here by its language code. Sphinx will then
translate text that it generates into that language.

For a list of supported codes, see
https://www.sphinx-doc.org/en/master/usage/configuration.html#confval-language.
> Project language [en]:

Creating file D:\Sphinx\docs\source\conf.py.
Creating file D:\Sphinx\docs\source\index.rst.
Creating file D:\Sphinx\docs\Makefile.
Creating file D:\Sphinx\docs\make.bat.

Finished: An initial directory structure has been created.

You should now populate your master file D:\Sphinx\docs\source\index.rst and create other documentation
source files. Use the Makefile to build the docs, like so:
   make builder
where "builder" is one of the supported builders, e.g. html, latex or linkcheck.


D:\Sphinx>sphinx-build -b html docs/source/ docs/build/html
Running Sphinx v5.0.0
making output directory... done
building [mo]: targets for 0 po files that are out of date
building [html]: targets for 1 source files that are out of date
updating environment: [new config] 1 added, 0 changed, 0 removed
reading sources... [100%] index
looking for now-outdated files... none found
pickling environment... done
checking consistency... done
preparing documents... done
writing output... [100%] index
generating indices... genindex done
writing additional pages... search done
copying static files... done
copying extra files... done
dumping search index in English (code: en)... done
dumping object inventory... done
build succeeded.

The HTML pages are in docs\build\html.

D:\Sphinx>
```

```python
# 推荐使用主题 sphinx_rtd_theme
# 安装主题 在conf.py中导入并使用
pip install sphinx_rtd_theme

import sphinx_rtd_theme

# html_theme = 'alabaster'
html_theme = 'sphinx_rtd_theme'
```

```shell
# 在Makefile同级目录运行
make html  # 生成html文件
make latexpdf  # 生成pdf文件
```

