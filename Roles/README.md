This is what they are all for:

files: This directory contains regular files that need to be transferred to the hosts you are configuring for this role. This may also include script files to run.
handlers: All handlers that were in your playbook previously can now be added into this directory.
meta: This directory can contain files that establish role dependencies. You can list roles that must be applied before the current role can work correctly.
templates: You can place all files that use variables to substitute information during creation in this directory.
tasks: This directory contains all of the tasks that would normally be in a playbook. These can reference files and templates contained in their respective directories without using a path.
vars: Variables for the roles can be specified in this directory and used in your configuration files.
Within all of the directories but the "files" and "templates", if a file called main.yml exists, its contents will be automatically added to the playbook that calls the role.


