# Based on Jethro's Braindump

This braindump is generated via [ox-hugo][ox-hugo] and uses the
[cortex][cortex] theme.

## Installation instructions

Clone this repository.

Install [hugo][hugo]. E.g., on a Mac with Homebrew:

    $ brew install hugo

Make sure the submodule containing the Hugo theme is installed:

    $ git submodule init
    $ git submodule update

Now run hugo to generate the files (find them in `/public`):

    $ hugo

Or run the following to get an immediately browsable website on localhost:

    $ hugo serve

[hugo]: https://gohugo.io/
[ox-hugo]: https://github.com/kaushalmodi/ox-hugo
[cortex]: https://github.com/jethrokuan/cortex
[org]: https://github.com/jethrokuan/braindump/tree/master/org


## Additional setup

First, we need some extra functions:

Creating a custom link type to correctly handle the `hugo` expected
format.

```emacs-lisp
(org-add-link-type "braindump" nil
                   '(lambda (path desc frmt)
                      (format "[%s]({{< relref \"/posts/%s\" >}} \"%s\")" desc path desc)))
```

Now, we need to process each file before `org` export to `markdown`
and find all their backlinks.

```emacs-lisp
(defun bk/org-roam--backlinks-list (file)
  "Find links referring to FILE."
  (if (org-roam--org-roam-file-p file)
      (--reduce-from
       (concat acc
               (let ((fld (car (split-string (file-relative-name (car it) org-roam-directory) ".orgr"))))
                 (format "- [[braindump:%s][%s]]\n"
                         fld
                         (org-roam--get-title-or-slug (car it)))))
       "" (org-roam-db-query [:select [from] :from links :where (= to $s1)] file))
    ""))
```

Once found, let's insert it in a specific heading. (The heading name
is important to CSS styling by the template)

```emacs-lisp

(defun bk/org-export-preprocessor (backend)
  "BACKEND passed by `org-export-before-processing-hook'."
  (let ((links (bk/org-roam--backlinks-list (buffer-file-name))))
    (save-excursion
      (goto-char (point-max))
      (insert (concat "\n* Links to this note\n"))
      (insert links))))

(add-hook 'org-export-before-processing-hook 'bk/org-export-preprocessor)
```

Also, in the content body of all your notes, you probably have more
links to other files. Let's change those as well:

```emacs-lisp
(defun bk/replace-file-handle (_backend)
  (save-excursion
    (goto-char (point-min))
    (while (re-search-forward "\\(file:\\).+\.org" nil t)
      (replace-match "braindump:"))))

(add-hook 'org-export-before-processing-hook 'bk/replace-file-handle)
```

Great, almost good to go!

Now, go in each `org-roam` file you have and add the following header:

```org
#+HUGO_BASE_DIR: ~/open-source/braindump
```

The path must match the directory where you cloned this project. Now,
you can use `ox-hugo` (do not forget to install it) and export your
note using `C-c C-e H A` and a new markdown file will be created at
`braindump/content/posts/`.

There is also a way to automatically export as you save your notes
using `org-hugo-auto-export-mode`.


### Customizations

Change the file `content/_index.md` to your liking.
