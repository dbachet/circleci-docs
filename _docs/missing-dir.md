---

title: A directory is missing from your repository
layout: doc

---

Rails expects certain directories to be present in your app, including
`log/`, `tmp/` and occasionally others.
If they are not in your repository, your tests will fail (though they ran fine locally because you created the directory a long time ago).

To fix this, simply add the directory to your repository.
Git doesn't allow you to add directories to it, but the standard idiom is instead to add a file called
`.gitkeep` within the directory. So this should solve your problem:

<pre>
$ touch log/.gitkeep
$ git add log/.gitkeep
$ git commit
</pre>

If this solution is unacceptable (perhaps you symlink your log directory in production), you can also add a post-checkout command to create the direction:

<pre>
checkout:
  post:
    - mkdir -p log
</pre>
