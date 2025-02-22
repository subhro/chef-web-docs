=====================================================
link resource
=====================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/resource_link.rst>`__

.. tag resource_link_summary

Use the **link** resource to create symbolic or hard links.

.. end_tag

A symbolic link---sometimes referred to as a soft link---is a directory entry that associates a file name with a string that contains an absolute or relative path to a file on any file system. In other words, "a file that contains a path that points to another file." A symbolic link creates a new file with a new inode that points to the inode location of the original file.

A hard link is a directory entry that associates a file with another file in the same file system. In other words, "multiple directory entries to the same file." A hard link creates a new file that points to the same inode as the original file.

Syntax
=====================================================
A **link** resource block creates symbolic or hard links. For example, to create a hard link from ``/tmp/file`` to ``/etc/file``:

.. code-block:: ruby

   link '/tmp/file' do
     to '/etc/file'
     link_type :hard
   end

Because the default value for ``link_type`` is symbolic, and because properties that are not specified in the resource block will be assigned their default values, the following example creates a symbolic link:

.. code-block:: ruby

   link '/tmp/file' do
     to '/etc/file'
   end

The full syntax for all of the properties that are available to the **link** resource is:

.. code-block:: ruby

  link 'name' do
    group            String, Integer
    link_type        String, Symbol # default value: :symbolic
    mode             Integer, String
    owner            String, Integer
    target_file      String # default value: 'name' unless specified
    to               String
    action           Symbol # defaults to :create if not specified
  end

where:

* ``link`` is the resource.
* ``name`` is the name given to the resource block.
* ``action`` identifies which steps the chef-client will take to bring the node into the desired state.
* ``group``, ``link_type``, ``mode``, ``owner``, ``target_file``, and ``to`` are properties of this resource, with the Ruby type shown. See "Properties" section below for more information about all of the properties that may be used with this resource.

Actions
=====================================================

The link resource has the following actions:

``:create``
   Default. Create a link. If a link already exists (but does not match), update that link to match.

``:delete``
   Delete a link.

``:nothing``
   .. tag resources_common_actions_nothing

   This resource block does not act unless notified by another resource to take action. Once notified, this resource block either runs immediately or is queued up to run at the end of the Chef Infra Client run.

   .. end_tag

Properties
=====================================================

The link resource has the following properties:

``group``
   **Ruby Type:** String, Integer

   A string or ID that identifies the group associated with a symbolic link.

``link_type``
   **Ruby Type:** String, Symbol | **Default Value:** ``:symbolic``

   The type of link: ``:symbolic`` or ``:hard``.

``mode``
   **Ruby Type:** Integer, String | **Default Value:** ``777``

   If ``mode`` is not specified and if the file already exists, the existing mode on the file is used. If ``mode`` is not specified, the file does not exist, and the ``:create`` action is specified, the chef-client assumes a mask value of ``'0777'`` and then applies the umask for the system on which the file is to be created to the ``mask`` value. For example, if the umask on a system is ``'022'``, the chef-client uses the default value of ``'0755'``.

   The behavior is different depending on the platform.

   UNIX- and Linux-based systems: A quoted 3-5 character string that defines the octal mode that is passed to chmod. For example: ``'755'``, ``'0755'``, or ``00755``. If the value is specified as a quoted string, it works exactly as if the ``chmod`` command was passed. If the value is specified as an integer, prepend a zero (``0``) to the value to ensure that it is interpreted as an octal number. For example, to assign read, write, and execute rights for all users, use ``'0777'`` or ``'777'``; for the same rights, plus the sticky bit, use ``01777`` or ``'1777'``.

   Microsoft Windows: A quoted 3-5 character string that defines the octal mode that is translated into rights for Microsoft Windows security. For example: ``'755'``, ``'0755'``, or ``00755``. Values up to ``'0777'`` are allowed (no sticky bits) and mean the same in Microsoft Windows as they do in UNIX, where ``4`` equals ``GENERIC_READ``, ``2`` equals ``GENERIC_WRITE``, and ``1`` equals ``GENERIC_EXECUTE``. This property cannot be used to set ``:full_control``. This property has no effect if not specified, but when it and ``rights`` are both specified, the effects are cumulative.

``owner``
   **Ruby Type:** String, Integer

   The owner associated with a symbolic link.

``target_file``
   **Ruby Type:** String | **Default Value:** ``The resource block's name``

   An optional property to set the target file if it differs from the resource block's name.

``to``
   **Ruby Type:** String

   The actual file to which the link is to be created.

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

Examples
=====================================================
The following examples demonstrate various approaches for using resources in recipes:

**Create symbolic links**

.. tag resource_link_create_symbolic

The following example will create a symbolic link from ``/tmp/file`` to ``/etc/file``:

.. code-block:: ruby

   link '/tmp/file' do
     to '/etc/file'
   end

.. end_tag

**Create hard links**

.. tag resource_link_create_hard

The following example will create a hard link from ``/tmp/file`` to ``/etc/file``:

.. code-block:: ruby

   link '/tmp/file' do
     to '/etc/file'
     link_type :hard
   end

.. end_tag

**Delete links**

.. tag resource_link_delete

The following example will delete the ``/tmp/file`` symbolic link and uses the ``only_if`` guard to run the ``test -L`` command, which verifies that ``/tmp/file`` is a symbolic link, and then only deletes ``/tmp/file`` if the test passes:

.. code-block:: ruby

   link '/tmp/file' do
     action :delete
     only_if 'test -L /tmp/file'
   end

.. end_tag

**Create multiple symbolic links**

.. tag resource_link_multiple_links_files

The following example creates symbolic links from two files in the ``/vol/webserver/cert/`` directory to files located in the ``/etc/ssl/certs/`` directory:

.. code-block:: ruby

   link '/vol/webserver/cert/server.crt' do
     to '/etc/ssl/certs/ssl-cert-name.pem'
   end

   link '/vol/webserver/cert/server.key' do
     to '/etc/ssl/certs/ssl-cert-name.key'
   end

.. end_tag

**Create platform-specific symbolic links**

.. tag resource_link_multiple_links_redhat

The following example shows installing a filter module on Apache. The package name is different for different platforms, and for the Red Hat Enterprise Linux family, a symbolic link is required:

.. code-block:: ruby

   include_recipe 'apache2::default'

   case node['platform_family']
   when 'debian'
     ...
   when 'suse'
     ...
   when 'rhel', 'fedora'
     ...

     link '/usr/lib64/httpd/modules/mod_apreq.so' do
       to      '/usr/lib64/httpd/modules/mod_apreq2.so'
       only_if 'test -f /usr/lib64/httpd/modules/mod_apreq2.so'
     end

     link '/usr/lib/httpd/modules/mod_apreq.so' do
       to      '/usr/lib/httpd/modules/mod_apreq2.so'
       only_if 'test -f /usr/lib/httpd/modules/mod_apreq2.so'
     end
   end

   ...

For the entire recipe, see https://github.com/onehealth-cookbooks/apache2/blob/68bdfba4680e70b3e90f77e40223dd535bf22c17/recipes/mod_apreq2.rb.

.. end_tag
