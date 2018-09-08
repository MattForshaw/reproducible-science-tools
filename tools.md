# Tools for reproducible research

This document will walk you through setting up your research code so that it
is easy to use and maintain. As a side effect it will also be reproducible.

The set of tools highlighted here are an opinionated choice. There are others
which you might prefer. Use those instead!

To keep this guide applied and concrete it will use a toy research project to
illustrate things. The "research" code is kept minimal and sometimes is a bit
artificial. The focus is on the tools, not the research code.

The guide will cover:

* setting up a simple GitHub/GitLab repository to host your work.
* specifying dependencies required to run your code
* building the environment to execute your code in
* fetching and storing data
* automated archiving of your code
* using Jupyter notebooks as fancy scripts
* automating your sanity checks


## Julia, Python or R

Most advice applies to any programming language. For simplicity we will stick
with Python with some brief excursions to R. If you are an expert in Julia (or
another language) you will probably know how to translate the advice here
to your universe.


## GitHub and GitLab

Both GitHub and GitLab are public websites that let you host your code,
its history, do project management, and collaborate with others. Both have
features that are only available to paying customers. Your university might
have a subscription to one of them.

GitHub is the defacto standard used by the vast majority of open-source
projects.

We will use GitHub today.

**Exercise:** create an account on github.com

**Exercise:** create a new repository called "zurich-bikes". Make sure to tick
the box for `Initialize this repository with a README`.

**Exercise:** create a file called `hello.py` via the web interface
with `print("Hello world")` on the first line.


### Run it!

One key take away from this guide is that you should try things as soon as
possible. The longer you wait the more changes there will be between the last
version that worked and your current one. This makes it much harder to figure
out why things are broken.

To run our code we will use https://mybinder.org a public infrastructure that
makes it easy to take a Git repository and run it from your browser. It is
powered by open-source software called BinderHub. You or your university
can setup your own copy of mybinder.org if you want to.

Head over to https://mybinder.org to get going.

The interface you see let's you specify the repository you want to start.

**Exercise**:

* Type the URL of your repository into the "GitHub repo or URL" box (should be
  something like github.com/YOURUSERNAME/zurich-bikes/)
* change the dropdown in the "Path to a notebook file" field from "File" to
  "URL" and type `/lab` into the field
* As you type the URL the webpage will generate a link you can share with
  others in the "Copy the URL below..." box. It should look something like: mybinder.org/v2/gh/YOURUSERNAME/zurich-bikes/master?urlpath=%2Flab
* Copy it, open a new tab and visit that URL.

> Tim's example link looks like this
> https://mybinder.org/v2/gh/betatim/zurich-bikes/master?urlpath=%2Flab

When you first visit that URL you will see a loading page. In the background
several things are happening while you wait. Binder is:

* fetching your repository from github.com,
* analysing the contents of the repository,
* creating a Docker image based on what it finds,
* launching the newly created image for you in the cloud, and
* finally connecting you to it via your browser.

Once it launched you should see the Jupyter lab interface. This is the new version
of the classic Jupyter notebook. It lets you open a terminal and execute code.

**Exercise:** open a new terminal and execute our small `hello.py` file with
`python hello.py`.


## Dependencies

To analyse our bike data we will need some extra libraries. There are several
package managers for Python, two of the most popular ones are `pip` and
`conda`.

### pip

Almost any Python package can be installed with `pip`. To install pandas and
matplotlib you would run `pip install pandas matplotlib`. However if you install
dependencies by hand like this you do not create a record of what was installed.

Instead the Python community adopted the convention of having a file called
`requirements.txt` which lists all packages that are required for your project.
The format of `requirements.txt` is very simple:

```
pandas==1.15.1
matplotlib==2.2.3
```

You can install all dependencies from a `requirements.txt` with
`pip install -r requirements.txt`. One important thing that people often forget
is to specify the versions of the packages you want to have installed.

You should only specify direct dependencies. In this example Numpy will be
installed automatically because it is a dependency of pandas. Unless you depend
on a specific version of Numpy you should not specify it in your
`requirements.txt`.

