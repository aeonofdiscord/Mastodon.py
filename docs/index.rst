Mastodon.py
===========
.. py:module:: mastodon
.. py:class: Mastodon

.. code-block:: python

   from mastodon import Mastodon

   # Register app - only once!
   '''
   Mastodon.create_app(
        'pytooterapp', 
         to_file = 'pytooter_clientcred.secret'
   )
   '''

   # Log in - either every time, or use persisted
   '''
   mastodon = Mastodon(client_id = 'pytooter_clientcred.secret')
   mastodon.log_in(
       'my_login_email@example.com',
       'incrediblygoodpassword', 
       to_file = 'pytooter_usercred.secret'
   )
   '''

   # Create actual instance
   mastodon = Mastodon(
       client_id = 'pytooter_clientcred.secret', 
       access_token = 'pytooter_usercred.secret'
   )
   mastodon.toot('Tooting from python!')

`Mastodon`_ is an ostatus based twitter-like federated social 
network node. It has an API that allows you to interact with its 
every aspect. This is a simple python wrapper for that api, provided
as a single python module. By default, it talks to the 
`Mastodon flagship instance`_, but it can be set to talk to any 
node running Mastodon.

A note about rate limits
------------------------
Mastodons API rate limits per IP. By default, the limit is 150 requests per 5 minute 
time slot. This can differ from instance to instance and is subject to change.
Mastodon.py has three modes for dealing with rate limiting that you can pass to 
the constructor, "throw", "wait" and "pace", "wait" being the default.

In "throw" mode, Mastodon.py makes no attempt to stick to rate limits. When
a request hits the rate limit, it simply throws a MastodonRateLimitError. This is
for applications that need to handle all rate limiting themselves (i.e. interactive apps), 
or applications wanting to use Mastodon.py in a multi-threaded context ("wait" and "pace" 
modes are not thread safe).

In "wait" mode, once a request hits the rate limit, Mastodon.py will wait until
the rate limit resets and then try again, until the request succeeds or an error
is encountered. This mode is for applications that would rather just not worry about rate limits
much, don't poll the api all that often, and are okay with a call sometimes just taking
a while.

In "pace" mode, Mastodon.py will delay each new request after the first one such that, 
if requests were to continue at the same rate, only a certain fraction (set in the
constructor as ratelimit_pacefactor) of the rate limit will be used up. The fraction can
be (and by default, is) greater than one. If the rate limit is hit, "pace" behaves like
"wait". This mode is probably the most advanced one and allows you to just poll in
a loop without ever sleeping at all yourself. It is for applications that would rather
just pretend there is no such thing as a rate limit and are fine with sometimes not
being very interactive.

A note about IDs
----------------
Mastodons API uses IDs in several places: User IDs, Toot IDs, ...

While debugging, it might be tempting to copy-paste in IDs from the
web interface into your code. This will not work, as the IDs on the web
interface and in the URLs are not the same as the IDs used internally
in the API, so don't do that.

Return values
-------------
Unless otherwise specified, all data is returned as python 
dictionaries, matching the JSON format used by the API.

User dicts
~~~~~~~~~~
.. code-block:: python

    mastodon.account(<numerical id>)
    # Returns the following dictionary:
    {
        'id': # Same as <numerical id>
        'username': # The username (what you @ them with)
        'acct': # The user's account name as username@domain (@domain omitted for local users)
        'display_name': # The user's display name
        'locked': # Denotes whether the account can be followed without a follow request
        'following_count': # How many people they follow
        'followers_count': # How many followers they have
        'statuses_count': # How many statuses they have
        'note': # Their bio
        'url': # Their URL; usually 'https://mastodon.social/users/<acct>'
        'avatar': # URL for their avatar
        'header': # URL for their header image
    }

