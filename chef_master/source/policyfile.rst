=====================================================
About Policyfile
=====================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/policyfile.rst>`__

.. tag policyfile_summary

A Policyfile is an optional way to manage role, environment, and community cookbook data with a single document that is uploaded to the Chef Infra Server. The file is associated with a group of nodes, cookbooks, and settings. When these nodes perform a Chef Infra Client run, they utilize recipes specified in the Policyfile run-list.

.. end_tag

Why Policyfile?
=====================================================
For some users of Chef, Policyfile will make it easier to test and promote code safely with a simpler interface. Policyfile improves the user experience and resolves real-world problems that some workflows built around Chef must deal with. The following sections discuss in more detail some of the good reasons to use Policyfile, including:

* Focus the workflow on the entire system
* Safer development workflows
* Less expensive computation
* Code visibility
* Role mutability
* Cookbook mutability
* Replaces Berkshelf and the environment cookbook pattern

Focused System Workflows
-----------------------------------------------------
The knife command line tool maps very closely to the Chef Infra Server API and the objects defined by it: roles, environments, run-lists, cookbooks, data bags, nodes, and so on. The Chef Infra Client assembles these pieces at run-time and configures a host to do useful work.

Policyfile focuses that workflow onto the entire system, rather than the individual components. For example, Policyfile describes whole systems, whereas each individual revision of the ``Policyfile.lock.json`` file uploaded to the Chef Infra Server describes a part of that system, inclusive of roles, environments, cookbooks, and the other Chef Infra Server objects necessary to configure that part of the system.

Safer Workflows
-----------------------------------------------------
Policyfile encourages safer workflows by making it easier to publish development versions of cookbooks to the Chef Infra Server without the risk of mutating the production versions and without requiring a complicated versioning scheme to work around cookbook mutability issues. Roles are mutable and those changes are applied only to the nodes specified by the policy. Policyfile does not require any changes to your normal workflows. Use the same repositories you are already using, the same cookbooks, and workflows. Policyfile will prevent an updated cookbook or role from being applied immediately to all machines.

Code Visibility
-----------------------------------------------------
When running Chef without Policyfile, the exact set of cookbooks that are applied to a node is defined by:

* The node's ``run_list`` property
* Any roles that are present in the node's run-list or recursively included by those roles
* The environment, which may restrict the set of valid cookbook versions for a node based on a variety of constraint operators
* Dependencies, as defined by each cookbook's metadata
* Dependency resolution picks the "best" set of cookbooks that meet dependency and environment criteria

These conditions are re-evaluated every time the Chef Infra Client runs, which can make it harder to know which cookbooks will be run by the Chef Infra Client or what the effects of updating a role or uploading a new cookbook will be.

Policyfile simplifies this behavior by computing the cookbook set on the workstation, and then producing a readable document of that solution: a ``Policyfile.lock.json`` file. This pre-computed file is uploaded to the Chef Infra Server, and is then used by all of the Chef Infra Client runs that are managed by that particular policy name and policy group.

Less Expensive Computation
-----------------------------------------------------
When running Chef without Policyfile, the Chef Infra Server loads dependency data for all known versions of all known cookbooks, and then runs an expensive computation to determine the correct set.

Policyfile moves this computation to the workstation, where it is done less frequently.

Role and Environment Mutability
-----------------------------------------------------
When running Chef without Policyfile roles and environments are global objects. Changes to roles and environments are applied immediately to any node that contains that role in its run-list or belong to a particular environment. This can make it hard to update roles and environments and in some use cases discourages using them at all.

Policyfile effectively replaces roles and environments. Policyfile files are versioned automatically and new versions are applied to systems only when promoted.

Cookbook Mutability
-----------------------------------------------------
When running Chef without Policyfile, existing versions of cookbooks are mutable. This is convenient for many use cases, especially if users upload in-development cookbook revisions to the Chef Infra Server. But this sometimes creates issues that are similar to role mutability by allowing those cookbook changes to be applied immediately to nodes that use that cookbook. Users account for this by rigorous testing processes to ensure that only fully integrated cookbooks are ever published. This process does enforce good development habits, but at the same time it shouldn't be a required part of a workflow that ends with publishing an in-development cookbook to the Chef Infra Server for testing against real nodes.

Policyfile solves this issue by using a cookbook publishing API for the Chef Infra Server that does not provide cookbook mutability. Name collisions are prevented by storing cookbooks by name and an opaque identifier that is computed from the content of the cookbook itself.

For example, name/version collisions can occur when users temporarily fork an upstream cookbook. Even if the user contributes their change and the maintainer is responsive, there may be a period of time where the user needs their fork in order to make progress. This situation presents a versioning dilemma: if the user doesn't update their own version, they must overwrite the existing copy of that cookbook on the Chef Infra Server, wheres if they do update the version number it might conflict with the version number of the future release of that upstream cookbook.

Opaque IDs
+++++++++++++++++++++++++++++++++++++++++++++++++++++
The opaque identifier that is computed from the content of a cookbook is the only place where an opaque identifier is necessary. When working with Policyfile, be sure to:

* Use the same names and version constraints as normal in the ``Policyfile.rb`` file
* Use the same references to cookbooks pulled from Chef Supermarket
* Use the same branch, tag, and revision patterns for cookbooks pulled from git
* Use the same paths for cookbooks pulled from disk

Extra metadata about the cookbook is stored and included in Chef Infra Server API responses and in the ``Policyfile.lock.json`` file, including the source of a cookbook (Chef Supermarket, git, local disk, etc.), as well as any upstream idenfiers, such as git revisions. For cookbooks that are loaded from the local disk that are in a git repo, the upstream URL, current revision ID, and the state of the repo are stored also.

The opaque identifier is mostly behind the scenes and is only visible once published to the Chef Infra Server. Cookbooks that are uploaded to the Chef Infra Server may have extended version numbers such as ``1.0.0-dev``.

Environment Cookbooks
-----------------------------------------------------
Policyfile replaces the environment cookbook pattern that is often required by Berkshelf, along with a dependency solver and fetcher. That said, Policyfile does not replace all Berkshelf scenarios.

Knife Commands
=====================================================
The following knife commands used to set the policy group and policy name on the Chef Infra Server. For example:

.. code-block:: bash

   $ knife node policy set test-node 'test-policy-group-name' 'test-policy-name'




Policyfile.rb
=====================================================
.. tag policyfile_rb

A Policyfile file allows you to specify in a single document the cookbook revisions and recipes that should be applied by the Chef Infra Client. A Policyfile file is uploaded to the Chef Infra Server, where it is associated with a group of nodes. When these nodes are configured by the Chef Infra Client, the Chef Infra Client will make decisions based on settings in the policy file, and will build a run-list based on that information. A Policyfile file may be versioned, and then promoted through deployment stages to safely and reliably deploy new configuration.

.. end_tag

Syntax
-----------------------------------------------------
.. tag policyfile_rb_syntax

A ``Policyfile.rb`` is a Ruby file in which run-list and cookbook locations are specified. The syntax is as follows:

.. code-block:: ruby

   name "name"
   run_list "ITEM", "ITEM", ...
   default_source :SOURCE_TYPE, *args
   cookbook "NAME" [, "VERSION_CONSTRAINT"] [, SOURCE_OPTIONS]

.. end_tag

Settings
-----------------------------------------------------
.. tag policyfile_rb_settings

A ``Policyfile.rb`` file may contain the following settings:

``name "name"``
   Required. The name of the policy. Use a name that reflects the purpose of the machines against which the policy will run.

``run_list "ITEM", "ITEM", ...``
   Required. The run-list the Chef Infra Client will use to apply the policy to one (or more) nodes.

``default_source :SOURCE_TYPE, *args``
   The location in which any cookbooks not specified by ``cookbook`` are located. Possible values: ``chef_repo``, ``chef_server``, ``:community``, ``:supermarket``, and ``:artifactory``. Use more than one ``default_source`` to specify more than one location for cookbooks.

   ``default_source :supermarket`` pulls cookbooks from the public Chef Supermarket.

   ``default_source :supermarket, "https://mysupermarket.example"`` pulls cookbooks from a named private Chef Supermarket.

   ``default_source :chef_server, "https://chef-server.example/organizations/example"`` pulls cookbooks from the Chef Infra Server.

   ``default_source :community`` is an alias for ``:supermarket``.

   ``default_source :chef_repo, "path/to/repo"`` pulls cookbooks from a monolithic cookbook repository. This may be a path to the top-level of a cookbook repository or to the ``/cookbooks`` directory within that repository.

   ``default_source :artifactory, "https://artifactory.example/api/chef/my-supermarket"`` pulls cookbooks from an Artifactory server. Requires either ``artifactory_api_key`` to be set in ``config.rb`` or ``ARTIFACTORY_API_KEY`` to be set in your environment.

   Multiple cookbook sources may be specified. For example from the public Chef Supermarket and a monolithic repository:

   .. code-block:: ruby

	  default_source :supermarket
	  default_source :chef_repo, "path/to/repo"

   or from both a public and private Chef Supermarket:

   .. code-block:: ruby

	  default_source :supermarket
	  default_source :supermarket, "https://supermarket.example"

   .. note:: If a run-list or any dependencies require a cookbook that is present in more than one source, be explicit about which source is preferred. This will ensure that a cookbook is always pulled from an expected source. For example, an internally-developed cookbook named ``chef-client`` will conflict with the public ``chef-client`` cookbook that is maintained by Chef. To specify a named source for a cookbook:

      .. code-block:: ruby

         default_source :supermarket
         default_source :supermarket, "https://supermarket.example" do |s|
           s.preferred_for "chef-client"
         end

      List multiple cookbooks on the same line:

      .. code-block:: ruby

         default_source :supermarket
         default_source :supermarket, "https://supermarket.example" do |s|
           s.preferred_for "chef-client", "nginx", "mysql"
         end

``cookbook "NAME" [, "VERSION_CONSTRAINT"] [, SOURCE_OPTIONS]``
   Add cookbooks to the policy, specify a version constraint, or specify an alternate source location, such as Chef Supermarket. For example, add a cookbook:

   .. code-block:: ruby

      cookbook "apache2"

   Specify a version constraint:

   .. code-block:: ruby

      run_list "jenkins::master"

      # Restrict the jenkins cookbook to version 2.x, greater than 2.1
      cookbook "jenkins", "~> 2.1"

   Specify an alternate source:

   .. code-block:: ruby

      cookbook 'my_app', path: 'cookbooks/my_app'

   or:

   .. code-block:: ruby

      cookbook 'mysql', github: 'opscode-cookbooks/mysql', branch: 'master'

   or:

   .. code-block:: ruby

      cookbook 'chef-ingredient', git: 'https://github.com/chef-cookbooks/chef-ingredient.git', tag: 'v0.12.0'

``named_run_list "NAME", "ITEM1", "ITEM2", ...``
   Specify a named run-list to be used as an alternative to the override run-list. This setting should be used carefully and for specific use cases, like running a small set of recipes to quickly converge configuration for a single application on a host or for one-time setup tasks. For example:

   .. code-block:: ruby

      named_run_list :update_app, "my_app_cookbook::default"

``include_policy "NAME", *args``
    Specify a policyfile lock to be merged with this policy. ChefDK supports pulling this lock from a local or remote file, from a Chef Infra Server, or from a git repository. When the policyfile lock is included, its run-list will appear before the current policyfile's run-list. This setting requires that the solved cookbooks appear as-is from the included policyfile lock. If conflicting attributes or cookbooks are provided, an error will be presented. See `RFC097 <https://github.com/chef/chef-rfc/blob/master/rfc097-policyfile-includes.md>`__ for the full specifications of this feature.


  Pull the policyfile lock from ``./NAME.lock.json``:

  .. code-block:: ruby

     include_policy "NAME", path: "."

  Pull the policyfile lock from ``./foo.lock.json``.

  .. code-block:: ruby

     include_policy "NAME", path: "./foo.lock.json"

  Pull the policyfile lock from ``./bar.lock.json`` with revision ID 'revision1'.

  .. code-block:: ruby

     include_policy "NAME", policy_revision_id: "revision1", path: "./bar.lock.json"

  Pull the policyfile lock from a remote server ``https://internal.example.com/foo.lock.json``.

  .. code-block:: ruby

     include_policy "NAME", remote: "https://internal.example.com/foo.lock.json"

  Pull the policyfile lock from a remote server ``https://internal.example.com/bar.lock.json`` and with revision ID 'revision1'.

  .. code-block:: ruby

     include_policy "NAME", policy_revision_id: "revision1", remote: "https://internal.example.com/foo.lock.json"

  Pull the policy ``NAME`` with revision ID ``revision1`` from the ``http://chef-server.example`` Chef Infra Server:

  .. code-block:: ruby

     include_policy "NAME", policy_revision_id: "revision1", server: "http://chef-server.example"

  Pull the policy ``foo`` with revision ID ``revision1``:

  .. code-block:: ruby

     include_policy "NAME", policy_name: "foo", policy_revision_id: "revision1", server: "http://chef-server.example"

  Pull and lock the current revision for policy ``foo`` in policy group ``prod``:

  .. code-block:: ruby

     include_policy "NAME", policy_name: "foo", policy_group: "prod", server: "http://chef-server.example"

