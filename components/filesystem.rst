.. index::
   single: Filesystem

The Filesystem Component
========================

    The Filesystem component provides basic utilities for the filesystem.

Installation
------------

.. code-block:: terminal

    $ composer require symfony/filesystem

Alternatively, you can clone the `<https://github.com/symfony/filesystem>`_ repository.

.. include:: /components/require_autoload.rst.inc

Usage
-----

The :class:`Symfony\\Component\\Filesystem\\Filesystem` class is the unique
endpoint for filesystem operations::

    use Symfony\Component\Filesystem\Filesystem;
    use Symfony\Component\Filesystem\Exception\IOExceptionInterface;

    $fileSystem = new Filesystem();

    try {
        $fileSystem->mkdir('/tmp/random/dir/'.mt_rand());
    } catch (IOExceptionInterface $exception) {
        echo "An error occurred while creating your directory at ".$exception->getPath();
    }

.. note::

    Methods :method:`Symfony\\Component\\Filesystem\\Filesystem::mkdir`,
    :method:`Symfony\\Component\\Filesystem\\Filesystem::exists`,
    :method:`Symfony\\Component\\Filesystem\\Filesystem::touch`,
    :method:`Symfony\\Component\\Filesystem\\Filesystem::remove`,
    :method:`Symfony\\Component\\Filesystem\\Filesystem::chmod`,
    :method:`Symfony\\Component\\Filesystem\\Filesystem::chown` and
    :method:`Symfony\\Component\\Filesystem\\Filesystem::chgrp` can receive a
    string, an array or any object implementing :phpclass:`Traversable` as
    the target argument.

mkdir
~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::mkdir` creates a directory recursively.
On POSIX filesystems, directories are created with a default mode value
`0777`. You can use the second argument to set your own mode::

    $fileSystem->mkdir('/tmp/photos', 0700);

.. note::

    You can pass an array or any :phpclass:`Traversable` object as the first
    argument.

.. note::

    This function ignores already existing directories.

.. note::

    The directory permissions are affected by the current `umask`_.
    Set the umask for your webserver, use PHP's :phpfunction:`umask`
    function or use the :phpfunction:`chmod` function after the
    directory has been created.

exists
~~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::exists` checks for the
presence of one or more files or directories and returns ``false`` if any of
them is missing::

    // this directory exists, return true
    $fileSystem->exists('/tmp/photos');

    // rabbit.jpg exists, bottle.png does not exist, return false
    $fileSystem->exists(array('rabbit.jpg', 'bottle.png'));

.. note::

    You can pass an array or any :phpclass:`Traversable` object as the first
    argument.

copy
~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::copy` makes a copy of a
single file (use :method:`Symfony\\Component\\Filesystem\\Filesystem::mirror` to
copy directories). If the target already exists, the file is copied only if the
source modification date is later than the target. This behavior can be overridden
by the third boolean argument::

    // works only if image-ICC has been modified after image.jpg
    $fileSystem->copy('image-ICC.jpg', 'image.jpg');

    // image.jpg will be overridden
    $fileSystem->copy('image-ICC.jpg', 'image.jpg', true);

touch
~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::touch` sets access and
modification time for a file. The current time is used by default. You can set
your own with the second argument. The third argument is the access time::

    // sets modification time to the current timestamp
    $fileSystem->touch('file.txt');
    // sets modification time 10 seconds in the future
    $fileSystem->touch('file.txt', time() + 10);
    // sets access time 10 seconds in the past
    $fileSystem->touch('file.txt', time(), time() - 10);

.. note::

    You can pass an array or any :phpclass:`Traversable` object as the first
    argument.

chown
~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::chown` changes the owner of
a file. The third argument is a boolean recursive option::

    // sets the owner of the lolcat video to www-data
    $fileSystem->chown('lolcat.mp4', 'www-data');
    // changes the owner of the video directory recursively
    $fileSystem->chown('/video', 'www-data', true);

.. note::

    You can pass an array or any :phpclass:`Traversable` object as the first
    argument.

chgrp
~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::chgrp` changes the group of
a file. The third argument is a boolean recursive option::

    // sets the group of the lolcat video to nginx
    $fileSystem->chgrp('lolcat.mp4', 'nginx');
    // changes the group of the video directory recursively
    $fileSystem->chgrp('/video', 'nginx', true);

.. note::

    You can pass an array or any :phpclass:`Traversable` object as the first
    argument.