You can find [more details on `pip` in the documentation](https://pip.pypa.io/en/stable/user_guide/#id1).


### conda

One drawback of `pip` is that it can only install Python packages. The `conda`
package manager let's you install packages in any language -- Python, R, Ruby,
Lua, Scala, Java, JavaScript, C/C++, FORTRAN. It is very popular in the Python
world.

The `conda` command is part of Anaconda and Miniconda. These are two widely
used Python distributions.

If you have `conda` installed you can use it by running `conda install numpy`.
This will install Numpy.

The conda equivalent of `requirements.txt` is called `environment.yml`.

It looks a little different:
```
name: reproducible-science-tools
channels:
  - conda-forge
  - defaults
dependencies:
  - numpy=1.14.3
```

**Exercise:** create an `environment.yml` in your GitHub repository that installs
pandas and matplotlib.

A nice feature of conda is that you can easily create environments that are
completely separate from each other. This is particularly useful if you have
more than one project. It means that you can install different versions of
the same package for each project or update the libraries you use for one
without worrying about breaking the other project.

The conda documentation has a good section on [managing conda environments](https://conda.io/docs/user-guide/tasks/manage-environments.html).

> conda or pip?
>
> I prefer conda because you can manage dependencies that are not Python
> packages as well. I find that with virtualenvs, the pip equivalent of conda
> environments the separation from the rest of the system isn't as good.


## Cross-platform dependency management

Sometimes there is no conda package for the software you want to install, or it
is easier to install it using your operating system's package manager.

One example of this is Octave. The instructions I found for running Octave only
explain how to install it using Ubuntu's package manager.

At home I have a linux desktop computer, however I use a Mac when I am
travelling. Some of the people I collaborate with use Windows. If we want to
make use of Octave in a project we need to find instructions that explain how to
install the same version of Octave on each of these operating systems. This is
a much bigger deal than it at first sounds.

An alternative solution is that we decide to maintain a virtual machine image
that each of us downloads. This way we all get the same environment and we only
need to figure out installation problems once. This is the "kitchen sink"
approach from earlier. Most people now prefer to use lightweight VMs which
are called containers. The most popular implementation is Docker.

> What is Docker?

Creating docker images is a science in itself. Creating good docker images is
an art. To install an extra Ubuntu package into your docker image takes six
lines:
```
RUN apt-get update && \
    apt-get install --yes --no-install-recommends \
       less && \
    apt-get purge && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```
You could use less but you either make the image bigger than needed or get other
weird side effects. If you wanted to do more complex tasks the number of lines
would get even bigger. Luckily there is a pattern to this, so we can
automate the generation of the image!

This is where a tool like repo2docker comes in handy. It lets you specify
dependencies in a way that is easy (`requirements.txt` and so on) and then
builds a docker image for you.


## Optional: Building transferable environments

repo2docker is a tool to build and run Docker images from source code
repositories. repo2docker fetches a repository (from GitHub or a local
directory) and builds a container image based on the configuration files found
in the repository.

The repo2docker documentation can be found at
https://repo2docker.readthedocs.io/en/latest/. Briefly it inspects your directory
for a set of files that it recognises and then uses what it learns from these
files to build the docker container for you.

As an example, when repo2docker sees a file called `apt.txt` it will assume that
it contains the name of Ubuntu packages that it should install. This means that
to install Octave we need to add a file called `apt.txt` with the following
content:
```
octave
octave-symbolic
octave-miscellaneous
gnuplot
ghostscript
```

You can combine this with an `environment.yml` to install tools via conda.

You can use repo2docker on your own laptop, without ever planning on sharing
your work, or when the possibility of sharing is still far away.

It is also exactly the tool that is used by mybinder.org to build the docker
containers for you.

**Exercise:** Add an `apt.txt` to your "zurich-bikes" repository that
installs the dependencies listed above to get Octave installed.

**Exercise:** Add `octave_kernel` to your list of dependencies in the
`environment.yml` of your "zurich-bikes" repository.

**Exercise**: Rebuild this repository on mybinder.org and create a notebook
that uses the Octave kernel. There should now be a new kernel to choose from
called Octave. If you don't know any Matlab to try out take a look at the
[binder-example/octave](https://github.com/binder-examples/octave/blob/master/index.ipynb) repository.


## Main points

The main take away from this section is that you should create a new virtual or
conda environment for every project you start. It costs you virtually nothing to
do so and it will save you a lot of hassle in the future when projects need
different versions of the same library. Or you want to find out if you can
upgrade to a newer version of a library you use.

Write down your dependencies in a `environment.yml` in your repository as you
install packages. Make sure to specify a version when you do this.

Try out your repository on mybinder.org once in a while to make sure you did not
forget a dependency. It sounds simple but it is by far the most common hurdle
I see when trying to run other people's code.

We have been using mybinder.org to do a lot of the heavy lifting for us. It lets
us get started very quickly. However you might want to work on things on your
laptop.

My recommendation for doing anything with Python is to install **miniconda** and
create conda environments for everything (I have over 100 different conda
environments on my laptop).

Install instructions for each operating system. Make sure you follow the
instructions for **miniconda**:
* https://conda.io/docs/user-guide/install/windows.html
* https://conda.io/docs/user-guide/install/macos.html
* https://conda.io/docs/user-guide/install/linux.html

If you need to manage things that are not supported by conda think about
switching to repo2docker. You can do almost everything you'd ever want to do,
it is a tool that has thousands of users, and it is maintained by someone that
is not you (free time!).

To install repo2docker on your computer you need to install Docker. This can
be anything from very easy to very hard. To install it follow the [get started guide](https://www.docker.com/get-started).

After you have installed Docker and made sure it works, [install repo2docker on your laptop](https://github.com/Build-a-binder/build-a-binder.github.io/blob/master/workshop/20-repo2docker.md).


## BREAK TIME

See you back here in 30 minutes.


## Back to counting bikes

We wanted to count bikes in Zurich. Let's get back to that.


## Fetching and storing data



## automated archiving of your code
## using Jupyter notebooks as fancy scripts

Papermill!


## automating your sanity checks