.. end_tag

Example
-----------------------------------------------------
.. tag policyfile_rb_example

For example:

.. code-block:: ruby

   name "jenkins-master"
   run_list "java", "jenkins::master", "recipe[policyfile_demo]"
   default_source :supermarket, "https://mysupermarket.example"
   cookbook "policyfile_demo", path: "cookbooks/policyfile_demo"
   cookbook "jenkins", "~> 2.1"
   cookbook "mysql", github: "chef-cookbooks/mysql", branch: "master"

.. end_tag

client.rb Settings
=====================================================
The following settings may be configured in the client.rb file for use with Policyfile:

``named_run_list``
   The run-list associated with a policy file.

``policy_group``
   The name of a policy group that exists on the Chef Infra Server. ``policy_name`` must also be specified.

``policy_name``
   The name of a policy, as identified by the ``name`` setting in a ``Policyfile.rb`` file. ``policy_group`` must also be specified.

``use_policyfile``
  The Chef Infra Client automatically checks the configuration, node JSON, and the stored node on the Chef Infra Server to determine if Policyfile files are being used, and then automatically updates this flag. Default value: ``false``.

knife bootstrap
=====================================================
A node may be bootstrapped to use Policyfile files. Use the following options as part of the bootstrap command:

``--policy-group POLICY_GROUP``
   The name of a policy group that exists on the Chef Infra Server.

