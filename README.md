# CPSC4620_MeTube
Semester project for CPSC4620, Clemson University Undergraduate Section.

In order to make use of this project, a
MySQL database would need to be created for metadata, account, comment, and message hosting using the SQL statements
in the file MeTube_U6.sql. A file config.php would then need to be created, setting variables to point to the created
database, $dbhost, $dbuser, $dbpass, and $database. Finally, the script setPermissions.sh should be run to ensure that
the permissions of the server's files are set in such a way to allow the site to function.

The following are the features included in this implementation, how to access them, and how they are implemented.

**Registration**

The registration page is accessed through an anchor tag link from the login page, so that
if one does not have an account yet with MeTube, they can create an account. From there, the
page is loaded from the register.php file, with that page containing an HTML form with text
fields for the user’s username, email, password, and a repeated entry of the user’s password.
Styling is handled through a combination of Bootstrap css classes and custom classes in
css/default.css .

As the user enters information, it is verified through JavaScript in the file
js/register_page.js . In that file, the username is validated by making sure that it is greater
than 0 characters, that it is less than or equal to 30 characters, that it only contains letters,
numbers or underscores, and that it is not already being used. This validation takes effect each
time the field is changed. The character composition check is accomplished by a comparison to a
regular expression. An AJAX request is used to check if the username is already being used,
calling an action in editAccountAjax.php which in turn calls a function in function.php
which returns 1 if the user exists, or another code based on whether an error was encountered or
the user did not exist. If any validation for the username fails, a message is printed inline to
notify the user. Email is verified only based on formatting, using a regular expression pattern
match to make sure that the email is formatted properly. Passwords are checked in JavaScript to
make sure that they are over 8 characters long and the entered and re-entered passwords match
each other. The email and passwords also give inline validation notifications.

Once the user has entered data in all fields and they have been verified, the submit button
will work for the user. When clicked, it sends an AJAX request to editAccountAjax.php
which, if all variables are posted correctly, calls the function add_account_to_db in
function.php . That function hashes the password using PHP’s password_hash function to
prevent storing the password in plaintext, then inserting a new entry into the account database
including the account’s username, password, and email. The function returns a 1 on success, or a
code on error. If the function succeeds, the AJAX request handler sets the PHP username session
variable to the new username (logging in the user) and echoes a success, otherwise echoing an
error string. If the JavaScript receives a successful code from the AJAX request, it redirects the
user to the index page. If it does not receive a successful code, it spawns an alert stating the error
to the user.

**Sign-In**

The sign-in page can be accessed through an anchor tag link from the header of any page
on the website or through a redirection from some pages which require the user to be logged in to
function. As much of the login functionality was already provided in the template for this project
provided by the TA, this page is implemented somewhat differently than the other functionality
of our project, using internal PHP code to handle the login post response rather than an AJAX
request to another file.

When loading login.php , a PHP script checks whether the page’s HTML form was
submitted before the page was loaded. If so, it make sure that the user has entered values for both
the username and password fields of the page’s HTML form, showing an error if they were not
both entered. Then, the script handles the login action, calling the function user_pass_check
in function.php , which queries the database for the password hash of the user with the given
username, using PHP’s password_verify function to check the hash against the entered
password. If the user entered incorrect information, the script then notifies the user, otherwise
setting the PHP username session variable to the user’s username with the capitalization patterns
retrieved from the database and redirecting them to whichever page was stored in the PHP
prevpage session variable which we use for storing the last page a user was on before a login
attempt. If no such page was set, the user is redirected to the index page.
Following that PHP script, the login.php file contains the HTML form containing the
user inputs needed to log in, being the username and password fields. These are formatted
through Bootstrap classes and custom classes in css/default.css as well as some inline
styling. The form submits data through the HTML POST method, simply refreshing the page
when submitted to allow for the internal PHP script to run and handle the login attempt.

**Profile Update**

