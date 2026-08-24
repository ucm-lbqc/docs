---
title: Accounts
next: access
weight: 300
---

Users with access to the cluster are **members** of the LBQC or **collaborators** of a member.

## Requesting an account

Email the laboratory director (see [Team]({{< relref "/team" >}})) and CC the administrator at [fadasme@ucm.cl](mailto:fadasme@ucm.cl).

Include:

- Full name and institutional email
- Whether you are a member or a collaborator (and which member you work with)
- A short description of what you will run
- An end date if the account should expire (required for visitors and students)

Requests are reviewed in about 1–7 business days.
You will receive a username, a temporary password (you must change it at first login), and VPN details if you need off-campus access ([Access]({{< relref "access" >}})).

> [!NOTE]
> Accounts can have an expiration date.
> After that date you will not be able to log in; ask for an extension before it lapses if the project continues.

## Members vs collaborators

| | Home | `$DATA_DIR` |
| --- | --- | --- |
| Member (Unix group = username) | `/home/$USER` | `/data/$USER` |
| Collaborator (Unix group = member) | under that member's home tree | `/data/<member>/collab/$USER` |

Scratch is always `/scratch/$USER` on compute nodes.

Do not share passwords. Collaborators should not write into another user's `$DATA_DIR` except the `collab` tree they were given.

## First login

Connect from the campus network or VPN:

```console
$ ssh <user>@{{% data "server.ip" %}}
```

Change the expired temporary password when prompted.
If you are locked out, contact [fadasme@ucm.cl](mailto:fadasme@ucm.cl).
