---
title: A Short Virtual Environment Primer
date: 2020-03-05T08:31:39-08:00
---

Virtual envs create a lightweight copy of your current Python binary and a clean, separate site packages folder.

To create a virtual environment, use the venv module:

```bash
ubuntu@ubuntu:~$ mkdir foo && cd foo
ubuntu@ubuntu:~/foo$ python3 -m venv my_env
ubuntu@ubuntu:~/foo$ ls -al
.  ..  my_env
```

The my_env folder here contains a lightweight copy of your current python distribution, with only basic packages installed

```bash
ubuntu@ubuntu:~/foo$ ls my_env
bin  include  lib  lib64  pyvenv.cfg  share
```

The my_env/bin directory contains scripts for activating the virtual environment, binaries for pip and easy_install, and symlinks to your system python.

```bash
ubuntu@ubuntu:~/foo$ ls -al my_env/bin
total 12
drwxrwxrwx 1 ubuntu ubuntu  512 Mar  4 15:27 .
drwxrwxrwx 1 ubuntu ubuntu  512 Mar  4 15:27 ..
-rwxrwxrwx 1 ubuntu ubuntu 2227 Mar  4 15:27 activate
-rwxrwxrwx 1 ubuntu ubuntu 1283 Mar  4 15:27 activate.csh
-rwxrwxrwx 1 ubuntu ubuntu 2447 Mar  4 15:27 activate.fish
-rwxrwxrwx 1 ubuntu ubuntu  315 Mar  4 15:27 easy_install
-rwxrwxrwx 1 ubuntu ubuntu  315 Mar  4 15:27 easy_install-3.6
-rwxrwxrwx 1 ubuntu ubuntu  287 Mar  4 15:27 pip
-rwxrwxrwx 1 ubuntu ubuntu  287 Mar  4 15:27 pip3
-rwxrwxrwx 1 ubuntu ubuntu  287 Mar  4 15:27 pip3.6
lrwxrwxrwx 1 ubuntu ubuntu    7 Mar  4 15:27 python -> python3
lrwxrwxrwx 1 ubuntu ubuntu   16 Mar  4 15:27 python3 -> /usr/bin/python3
```

The virtual environment does nothing to your system until you `activate` it using the relevant script for your shell.

```bash
ubuntu@ubuntu:~/foo$ source my_env/bin/activate
(my_env) ubuntu@ubuntu:~/foo$
```

Activating the virtualenv does a few things and is fully reversible with deactivate

- Prepends my_env/bin to $PATH
- Exports the VIRTUAL_ENV variable
- Tries to add a marker to the shell prompt with the environment's name

Running `deactivate` from the shell will revert any activated virtual env. If you activated a virtual environment from another virtual environment, it will revert to the base.

```bash
ubuntu@ubuntu:~/foo$ source my_env/bin/activate
(my_env) ubuntu@ubuntu:~/foo$ python3 -m venv venv
(my_env) ubuntu@ubuntu:~/foo$ source venv/bin/activate
(venv) ubuntu@ubuntu:~/foo$
(venv) ubuntu@ubuntu:~/foo$ deactivate
ubuntu@ubuntu:~/foo$
```

Packages installed from apt or pip outside of the virtualenv are not accessible inside

```bash
ubuntu@ubuntu:~/foo$ pip3 install mypy
...
ubuntu@ubuntu:~/foo$ python3 -m mypy --version
mypy 0.560
ubuntu@ubuntu:~/foo$ source my_env/bin/activate
(my_env) ubuntu@ubuntu:~/foo$ python3 -m mypy --version
/home/ubuntu/foo/my_env/bin/python3: No module named mypy
```

and

```bash
ubuntu@ubuntu:~/foo$ sudo apt install python3-mypy
...
ubuntu@ubuntu:~/foo$ python3 -m mypy --version
mypy 0.560
ubuntu@ubuntu:~/foo$ source my_env/bin/activate
(my_env) ubuntu@ubuntu:~/foo$ python3 -m mypy --version
/home/ubuntu/foo/my_env/bin/python3: No module named mypy
```

> ## Gotcha
>
> If any script or your .bashrc modifies $PATH or $PYTHONPATH before or after activating the environment, those changes will be included in the virtual environment. Likewise, deactivating the virtual environment will undo the effect of scripts run while activated.
>
> For instance, sourcing a ROS distribution in .bashrc means those packages will remain accessible even inside the virtual envrionment. I recommend not doing this, as it can hide dependencies you left out of your `requirements.txt`.

When in a virtual environment, all packages should be installed with pip3 (aliased to pip for convenience).

```bash
(my_env) ubuntu@ubuntu:~/foo$ pip install mypy
...
(my_env) ubuntu@ubuntu:~/foo$ python3 -m mypy --version
mypy 0.761
(my_env) ubuntu@ubuntu:~/foo$ ls my_env/lib64/python3.6/site_packages
97d839f88ac1187b7870__mypyc.cpython-36m-x86_64-linux-gnu.so  pkg_resources
easy_install.py                                              pkg_resources-0.0.0.dist-info
mypy                                                         __pycache__
mypy-0.761.dist-info                                         setuptools
mypyc                                                        setuptools-39.0.1.dist-info
mypy_extensions-0.4.3.dist-info                              typed_ast
mypy_extensions.py                                           typed_ast-1.4.1.dist-info
pip                                                          typing_extensions-3.7.4.1.dist-info
pip-9.0.1.dist-info                                          typing_extensions.py
```

The best practice is to collect the dependencies you care about into a requirements.txt file in your project.

```bash
(my_env) ubuntu@ubuntu:~/foo$ cat requirements.txt
mypy==0.761
```

If you started installing packages without maintaining a `requirements.txt`, you can use `pip freeze` to output the current environment in its entirety to construct one. Be aware that all dependencies will be included in this output, which makes the file less useful to a human reader who may want to glean the top level dependencies.

```bash
(my_env) ubuntu@ubuntu:~/foo$ pip freeze > requirements.txt
(my_env) ubuntu@ubuntu:~/foo$ cat requirements.txt
mypy==0.761
mypy-extensions==0.4.3
pkg-resources==0.0.0
typed-ast==1.4.1
typing-extensions==3.7.4.1
```

> Extra packages may appear here if PYTHONPATH was modified elsewhere, as stated in the 'Gotcha' above.
>
> In the worst case, use `$ env -i bash --norc --noprofile` to start a new shell and then activate the environment to export a clean version of your virtual environment.

Pip can intake `requirements.txt` seamlessly. You can use this to duplicate, backup, and restore environments.

```bash
(my_env) ubuntu@ubuntu:~/foo$ pip install -r requirements.txt
```

`requirements.txt` should be version controlled and included with the source.
