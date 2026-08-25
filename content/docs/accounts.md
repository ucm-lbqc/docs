---
title: Accounts
next: access
weight: 300
---

Access to the cluster is limited to **members** of the LBQC and to **collaborators** of a member (students, visitors, or external partners working with that member).

## Requesting an account

Email the laboratory director (see [Team]({{< relref "/team" >}})) and CC the administrator at [fadasme@ucm.cl](mailto:fadasme@ucm.cl).

Include:

- Full name and institutional email
- Whether you are a member or a collaborator, and which member you work with
- A short description of what you will run
- An end date if the account should expire (required for visitors and students)

Requests are reviewed in about 1–7 business days.
You will receive a username, a temporary password (you must change it at first login), and VPN details if you need off-campus access ([Access]({{< relref "access" >}})).

> [!NOTE]
> Accounts can have an expiration date.
> After that date you will not be able to log in; ask for an extension before it lapses if the project continues.

## Members and collaborators

A **member** has a Unix group equal to their username. Their directories sit at the top of the trees:

- Home: `/home/$USER`
- Data: `/data/$USER`

A **collaborator** is created under a member. Their directories live inside that member’s `collab` folders:

- Home: `/home/<member>/collab/$USER`
- Data: `/data/<member>/collab/$USER`

In both cases the environment variables `$HOME` and `$DATA_DIR` already point to the right places; prefer those names in scripts so you do not hard-code another user’s path.
Scratch is always `/scratch/$USER` on each compute node.

How to use these locations (what to put where, lifetime, backups): [Storage]({{< relref "storage" >}}).

Do not share passwords.
Collaborators should work only in the `collab` tree they were given, not in the member’s own `$DATA_DIR`.

## First login

Connect from the campus network or VPN:

```console
$ ssh <user>@{{% data "server.ip" %}}
```

Change the expired temporary password when prompted.
If you are locked out, contact [fadasme@ucm.cl](mailto:fadasme@ucm.cl).
