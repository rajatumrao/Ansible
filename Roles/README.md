This is what they are all for:

files: This directory contains regular files that need to be transferred to the hosts you are configuring for this role. This may also include script files to run.
handlers: All handlers that were in your playbook previously can now be added into this directory.
meta: This directory can contain files that establish role dependencies. You can list roles that must be applied before the current role can work correctly.
templates: You can place all files that use variables to substitute information during creation in this directory.
tasks: This directory contains all of the tasks that would normally be in a playbook. These can reference files and templates contained in their respective directories without using a path.
vars: Variables for the roles can be specified in this directory and used in your configuration files.
Within all of the directories but the "files" and "templates", if a file called main.yml exists, its contents will be automatically added to the playbook that calls the role.


###Role Directory Structure
Here is what a typical role directory structure may look like:

common

├── handlers

│   └── main.yml
├── meta

│   └── main.yml
├── tasks

│   └── main.yml
├── templates

└── vars

    ├── Debian.yml
    ├── Ubuntu.yml
    └── main.yml
The 'tasks/main.yml' file is the where all the tasks are defined. Each task corresponds to an Ansible command that typically uses a module.

The 'meta/main.yml' file will contain a list of other roles that the current role depends on. Those roles' tasks will be executed before the current role, so it can be sure all its prerequisites are met.

The 'handlers/main.yml' file is where you keep your handlers, like the handler you saw earlier that starts Nginx after installation.

The templates directory is where you keep Jinja2 templates of configuration and other files that you want to populate and copy to the target system.

The vars directory contains various variables and can conditionally contain different values for different operating systems (very common use case).

It's important to note that Ansible is very flexible and you can put anything almost anywhere. This is just one possible structure that makes sense to me. If you look at other people's directory structures, you may see something completely different. That's totally fine. Don't be alarmed. Ansible is not prescriptive, although it does provide guidance for best practices.