The account edit page is accessed through a link on the left of the account page, with the
main page being contained in editaccount.php . There are two main forms associated with
account editing which are dynamically loaded through AJAX. Those two actions are editing
profile metadata and changing one’s password, with the forms for each contained in
editprofileinfo.php and updatepassword.php . The account editing page itself contains
HTML markup for a sidenav, formatted with custom CSS classes from the css/default.css
file. In addition, there is an empty div which is used to contain whichever form is loaded,
defaulting to the edit profile info form. The sidenav is given functionality through the
js/edit_account_page.js file. When a form is loaded, its validation and submission scripts
are automatically set through JavaScript. The main edit account page also has a PHP script to
redirect the user to log in if they are not already, setting the PHP prevpage session variable so
that the user will be redirected back to the edit account page after logging in.

The editprofileinfo.php page is used to change the email and bio for an account,
containing text fields for each piece of information. A database query is used to fill in the user’s
current email and bio by default. JavaScript is used to validate the email, making sure that one is
entered, that it is not too long, and that it is the correct format for an e-mail. The format
validation is done through a regular expression pattern match. The bio is also validated through
JavaScript, making sure that it does not surpass 750 characters. When both items are successfully
validated, the form can be submitted. Once submitted, an AJAX request is submitted to the
editAccountAjax.php file, which calls the update_account_info function in
function.php if the user is logged in and both pieces of data are passed correctly. That
function submits a query to update the account information in the database, returning true on
success, or false on failure. The AJAX request handler then echoes success if it gets a true value,
or echoes an error message if a false value is received. Finally, the JavaScript which receives the
AJAX response prints an inline validation message to the user to show a success or failure.

The updatepassword.php page is similar, but is used to update the user’s password,
containing fields for the current password, the new password, and a repeat of the new password.
JavaScript is used to validate the form, making sure that the current password is entered, the new
passwords are entered, that the new passwords match, and that the new passwords are at least 8
characters in length. If a validation step is failed, an error is printed inline. When all validation
steps are passed, the user can submit the form. Once submitted, an AJAX request is submitted to
the editAccountAjax.php file, which calls the update_user_pass function in
function.php if the user is logged in and all pieces of data are passed correctly. That function
first checks to make sure the user provided the correct current password, returning an error code
if it is incorrect. If correct, the new password is hashed using PHP’s password_hash function,
then set in the database through a query. A code of 0 is returned on success, or a 1 on failure. The
AJAX request handler the echoes echoes success if the attempt was successful, or an error
message depending on the error encountered. Once the JQuery code receives the response, it
prints an inline message informing the user of the password change request success or failure.

**Upload**

The Upload page is accessed via the Account page by clicking the button that says “New
Upload.” This button only appears when the user is logged in and on their own account page. On
click, the user is redirected to media_upload.php , which contains forms for the file
submission, title, description, category, keywords, and comment permissions of a media to be
uploaded. The page also displays the supported file types and the maximum file size allowed.

Submission of the form, via the “Upload” button, is blocked until all fields have valid
values. This Validation is implemented in the file js/media_upload.js . File input is
validated by checking the file extension on the file to be uploaded. If the extension is not of one
of the supported types, then the upload is prevented. The “Title” field is validated by ensuring it
is nonempty. The “Title” field is also restricted to a maximum length of 40 characters through
html. The “Description” field is similarly limited to 998 characters in html. The “Keywords”
field requires a space delimited list of “words,” which for the implementation are just strings
consisting solely of the characters a-z. The “Keywords” field converts uppercase letters input to
it to lowercase, then, using regular expression matching, the field is checked to ensure proper
formatting. When the “Upload” button is pressed and all of the forms are properly validated, the
form is allowed to POST.

The POST is handled in the file media_upload_process.php . First, the file checks to
see if a directory exists for the user’s uploads, and, if not, creates that directory. Next, the process
checks to see if there were file errors. If there was an error, the user is redirected to the index. If
there was not an error, then the uploaded file is moved to its location under the user’s directory,
and a SQL statement is prepared. Form data is added to this prepared statement from the PHP
_POST variable, and the query is executed. Since keywords are maintained in a separate table,
the process makes use of the array_from_keywords function in function.php to get an
array of strings corresponding to the string sent from the “Keywords” field. For each of these
strings, the process then makes use of the add_media_keyword function in function.php to
add these keywords to the database. Once all of this is finished, the user is redirected to
index.php.

