git-sha-x
=========

git-sha-x illustrates a potential solution for stronger cryptographic content
verification in Git while keeping SHA1 content pointers.

git-sha-x is available at <https://github.com/sprohaska/git-sha-x>.

git-sha-x computes a hash tree similar to the SHA1-based Git history but using
a different hash function.  The hashes can be added to the commit message and
signed with GPG to confirm the tree and the entire history with a cryptographic
strength that no longer depends on SHA1 but on the chosen git-sha-x hash and
the GPG configuration.  See `git-sha-x --help`.  Examples:

```
git-sha-x commit HEAD
git-sha-x tree HEAD
git-sha-x --sha512 commit HEAD

git-sha-x amend && git-sha-x --sha512 amend && git commit -S --amend -C HEAD
```

git-sha-x is only a proof of concept to illustrate the idea.  I do not intend
to develop it further.

If a similar approach was chosen for Git, the hashes could be managed by Git
and somehow represented in its object store as supplementary information.  Git
could incrementally compute additional hashes while it constructs history and
verify them when transferring data.

The strength of bare SHA1 ids is obviously not increased.  The strength is only
increased if the additional hashes are communicated in a verifiable way, too.
GPG signatures are one way.  Another way could be to communicate them via
a secure channel and pass them to git fetch for verification.  Assuming such an
implementation, a fetch for a commit from this repo could look like:

```bash
git fetch origin \
    --sha256=8a3c72de658a4797e36bb29fc3bdc2a2863c04455a1b394ed9331f11f65ba802 \
    --sha512=729de81500ce4ad70716d7641a115bd0a67984acc4d674044b25850e36d940bf631f9f6aa881111768743690545ac899888fb54f65840f84853f9a8811aeb9ca \
    ef2a4b7d216ab79630b9cd17e072a86e57f044fa
```

For practical purposes, supplementary hashes in the commit in combination with
GPG signatures should provide sufficient protection against attackers that try
to manipulate SHA1s.  For convenience, supplementary hashes could be stored in
the commit header, similar to `gpgsig`.  A hypothetical commit object could
look like:

```
tree 365c7e42fd004a1778c6d79c0437f970397a59b8
parent c2bfff12099b32425a3bcc4d0c7e6e6a169392d8
tree-sha256 2f588b9308b5203212d646fb56201608449cb4d83a5ffd6b7e6213d175a8077c
parent-sha256 090d9a3e69aa3369efac968abde859a6e42d05b631ece6d533765a35e998336c
tree-sha512 12ae91b23733d52fa2f42b8f0bb5aeaeb111335688f387614c3b108a8cb86fa0e2cd6d19bf050f8a9308f8c1e991080507c91df53e0fc4cace3f746ec89a789a
parent-sha512 d319889a40cf945d8c61dbe6d816e10badd49845c547df85ace4327676275eeb5ba2cd962712ddbb8f08f2db17dbc9eb46b59b5f7b7a7e05eab7df0ef89dec65
author Steffen Prohaska <prohaska@zib.de> 1488122961 +0100
committer Steffen Prohaska <prohaska@zib.de> 1488123452 +0100
gpgsig ...
```

GPG signatures would automatically cover the supplementary hashes.
Verification code paths would have to be added to compute the hashes from the
content to confirm that it has not been tampered with.

Since content verification would become independent from the content address,
the interpretation of the content address could be changed in the future.  The
size of 160 bits could be kept for simplicity.  But the meaning could be
changed.  For example, the first 160 bits of SHA256 could be uses as the
content address.  The remaining bits could be stored in an object supplement.
Verification code paths would combine the content address with the additional
bits to verify the SHA256.  Content pointers would keep their size.  Only the
additional SHA256 bits would be stored and used for verification.