``--policy-name POLICY_NAME``
   The name of a policy, as identified by the ``name`` setting in a ``Policyfile.rb`` file.

For a customized bootstrap process, add ``policy_name`` and ``policy_group`` to the first-boot JSON file that is passed to the Chef Infra Client.

knife search
=====================================================
The ``policy_name`` and ``policy_group`` settings for a node are stored as searchable attributes and as such are available when using a fuzzy matching search pattern. For example: ``knife search dev`` will return nodes that are part of the ``dev`` policy group.

Test w/Kitchen
=====================================================
Kitchen may be used to test Policyfile files. Add the following to .kitchen.yml:

.. code-block:: yaml

   provisioner:
     name: chef_zero

A named run-list may be used on a per-suite basis:

.. code-block:: yaml

   suites:
     - name: client
       provisioner:
         named_run_list: test_client_recipe
     - name: server
       provisioner:
         named_run_list: test_server_recipe

or globally:

.. code-block:: yaml

   provisioner:
     name: chef_zero
     named_run_list: integration_test_run_list

or testing with policies per-suite, once the Policyfile files are available in your repo:

.. code-block:: yaml

   suites:
      - name: defaultmega
         provisioner:
            policyfile: policies/default.rb
         attributes:
      - name: defaultultra
         provisioner:
            policyfile: policies/defaulttwo.rb
         attributes

