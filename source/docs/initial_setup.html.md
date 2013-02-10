## Initial Setup

At this point, Triage should be installed and running on port 80. Point your browser to `http://your-server-ip-or-hostname/` and login using username `admin` and password `administrator`.

### Administration

The administration section can be accessed using the "Admin" link at the top right of the screen. Once in the admin section, you can get back to the main Triage application by clicking the "Triage" link at the top left of the screen.

### Managing Users

#### User Types

Triage supports two types of users: directory and database. Directory users are authenticated via a directory server, using LDAP (as configured in the [installation instructions](/docs/)). As a result, they don't store passwords, deferring to the directory server for password authentication. Database users have their encrypted password stored in the database, which they're able to update themselves as part of their account settings.

At this point, there's only the administrator user (which is a database user). Click the "Database Users" link at the top, and then click the "New Database User" button at the top right to add a new user. Here's a rundown on the user attributes:

* #### Username

    Used to login, as well as with Twitter-style mentions (@*username*).

    <p class="alert alert-info">
      The username attribute is used for binding directory users (using the attribute specified in <code>config/ldap.yml</code>).
    </p>

* #### Email

    Used for notifications.

* #### Name

    Proper name used for display purposes throughout Triage, as well as in notification emails.

* #### Role

    Controls the permissions given to the user. Also used for display purposes throughout Triage, as well as in notification emails. Here are the role types and their associated privileges:

* * ##### Administrator

    This role provides full access to all of Triage's functionality, including the administrative section, as well as the ability to view and modify all requests.

* * ##### Provider

    This role is meant for users that are providing support to others. They can read any request, can create requests of their own, and can comment and add attachments on any requests. They can also delete attachments that they've uploaded. However, they cannot delete requests.

* * ##### Requester

    This role is meant for end-users that will need support. They can create requests, and can comment and attach/delete files to these requests. They can only read requests they've created themselves.

<p class="alert alert-info">
  Triage does not provide the ability to delete requests or comments (at least not through the user interface). This is by design; requests should be closed if they have been resolved or are no longer relevant, and deleting comments can lead to incomprehensible conversations.
</p>

* #### Password

  Used for authentication purposes; as previously mentioned, this field is not present for directory users.

* #### Available

  Indicates whether or not the user is available for assignments. If the user is responsible for a system (see Responsibilities) and they are set to available, they'll be included as a possible assignee in the auto-assignment algorithm.

#### Responsibilities

You can associate any number of systems with a particular user by selecting them in the "Responsibilities" section (hold down the Ctrl or Cmnd key to select multiple systems). This will include the user when automatically assigning requests for the system.

### Managing Systems

Systems are what you and your company plan to support. When requests are created, they're associated with a system. Click the "Systems" link in the navigation panel on the left side of the screen to view the list of existing systems. Triage is added as an initial system by default; feel free to delete this entry if you don't want to allow requests relating to Triage.

#### Responsibilities

You can associate any number of users with a particular system by selecting them in the "Responsible Users" section (hold down the Ctrl or Cmnd key to select multiple users). These users will be used to automatically assign requests for this system.

### Managing Statuses

Statuses are used to signify a request's state. The default statuses are new, assigned, and closed, although you can replace these if you like. Aside from a title, statuses have two other key fields:

* #### Default

  When creating a request, user's aren't asked for a status, since all requests start with the same status. Which status they start with is dictated by this attribute. Only one status can be the default; this is typically the "new" status.

* #### Closed

  When requests have been completed, or are deemed invalid, they should be closed. This allows requesters and providers to be able to distinguish which requests still require work, and which have been completed. Typical examples of statuses that have this attribute checked are "closed/fixed" and "rejected".
