---
title: "How to batch sign previous commits"
description: ''
pubDate: 'May 18 2024'
---

Find the earliest commit that has not been signed yet using `git log --show-signatures` (you may want to use `--reverse` flag to start from the initial commit).

Use the hash you got from the first part and insert that commit's hash to `<HASH>` in the command below.

```sh
$ git rebase --exec 'git commit --amend --no-edit -n -s' -i <HASH>~1
```

This will sign all of your commits starting from `<HASH>` to the most current commit.

---

If you want to sign all of the commits from the initial commit to the most current one, you can simply use `--root` flag instead of inserting the `<HASH>`.

```sh
$ git rebase --exec 'git commit --amend --no-edit -n -s' -i --root
```

---

## Reference
- [Signing commits — Hyperledger Indy SDK documentation](https://hyperledger-indy.readthedocs.io/projects/sdk/en/latest/docs/contributors/signing-commits.html#:~:text=If%20you%20need%20to%20re%2Dsign%20a%20bunch%20of%20previous,s'%20%2Di%20HASH%60)