.. note:: As ``chef_zero`` explicitly tests outside the context of a Chef Infra Server, the ``policy_groups`` concept is not applicable. The value of ``policy_group`` during a converge will be set to ``local``.

chef Commands
=====================================================
.. tag policyfile_chef_commands

The following commands are built into the ``chef`` executable and support the use of Policyfile files.

.. end_tag

chef clean-policy-cookbooks
-----------------------------------------------------
.. tag ctl_chef_clean_policy_cookbooks

Use the ``chef clean-policy-cookbooks`` subcommand to delete cookbooks that are not used by Policyfile files. Cookbooks are considered unused when they are not referenced by any policy revisions on the Chef Infra Server.

.. note:: Cookbooks that are referenced by orphaned policy revisions are not removed. Use ``chef clean-policy-revisions`` to remove orphaned policies.

.. end_tag

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_clean_policy_cookbooks_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef clean-policy-cookbooks (options)

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_clean_policy_cookbooks_options

This subcommand has the following options:

``-c CONFIG_FILE``, ``--config CONFIG_FILE``
   The path to the knife configuration file.

``-D``, ``--debug``
   Enable stack traces and other debug output. Default value: ``false``.

``-h``, ``--help``
   Show help for the command.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag

chef clean-policy-revisions
-----------------------------------------------------
.. tag ctl_chef_clean_policy_revisions

