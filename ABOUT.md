# Action: Manage Groups.io lists From GitHub, autogenerate a member directory of group members

This GitHub Action enables you to manage membership of Groups.io lists using a YAML file in a GitHub repo. It also autogenerates a nicely formatted directory.

_**Note:** A Groups.io Premium or Enterprise subscription is required for this to work, as it uses the "Direct Add" functionality of Groups.io._

At a high level, this Action parses a directory of YAML files containing Groups.io subgroup membership data.  It then updates each Groups.io subgroup appropriately, adding and removing list subscribers.

Once the subgroups are managed, it then autogenerates a directory README.md in markdown.  Users can opt to add additional information, such as their roles in the project, their preferred pronouns, where to find them on Twitter or LinkedIn, a profile photo, etc.

In order to manage a subgroup, you must add a moderator account to the group with certain permissions (more below).  To avoid undesirable outcomes, this Action can **only** manage subgroups which contain this moderator.

The YAML file uses a flexible and intuitive structure:


```
minimal-subgroup:
  name: A minimal subgroup
  description: This is an example of the bare minimum information required
  list-members:
    - email: memberEmail@example.com
      name: Member Name
    - email: member2Email@example.com
      name: Member2 Name
...
```

... and so on.  This script will keep group members' emails synced to the relevant subgroups.  Group members with Moderator or Owner status are ignored.

In addition, this Action can manage a meta list containing all of the members.

## Prerequisites

### You need a Premium or Enterprise Groups.io subscription:

The Direct Add functionality is only available to Premium and Enterprise subscribers.  This Action won't work on free accounts.

### You should create a groups.io account with moderator permissions:

You must have a groups.io account with sufficient permissions to add and remove members from the subgroups you plan to manage.  You should create a dummy account for this purpose, because although the credentials are encrypted using GitHub Secrets, you shouldn't be using *your* credentials for GitHub Actions.

At a high level, you will create a new user, and make it a **moderator** on the main list with a limited set of permissions.  You'll then Direct Add that user to each subgroup you want to manage, make it a **moderator** in these subgroups, and again grant it a limited set of permissions.

You may be tempted to just make it a group Owner.  Please don't.  It will work, but *it grants anyone with access to this configuration file the ability to add/remove users from any list in your group*.  You don't want this.

Here's how to do this:

1. Use Direct Add to add a user on **main**, for example (with email delivery disabled):
   * `Subgroup Moderator <youremail+moderator@yourdomain.org> nomail`

1. Once you've created this account, make it a **moderator** in the **main** group, and grant it only these permissions:
   * `Invite Members`
   * `Modify Group Settings`
   * `View Member List`

1. Next, navigate to a Subgroup you want this Action to manage.  Use Direct Add to add the account **to the subgroup**, change it to a **Moderator** within the subgroup, and grant the following permissions only: 
   * `Add Members`
   * `Remove Members`

1. Repeat the previous step with each subgroup you want this file to manage (don't forget the meta list).

### You need to find your internal Group name:

Groups.io uses a special internal name for your group.  It is not the name which appears on the website, nor is it your URL.  You can find it by going to Settings &raquo; Export Group Data.  Select "Group Info" and wait for the download.  In the zip file, look for `group.json` and find the value of `name:`.  That is your group name.

### You need to restrict membership and tell your users they now join the list via GitHub

When a list is managed by the Action, it has to be the *only* thing managing membership.  This is because it adds or removes members *solely* based upon the contents of the YAML file.  You need to configure the subgroup information page to inform users they cannot join through the Groups.io interface, and point them to your YAML file.

1. Under your Subgroup settings, make sure `Restricted membership` is checked.

1. Update the Group Description to include a link to this repo where membership is managed.

## Installation

1. Make a backup of your group membership, just in case.  Do this from the Settings panel of your main group.

1. Clone this repo to a new repo within your organization called `directory`

1. (Optional) Update the text of `groups/assets/INDEX_TEMPLATE.txt.default` if you want.  You should rename the INDEX_TEMPLATE file so it isn't overwritten if you merge in upstream changes.  Keep track of the new filename for the next step.

1. Update `groups/assets/config.yml` with your settings, including the renamed index template.

1. Once everything is ready to go, push the repo.

1. In the `directory` repo on GitHub, create a branch protection rule to prevent pushing directly to `master`.  (This is because the directory is created when you push to a branch, and Groups.io is updated when those changes are merged.)
   1. In the `directory` repo, go to Settings.
   1. Under **Branch protection rules** click *Add rule*.
   1. Branch name pattern is *master*.
   1. Select the boxes for *Require status checks to pass before merging* and *Allow force pushes*.

1. Add the moderator account and password you created above as GitHub Secrets:
   1. In the `directory` repo, go to Settings.
	1. Under **Secrets** click *Add a new secret* and add the following two secrets:

	   Name: `GROUPSIO_USERNAME` Value: `(the moderator username you created earlier)`

	   Name: `GROUPSIO_PASSWORD` Value: `(the moderator password you created earlier)`

## Configuring groups

1. Create your YAML files to match the subgroups you created in advance. You can use [`groups/minimal_group_example.yml.txt`](groups/minimal_group_example.yml.txt), [`groups/full_group_example.yml.txt`](groups/full_group_example.yml.txt) to help you get started. You can define multiple groups in one file, or have multiple files. Any file in `groups/` with a `.yml` extension will be parsed, although files that don't contain the right info will be ignored.

   IMPORTANT: If you are managing a group which already exists, make sure those members are defined in one of the `.yml` files **before** you go to the next step.  Once you start a sync, it will prune anybody who doesn't appear in the YAML file.
   
   The only exception is Groups.io moderators and admins.  They are never pruned, even if they don't appear in a YAML file.

1. Once you're done, open a PR and follow the instructions.

That should be everything you need to get going.  Once you open a PR that changes a `.yml` file in `groups/`, it will attempt to build a directory.  When that branch is merged to master, it will update Groups.io.

Please note that you cannot make any manual changes to the auto-generated directory.  When this Action rebuilds the README.md and directory files, it will overwrite anything which is there.

## Contact info

If you have an issue and it is not security-related, please open an issue at [https://github.com/brianwarner/manage-groupsio-lists-from-github-action](https://github.com/brianwarner/manage-groupsio-lists-from-github-action).  If it is security-related, please contact me directly at <bwarner@linuxfoundation.org>.

---

**Brian Warner** | [The Linux Foundation](https://linuxfoundation.org) | <bwarner@linuxfoundation.org>, <brian@bdwarner.com> | [@realBrianWarner](https://twitter.com/realBrianWarner)
