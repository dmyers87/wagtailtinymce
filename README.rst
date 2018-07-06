===============
Wagtail TinyMCE
===============

Wagtail TinyMCE offers integration of the
`TinyMCE rich text editor <http://www.tinymce.com>`_ into the
`Wagtail CMS <http://wagtail.io>`_.

As of Wagtail 1.5 this integrates using Wagtail's alternative rich text editor feature and requires no extra customisation or patching.

Installation
============

Add ``wagtailtinymce`` to your |INSTALLED_APPS Django setting|_.

.. |INSTALLED_APPS Django setting| replace:: ``INSTALLED_APPS`` Django setting
.. _`INSTALLED_APPS Django setting`: https://docs.djangoproject.com/en/1.9/ref/settings/#installed-apps

Add or change ``WAGTAILADMIN_RICH_TEXT_EDITORS`` in your settings, to use the widget ``wagtailtinymce.rich_text.TinyMCERichTextArea``.

For example, to use TinyMCE for all ``RichTextField`` and ``RichTextBlock`` instances:

.. code-block:: python

    WAGTAILADMIN_RICH_TEXT_EDITORS = {
        'default': {
            'WIDGET': 'wagtailtinymce.rich_text.TinyMCERichTextArea'
        },
    }

Or, to use TinyMCE for certain instances...

.. code-block:: python

    WAGTAILADMIN_RICH_TEXT_EDITORS = {
        'default': {
            'WIDGET': 'wagtail.wagtailadmin.rich_text.HalloRichTextArea'
        },
        'tinymce': {
            'WIDGET': 'wagtailtinymce.rich_text.TinyMCERichTextArea'
        },
    }

...and declare fields with the corresponding key in the ``editor`` parameter:

.. code-block:: python

    html_field = RichTextField(editor='tinymce', ...)
    stream_field = StreamField([('html', RichTextBlock(editor='tinymce', ...)), ...])

TinyMCE configuration
=====================

Wagtail does not currently allow for passing parameters to the TinyMCERichTextArea constructor. To change the configuration you must create and register a subclass of ``TinyMCERichTextArea`` and pass these parameters or amend defaults in the subclass constructor.

The default configuration for WagtailTinyMCE is:

.. code-block:: javascript
    {
        'browser_spellcheck': True,
        'menubar': False,
        'plugins': 'hr code fullscreen noneditable paste table',
        'toolbar': 'undo redo | styleselect | bold italic | bullist numlist outdent indent | table| link unlink | wagtaildoclink wagtailimage wagtailembed | pastetext fullscreen | removeformat',
        'tools': 'inserttable',
    }

Additional configuration options for can be found in the TinyMCE documentation: https://www.tinymce.com/docs/


TinyMCE plugins and tools
=========================


To add external plugins and tools to TinyMCE, use the
``insert_tinymce_js`` and ``insert_tinymce_css`` hooks. Once you have the hook in place use the
following JavaScript to register the plugin with TinyMCE:

.. code-block:: javascript

    registerMCEExternalPlugin(name, path, language);

For example:

.. code-block:: javascript

    registerMCEExternalPlugin('myplugin', '/static/js/my-tinymce-plugin.js', 'en_GB');

The ``language`` parameter is optional and can be omitted.

A complete ``wagtail_hooks.py`` file example:

.. code-block:: python

    import json

    from django.templatetags.static import static
    from django.utils import translation
    from django.utils.html import format_html
    from django.utils.safestring import mark_safe
    from wagtail.wagtailcore import hooks

    @hooks.register('insert_tinymce_js')
    def my_plugin_js():
        return format_html(
            """
            <script>
                registerMCEPlugin("myplugin", {});
            </script>
            """,
            mark_safe(json.dumps(static('js/my-tinymce-plugin.js'))),
            to_js_primitive(translation.to_locale(translation.get_language())),
        )

How to upgrade TinyMCE
======================

1. Clone [TinyMCE](https://github.com/tinymce/tinymce) repo in a different folder.
1. Follow the instructions on this repo to build it using Grunt.
1. As of version 4.6.4, a `/js` folder will be generated in the root of the TinyMCE repo.
Copy its contents to `wagtailtinymce/wagtailtinymce/static/wagtailtinymce/js/vendor`, replacing the contents of this folder.
1. Create a new branch with the versioning instructions below so that this new version is accessible with `pip`.
The branch for the version `4.6.4` is `TinyMCE4.6.4`.

Versioning
==========
The version number of this package is the TinyMCE version, followed by
the release number of this package for that TinyMCE version.