Use the ``chef clean-policy-revisions`` subcommand to delete orphaned policy revisions to Policyfile files from the Chef Infra Server. An orphaned policy revision is not associated to any policy group and therefore is not in active use by any node. Use ``chef show-policy --orphans`` to view a list of orphaned policy revisions.

.. end_tag

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_clean_policy_revisions_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef clean-policy-revisions (options)

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_clean_policy_revisions_options

This subcommand has the following options:

``-c CONFIG_FILE``, ``--config CONFIG_FILE``
   The path to the knife configuration file.

``-D``, ``--debug``
   Enable stack traces and other debug output. Default value: ``false``.

``-h``, ``--help``
   Show help for the command.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag

chef delete-policy
-----------------------------------------------------
.. tag ctl_chef_delete_policy

Use the ``chef delete-policy`` subcommand to delete all revisions of the named policy that exist on the Chef Infra Server. (The state of the policy revision is backed up locally and may be restored using the ``chef undelete`` subcommand.)

.. end_tag

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_delete_policy_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef delete-policy POLICY_NAME (options)

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_delete_policy_options

This subcommand has the following options:

``-c CONFIG_FILE``, ``--config CONFIG_FILE``
   The path to the knife configuration file.

``-D``, ``--debug``
   Enable stack traces and other debug output. Default value: ``false``.

``-h``, ``--help``
   Show help for the command.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag

chef delete-policy-group
-----------------------------------------------------
.. tag ctl_chef_delete_policy_group

Use the ``chef delete-policy-group`` subcommand to delete the named policy group from the Chef Infra Server. Any policy revision associated with that policy group is not deleted. (The state of the policy group is backed up locally and may be restored using the ``chef undelete`` subcommand.)

.. end_tag

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_delete_policy_group_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef delete-policy-group POLICY_GROUP (options)

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_delete_policy_group_options

This subcommand has the following options:

``-c CONFIG_FILE``, ``--config CONFIG_FILE``
   The path to the knife configuration file.

``-D``, ``--debug``
   Enable stack traces and other debug output. Default value: ``false``.

``-h``, ``--help``
   Show help for the command.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag

chef diff
-----------------------------------------------------
.. tag ctl_chef_diff

Use the ``chef diff`` subcommand to display an itemized comparison of two revisions of a ``Policyfile.lock.json`` file.

.. end_tag

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_diff_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef diff POLICY_FILE --head | --git POLICY_GROUP | POLICY_GROUP...POLICY_GROUP (options)

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_diff_options

