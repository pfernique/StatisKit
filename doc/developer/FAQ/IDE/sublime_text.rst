Sublime Text
============

With **Sublime Text**, it is recommanded to use the following addons:

* **Package Control**, see this `page <https://packagecontrol.io>`_ for more details.
* **Terminal**, see this `page <https://packagecontrol.io/packages/Terminal>`_ for more details.
* **Git**, see this `page <https://packagecontrol.io/packages/Git>`_ for more details.
* **ProjectManager**, see this `page <https://packagecontrol.io/packages/ProjectManager>`_ for more details.

Moreover, to build **StatisKit** sublime text projects, you can add a custom `build system <https://www.sublimetext.com/docs/3/build_systems.html>`_.
To do so,

* Select the :code:`Tools > Build System > New Build System` menu item.
* In the new opened file, replace the existing content with the following one

  .. code-block:: json

    {
        "target": "statiskit_build",
        "cancel": {"kill": true},
        "system": "",
        "prefix": "",
        "environment": "",
        "variants": [
            {
                "name": "C++",
                "cpp": true,
            },
            {
                "name": "Python",
                "py": true,
            },
            {
                "name": "Test",
                "test": true,
            },
            {
                "name": "AutoWIG",
                "autowig": true,
            },
            {
                "name": "Clean",
                "clean": true,
            },
            {
                "name": "Sphinx",
                "sphinx": true
            }
        ]
    }

  Save it with under the :code:`StatisKit.sublime-build` filename in the proposed directory.

  .. warning::

    Replace the :code:`"system"`, :code:`"prefix"` and :code:`"environment"` values (here :code:`""`) by the appropriate one.
    The :code:`"system"` value should be one of :code:`"linux"`, :code:`"osx"` or :code:`"win"`.
    The :code:`"prefix"` value should the complete path to your **Anaconda** or **Miniconda** root directory.
    The :code:`"environment"` value should the name of your **Conda** environment where the **StatisKit** toolchain packages were installed.
    All values must be surrounded by :code:`"` as for the system candidate values.

