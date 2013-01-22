### Initial Setup

#### Login

At this point, Triage should be installed and running on port 80. Point your browser to `http://your-server-ip-or-hostname/` and login using username `admin` and password `administrator`.

#### Administration

The administration section can be accessed using the "Admin" link at the top right of the screen. Once in the admin section, you can get back to the main Triage interface by clicking the "Home" link at the top of the screen.

##### Managing Users

Click the "Users" link in the navigation panel on the left side of the screen to view the list of existing users. At this point, there's only the administrator account. Click the "Add New" tab above the user listing to add a new user. Here's a rundown on the user attributes:

###### Username

Used to login, as well as with Twitter-style mentions (@*username*).

<p class="note">
  The username attribute is used to bind the user in LDAP (using the attribute specified in <code>config/ldap.yml</code>).
</p>
