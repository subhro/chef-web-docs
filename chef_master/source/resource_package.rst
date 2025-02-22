=====================================================
package resource
=====================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/resource_package.rst>`__

.. tag resource_package_summary

Use the **package** resource to manage packages. When the package is installed from a local file (such as with RubyGems, dpkg, or RPM Package Manager), the file must be added to the node using the **remote_file** or **cookbook_file** resources.

.. end_tag

This resource is the base resource for several other resources used for package management on specific platforms. While it is possible to use each of these specific resources, it is recommended to use the **package** resource as often as possible.

For more information about specific resources for specific platforms, see the following topics:

* `apt_package </resource_apt_package.html>`__
* `bff_package </resource_bff_package.html>`__
* `cab_package </resource_cab_package.html>`__
* `chef_gem </resource_chef_gem.html>`__
* `chocolatey_package </resource_chocolatey_package.html>`__
* `dnf_package </resource_dnf_package.html>`__
* `dpkg_package </resource_dpkg_package.html>`__
* `freebsd_package </resource_freebsd_package.html>`__
* `gem_package </resource_gem_package.html>`__
* `homebrew_package </resource_homebrew_package.html>`__
* `ips_package </resource_ips_package.html>`__
* `macports_package </resource_macports_package.html>`__
* `openbsd_package </resource_openbsd_package.html>`__
* `pacman_package </resource_pacman_package.html>`__
* `paludis_package </resource_paludis_package.html>`__
* `portage_package </resource_portage_package.html>`__
* `rpm_package </resource_rpm_package.html>`__
* `smartos_package </resource_smartos_package.html>`__
* `solaris_package </resource_solaris_package.html>`__
* `windows_package </resource_windows_package.html>`__
* `yum_package </resource_yum_package.html>`__
* `zypper_package </resource_zypper_package.html>`__

Syntax
=====================================================
A **package** resource block manages a package on a node, typically by installing it. The simplest use of the **package** resource is:

.. code-block:: ruby

   package 'httpd'

which will install Apache using all of the default options and the default action (``:install``).

For a package that has different package names, depending on the platform, use a ``case`` statement within the **package**:

.. code-block:: ruby

   package 'Install Apache' do
     case node[:platform]
     when 'redhat', 'centos'
       package_name 'httpd'
     when 'ubuntu', 'debian'
       package_name 'apache2'
     end
   end

where ``'redhat', 'centos'`` will install Apache using the ``httpd`` package and ``'ubuntu', 'debian'`` will install it using the ``apache2`` package

The full syntax for all of the properties that are available to the **package** resource is:

.. code-block:: ruby

   package 'name' do
     allow_downgrade            true, false # Yum, RPM packages only
     arch                       String, Array # Yum packages only
     default_release            String # Apt packages only
     flush_cache                Array
     gem_binary                 String
     homebrew_user              String, Integer # Homebrew packages only
     notifies                   # see description
     options                    String
     package_name               String, Array # defaults to 'name' if not specified
     response_file              String # Apt packages only
     response_file_variables    Hash # Apt packages only
     source                     String
     subscribes                 # see description
     timeout                    String, Integer
     version                    String, Array
     action                     Symbol # defaults to :install if not specified
   end

where:

* ``package`` tells the chef-client to manage a package; the chef-client will determine the correct package provider to use based on the platform running on the node
* ``'name'`` is the name of the package
* ``action`` identifies which steps the chef-client will take to bring the node into the desired state
* ``allow_downgrade``, ``arch``, ``default_release``, ``flush_cache``, ``gem_binary``, ``homebrew_user``, ``options``, ``package_name``, ``response_file``, ``response_file_variables``, ``source``, ``recursive``, ``timeout``, and ``version`` are properties of this resource, with the Ruby type shown. See "Properties" section below for more information about all of the properties that may be used with this resource.

Gem Package Options
-----------------------------------------------------
.. tag resource_package_options

The RubyGems package provider attempts to use the RubyGems API to install gems without spawning a new process, whenever possible. A gems command to install will be spawned under the following conditions:

* When a ``gem_binary`` property is specified (as a hash, a string, or by a .gemrc file), the chef-client will run that command to examine its environment settings and then again to install the gem.
* When install options are specified as a string, the chef-client will span a gems command with those options when installing the gem.
* The Chef installer will search the ``PATH`` for a gem command rather than defaulting to the current gem environment. As part of ``enforce_path_sanity``, the ``bin`` directories area added to the ``PATH``, which means when there are no other proceeding RubyGems, the installation will still be operated against it.

.. end_tag

.. warning:: Gem package options should only be used when gems are installed into the system-wide instance of Ruby, and not the instance of Ruby dedicated to the chef-client.

Specify with Hash
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag resource_package_options_hash

If an explicit ``gem_binary`` parameter is not being used with the ``gem_package`` resource, it is preferable to provide the install options as a hash. This approach allows the provider to install the gem without needing to spawn an external gem process.

The following RubyGems options are available for inclusion within a hash and are passed to the RubyGems DependencyInstaller:

* ``:env_shebang``
* ``:force``
* ``:format_executable``
* ``:ignore_dependencies``
* ``:prerelease``
* ``:security_policy``
* ``:wrappers``

For more information about these options, see the RubyGems documentation: http://rubygems.rubyforge.org/rubygems-update/Gem/DependencyInstaller.html.

.. end_tag

**Example**

.. tag resource_package_install_gem_with_hash_options

.. To install a gem with a |hash| of options:

.. code-block:: ruby

   gem_package 'bundler' do
     options(:prerelease => true, :format_executable => false)
   end

.. end_tag

Specify with String
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag resource_package_options_string

When using an explicit ``gem_binary``, options must be passed as a string. When not using an explicit ``gem_binary``, the chef-client is forced to spawn a gems process to install the gems (which uses more system resources) when options are passed as a string. String options are passed verbatim to the gems command and should be specified just as if they were passed on a command line. For example, ``--prerelease`` for a pre-release gem.

.. end_tag

**Example**

.. tag resource_package_install_gem_with_options_string

.. To install a gem with an options string:

.. code-block:: ruby

   gem_package 'nokogiri' do
     gem_binary('/opt/ree/bin/gem')
     options('--prerelease --no-format-executable')
   end

.. end_tag

Specify with .gemrc File
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag resource_package_options_gemrc

Options can be specified in a .gemrc file. By default the ``gem_package`` resource will use the Ruby interface to install gems which will ignore the .gemrc file. The ``gem_package`` resource can be forced to use the gems command instead (and to read the .gemrc file) by adding the ``gem_binary`` attribute to a code block.

.. end_tag

**Example**

.. tag resource_package_install_gem_with_gemrc

A template named ``gemrc.erb`` is located in a cookbook's ``/templates`` directory:

.. code-block:: ruby

   :sources:
   - http://<%= node['gem_file']['host'] %>:<%= node['gem_file']['port'] %>/

A recipe can be built that does the following:

* Builds a ``.gemrc`` file based on a ``gemrc.erb`` template
* Runs a ``Gem.configuration`` command
* Installs a package using the ``.gemrc`` file

.. code-block:: ruby

   template '/root/.gemrc' do
     source 'gemrc.erb'
     action :create
     notifies :run, 'ruby_block[refresh_gemrc]', :immediately
   end

   ruby_block 'refresh_gemrc' do
     action :nothing
     block do
       Gem.configuration = Gem::ConfigFile.new []
     end
   end

   gem_package 'di-ruby-lvm' do
     gem_binary '/opt/chef/embedded/bin/gem'
     action :install
   end

.. end_tag

Actions
=====================================================

The package resource has the following actions:

``:install``
   Default. Install a package. If a version is specified, install the specified version of the package.

``:nothing``
   .. tag resources_common_actions_nothing

   This resource block does not act unless notified by another resource to take action. Once notified, this resource block either runs immediately or is queued up to run at the end of the Chef Infra Client run.

   .. end_tag

``:purge``
   Purge a package. This action typically removes the configuration files as well as the package. (Debian platform only; for other platforms, use the ``:remove`` action.)

``:reconfig``
   Reconfigure a package. This action requires a response file.

``:remove``
   Remove a package.

``:upgrade``
   Install a package and/or ensure that a package is the latest version.

Properties
=====================================================

The package resource has the following properties:

``allow_downgrade``
   **Ruby Type:** true, false | **Default Value:** ``true``

   **yum_package** resource only. Downgrade a package to satisfy requested version requirements.

``arch``
   **Ruby Type:** String, Array

   **yum_package** resource only. The architecture of the package to be installed or upgraded. This value can also be passed as part of the package name.

``default_release``
   **Ruby Type:** String

   **apt_package** resource only. The default release. For example: ``stable``.

``flush_cache``
   **Ruby Type:** Array

   Flush the in-memory cache before or after a Yum operation that installs, upgrades, or removes a package. Default value: ``[ :before, :after ]``. The value may also be a Hash: ``( { :before => true/false, :after => true/false } )``.

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

``gem_binary``
   **Ruby Type:** String

   A property for the ``gem_package`` provider that is used to specify a gems binary.

``homebrew_user``
   **Ruby Type:** String, Integer

   **homebrew_package** resource only. The name of the Homebrew owner to be used by the chef-client when executing a command.

``ignore_failure``
   **Ruby Type:** true, false | **Default Value:** ``false``

   Continue running a recipe if a resource fails for any reason.

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

``options``
   **Ruby Type:** String

   One (or more) additional options that are passed to the command.

``package_name``
   **Ruby Type:** String, Array

   The name of the package. Default value: the ``name`` of the resource block. See "Syntax" section above for more information.

``response_file``
   **Ruby Type:** String

   **apt_package** and **dpkg_package** resources only. The direct path to the file used to pre-seed a package.

``response_file_variables``
   **Ruby Type:** Hash

   **apt_package** and **dpkg_package** resources only. A Hash of response file variables in the form of ``{"VARIABLE" => "VALUE"}``.

``retries``
   **Ruby Type:** Integer | **Default Value:** ``0``

   The number of attempts to catch exceptions and retry the resource.

``retry_delay``
   **Ruby Type:** Integer | **Default Value:** ``2``

   The retry delay (in seconds).

``source``
   **Ruby Type:** String

   Optional. The path to a package in the local file system.

   .. note:: The AIX platform requires ``source`` to be a local file system path because ``installp`` does not retrieve packages using HTTP or FTP.

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

``timeout``
   **Ruby Type:** String, Integer

   The amount of time (in seconds) to wait before timing out.

``version``
   **Ruby Type:** String, Array

   The version of a package to be installed or upgraded.

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

**Install a gems file for use in recipes**

.. tag resource_package_install_gems_for_chef_recipe

.. To install a gem only for use in recipes:

.. code-block:: ruby

   chef_gem 'right_aws' do
     action :install
   end

   require 'right_aws'

.. end_tag

**Install a gems file from the local file system**

.. tag resource_package_install_gems_from_local

.. To install a gem from the local file system:

.. code-block:: ruby

   gem_package 'right_aws' do
     source '/tmp/right_aws-1.11.0.gem'
     action :install
   end

.. end_tag

**Install a package**

.. tag resource_package_install

.. To install a package:

.. code-block:: ruby

   package 'tar' do
     action :install
   end

.. end_tag

**Install a package version**

.. tag resource_package_install_version

.. To install a specific package version:

.. code-block:: ruby

   package 'tar' do
     version '1.16.1-1'
     action :install
   end

.. end_tag

**Install a package with options**

.. tag resource_package_install_with_options

.. To install a package with options:

.. code-block:: ruby

   package 'debian-archive-keyring' do
     action :install
     options '--force-yes'
   end

.. end_tag

**Install a package with a response_file**

.. tag resource_package_install_with_response_file

Use of a ``response_file`` is only supported on Debian and Ubuntu at this time. Custom resources must be written to support the use of a ``response_file``, which contains debconf answers to questions normally asked by the package manager on installation. Put the file in ``/files/default`` of the cookbook where the package is specified and the chef-client will use the **cookbook_file** resource to retrieve it.

To install a package with a ``response_file``:

.. code-block:: ruby

   package 'sun-java6-jdk' do
     response_file 'java.seed'
   end

.. end_tag

**Install a specified architecture using a named provider**

.. tag resource_package_install_with_yum

.. To install a Yum package with a specified architecture:

.. code-block:: ruby

   yum_package 'glibc-devel' do
     arch 'i386'
   end

.. end_tag

**Purge a package**

.. tag resource_package_purge

.. To purge a package:

.. code-block:: ruby

   package 'tar' do
     action :purge
   end

.. end_tag

**Remove a package**

.. tag resource_package_remove

.. To remove a package:

.. code-block:: ruby

   package 'tar' do
     action :remove
   end

.. end_tag

**Upgrade a package**

.. tag resource_package_upgrade

.. To upgrade a package

.. code-block:: ruby

   package 'tar' do
     action :upgrade
   end

.. end_tag

**Use the ignore_failure common attribute**

.. tag resource_package_use_ignore_failure_attribute

.. To use the ``ignore_failure`` common attribute in a recipe:

.. code-block:: ruby

   gem_package 'syntax' do
     action :install
     ignore_failure true
   end

.. end_tag

**Avoid unnecessary string interpolation**

.. tag resource_package_avoid_unnecessary_string_interpolation

.. To avoid unnecessary string interpolation

Do this:

.. code-block:: ruby

   package 'mysql-server' do
     version node['mysql']['version']
     action :install
   end

and not this:

.. code-block:: ruby

   package 'mysql-server' do
     version "#{node['mysql']['version']}"
     action :install
   end

.. end_tag

**Install a package in a platform**

.. tag resource_package_install_package_on_platform

The following example shows how to use the **package** resource to install an application named ``app`` and ensure that the correct packages are installed for the correct platform:

.. code-block:: ruby

   package 'app_name' do
     action :install
   end

   case node[:platform]
   when 'ubuntu','debian'
     package 'app_name-doc' do
       action :install
     end
   when 'centos'
     package 'app_name-html' do
       action :install
     end
   end

.. end_tag

**Install sudo, then configure /etc/sudoers/ file**

.. tag resource_package_install_sudo_configure_etc_sudoers

The following example shows how to install sudo and then configure the ``/etc/sudoers`` file:

.. code-block:: ruby

   #  the following code sample comes from the ``default`` recipe in the ``sudo`` cookbook: https://github.com/chef-cookbooks/sudo

   package 'sudo' do
     action :install
   end

   if node['authorization']['sudo']['include_sudoers_d']
     directory '/etc/sudoers.d' do
       mode        '0755'
       owner       'root'
       group       'root'
       action      :create
     end

     cookbook_file '/etc/sudoers.d/README' do
       source      'README'
       mode        '0440'
       owner       'root'
       group       'root'
       action      :create
     end
   end

   template '/etc/sudoers' do
     source 'sudoers.erb'
     mode '0440'
     owner 'root'
     group platform?('freebsd') ? 'wheel' : 'root'
     variables(
       :sudoers_groups => node['authorization']['sudo']['groups'],
       :sudoers_users => node['authorization']['sudo']['users'],
       :passwordless => node['authorization']['sudo']['passwordless']
     )
   end

where

* the **package** resource is used to install sudo
* the ``if`` statement is used to ensure availability of the ``/etc/sudoers.d`` directory
* the **template** resource tells the chef-client where to find the ``sudoers`` template
* the ``variables`` property is a hash that passes values to template files (that are located in the ``templates/`` directory for the cookbook

.. end_tag

**Use a case statement to specify the platform**

.. tag resource_package_use_case_statement

The following example shows how to use a case statement to tell the chef-client which platforms and packages to install using cURL.

.. code-block:: ruby

   package 'curl'
     case node[:platform]
     when 'redhat', 'centos'
       package 'package_1'
       package 'package_2'
       package 'package_3'
     when 'ubuntu', 'debian'
       package 'package_a'
       package 'package_b'
       package 'package_c'
     end
   end

where ``node[:platform]`` for each node is identified by Ohai during every chef-client run. For example:

.. code-block:: ruby

   package 'curl'
     case node[:platform]
     when 'redhat', 'centos'
       package 'zlib-devel'
       package 'openssl-devel'
       package 'libc6-dev'
     when 'ubuntu', 'debian'
       package 'openssl'
       package 'pkg-config'
       package 'subversion'
     end
   end

.. end_tag

**Use symbols to reference attributes**

.. tag resource_package_use_symbols_to_reference_attributes

.. To use symbols to reference attributes

Symbols may be used to reference attributes:

.. code-block:: ruby

   package 'mysql-server' do
     version node[:mysql][:version]
     action :install
   end

instead of strings:

.. code-block:: ruby

   package 'mysql-server' do
     version node['mysql']['version']
     action :install
   end

.. end_tag

**Use a whitespace array to simplify a recipe**

.. tag resource_package_use_whitespace_array

The following examples show different ways of doing the same thing. The first shows a series of packages that will be upgraded:

.. code-block:: ruby

   package 'package-a' do
     action :upgrade
   end

   package 'package-b' do
     action :upgrade
   end

   package 'package-c' do
     action :upgrade
   end

   package 'package-d' do
     action :upgrade
   end

and the next uses a single **package** resource and a whitespace array (``%w``):

.. code-block:: ruby

   package %w{package-a package-b package-c package-d} do
     action :upgrade
   end

.. end_tag

**Specify the Homebrew user with a UUID**

.. tag resource_homebrew_package_homebrew_user_as_uuid

.. To specify the Homebrew user as a UUID:

.. code-block:: ruby

   homebrew_package 'emacs' do
     homebrew_user 1001
   end

.. end_tag

**Specify the Homebrew user with a string**

.. tag resource_homebrew_package_homebrew_user_as_string

.. To specify the Homebrew user as a string:

.. code-block:: ruby

   homebrew_package 'vim' do
     homebrew_user 'user1'
   end

.. end_tag