* Select the :code:`Tools > Developer > New Plugin` menu item.
* In the new opened file, replace the existing content with the following one (this content is an extension of the example given on this `page <https://www.sublimetext.com/docs/3/build_systems.html>`_)

  .. code-block:: python

    import sublime
    import sublime_plugin

    import subprocess
    import threading
    import os
    import re

    class StatiskitBuildCommand(sublime_plugin.WindowCommand):

        encoding = 'utf-8'
        killed = False
        proc = None
        panel = None
        panel_lock = threading.Lock()

        file_regexes = ["^[ ]*File \"(...*?)\", line ([0-9]*)",
                        "(^/.*):(\\d+):(\\d+): ",
                        "^(..[^:]*):([0-9]+):?([0-9]+)?:? error: (.*)$"]

        variant_dirs = {'build/(.*)' : '',
                        '^.*/build/(.*)' : '',
                        '^.*/site-packages/(.*)' : 'src/py/',
                        '^.*/include/statiskit/[^/]*/(.*)' : 'src/cpp/'}

        def is_enabled(self, system, prefix, environment, cpu_count=1, cpp=False, py=False, test=False, autowig=False, clean=False, sphinx=False, kill=False):
            # The Cancel build option should only be available
            # when the process is still running
            if kill:
                return self.proc is not None and self.proc.poll() is None
            return True

        def run(self, system, prefix, environment, cpu_count=1, cpp=False, py=False, test=False, autowig=False, clean=False, sphinx=False, kill=False):
            if kill:
                if self.proc:
                    self.killed = True
                    self.proc.terminate()
                return

            vars = self.window.extract_variables()
            self.working_dir = vars['folder']

            # A lock is used to ensure only one thread is
            # touching the output panel at a time
            with self.panel_lock:
                # Creating the panel implicitly clears any previous contents
                self.panel = self.window.create_output_panel('exec')

                # Enable result navigation. The result_file_regex does
                # the primary matching, but result_line_regex is used
                # when build output includes some entries that only
                # contain line/column info beneath a previous line
                # listing the file info. The result_base_dir sets the
                # path to resolve relative file names against.

                settings = self.panel.settings()
                if not sphinx:
                    settings.set(
                        'result_file_regex',
                        r"^\[Build error - file \"(...*?)\" at line ([0-9]*), (.*)\]$"
                    )
                    settings.set('result_base_dir', self.working_dir)

                self.window.run_command('show_panel', {'panel': 'output.exec'})

            if self.proc is not None:
                self.proc.terminate()
                self.proc = None

            if system in ["linux", "osx"]:
                cmd = "bash -c 'source " + prefix + "/bin/activate " + environment
                sep = " && "
                end = "'"
            elif system == "win":
                cmd = "call " + prefix + "\\Scripts\\activate.bat " + environment
                sep = " & "
                end = ""

            if not sphinx:
                scons = "scons -j" + str(cpu_count)
                if system == "linux":
                    scons += " --diagnostics-color=never"

                targets = []
                if cpp:
                    targets.append("cpp")
                elif py:
                    targets.append("py")
                elif test:
                    targets.append("test")
                elif autowig:
                    targets.extend(["cpp",
                                    "autowig"])
                elif clean:
                    targets.append("-c")
                else:
                    targets.extend(["cpp",
                                    "autowig",
                                    "py",
                                    "test"])

                for target in targets:
                    cmd += sep + scons + " " + target
            else:
                cmd += sep + "cd doc" + sep + "make html"

            if end:
                cmd += end

            self.proc = subprocess.Popen(
                cmd,
                stdout=subprocess.PIPE,
                stderr=subprocess.STDOUT,
                cwd=self.working_dir,
                shell=True,
            )
            self.killed = False

            threading.Thread(
                target=self.read_handle,
                args=(self.proc.stdout,)
            ).start()

        def read_handle(self, handle):
            chunk_size = 2 ** 13
            out = b''
            while True:
                try:
                    data = os.read(handle.fileno(), chunk_size)
                    # If exactly the requested number of bytes was
                    # read, there may be more data, and the current
                    # data may contain part of a multibyte char
                    out += data
                    if len(data) == chunk_size:
                        continue
                    if data == b'' and out == b'':
                        raise IOError('EOF')
                    # We pass out to a function to ensure the
                    # timeout gets the value of out right now,
                    # rather than a future (mutated) version
                    self.queue_write(out.decode(self.encoding))
                    if data == b'':
                        raise IOError('EOF')
                    out = b''
                except (UnicodeDecodeError) as e:
                    msg = 'Error decoding output using %s - %s'
                    self.queue_write(msg  % (self.encoding, str(e)))
                    break
                except (IOError):
                    if self.killed:
                        msg = 'Cancelled'
                    else:
                        msg = 'Finished'
                    self.queue_write('\n[%s]' % msg)
                    break

        def queue_write(self, text):
            sublime.set_timeout(lambda: self.do_write(text), 1)

        def do_write(self, text):
            with self.panel_lock:

                at_line = text.endswith('\n')
                text = text.splitlines()

                cache = set()
                for index in range(len(text)):
                    matchline = None
                    for file_regex in self.file_regexes:
                        matchline = re.search(file_regex, text[index])
                        if matchline and text[index] not in cache:
                            cache.add(text[index])
                            file, line = matchline.group(1, 2)
                            for variant_dir in self.variant_dirs:
                                matchvariant = re.match(variant_dir, file)
                                if matchvariant:
                                    file = self.variant_dirs[variant_dir] + matchvariant.group(1)
                                    break
                            if os.path.exists(os.path.join(self.working_dir, file)):
                                text[index] = text[index] + '\n[Build error - file \"' + file + '" at line ' +  line + ", see build results]" 

                text = '\n'.join(text)
                if at_line:
                    text += '\n'

                self.panel.run_command('append', {'characters': text})

  Save it with under the :code:`statiskit_build.py` filename in the proposed directory.