**Media Metadata Input**

Media metadata can be adjusted originally through the use of media_upload.php (see
Upload ). Once uploaded, media metadata can be edited by clicking the “edit” icon in the media
box on the uploader’s account page. This redirects the user to editmedia.php?id=<id> ,
where <id> is the mediaid of the media.

editmedia.php first ensures that the user is logged in, that the id field in the GET
request is set, and that the currently logged in user is the uploader of the media with that id. If
any of these checks fails, then the user is redirected to index.php . Otherwise, the page is
loaded as normal with prepopulated fields for title, description, category, keywords, and
comment permissions. Javascript is used to provide validation to the title and keyword fields
prior to POST in a similar manner described in Upload.

The POST request is handled by media_edit_process.php. This process first checks
to make sure all required fields are set, and, if not, redirects the user to index.php. If the checks
pass, then the media metadata is edited through use of the update_media_metadata function
defined in function.php. The user is then redirected back to their account page.

**Download/View**

The view and download actions are both accessed through the media.php page, which is
linked to from most other pages on the site where media items are shown. That page displays a
media item based on a media id passed through the GET method, as well as playlist information
based on a playlist id passed through the GET method. These are passed in the format
media.php?id=<mediaid>&playlistid=<playlistid> through the URL. Most of the
page is loaded dynamically based on PHP scripts, with the page being formatted through
Bootstrap CSS and some custom CSS classes in css/default.css.

When loaded, the page first checks whether a media id was supplied. If not, an error
message is displayed in a Bootstrap alert div. The page then attempts to retrieve the information
from the database for the selected media item. If the information is found, the page continues,
otherwise displaying an error div in place of the media item. Once the media item’s information
is loaded, the page begins to be populated. The recommendations container at the side of the
media item is set first. If a playlist id is set, then it shows all the media items in that playlist other
than the current media item, querying the database to find all other media items in the playlist
with the given id. A message is displayed if the playlist cannot be found, or no other media exist
within the given playlist. If the playlist id was not set at all, then the recommendations box
shows media in the same category as the current media item, querying for the ten most recently
uploaded media items which are in the same category as the current media item. In each case, the
media items displayed are given with links to them so that the user can immediately view those
media items.

Once the similar media container is loaded, the media item’s title and the media item
itself are shown, printing a heading and an image, audio, or video element based on the type of
media which was loaded, as retrieved from the database. The source for that element is the
retrieved filepath from the database. Below the media, the media details container is printed,
showing the other details retrieved from the media item’s entry in the database, such as the
uploader, upload time, description, category, and keywords. If the user is logged in, buttons are
displayed for favoriting/unfavoriting the media item and a dropdown for adding/removing the
media item from playlists. The favoriting utility is described in further depth in the Favorites
section, and the playlist utility is described further in the Playlists section. In addition, a
download button is always visible. This allows for the downloading of media files by the user,
linking through an anchor tag to the media item’s location in the file system. When clicked, a
JavaScript function in js/media_view.js submits a post request to the file
media_download_process.php which records the download, username (if the user is logged
in), download time, and download IP of the download action in the download table of the
database.

Beneath the media item and its metadata display is the comments section, which is only
displayed if the media owner has enabled comments on their media upload. Otherwise, a heading
is displayed stating that comments have been disabled for the media item in question. If the user
is logged in and comments are enabled, a comment creation form is shown above the previous
comments, with a comment textarea and a submit button for the form. Below that, a list of
current comments is displayed, as retrieved from a query to the database. If no comments exist, a
message is shown that no comments have been made yet. If a user is logged in and was the one
who made one or more of the comments on the video, then those comments are loaded with
delete and edit buttons for the user. The comment functionality is further described in the
Commenting section.

**Browse by Category**

Users can browse media by category through index.php . On this page, there is a
sidenav allowing the user to navigate between the categories to view recently added media in
that category. The categories thus implemented are: Funny, Music, Sports, Informative, and
Other. In addition to these categories, the sidenav includes “Home,” which displays recent media
regardless of category, recently added playlists, and, if the user is logged in, recent subscription
activity.

