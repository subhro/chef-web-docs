=====================================================
yum_package resource
=====================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/resource_yum_package.rst>`__

.. tag resource_package_yum

Use the **yum_package** resource to install, upgrade, and remove packages with Yum for the Red Hat and CentOS platforms. The **yum_package** resource is able to resolve ``provides`` data for packages much like Yum can do when it is run from the command line. This allows a variety of options for installing packages, like minimum versions, virtual provides, and library names.

.. end_tag

.. note:: Support for using file names to install packages (as in ``yum_package "/bin/sh"``) is not available because the volume of data required to parse for this is excessive.

.. note:: .. tag notes_resource_based_on_package

          In many cases, it is better to use the **package** resource instead of this one. This is because when the **package** resource is used in a recipe, the chef-client will use details that are collected by Ohai at the start of the chef-client run to determine the correct package application. Using the **package** resource allows a recipe to be authored in a way that allows it to be used across many platforms.

          .. end_tag

Syntax
=====================================================
A **yum_package** resource block manages a package on a node, typically by installing it. The simplest use of the **yum_package** resource is:

.. code-block:: ruby

   yum_package 'package_name'

which will install the named package using all of the default options and the default action (``:install``).

The yum_package resource has the following syntax:

.. code-block:: ruby

  yum_package 'name' do
    allow_downgrade      true, false # default value: true
    arch                 String, Array
    flush_cache          Hash # default value: {"before"=>false, "after"=>false}
    options              String, Array
    package_name         String, Array
    source               String
    timeout              String, Integer
    version              String, Array
    yum_binary           String
    action               Symbol # defaults to :install if not specified
  end

where:

* ``yum_package`` is the resource.
* ``name`` is the name given to the resource block.
* ``action`` identifies which steps the chef-client will take to bring the node into the desired state.
* ``allow_downgrade``, ``arch``, ``flush_cache``, ``options``, ``package_name``, ``source``, ``timeout``, ``version``, and ``yum_binary`` are the properties available to this resource.

Actions
=====================================================

The yum_package resource has the following actions:

``:install``
   Default. Install a package. If a version is specified, install the specified version of the package.

``:lock``
   Locks the yum package to a specific version.

``:nothing``
   .. tag resources_common_actions_nothing

   This resource block does not act unless notified by another resource to take action. Once notified, this resource block either runs immediately or is queued up to run at the end of the Chef Infra Client run.

   .. end_tag

``:purge``
   Purge a package. This action typically removes the configuration files as well as the package.

``:remove``
   Remove a package.

``:unlock``
   Unlocks the yum package so that it can be upgraded to a newer version.

``:upgrade``
   Install a package and/or ensure that a package is the latest version. This action will ignore the ``version`` attribute.

Properties
=====================================================

The yum_package resource has the following properties:

``allow_downgrade``
   **Ruby Type:** true, false | **Default Value:** ``true``

   Downgrade a package to satisfy requested version requirements.

``arch``
   **Ruby Type:** String, Array

   The architecture of the package to be installed or upgraded. This value can also be passed as part of the package name.

``flush_cache``
   **Ruby Type:** Hash | **Default Value:** ``{"before"=>false, "after"=>false}``

   Flush the in-memory cache before or after a Yum operation that installs, upgrades, or removes a package. Accepts a Hash in the form: { :before => true/false, :after => true/false } or an Array in the form [ :before, :after ].

   .. tag resources_common_package_yum_cache

   Yum automatically synchronizes remote metadata to a local cache. The chef-client creates a copy of the local cache, and then stores it in-memory during the chef-client run. The in-memory cache allows packages to be installed during the chef-client run without the need to continue synchronizing the remote metadata to the local cache while the chef-client run is in-progress.

   .. end_tag

   As an array:

   .. code-block:: ruby

      yum_package 'some-package' do
        #...
        flush_cache [ :before ]
        #...
      end

   and as a Hash:

   .. code-block:: ruby

      yum_package 'some-package' do
        #...
        flush_cache( { :after => true } )
        #...
      end

   .. note:: The ``flush_cache`` property does not flush the local Yum cache! Use Yum tools---``yum clean headers``, ``yum clean packages``, ``yum clean all``---to clean the local Yum cache.

``options``
   **Ruby Type:** String, Array

   One (or more) additional command options that are passed to the command.

``package_name``
   **Ruby Type:** String, Array

   One of the following: the name of a package, the name of a package and its architecture, the name of a dependency. Default value: the ``name`` of the resource block. See "Syntax" section above for more information.

``source``
   **Ruby Type:** String

   The optional path to a package on the local file system.

``timeout``
   **Ruby Type:** String, Integer

   The amount of time (in seconds) to wait before timing out.

``version``
   **Ruby Type:** String, Array

   The version of a package to be installed or upgraded. This property is ignored when using the ``:upgrade`` action.

``yum_binary``
   **Ruby Type:** String

Common Resource Functionality
=====================================================

Chef resources include common properties, notifications, and resource guards.

Common Properties
-----------------------------------------------------

.. tag resources_common_properties

The following properties are common to every resource:

``ignore_failure``
  **Ruby Type:** true, false | **Default Value:** ``false``

  Continue running a recipe if a resource fails for any reason.

``retries``
  **Ruby Type:** Integer | **Default Value:** ``0``

  The number of attempts to catch exceptions and retry the resource.

``retry_delay``
  **Ruby Type:** Integer | **Default Value:** ``2``

  The retry delay (in seconds).

``sensitive``
  **Ruby Type:** true, false | **Default Value:** ``false``

  Ensure that sensitive resource data is not logged by the Chef Infra Client.

.. end_tag

Notifications
-----------------------------------------------------

``notifies``
  **Ruby Type:** Symbol, 'Chef::Resource[String]'

  .. tag resources_common_notification_notifies

  A resource may notify another resource to take action when its state changes. Specify a ``'resource[name]'``, the ``:action`` that resource should take, and then the ``:timer`` for that action. A resource may notify more than one resource; use a ``notifies`` statement for each resource to be notified.

  .. end_tag

.. tag resources_common_notification_timers

A timer specifies the point during the Chef Infra Client run at which a notification is run. The following timers are available:

``:before``
   Specifies that the action on a notified resource should be run before processing the resource block in which the notification is located.

``:delayed``
   Default. Specifies that a notification should be queued up, and then executed at the end of the Chef Infra Client run.

``:immediate``, ``:immediately``
   Specifies that a notification should be run immediately, per resource notified.

.. end_tag

.. tag resources_common_notification_notifies_syntax

The syntax for ``notifies`` is:

.. code-block:: ruby

  notifies :action, 'resource[name]', :timer

.. end_tag

``subscribes``
  **Ruby Type:** Symbol, 'Chef::Resource[String]'

.. tag resources_common_notification_subscribes

A resource may listen to another resource, and then take action if the state of the resource being listened to changes. Specify a ``'resource[name]'``, the ``:action`` to be taken, and then the ``:timer`` for that action.

Note that ``subscribes`` does not apply the specified action to the resource that it listens to - for example:

.. code-block:: ruby

 file '/etc/nginx/ssl/example.crt' do
   mode '0600'
   owner 'root'
 end

 service 'nginx' do
   subscribes :reload, 'file[/etc/nginx/ssl/example.crt]', :immediately
 end

In this case the ``subscribes`` property reloads the ``nginx`` service whenever its certificate file, located under ``/etc/nginx/ssl/example.crt``, is updated. ``subscribes`` does not make any changes to the certificate file itself, it merely listens for a change to the file, and executes the ``:reload`` action for its resource (in this example ``nginx``) when a change is detected.

.. end_tag

.. tag resources_common_notification_timers

A timer specifies the point during the Chef Infra Client run at which a notification is run. The following timers are available:

``:before``
   Specifies that the action on a notified resource should be run before processing the resource block in which the notification is located.

``:delayed``
   Default. Specifies that a notification should be queued up, and then executed at the end of the Chef Infra Client run.

``:immediate``, ``:immediately``
   Specifies that a notification should be run immediately, per resource notified.

.. end_tag

.. tag resources_common_notification_subscribes_syntax

The syntax for ``subscribes`` is:

.. code-block:: ruby

   subscribes :action, 'resource[name]', :timer

.. end_tag

Guards
-----------------------------------------------------

.. tag resources_common_guards

A guard property can be used to evaluate the state of a node during the execution phase of the Chef Infra Client run. Based on the results of this evaluation, a guard property is then used to tell the Chef Infra Client if it should continue executing a resource. A guard property accepts either a string value or a Ruby block value:

* A string is executed as a shell command. If the command returns ``0``, the guard is applied. If the command returns any other value, then the guard property is not applied. String guards in a **powershell_script** run Windows PowerShell commands and may return ``true`` in addition to ``0``.
* A block is executed as Ruby code that must return either ``true`` or ``false``. If the block returns ``true``, the guard property is applied. If the block returns ``false``, the guard property is not applied.

A guard property is useful for ensuring that a resource is idempotent by allowing that resource to test for the desired state as it is being executed, and then if the desired state is present, for the Chef Infra Client to do nothing.

.. end_tag

.. tag resources_common_guards_properties

The following properties can be used to define a guard that is evaluated during the execution phase of the Chef Infra Client run:

``not_if``
  Prevent a resource from executing when the condition returns ``true``.

``only_if``
  Allow a resource to execute only if the condition returns ``true``.

.. end_tag

Multiple Packages
-----------------------------------------------------
.. tag resources_common_multiple_packages

A resource may specify multiple packages and/or versions for platforms that use Yum, DNF, Apt, Zypper, or Chocolatey package managers. Specifying multiple packages and/or versions allows a single transaction to:

* Download the specified packages and versions via a single HTTP transaction
* Update or install multiple packages with a single resource during the chef-client run

For example, installing multiple packages:

.. code-block:: ruby

   package %w(package1 package2)

Installing multiple packages with versions:

.. code-block:: ruby

   package %w(package1 package2) do
     version [ '1.3.4-2', '4.3.6-1']
   end

Upgrading multiple packages:

.. code-block:: ruby

   package %w(package1 package2)  do
     action :upgrade
   end

Removing multiple packages:

.. code-block:: ruby

   package %w(package1 package2)  do
     action :remove
   end

Purging multiple packages:

.. code-block:: ruby

   package %w(package1 package2)  do
     action :purge
   end

Notifications, via an implicit name:

.. code-block:: ruby

   package %w(package1 package2)  do
     action :nothing
   end

   log 'call a notification' do
     notifies :install, 'package[package1, package2]', :immediately
   end

.. note:: Notifications and subscriptions do not need to be updated when packages and versions are added or removed from the ``package_name`` or ``version`` properties.

.. end_tag

Examples
=====================================================
The following examples demonstrate various approaches for using resources in recipes:

**Install an exact version**

.. tag resource_yum_package_install_exact_version

.. To install an exact version:

.. code-block:: ruby

   yum_package 'netpbm = 10.35.58-8.el5'

.. end_tag

**Install a minimum version**

.. tag resource_yum_package_install_minimum_version

.. To install a minimum version:

.. code-block:: ruby

   yum_package 'netpbm >= 10.35.58-8.el5'

.. end_tag

**Install a minimum version using the default action**

.. tag resource_yum_package_install_package_using_default_action

.. To install the same package using the default action:

.. code-block:: ruby

   yum_package 'netpbm'

.. end_tag

**To install a package**

.. tag resource_yum_package_install_package

.. To install a package:

.. code-block:: ruby

   yum_package 'netpbm' do
     action :install
   end

.. end_tag

**To install a partial minimum version**

.. tag resource_yum_package_install_partial_minimum_version

.. To install a partial minimum version:

.. code-block:: ruby

   yum_package 'netpbm >= 10'

.. end_tag

**To install a specific architecture**

.. tag resource_yum_package_install_specific_architecture

.. To install a specific architecture:

.. code-block:: ruby

   yum_package 'netpbm' do
     arch 'i386'
   end

or:

.. code-block:: ruby

   yum_package 'netpbm.x86_64'

.. end_tag

**To install a specific version-release**

.. tag resource_yum_package_install_specific_version_release

.. To install a specific version-release:

.. code-block:: ruby

   yum_package 'netpbm' do
     version '10.35.58-8.el5'
   end

.. end_tag

**Handle cookbook_file and yum_package resources in the same recipe**

.. To handle cookbook_file and package when both called in the same recipe

When a **cookbook_file** resource and a **yum_package** resource are both called from within the same recipe, use the ``flush_cache`` attribute to dump the in-memory Yum cache, and then use the repository immediately to ensure that the correct package is installed:

.. code-block:: ruby

   cookbook_file '/etc/yum.repos.d/custom.repo' do
     source 'custom'
     mode '0755'
   end

   yum_package 'only-in-custom-repo' do
     action :install
     flush_cache [ :before ]
   end


