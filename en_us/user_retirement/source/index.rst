.. _document index:

#################################################
User Retirement Guide for Open edX Administrators
#################################################

GDPR (or the EU General Data Protection Regulation) is a sweeping change to
privacy laws intended to change the way that business think about and handle
Personally Identifiable Information (PII) data.  In order to comply with this
law, edX has implemented APIs and tooling for "retiring" users.  Once
configured, the user retirement tooling can automatically erase all PII for a
given user from internal systems (i.e. lms, forums, credentials, and all other
IDAs) and external (e.g. third party marketing services).

.. toctree::
    :numbered:
    :maxdepth: 2

    implementation_overview
    service_setup
    driver_setup
    special_cases