Toot dicts
~~~~~~~~~~
.. code-block:: python

    mastodon.toot("Hello from Python")
    # Returns the following dictionary:
    {
        'id': # Numerical id of this toot
        'uri': # Descriptor for the toot
            # EG 'tag:mastodon.social,2016-11-25:objectId=<id>:objectType=Status'
        'url': # URL of the toot
        'account': # Account dict for the account which posted the status
        'in_reply_to_id': # Numerical id of the toot this toot is in response to
        'in_reply_to_account_id': # Numerical id of the account this toot is in response to
        'reblog': # Denotes whether the toot is a reblog
        'content': # Content of the toot, as HTML: '<p>Hello from Python</p>'
        'created_at': # Creation time
        'reblogs_count': # Number of reblogs
        'favourites_count': # Number of favourites
        'reblogged': # Denotes whether the logged in user has boosted this toot
        'favourited': # Denotes whether the logged in user has favourited this toot
        'sensitive': # Denotes whether media attachments to the toot are marked sensitive
        'spoiler_text': # Warning text that should be displayed before the toot content
        'visibility': # Toot visibility ('public', 'unlisted', 'private', or 'direct')
        'mentions': # A list of account dicts mentioned in the toot
        'media_attachments': # list of media dicts of attached files. Only present
                            # when there are attached files.
        'tags': # A list of hashtag dicts used in the toot
        'application': # Application dict for the client used to post the toot
    }

Relationship dicts
~~~~~~~~~~~~~~~~~~
.. code-block:: python

    mastodon.account_follow(<numerical id>)
    # Returns the following dictionary:
    {
        'id': # Numerical id (same one as <numerical id>)
        'following': # Boolean denoting whether you follow them
        'followed_by': # Boolean denoting whether they follow you back
        'blocking': # Boolean denoting whether you are blocking them
        'muting': # Boolean denoting whether you are muting them
        'requested': # Boolean denoting whether you have sent them a follow request
    }

Notification dicts
~~~~~~~~~~~~~~~~~~
.. code-block:: python

    mastodon.notifications()[0]
    # Returns the following dictionary:
    {
        'id': # id of the notification.
        'type': # "mention", "reblog", "favourite" or "follow".
        'created_at': # The time the notification was created.
        'account': # User dict of the user from whom the notification originates.
        'status': # In case of "mention", the mentioning status. 
                  # In case of reblog / favourite, the reblogged / favourited status.
    }

Context dicts
~~~~~~~~~~~~~
.. code-block:: python

    mastodon.status_context(<numerical id>)
    # Returns the following dictionary:
    {
        'ancestors': # A list of toot dicts
        'descendants': # A list of toot dicts
    }

Media dicts
~~~~~~~~~~~
.. code-block:: python

    mastodon.media_post("image.jpg", "image/jpeg")
    # Returns the following dictionary:
    {
        'id': # The ID of the attachment.
        'type': # Media type, EG 'image'
        'url': # The URL for the image in the local cache
        'remote_url': # The remote URL for the media (if the image is from a remote instance)
        'preview_url': # The URL for the media preview
        'text_url': # The display text for the media (what shows up in toots)
    }

Card dicts
~~~~~~~~~~
..code-block:: python

    mastodon.status_card(<numerical id>):
    # Returns the following dictionary:
    {
        'url': The URL of the card.
        'title': The title of the card.
        'description': The description of the card.
        'image': (optional) The image associated with the card.
    }

App registration and user authentication
----------------------------------------
Before you can use the mastodon API, you have to register your 
application (which gets you a client key and client secret) 
and then log in (which gets you an access token). These functions 
allow you to do those things.
For convenience, once you have a client id, secret and access token, 
you can simply pass them to the constructor of the class, too!

Note that while it is perfectly reasonable to log back in whenever 
your app starts, registering a new application on every 
startup is not, so don't do that - instead, register an application 
once, and then persist your client id and secret. Convenience
methods for this are provided.

.. automethod:: Mastodon.create_app
.. automethod:: Mastodon.__init__
.. automethod:: Mastodon.log_in
.. automethod:: Mastodon.auth_request_url

Reading data: Instance
-----------------------
This function allows you to fetch information associated with the
current instance.