chmod
~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::chmod` changes the mode or
permissions of a file. The fourth argument is a boolean recursive option::

    // sets the mode of the video to 0600
    $fileSystem->chmod('video.ogg', 0600);
    // changes the mod of the src directory recursively
    $fileSystem->chmod('src', 0700, 0000, true);

.. note::

    You can pass an array or any :phpclass:`Traversable` object as the first
    argument.

remove
~~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::remove` deletes files,
directories and symlinks::

    $fileSystem->remove(array('symlink', '/path/to/directory', 'activity.log'));

.. note::

    You can pass an array or any :phpclass:`Traversable` object as the first
    argument.

rename
~~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::rename` changes the name
of a single file or directory::

    // renames a file
    $fileSystem->rename('/tmp/processed_video.ogg', '/path/to/store/video_647.ogg');
    // renames a directory
    $fileSystem->rename('/tmp/files', '/path/to/store/files');

symlink
~~~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::symlink` creates a
symbolic link from the target to the destination. If the filesystem does not
support symbolic links, a third boolean argument is available::

    // creates a symbolic link
    $fileSystem->symlink('/path/to/source', '/path/to/destination');
    // duplicates the source directory if the filesystem
    // does not support symbolic links
    $fileSystem->symlink('/path/to/source', '/path/to/destination', true);

readlink
~~~~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::readlink` read links targets.

PHP's ``readlink()`` function returns the target of a symbolic link. However, its behavior
is completely different under Windows and Unix. On Windows systems, ``readlink()``
resolves recursively the children links of a link until a final target is found. On
Unix-based systems ``readlink()`` only resolves the next link.

The :method:`Symfony\\Component\\Filesystem\\Filesystem::readlink` method provided
by the Filesystem component always behaves in the same way::

    // returns the next direct target of the link without considering the existence of the target
    $fs->readlink('/path/to/link');

    // returns its absolute fully resolved final version of the target (if there are nested links, they are resolved)
    $fs->readlink('/path/to/link', true);

Its behavior is the following::

    public function readlink($path, $canonicalize = false)

* When ``$canonicalize`` is ``false``:
    * if ``$path`` does not exist or is not a link, it returns ``null``.
    * if ``$path`` is a link, it returns the next direct target of the link without considering the existence of the target.

* When ``$canonicalize`` is ``true``:
    * if ``$path`` does not exist, it returns null.
    * if ``$path`` exists, it returns its absolute fully resolved final version.

makePathRelative
~~~~~~~~~~~~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::makePathRelative` takes two
absolute paths and returns the relative path from the second path to the first one::

    // returns '../'
    $fileSystem->makePathRelative(
        '/var/lib/symfony/src/Symfony/',
        '/var/lib/symfony/src/Symfony/Component'
    );
    // returns 'videos/'
    $fileSystem->makePathRelative('/tmp/videos', '/tmp')

mirror
~~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::mirror` copies all the
contents of the source directory into the target one (use the
:method:`Symfony\\Component\\Filesystem\\Filesystem::copy` method to copy single
files)::

    $fileSystem->mirror('/path/to/source', '/path/to/target');

isAbsolutePath
~~~~~~~~~~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::isAbsolutePath` returns
``true`` if the given path is absolute, ``false`` otherwise::

    // returns true
    $fileSystem->isAbsolutePath('/tmp');
    // returns true
    $fileSystem->isAbsolutePath('c:\\Windows');
    // returns false
    $fileSystem->isAbsolutePath('tmp');
    // returns false
    $fileSystem->isAbsolutePath('../dir');

dumpFile
~~~~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::dumpFile` saves the given
contents into a file. It does this in an atomic manner: it writes a temporary
file first and then moves it to the new file location when it's finished.
This means that the user will always see either the complete old file or
complete new file (but never a partially-written file)::

    $fileSystem->dumpFile('file.txt', 'Hello World');

The ``file.txt`` file contains ``Hello World`` now.

appendToFile
~~~~~~~~~~~~

:method:`Symfony\\Component\\Filesystem\\Filesystem::appendToFile` adds new
contents at the end of some file::

    $fs->appendToFile('logs.txt', 'Email sent to user@example.com');

If either the file or its containing directory doesn't exist, this method
creates them before appending the contents.

Error Handling
--------------

Whenever something wrong happens, an exception implementing
:class:`Symfony\\Component\\Filesystem\\Exception\\ExceptionInterface` or
:class:`Symfony\\Component\\Filesystem\\Exception\\IOExceptionInterface` is thrown.

.. note::

    An :class:`Symfony\\Component\\Filesystem\\Exception\\IOException` is
    thrown if directory creation fails.

.. _`Packagist`: https://packagist.org/packages/symfony/filesystem
.. _`umask`: https://en.wikipedia.org/wiki/Umask