This subcommand has the following options:

``-c CONFIG_FILE``, ``--config CONFIG_FILE``
   The path to the knife configuration file.

``-D``, ``--debug``
   Enable stack traces and other debug output. Default value: ``false``.

``-g GIT_REF``, ``--git GIT_REF``
   Compare the specified git reference against the current revision of a ``Policyfile.lock.json`` file or against another git reference.

``-h``, ``--help``
   Show help for the command.

``--head``
   A shortcut for ``chef diff --git HEAD``. When a git-specific flag is not provided, the on-disk ``Policyfile.lock.json`` file is compared to one on the Chef Infra Server or (if a ``Policyfile.lock.json`` file is not present on-disk) two ``Policyfile.lock.json`` files in the specified policy group on the Chef Infra Server are compared.

``--[no-]pager``
   Use ``--pager`` to enable paged output for a ``Policyfile.lock.json`` file. Default value: ``--pager``.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag

Examples
+++++++++++++++++++++++++++++++++++++++++++++++++++++

**Compare current lock to latest commit on latest branch**

.. tag ctl_chef_diff_current_lock_latest_branch

.. To compare current lock to latest commit on latest branch:

.. code-block:: bash

   $ chef diff --git HEAD

.. end_tag

**Compare current lock with latest commit on master branch**

.. tag ctl_chef_diff_current_lock_master_branch

.. To compare current lock with latest commit on master branch:

.. code-block:: bash

   $ chef diff --git master

.. end_tag

**Compare current lock to specified revision**

.. tag ctl_chef_diff_current_lock_specified_revision

.. To compare current lock to specified revision:

.. code-block:: bash

   $ chef diff --git v1.0.0

.. end_tag

**Compare lock on master branch to lock on revision**

.. tag ctl_chef_diff_master_lock_revision_lock

.. To compare lock on master branch to lock on revision:

.. code-block:: bash

   $ chef diff --git master...dev

.. end_tag

**Compare lock for version with latest commit on master branch**

.. tag ctl_chef_diff_version_lock_master_branch

.. To compare lock for version with latest commit on master branch:

.. code-block:: bash

   $ chef diff --git v1.0.0...master

.. end_tag

**Compare current lock with latest lock for policy group**

.. tag ctl_chef_diff_current_lock_policy_group

.. To compare current lock with latest lock for policy group:

.. code-block:: bash

   $ chef diff staging

.. end_tag

**Compare locks for two policy groups**

.. tag ctl_chef_diff_two_policy_groups

.. To compare locks for two policy groups:

.. code-block:: bash

   $ chef diff production...staging

.. end_tag

chef export
-----------------------------------------------------
.. tag ctl_chef_export

Use the ``chef export`` subcommand to create a chef-zero-compatible chef-repo that contains the cookbooks described by a ``Policyfile.lock.json`` file. After a chef-zero-compatible chef-repo is copied to a node, the policy can be applied locally on that machine by running ``chef-client -z`` (local mode).

.. end_tag

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_export_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef export POLICY_FILE DIRECTORY (options)

.. end_tag

Configuration Settings
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_export_config

The client.rb file on that machine requires the following settings:

``deployment_group``
   This setting should be set to ``'$POLICY_NAME-local'``.

``policy_document_native_api``
   This setting should be set to ``false``.

``use_policyfile``
   This setting should be set to ``true``.

``versioned_cookbooks``
   This setting should be set to ``true``.

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_export_options

This subcommand has the following options:

``-a``, ``--archive``
   Export an archive as a tarball, instead as a directory. Default value: ``false``.

``-D``, ``--debug``
   Enable stack traces and other debug output. Default value: ``false``.

``-f``, ``--force``
   Remove the contents of the destination directory if that directory is not empty. Default value: ``false``.

``-h``, ``--help``
   Show help for the command.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag

chef generate policyfile
-----------------------------------------------------
.. tag ctl_chef_generate_policyfile

Use the ``chef generate policyfile`` subcommand to generate a file to be used with Policyfile.