.. automethod:: Mastodon.instance

Reading data: Timelines
-----------------------
This function allows you to access the timelines a logged in
user could see, as well as hashtag timelines and the public timeline.

.. automethod:: Mastodon.timeline
.. automethod:: Mastodon.timeline_home
.. automethod:: Mastodon.timeline_local
.. automethod:: Mastodon.timeline_public
.. automethod:: Mastodon.timeline_hashtag

Reading data: Statuses
----------------------
These functions allow you to get information about single statuses.

.. automethod:: Mastodon.status
.. automethod:: Mastodon.status_context
.. automethod:: Mastodon.status_reblogged_by
.. automethod:: Mastodon.status_favourited_by
.. automethod:: Mastodon.status_card

Reading data: Notifications
---------------------------
This function allows you to get information about a users notifications.

.. automethod:: Mastodon.notifications

Reading data: Accounts
----------------------
These functions allow you to get information about accounts and
their relationships.

.. automethod:: Mastodon.account
.. automethod:: Mastodon.account_verify_credentials
.. automethod:: Mastodon.account_statuses
.. automethod:: Mastodon.account_following
.. automethod:: Mastodon.account_followers
.. automethod:: Mastodon.account_relationships
.. automethod:: Mastodon.account_search

Reading data: Follows
---------------------

.. automethod:: Mastodon.follows

Reading data: Searching
-----------------------
This function allows you to search for content.

.. automethod:: Mastodon.search


Reading data: Mutes and blocks
------------------------------
These functions allow you to get information about accounts that are
muted or blocked by the logged in user.

.. automethod:: Mastodon.mutes
.. automethod:: Mastodon.blocks

Reading data: Reports
------------------------------
These functions allow you to retrieve information about reports filed
by the authenticated user, and file a report against a user.

.. automethod:: Mastodon.reports
.. automethod:: Mastodon.report

Reading data: Favourites
------------------------
This function allows you to get information about statuses favourited
by the authenticated user.

.. automethod:: Mastodon.favourites

Reading data: Follow requests
-----------------------------
This function allows you to get a list of pending incoming follow
requests for the authenticated user.

.. automethod:: Mastodon.follow_requests

Writing data: Statuses
----------------------
These functions allow you to post statuses to Mastodon and to
interact with already posted statuses.

.. automethod:: Mastodon.status_post
.. automethod:: Mastodon.toot
.. automethod:: Mastodon.status_reblog
.. automethod:: Mastodon.status_unreblog
.. automethod:: Mastodon.status_favourite
.. automethod:: Mastodon.status_unfavourite
.. automethod:: Mastodon.status_delete

Writing data: Accounts
----------------------
These functions allow you to interact with other accounts: To (un)follow and
(un)block.

.. automethod:: Mastodon.account_follow
.. automethod:: Mastodon.follows
.. automethod:: Mastodon.account_unfollow
.. automethod:: Mastodon.account_block
.. automethod:: Mastodon.account_unblock
.. automethod:: Mastodon.account_mute
.. automethod:: Mastodon.account_unmute
.. automethod:: Mastodon.account_update_credentials

Writing data: Follow requests
-----------------------------
These functions allow you to accept or reject incoming follow requests.

.. automethod:: Mastodon.follow_request_authorize
.. automethod:: Mastodon.follow_request_reject

Writing data: Media
-------------------
This function allows you to upload media to Mastodon. The returned
media IDs (Up to 4 at the same time) can then be used with post_status
to attach media to statuses.

.. automethod:: Mastodon.media_post

Streaming
---------
These functions allow access to the streaming API.

.. automethod:: Mastodon.user_stream
.. automethod:: Mastodon.public_stream
.. automethod:: Mastodon.hashtag_stream


.. _Mastodon: https://github.com/tootsuite/mastodon
.. _Mastodon flagship instance: http://mastodon.social/
.. _Mastodon api docs: https://github.com/tootsuite/documentation/
