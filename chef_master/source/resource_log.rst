=====================================================
log resource
=====================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/resource_log.rst>`__

.. tag resource_log_summary

Use the **log** resource to create log entries. The **log** resource behaves like any other resource: built into the resource collection during the compile phase, and then run during the execution phase. (To create a log entry that is not built into the resource collection, use ``Chef::Log`` instead of the **log** resource.)

.. note:: By default, every log resource that executes will count as an updated resource in the updated resource count at the end of a Chef run. You can disable this behavior by adding ``count_log_resource_updates false`` to your Chef ``client.rb`` configuration file.

.. end_tag

Syntax
=====================================================
.. tag resource_log_syntax

A **log** resource block adds messages to the log file based on events that occur during the Chef Infra Client run:

.. code-block:: ruby

   log 'message' do
     message 'A message add to the log.'
     level :info
   end

The full syntax for all of the properties that are available to the **log** resource is:

.. code-block:: ruby

  log 'name' do
    level        Symbol # default value: :info
    message      String # default value: 'name' unless specified
    action       Symbol # defaults to :write if not specified
  end

where:

* ``log`` is the resource.
* ``name`` is the name given to the resource block.
* ``action`` identifies which steps the Chef Infra Client will take to bring the node into the desired state.
* ``level`` and ``message`` are the properties available to this resource.

.. end_tag

Actions
=====================================================
.. tag resource_log_actions

The log resource has the following actions:

``:nothing``
   .. tag resources_common_actions_nothing

   This resource block does not act unless notified by another resource to take action. Once notified, this resource block either runs immediately or is queued up to run at the end of the Chef Infra Client run.

   .. end_tag

``:write``
   Default. Write to log.

.. end_tag

Properties
=====================================================
.. tag resource_log_properties

The log resource has the following properties:

``level``
   **Ruby Type:** Symbol | **Default Value:** ``:info``

   The logging level for displaying this message.. Options (in order of priority): ``:debug``, ``:info``, ``:warn``, ``:error``, and ``:fatal``.

``message``
   **Ruby Type:** String | **Default Value:** ``The resource block's name``

   The message to be added to a log file. Default value: the ``name`` of the resource block. See "Syntax" section above for more information.

.. end_tag

Chef::Log Entries
=====================================================
.. tag ruby_style_basics_chef_log

``Chef::Log`` extends ``Mixlib::Log`` and will print log entries to the default logger that is configured for the machine on which the Chef Infra Client is running. (To create a log entry that is built into the resource collection, use the **log** resource instead of ``Chef::Log``.)

The following log levels are supported:

.. list-table::
   :widths: 150 450
   :header-rows: 1

   * - Log Level
     - Syntax
   * - Fatal
     - ``Chef::Log.fatal('string')``
   * - Error
     - ``Chef::Log.error('string')``
   * - Warn
     - ``Chef::Log.warn('string')``
   * - Info
     - ``Chef::Log.info('string')``
   * - Debug
     - ``Chef::Log.debug('string')``

.. note:: The parentheses are optional, e.g. ``Chef::Log.info 'string'`` may be used instead of ``Chef::Log.info('string')``.

.. end_tag

.. tag ruby_class_chef_log_fatal

The following example shows a series of fatal ``Chef::Log`` entries:

.. code-block:: ruby

   unless node['splunk']['upgrade_enabled']
     Chef::Log.fatal('The chef-splunk::upgrade recipe was added to the node,')
     Chef::Log.fatal('but the attribute `node["splunk"]["upgrade_enabled"]` was not set.')
     Chef::Log.fatal('I am bailing here so this node does not upgrade.')
     raise
   end

   service 'splunk_stop' do
     service_name 'splunk'
     supports status: true
     action :stop
   end

   if node['splunk']['is_server']
     splunk_package = 'splunk'
     url_type = 'server'
   else
     splunk_package = 'splunkforwarder'
     url_type = 'forwarder'
   end

   splunk_installer splunk_package do
     url node['splunk']['upgrade']["#{url_type}_url"]
   end

   if node['splunk']['accept_license']
     execute 'splunk-unattended-upgrade' do
       command "#{splunk_cmd} start --accept-license --answer-yes"
     end
   else
     Chef::Log.fatal('You did not accept the license (set node["splunk"]["accept_license"] to true)')
     Chef::Log.fatal('Splunk is stopped and cannot be restarted until the license is accepted!')
     raise
   end

The full recipe is the ``upgrade.rb`` recipe of the `chef-splunk cookbook <https://github.com/chef-cookbooks/chef-splunk/>`_ that is maintained by Chef.

.. end_tag

.. tag ruby_class_chef_log_multiple

The following example shows using multiple ``Chef::Log`` entry types:

.. code-block:: ruby

   ...

   begin
     aws = Chef::DataBagItem.load(:aws, :main)
     Chef::Log.info("Loaded AWS information from DataBagItem aws[#{aws['id']}]")
   rescue
     Chef::Log.fatal("Could not find the 'main' item in the 'aws' data bag")
     raise
   end

   ...

The full recipe is in the ``ebs_volume.rb`` recipe of the `database cookbook <https://github.com/chef-cookbooks/database/>`_ that is maintained by Chef.

.. end_tag

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

**Set default logging level**

.. tag resource_log_set_info

.. To set the info (default) logging level:

.. code-block:: ruby

   log 'a string to log'

.. end_tag

**Set debug logging level**

.. tag resource_log_set_debug

.. To set the debug logging level:

.. code-block:: ruby

   log 'a debug string' do
     level :debug
   end

.. end_tag

**Add a message to a log file**

.. tag resource_log_add_message

.. To add a message to a log file:

.. code-block:: ruby

   log 'message' do
     message 'This is the message that will be added to the log.'
     level :info
   end

.. end_tag