.. end_tag

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_generate_policyfile_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef generate policyfile POLICY_NAME (options)

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_generate_policyfile_options

This subcommand has the following options:

``-h``, ``--help``
   Show help for the command.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag

chef generate repo
-----------------------------------------------------
.. tag ctl_chef_generate_repo

Use the ``chef generate repo`` subcommand to create a chef-repo. By default, the repo is a cookbook repo with options available to support generating a cookbook that supports Policyfile.

.. end_tag

.. note:: This subcommand requires using one (or more) of the options (below) to support Policyfile files.

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_generate_repo_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef generate repo REPO_NAME (options)

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_generate_repo_options

This subcommand has the following options:

``-h``, ``--help``
   Show help for the command.

``-p``, ``--policy-only``
   Create a repository that does not store cookbook files, only Policyfile files.

``-P``, ``--policy``
   Use Policyfile instead of Berkshelf.

``-r``, ``--roles``
   Create directories for ``/roles`` and ``/environments`` instead of creating directories for Policyfile.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag

chef install
-----------------------------------------------------
.. tag ctl_chef_install

Use the ``chef install`` subcommand to evaluate a policy file and find a compatible set of cookbooks, build a run-list, cache it locally, and then emit a ``Policyfile.lock.json`` file that describes the locked policy set. The ``Policyfile.lock.json`` file may be used to install the locked policy set to other machines and may be pushed to a policy group on the Chef Infra Server to apply that policy to a group of nodes that are under management by Chef.

.. end_tag

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_install_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef install POLICY_FILE (options)

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_install_options

This subcommand has the following options:

``-D``, ``--debug``
   Enable stack traces and other debug output.

``-h``, ``--help``
   Show help for the command.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag

Policyfile.lock.json
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag policyfile_lock_json

When the ``chef install`` command is run, ChefDK caches any necessary cookbooks and emits a ``Policyfile.lock.json`` file that describes:

* The versions of cookbooks in use
* A Hash of cookbook content
* The source for all cookbooks

A ``Policyfile.lock.json`` file is associated with a specific policy group, i.e. is associated with one (or more) nodes that use the same revision of a given policy.

.. end_tag

.. tag policyfile_lock_json_example

A ``Policyfile.lock.json`` file is similar to:

