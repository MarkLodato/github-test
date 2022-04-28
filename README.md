# GitHub Features Test

This repository exists for me to test some features of GitHub.

## Refs other than branches and tags

Q: Can GitHub support refs that are neither branches (refs/heads/...) or tags
(refs/heads/...)?

A: **YES**

Setup:

```bash
$ echo 'This is not a branch or tag.' | git hash-object -w --stdin
43bf49d99f03a69124d5c071516a97cdd18fd331
$ printf '100644 blob 43bf49d99f03a69124d5c071516a97cdd18fd331\tREADME.md\n' | git mktree
bc6f7249ff10022298c77b1110afdbfed205cfaa
$ printf 'tree bc6f7249ff10022298c77b1110afdbfed205cfaa\nauthor x <x> 0 +0000\ncommitter x <x> 0 +0000\n\n'| git hash-object -t commit --stdin -w
8c706033792f6152b108d9e1342e607433be0396
$ git update-ref refs/other/blob 43bf49d99f03a69124d5c071516a97cdd18fd331
$ git update-ref refs/other/tree bc6f7249ff10022298c77b1110afdbfed205cfaa
$ git update-ref refs/other/commit 8c706033792f6152b108d9e1342e607433be0396
```

Writing works:

```bash
$ git push origin 'refs/other/*:refs/other/*'
```

Reading works via the git protocol:

```bash
$ git fetch origin '+refs/other/*:refs/other-remotes/origin/*'
$ git show-ref refs/other-remotes/origin/{blob,tree,commit}
43bf49d99f03a69124d5c071516a97cdd18fd331 refs/other-remotes/origin/blob
8c706033792f6152b108d9e1342e607433be0396 refs/other-remotes/origin/commit
bc6f7249ff10022298c77b1110afdbfed205cfaa refs/other-remotes/origin/tree
```

Reading via the website does not. (Interestingly, you can view the commit and
the README but nothing else.)

*   Semi-works: https://github.com/MarkLodato/github-test/tree/refs/other/commit
*   Doesn't work: https://github.com/MarkLodato/github-test/tree/refs/other/tree
*   Doesn't work: https://github.com/MarkLodato/github-test/blob/refs/other/blob
*   Doesn't work: https://github.com/MarkLodato/github-test/blob/refs/other/blob
*   Doesn't work: https://raw.githubusercontent.com/MarkLodato/github-test/refs/other/commit/README.md
*   Doesn't work: https://raw.githubusercontent.com/MarkLodato/github-test/refs/other/tree/README.md
*   Doesn't work: https://raw.githubusercontent.com/MarkLodato/github-test/refs/other/blob

Reading via the GitHub Git Database API does:

```bash
$ curl https://api.github.com/repos/MarkLodato/github-test/git/ref/other/blob
{
  "ref": "refs/other/blob",
  "node_id": "REF_kwDOHQDYha9yZWZzL290aGVyL2Jsb2I",
  "url": "https://api.github.com/repos/MarkLodato/github-test/git/refs/other/blob",
  "object": {
    "sha": "43bf49d99f03a69124d5c071516a97cdd18fd331",
    "type": "blob",
    "url": "https://api.github.com/repos/MarkLodato/github-test/git/blobs/43bf49d99f03a69124d5c071516a97cdd18fd331"
  }
}
$ curl https://api.github.com/repos/MarkLodato/github-test/git/ref/other/tree
{
  "ref": "refs/other/tree",
  "node_id": "REF_kwDOHQDYha9yZWZzL290aGVyL3RyZWU",
  "url": "https://api.github.com/repos/MarkLodato/github-test/git/refs/other/tree",
  "object": {
    "sha": "bc6f7249ff10022298c77b1110afdbfed205cfaa",
    "type": "tree",
    "url": "https://api.github.com/repos/MarkLodato/github-test/git/trees/bc6f7249ff10022298c77b1110afdbfed205cfaa"
  }
}
$ curl https://api.github.com/repos/MarkLodato/github-test/git/ref/other/commit
{
  "ref": "refs/other/commit",
  "node_id": "REF_kwDOHQDYhbFyZWZzL290aGVyL2NvbW1pdA",
  "url": "https://api.github.com/repos/MarkLodato/github-test/git/refs/other/commit",
  "object": {
    "sha": "8c706033792f6152b108d9e1342e607433be0396",
    "type": "commit",
    "url": "https://api.github.com/repos/MarkLodato/github-test/git/commits/8c706033792f6152b108d9e1342e607433be0396"
  }
}
```
