Chatbot Portal
==============

A Chatbot Portal for rea websites.

.. image:: https://img.shields.io/badge/built%20for-ClikHome%2C%20Inc.-ff69b4.svg
     :target: https://www.apartmentocean.com/
     :alt: Built with Love


Settings
--------

Moved to settings_.

.. _settings: http://cookiecutter-django.readthedocs.io/en/latest/settings.html


Basic Commands
--------------

Setting Up Your Users
^^^^^^^^^^^^^^^^^^^^^

* To create a **normal user account**, just go to Sign Up and fill out the form. Once you submit it, you'll see a "Verify Your E-mail Address" page. Go to your console to see a simulated email verification message. Copy the link into your browser. Now the user's email should be verified and ready to go.

* To create an **superuser account**, use this command::

    $ python manage.py createsuperuser

For convenience, you can keep your normal user logged in on Chrome and your superuser logged in on Firefox (or similar), so that you can see how the site behaves for both kinds of users.

Test coverage
^^^^^^^^^^^^^

To run the tests, check your test coverage, and generate an HTML coverage report::

    $ coverage run manage.py test
    $ coverage html
    $ open htmlcov/index.html

Running tests with py.test
~~~~~~~~~~~~~~~~~~~~~~~~~~

::

  $ py.test

Live reloading and Sass CSS compilation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Moved to `Live reloading and SASS compilation`_.

.. _`Live reloading and SASS compilation`: http://cookiecutter-django.readthedocs.io/en/latest/live-reloading-and-sass-compilation.html

Celery
^^^^^^

This app comes with Celery.

To run a celery worker:

.. code-block:: bash

    cd webchat
    celery -A webchat.taskapp worker -l info

Please note: For Celery's import magic to work, it is important *where* the celery commands are run. If you are in the same folder with *manage.py*, you should be right.

Email Server
^^^^^^^^^^^^

In development, it is often nice to be able to see emails that are being sent from your application. For that reason local SMTP server `MailHog`_ with a web interface is available as docker container.

.. _mailhog: https://github.com/mailhog/MailHog

Container mailhog will start automatically when you will run all docker containers.
Please check `cookiecutter-django Docker documentation`_ for more details how to start all containers.

**Alternative running**:

MailHog is built with Go so there are no dependencies. To use MailHog:

1. `Download the latest release`_ for your operating system.
2. Rename the executable to ``mailhog`` and copy it to the root of your project directory.
3. Make sure it is executable (e.g. ``chmod +x mailhog``).
4. Execute mailhog from the root of your project in a new terminal window (e.g. ``./mailhog``).


With MailHog running, to view messages that are sent by your application, open your browser and go to ``http://127.0.0.1:8025``.

Alternatively simply output emails to the console via::

    DJANGO_EMAIL_BACKEND='django.core.mail.backends.console.EmailBackend'

Sentry
^^^^^^

Sentry is an error logging aggregator service. You can sign up for a free account at  https://getsentry.com/signup/?code=cookiecutter  or download and host it yourself.
The system is setup with reasonable defaults, including 404 logging and integration with the WSGI application.

You must set the DSN url in production.


Deployment
----------

The following details how to deploy this application.

Getting Up and Running Locally
^^^^^^

The steps below will get you up and running with a local development environment. We assume you have the following installed:

* pip
* virtualenv
* PostgreSQL

Step 1: Create a PostgreSQL user and database with the following commands::

    $ sudo su - postgres
    $ createuser <owning_user>
    $ createdb webchat

Step 2: First make sure to create and activate a `virtualenv`_.

Step 3: Clone this repository.

Step 4: Then install the requirements for your local development::

    $ pip install -r requirements/local.txt
    $ pip install -r requirements/test.txt

Step 5: Create ``.env`` file and add your own environment variables to it.

Step 6: Setup your email backend (see "Email Server" chapter).

Step 7: You can now run the usual Django ``migrate`` and ``runserver`` commands::

    $ python manage.py migrate
    $ python manage.py runserver

Go to http://127.0.0.1:8000/

Heroku
^^^^^^

See detailed `cookiecutter-django Heroku documentation`_.

.. _`cookiecutter-django Heroku documentation`: http://cookiecutter-django.readthedocs.io/en/latest/deployment-on-heroku.html

Docker
^^^^^^

See detailed `cookiecutter-django Docker documentation`_.

.. _`cookiecutter-django Docker documentation`: http://cookiecutter-django.readthedocs.io/en/latest/deployment-with-docker.html
.. _`Download the latest release`: https://github.com/mailhog/MailHog/releases
.. _`PostgreSQL`: https://www.postgresql.org/
.. _`virtualenv`: http://docs.python-guide.org/en/latest/dev/virtualenvs/


Configurations
--------------

Managing chatbot engine IBMWatson_.

.. _IBMWatson: https://www.ibmwatsonconversation.com/

Adding new chatbot to admin panel
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
First of all we need to make sure that we have ``env`` variable called ``WATSON_CONVERSATION_WORKSPACE_ID``
with specific value.

You can add/change it:
    * in ``.env`` file
    * with heroku command ``heroku config:set WATSON_CONVERSATION_WORKSPACE_ID=***************``
    * directly in `Heroku Dashboard`_ > Settings > Reveal Config Vars
.. _Heroku Dashboard: https://dashboard.heroku.com/`

To add new bot you need to navigate ``/ADMIN_URL/portal/chatbot/``

Select **Bot Engine** as ``IBM Watson Conversation``. Don't forget to fill ``Access ID`` field. You can find it on
IBMWatson_ > Deploy > Credentials > Workspace Details and copy ``Workspace ID``

Passing context from chatbot
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
All response from new IBMWatson_ can be saved and processed on our side. All we have to do is add context variable
in bot's questions. To do that we need to open ``Dialog`` tab in IBMWatson_ dashboard and add following lines:

    $ "context": {
    $   "variable_name": "<? input_text ?>"
    $ },

Replace ``variable_name`` with desired keyword.

Working with backend
^^^^^^^^^^^^^^^^^^^^
Most of our bots are gathering specific data such as First Name, Phonenumber, Emails etc. In most cases all data
are saved in Leads for further processing.

Make sure that Widget setting is associated with new chatbot.

So far we have two internal services for handling data from the bots:
    * Entrata
    * Leadmailbox

Entrata
^^^^^^^
To enable Entrata requests simply add Chatbot with specific ``access_id`` and select checkbox called ``Is entrata``.
In this case all responses from user will be saved in a JSON ``request_data`` field in model ``EntrataRequest``.

Leadmailbox
^^^^^^^^^^^
Make sure to enable checkboxes ``Is leadmailbox`` and ``Is persist data``. See more comments
in ``webchat.chatbots.watsonbot.py`` function called ``send_to_leadmailbox``

Data for this service will be send right after ``end_conversation`` flag from the bot.
In case of fail - log will be saved.

* Additional information about methods/functions and logic you can find in docstring in files like
``views.py``, ``models.py`` *