.. code-block:: javascript

   {
     "revision_id": "288ed244f8db8bff3caf58147e840bbe079f76e0",
     "name": "jenkins",
     "run_list": [
       "recipe[java::default]",
       "recipe[jenkins::master]",
       "recipe[policyfile_demo::default]"
     ],
     "cookbook_locks": {
       "policyfile_demo": {
         "version": "0.1.0",
         "identifier": "f04cc40faf628253fe7d9566d66a1733fb1afbe9",
         "dotted_decimal_identifier": "67638399371010690.23642238397896298.25512023620585",
         "source": "cookbooks/policyfile_demo",
         "cache_key": null,
         "scm_info": null,
         "source_options": {
           "path": "cookbooks/policyfile_demo"
         }
       },
     "java": {
       "version": "1.24.0",
       "identifier": "4c24ae46a6633e424925c24e683e0f43786236a3",
       "dotted_decimal_identifier": "21432429158228798.18657774985439294.16782456927907",
       "cache_key": "java-1.24.0-supermarket.chef.io",
       "origin": "https://supermarket.chef.io/api/v1/cookbooks/java/versions/1.24.0/download",
       "source_options": {
         "artifactserver": "https://supermarket.chef.io/api/v1/cookbooks/java/versions/1.24.0/download",
         "version": "1.24.0"
       }

.. end_tag

chef push
-----------------------------------------------------
.. tag ctl_chef_push

Use the ``chef push`` subcommand to upload an existing ``Policyfile.lock.json`` file to the Chef Infra Server, along with all of the cookbooks that are contained in the file. The ``Policyfile.lock.json`` file will be applied to the specified policy group, which is a set of nodes that share the same run-list and cookbooks.

.. end_tag

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_push_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef push POLICY_GROUP POLICY_FILE (options)

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_push_options

This subcommand has the following options:

``-c CONFIG_FILE``, ``--config CONFIG_FILE``
   The path to the knife configuration file.

``-D``, ``--debug``
   Enable stack traces and other debug output.

``-h``, ``--help``
   Show help for the command.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag

chef push-archive
-----------------------------------------------------
.. tag ctl_chef_push_archive

The ``chef push-archive`` subcommand is used to publish a policy archive file to the Chef Infra Server. (A policy archive is created using the ``chef export`` subcommand.) The policy archive is assigned to the specified policy group, which is a set of nodes that share the same run-list and cookbooks.

.. end_tag

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_push_archive_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef push-archive POLICY_GROUP ARCHIVE_FILE (options)

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_push_archive_options

This subcommand has the following options:

``-c CONFIG_FILE``, ``--config CONFIG_FILE``
   The path to the knife configuration file.

``-D``, ``--debug``
   Enable stack traces and other debug output. Default value: ``false``.

``-h``, ``--help``
   Show help for the command.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag

chef show-policy
-----------------------------------------------------
.. tag ctl_chef_show_policy

Use the ``chef show-policy`` subcommand to display revisions for every ``Policyfile.rb`` file that is on the Chef Infra Server. By default, only active policy revisions are shown. When both a policy and policy group are specified, the contents of the active ``Policyfile.lock.json`` file for the policy group is returned.

.. end_tag

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_show_policy_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef show-policy POLICY_NAME POLICY_GROUP (options)

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_show_policy_options

This subcommand has the following options:

``-c CONFIG_FILE``, ``--config CONFIG_FILE``
   The path to the knife configuration file.

``-D``, ``--debug``
   Enable stack traces and other debug output. Default value: ``false``.

``-h``, ``--help``
   Show help for the command.

``-o``, ``--orphans``
   Show policy revisions that are not currently assigned to any policy group.

``--[no-]pager``
   Use ``--pager`` to enable paged output for a ``Policyfile.lock.json`` file. Default value: ``--pager``.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag

chef undelete
-----------------------------------------------------
.. tag ctl_chef_undelete

Use the ``chef undelete`` subcommand to recover a deleted policy or policy group. This command:

* Does not detect conflicts. If a deleted item has been recreated, running this command will overwrite it
* Does not include cookbooks that may be referenced by policy files; cookbooks that are cleaned after running this command may not be fully restorable to their previous state
* Does not store access control data

.. end_tag

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_undelete_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef undelete (options)

When run with no arguments, returns a list of available operations.

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_undelete_options

This subcommand has the following options:

``-c CONFIG_FILE``, ``--config CONFIG_FILE``
   The path to the knife configuration file.

``-D``, ``--debug``
   Enable stack traces and other debug output.

``-h``, ``--help``
   Show help for the command.

``-i ID``, ``--id ID``
   Undo the delete operation specified by ``ID``.

``-l``, ``--last``
   Undo the most recent delete operation.

``--list``
   Default. Return a list of available operations.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag

chef update
-----------------------------------------------------
.. tag ctl_chef_update

Use the ``chef update`` subcommand to read the ``Policyfile.rb`` file, and then apply any changes. This will resolve dependencies and will create a ``Policyfile.lock.json`` file. The locked policy will reflect any changes to the run-list and will pull in any cookbook updates that are compatible with any version constraints defined in the ``Policyfile.rb`` file.

.. end_tag

Syntax
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_update_syntax

This subcommand has the following syntax:

.. code-block:: bash

   $ chef update POLICY_FILE (options)

.. end_tag

Options
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag ctl_chef_update_options

This subcommand has the following options:

``-a``, ``--attributes``
   Update attributes. Default value: ``false``.

``-D``, ``--debug``
   Enable stack traces and other debug output. Default value: ``false``.

``-h``, ``--help``
   Show help for the command.

``-v``, ``--version``
   The version of the Chef Infra Client.

.. end_tag
