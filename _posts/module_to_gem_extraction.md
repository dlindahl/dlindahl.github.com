---
title: Extracting a Ruby Module into a Gem
---

# DRAFT

# Extracting a Ruby Module into a Gem

**Disclaimer: I HAVE NO IDEA WHAT I AM DOING.**

How to use git to extract a Module living in `lib/` of your monorail application into its own gem, saving its commit
history and including its tests.

If you are like me, your module probably lives in the following directory structure:

    lib/
      conductor.rb
      conductor/
        headgear.rb
        demeanor.rb
        demeanor/
          laxidasical.rb
          enthusiastic.rb
          comic.rb
      railcar.rb
      railcar/
        manufacturer_origin.rb
        manufactyrer_origin/
          italy.rb
          germany.rb
    spec/
      models/
        conductor_spec.rb
        conductor/
          demeanor_spec.rb
        railcar_spec.rb
        railcar/
          italy_spec.rb
          germany_spec.rb

Most StackOverflow solutions only solve for the sub-directories. Nothing allowed
the ability to include the root module file (i.e. `lib/conductor.rb`)

### Clone the monorail application repo

Start with a clean copy of your application. There is a good chance you're about to mess
things up real badly:

    git clone --no-hardlinks git@github.com:jdoe/monorail.git extract_conductor
    cd extract_conductor

### Extract all the folders we care about:

Pull your `lib/` and `spec/models` folders into their own separate branches:

    git filter-branch -f --tag-name-filter cat --subdirectory-filter lib --prune-empty lib-extraction
    git filter-branch -f --tag-name-filter cat --subdirectory-filter spec/models --prune-empty spec-models-extraction

Repeat as neccessary.

If you checkout those branches, you'll see that the contents of `lib/` and `spec/models` are now in the root directory
of the repo, including some modules that we don't care about. Not quite what we want, but it solves the problem of
losing our `lib/conductor.rb` file.

### Remove unwanted modules

We now have more than what we want, so let's ditch the modules we aren't extracting:

    git checkout lib-extraction
    git filter-branch -f --index-filter 'git rm -r --cached --ignore-unmatch railcar' --prune-empty
    git filter-branch -f --index-filter 'git rm -r --cached --ignore-unmatch railcar.rb' --prune-empty
    ... etc. until you are left with just the files you want.

    git checkout spec-models-extraction
    git filter-branch -f --index-filter 'git rm -r --cached --ignore-unmatch railcar' --prune-empty
    git filter-branch -f --index-filter 'git rm -r --cached --ignore-unmatch railcar_spec.rb' --prune-empty

If I understand it correctly, the above command goes through the entire commit history and removes all references to the
files you delete through the `git rm -r` sub-command thingie.

Did I mention that I have no idea what I'm doing here?

You should now be left with two branches. One for `lib/` and one for `spec/models` that only contain your module.

### Rebuilt the directory structure

As previously mentioned, the unfortunate effect of filter-branch is that it removes the `lib` and `spec` folders. To make
the merge procedure we'll make later easier, let's rebuild that directory structure:

    git checkout lib-extraction
    mkdir lib
    git mv conductor* lib/.
    git add .
    git commit -m "Gem extraction: Moved Conductor's lib files into their own `lib` folder."

    git checkout spec-models-extraction
    mkdir spec
    git mv conductor* spec/.
    git commit -m "Gem extraction: Moved Conductor's spec files into their own `spec` folder."

### Create your Gem

Sweet. Both of our branches now have the files we want and in a directory structure we want. Let's make our gem.

    cd ..
    bundle gem conductor
    cd conductor

By default, Bundler will create your lib files for you. If you commit those now, they will take priority over the merge
you are about to do, which would make this whole excercise useless. So let's stash those away for a minute:

    git rm --cached lib
    git commit -m "Initial commit."
    git add lib
    git stash save "Initial lib files."

### Pull in your extracted code.

Great. We now have a proper git repo with the standard set of files required for a gem. Let's get our code into this
puppy.

First, let's create a branch in case we mess something up:

    git branch import master

Now let's connect your new gem repo to your old and busted application repo where the extracted code is waiting:

    git remote add ~/Work/monorail conductor-extraction

This creates a link called "conductor-extraction" from your gem repo to your monorail application's repo. Call it
whatever you want.

    git pull conductor-extraction lib-extraction
    git pull conductor-extraction spec-models-extraction

Voila! Your extracted code is now in your gem's repo!

Poke around, kick the tires, make sure everything looks as it should. If it doesn't, you messed something up, not me!

### Merge into master

If everything looks OK, merge everything into master.

    git checkout master
    git merge import

### Bring back the stashed files.

A gem isn't a gem unless it has a version. Either create the files manually, or retrieve the previously stashed files:

    git stash pop

  You will almost certainly run into a merge conflict in `lib/conductor.rb`. Resolve appropriately.

That's it! Push and release unto the world!

#### References

http://stackoverflow.com/questions/359424/detach-subdirectory-into-separate-git-repository
http://stackoverflow.com/questions/4669795/git-surgery-splitting-out-a-single-repository-into-many-repositories
http://help.github.com/remove-sensitive-data/

#### Questions? Corrections?

File a GitHub Issue or submit a Pull Request! :+1:

    











