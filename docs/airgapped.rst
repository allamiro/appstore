====================================
Air-gapped Environment Installation
====================================

This guide describes how to set up a fully functional internal App Store for a disconnected (air-gapped) environment. This requires a "Sync and Capture" operation on an internet-connected machine before moving the data.

Step 1: Configure the "Bridge" Environment
------------------------------------------
On your internet-connected machine, create a production settings file that mimics your target internal environment.

1. Create the file: ``nextcloudappstore/settings/production.py``
2. Define Local Storage:

.. code-block:: python

    from nextcloudappstore.settings.baseproduction import *

    # The internal URL Nextcloud will use to download apps
    MEDIA_URL = 'https://internal-appstore.local/media/'
    
    # Physical folders on your server
    MEDIA_ROOT = '/srv/media/'
    STATIC_ROOT = '/srv/static/'

3. Set the environment variable:

.. code-block:: bash

    export DJANGO_SETTINGS_MODULE=nextcloudappstore.settings.production

Step 2: Sync Metadata and Download Apps
---------------------------------------
Pull the app information and files into local storage while connected.

1. Sync Nextcloud Releases:

.. code-block:: bash

    python manage.py syncnextcloudreleases --oldest-supported="28.0.0"

2. Download App Packages:

.. code-block:: bash

    make prod-data prod_version=28.0.0

This command downloads the .tar.gz files into your ``MEDIA_ROOT``.

Step 3: Verification
--------------------
Before moving, verify:
* **The Files**: Check ``/srv/media/`` for the downloaded packages.
* **The Database**: PostgreSQL now contains local paths for these apps.

Step 4: Moving to the Disconnected Environment
----------------------------------------------
Export the "Engine" (Docker), "Brain" (Database), and "Body" (Media).

1. Export the Image: ``docker save -o nextcloudappstore.tar appstore_production``
2. Export the Database:

.. code-block:: bash

    docker exec -t your_db_container pg_dump -U nextcloudappstore nextcloudappstore > appstore_db_dump.sql

3. Transfer the media folder and files to your offline storage.

Step 5: Final Internal Setup
----------------------------
On the disconnected server:

1. Load the Image: ``docker load -i nextcloudappstore.tar``
2. Import the Database: ``psql -U nextcloudappstore -d nextcloudappstore < appstore_db_dump.sql``
3. Update Nextcloud config (``config/config.php``):

.. code-block:: php

    'appstoreurl' => 'https://internal-appstore.local/api/v1',