When a category on the sidenav in index.php is clicked, an AJAX POST request is sent
from the accompanying javascript file, browse_page.js ,to the server and the data returned
refreshes the body content in the page corresponding to the category the user clicked. The POST
request is processed in a file called browsecategory.php . The category to be retrieved is
passed as a parameter in the AJAX call. In browsecategory.php , the parameter is used to
select up to 40 media corresponding to that category string. The results of the query are then
formatted in html and sent back to the user to refresh the body content of index.php .

**Channels**

The channel system is handled on the page account.php . The page is accessible by
clicking on “Account” at the top left of any page to view the currently logged in user’s page, or
by clicking any username on the website, such as on a media page or search results. The page is
divided into a left section which contains information about the user whose page is being viewed.
The right side is split up by the user’s uploads, playlists, and favorites. The layout is based on a
Bootstrap container along with some custom CSS which places media items in boxes on the
page.

When the page is loaded, there are a few possibilities that can run. The channel you are
viewing is specified in the URL using the GET method by ?username=<username> .

1. If a correct username is provided, that page is shown.

2. If a nonexistent username is provided, the page displays an error message.

3. If no username is provided and the current user is logged in, they are redirected to their own channel page.

4. If no username is provided and the current user is not logged in, the user is redirected to index.php.

All channels show the user’s email, bio, subscriptions, uploads, playlists, and favorites.
These are retrieved from the database by SELECT ing based on the username given by the GET
method. The media contents are ordered by upload date, most recent first. Displaying playlists
requires two queries: one to fetch each of the user’s created playlists and another that SELECT s
each video from each playlist inside the loop fetching the first query’s results.

Javascript JQuery handles the logic for the buttons on the page in the file
js/account_view.js . Every channel page has a button underneath the channel username;
Javascript sets the text and logic based on the login status of the current user and a value field in
the HTML button element which is set after a query to check if the current user is subscribed to
the channel they are viewing. If the user is viewing their own page, it allows them to edit their
page by redirecting them to editaccount.php . If the current user is viewing another person’s
channel page and is logged in, the button allows them to subscribe if they are not subscribed or
unsubscribe if they are subscribed already. If the current user is not logged in, the button will
read “Log in to subscribe” and redirect them to the login page. The user will be returned to the
channel page they were viewing if they log in successfully. When the user is logged in and
viewing their own page, three other buttons also appear: “Messages,” “New Upload,” and “Add
Playlist.” These allow you to view your message page, upload new media, and create a new
playlist, respectively; their functionality will be discussed in other sections. When logged in and
viewing another user’s page, the current user also has the ability to send that user a message
directly from the left pane in a message box. In order to decide which of the variable elements
should be on the page, a PHP _SESSION variable is used to determine the user’s login status
along with the username passed through the GET method.

To process subscriptions and content deletions, AJAX requests are sent through the
HTTP POST method to separate PHP files via Javascript JQuery. Subscribing, unsubscribing,
deleting an upload, and sending a message directly to the user are handled by the same PHP file,
accountViewAjax.php (sending messages from the message page is handled separately). The
request is done by sending the username of the channel you are viewing and an “action” number.
The action number determines what the user requested: 0 to subscribe, 1 to unsubscribe, 2 to
send the message, and 3 to delete a media item. The PHP code will echo its response which will
be used to tell the user of success or failure. The functions in function.php are used to attempt
an insertion to or deletion from the database: add_subscription , remove_subscription ,
add_message , and remove_media . In the case of the remove_media function, the media
item is also removed from the filesystem as it is removed from the database. If successful,
“success” is echoed; otherwise, “failure” is echoed for subscribing issues or a message
describing the problem with the message if the problem is caused by the user. In this way, the
validation for messages is handled on the back-end. Upon success, an HTML element is made
visible for messages. For a successful subscription operation, the text of the subscription button
is flipped to “Subscribe” or “Unsubscribe” depending on if the user was previously subscribed or
not. The value of the button is also changed so the next click will have the correct logic attached
to in in Javascript. Content deletions from playlists and favorites are handled by the page
mediaViewAjax.php using the same method as before: an “action” is requested, this time with
an action number, media ID, and playlist ID if deleting from a playlist. Playlist deletions are
assigned to action 7 and favorite deletions are action 4. function.php again handles the
querying, attempting to make the requested deletions from the database through the functions
remove_playlist_media and remove_favorited_media respectively.

**Playlists**

The MeTube system allows logged in users to create, edit, and delete playlists and allows
all users, logged in or not, to view the media inside playlists. Playlist functionality is split accross
a few pages. On a channel page, a user can view another user’s playlists or view and edit their
own. Clicking on any playlist name on the website will send the user to playlist.php which
contains all of the metadata and contents of a playlist. A user can add a media item to a playlist
only while viewing the media item. Under the media item, a playlists dropdown menu will allow
the user to add the media to or remove it from any of their playlists.

When viewing a user’s channel (see Channels ), anyone can see the user’s playlists.
However, if the logged in user is viewing their own page, they also have access to an Add
Playlist button. The button’s logic is set using Bootstrap and the HTML tag data-toggle =
”modal” to open a dialogue box. The dialogue box prompts for a new playlist name. Clicking
cancel closes the dialogue. The Submit button’s logic is set through JavaScript JQuery. Clicking
submit will send an AJAX request from js/account_page.js to playlistViewAjax.php .
Validation for the playlist name is handled on the front-end using JavaScript before being sent in
the AJAX request. The PHP page that handles the request uses the function create_playlist
in function.php to add a playlist to the database. If the action was successful, “success” is
echoed and the account page is refreshed in order to show the new playlist to the user. By
default, the playlist is empty and has no metadata set. It must be edited from the playlist page or
added to on a media page. The logged in user may also delete videos from their playlists directly
from their channel page by clicking the small “x” at the top right of each media item in the list.
This is handled in the same manner by sending an AJAX request to call a function to remove the
media from the playlist. If it is successful, the <div> which contained the media item is deleted
so that the page does not have to be refreshed.

When viewing a media item, a logged in user will see a dropdown menu under the media
item and next to the download button. The dropdown menu will contain all of the user’s playlists
so they may add the current media item to a playlist or remove it if it is already in a playlist. The
button for the dropdown and the unordered list it is linked to is included by a separate PHP page,
playlistDropdown.php . This page queries for all of the logged in user’s playlists and checks
if the current media is or is not in the playlist. This also determines whether the button in the
dropdown list for a playlist says “add” or “remove.” The logic for the add and remove buttons in
the dropdown menu is set by JavaScript in js/media_view.js . An AJAX request is sent to
mediaViewAjax.php with an “action” number, media ID, and playlist ID. Action 6 is to add to
and action 7 is to remove from the playlist. The playlist ID determines what playlist is being
modified and the media ID determines what is being added or removed. If “success” is echoed,
the dropdown list is refreshed from playlistDropdown.php .

The primary way to view all of the information about a playlist is via
playlist.php?id=<playlistid> where the playlist ID is an ID that the database assigned
to a playlist when it was created. When the page loads, the playlist ID is retrieved from GET and
the information regarding the playlist is queried from the database on the matching ID. If no ID
is specified in the GET method, the user is redirected to index.php . First the metadata from the
query is displayed at the top of the page, then each media ID in the playlist is looped through to
retrieve their information from the database to be displayed on the page. If the logged in user
owns the playlist they are viewing, there are also edit and delete buttons on the metadata pane at
the top of the page, and each media item has a delete button on it to remove it from the playlist.
To see if the current user owns the playlist, the username of the creator stored in the database is
matched against a PHP _SESSION variable. The logic for each of the delete and edit buttons is
set in JavaScript by js/playlist_view.js . When the buttons are clicked, an AJAX request is
sent to playlistViewAjax.php which uses an “action,” media ID, and playlist ID to make
calls to the functions in function.php . When the edit button for the playlist metadata is
clicked, the user is taken to editplaylist.php?id=<playlistid> and the user is able to
enter a name, description, and keywords for their playlist. The fields are prefilled with the
playlist’s metadata queried from the playlist. When the user makes a change and clicks submit,
another AJAX is sent to update the database entry for the playlist. If the user clicks cancel, the
changes are discarded and the user is taken back to the main playlist page.

**Favorites**

The favoriting functionality can be accessed in two ways: on a media item on the
media.php page, or on a user’s own channel page in account.php . If a user is logged in, then
whenever they view a media item, a star icon button will be displayed, with the icon being set
based on whether or not the user has already favorited the viewed media item. As such, a filled
star is shown and the action is set to unfavorite the media item if the user has already favorited it,
while an empty star is shown and the action is set to favorite the media item if the user has not
already favorited it. When clicked, the favoriting or unfavoriting action is done through
JavaScript in js/media_view.js . The onclick function there sends an AJAX request to the
mediaViewAjax.php page with the action depending on whether the user is favoriting or
unfavoriting the media item. If the media id is properly set in the request and the user is logged
in, the AJAX request handler then calls either add_favorited_media or
remove_favorited_media in function.php depending on the action selected. Those two
functions attempt to either add an item to the favorited_media table in the database for the given
username and media item, or remove the entry for the given username and media item
respectively. Each returns true on success and false on failure. The AJAX request handler then
echoes success if the action was completed successfully, or echoes an error message otherwise.
Once the AJAX request response is returned to JavaScript, the AJAX request sender shows an
alert if an error was encountered. If the request was successful, the favorite/unfavorite button is
changed to the opposite state to what it was, changing to the favorite media button if the media
item was just unfavorited, or changing to the unfavorite media button if the media item was just
favorited.

As described in the Channels section, a user’s favorites are displayed on their account
page. If the user is logged in and on their own account page, a delete from favorites button is also
shown on each favorited media item. The js/account_view.js script file handles clicks to
those buttons, with a click to one sending a request to the same action in mediaViewAjax as the
unfavorite button on the media view page. If successful, the div containing that media item’s
information in the favorites list is deleted from the account page. On failure, an alert is displayed
to the user describing the failure.

**Messaging**

Messaging in the MeTube system is possible in two ways: a user can send a message
directly to another user via that user’s channel page at account.php?username=<username>
or to multiple users at once from the main messages page at messages.php . A user can also
view their sent and received messages from this page. The messages page is accessible from the
current user’s channel page by clicking the “Messages” button next to “Edit profile.” The first
method of sending a message is described in the Channels section.

Upon loading the messages page, a PHP _SESSION variable is checked to see if the user
is logged in. If the user is not logged in, they are redirected to login.php to login. After
successful login, the user is redirected back to the messages page. If the user is logged in to
MeTube, they are shown the messages page which is laid out in three sections using a Bootstrap
container: Received Messages, Sent Messages, and Create a message. The received and sent
messages sections are each separately scrollable so a user can look through each section
independently of the other. When the page is loaded the database is queried for all messages sent
to the logged in user and the messages are displayed along with a reply button. Sent messages
require two queries, because the messages are grouped by recipients. First, all messages sent
from the current user are retrieved from the database. While looping through each entry, another
query uses the message ID to check for each user who received that specific message. By doing
this, the Sent Messages section is able to avoid showing the user multiple copies of the same
message instance sent to different users. Instead, each message entry lists all of the users to
whom it was sent. In both the Received and Sent Messages sections, the message entries are
sorted by date, most recent first, using ORDER BY DESC in the MySQL query.

In the third pane of the messages page, a user may send a message to one or more users
by specifying their usernames in the Recipients field, separated by spaces. If all the specified
users exist and the message is not too short or too long, the message will be sent upon clicking
the send button. The logic for both the send button and the reply buttons are set using JavaScript
in the file js/message_page.js . Using an HTML id field on the button elements, the logic is
set when the page is finished loading. Each reply button also contains a value field which is set
to the username of the person who sent the message when the message queries run while the
page is loading. When the button is clicked, JavaScript sets the Recipients field to the username
of the person who sent the message using the button’s value. When the send button is clicked,
Javascript sends an AJAX request to messagePageAjax.php using the POST method with an
“action” number, the list of recipients, and the message. The only action that will ever be
requested by this page is action 0, which attempts to send a message. In the PHP file the recipient
list is split into an array using a function in function.php , array_from_keywords . This
function was designed to parse keywords for media uploads, but also works to parse message
recipients into an array. All whitespace is ignored so two valid recipients can be separated by an
arbitrary number of spaces. Once the usernames are in the array, they are each checked against
the database to ensure they exist using the function user_exist_check . If any user in the list
did not exist, the message is not sent to any of the recipients and an alert is shown to the user
telling them which usernames were incorrect. If all usernames were correct, the message is then
validated. If the message was not too short or too long and all the usernames existed, “success” is
echoed; otherwise, “failure” is echoed for AJAX failures or a specific error message if the error
originated from the user. Upon success, JavaScript clears the recipients list and message and sets
a success message in HTML to be visible. Upon system failure, an alert is shown. Upon a user
error, an error HTML message is set to be visible, unless the error was due to incorrect
recipients; then an alert is shown in order to preserve the layout of the page if the list of incorrect
recipients was very long.

When a message is sent successfully, the Sent Messages section is refreshed by sending
an AJAX request to messagessent.php which performs the same query that ran when the
page loaded in order to repopulate the Sent Messages section including the newly sent message.
This query is housed in a different PHP file so it can be called multiple times from the messages
page.

**Commenting**

Comments are displayed on the media.php page if the uploader has allowed comments.
Under the media player, a form is displayed consisting of a textarea for the comment text and a
submit button. When the user submits the comment, an AJAX POST request is sent from the
accompanying javascript file, media_view.js , to the server to be processed by a file called
mediaViewAjax.php , which responds to all AJAX requests from media.php . Upon
successful submission, the comments are refreshed, such that newly added comments are
displayed.

Users are also allowed to edit or delete comments on the media.php page. When a
comment has been submitted by the current user, an edit and a delete icon are displayed in the
top right of the comment box. Upon clicking the edit icon, an AJAX POST request is sent to
mediaViewAjax.php , which then provides a form consisting of a textarea prefilled with the
comment text, a submit button, and a cancel button. Pressing either submit or cancel will send
yet another AJAX request refreshing the comments section, but cancel will not update the
current comment text. Upon pressing the delete icon, an AJAX request is sent to
mediaViewAjax.php removing the current comment from the database. The comments section
is then refreshed to show the changes.

Comments are validated before submission in media_view.js by checking the length
of the string contained in the textarea. If the string is less than 10 characters or greater that 1000
characters, an error message is displayed in the comment box and no AJAX requests are sent.

**Keyword Search**

The page search.php handles MeTube’s search functionality. The search page can be
reached by typing into the search bar at the top of any MeTube page and clicking the search
button, and the search terms will be sent to the search page via the GET method using
?query=<searchterms> MeTube supports searching for media by its title, keywords, or
uploader. The search page displays up to forty media results per page, and pages can be changed
using the page button in the top right of the search results next to the current page number.

Upon loading the page, the search query is taken from the GET method and used to make
three MySQL queries to the database: each term in the search query is matched against each
word in the titles of all media and their keywords as well as matched against each username
using LIKE with an expression in the format “%searchterm%” which will match any appearance
of the search term as a substring in media titles and keywords or usernames. When a piece of
media is matched using this method, it is placed in an associative array with a key/value pair.
The key is the media ID and the value is the number of times it was matched. Once the queries
are completed, the resulting media IDs in the array are used to perform a final SELECT and
populate the search results in descending order by the values in the key/value pair. Since the
value represents the number of times the media ID was matched, this means the search results
are ordered by relevance to the search terms -- the more matches they had, the further up the list
they appear. If no search terms are provided, the user is shown an error message on the page. The
results which appear on the page are also determined by which result page the user is viewing.
Since there are forty results per page, the results are limited by looping through all of the media
IDs in the associative array, but limiting which ones are shown by a loop index. If the index is
between 40 * previous page number and 40 * page number , the results are shown.
The page number is determined through the page number in the URL passed via the GET
method. If the page number is not present in GET, it is assumed to be page one. The logic for the
buttons “Prev” and “Next” to navigate the pages is simply to load the search page again with a
page number passed to the GET method along with the search terms that were originally passed.
It is passed as ?page=<pagenumber> .
